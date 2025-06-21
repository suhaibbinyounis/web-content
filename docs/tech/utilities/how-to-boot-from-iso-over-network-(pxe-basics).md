---
categories:
- Networking
- System Administration
- Tutorials
- Linux
comments: true
cover:
  image: https://images.pexels.com/photos/9470919/pexels-photo-9470919.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 11:22:34.549000
description: Dive into the world of network booting with PXE! This comprehensive guide
  explains how to set up a PXE server to boot various ISO images over your network,
  covering DHCP, TFTP, and serving ISO content for efficient system deployments and
  recovery.
tags:
- PXE
- Network Boot
- ISO
- Linux
- Windows
- SysAdmin
- DevOps
- Infrastructure
- Bare Metal
- Deployment
title: How to Boot from ISO Over Network (PXE Basics)
---

![Detailed shot of a bicycle frame featuring the Pexels logo with focus on texture and design.](https://images.pexels.com/photos/9470919/pexels-photo-9470919.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Detailed shot of a bicycle frame featuring the Pexels logo with focus on texture and design.")

## How to Boot from ISO Over Network (PXE Basics)

Network booting, specifically using PXE (Preboot Execution Environment), is an incredibly powerful capability for IT professionals, system administrators, and anyone dealing with multiple machines. Imagine provisioning a new server without fumbling with USB sticks or optical drives, or quickly booting a live rescue environment on a client machine from a central server. This is where PXE shines.

In this detailed guide, we'll peel back the layers of PXE to understand its core components and provide a practical walkthrough for setting up a PXE server to boot from ISO images over your network. While the principles apply broadly, our practical examples will lean towards a Linux-based PXE server and booting Linux ISOs, as it offers the most flexibility and open-source tooling.

## The Power of PXE: Why Network Boot?

At its heart, PXE allows a client machine to boot an operating system or utility directly from a server over a network, without needing local storage or media. This offers several compelling advantages:

*   **Centralized Control:** Manage all your bootable images from a single server.
*   **Rapid Deployment:** Accelerate OS installations and updates across numerous machines.
*   **Streamlined Troubleshooting:** Quickly boot into diagnostic or recovery environments.
*   **Efficiency:** Reduce the need for physical media (CDs, DVDs, USB drives).
*   **Scalability:** Ideal for data centers, labs, or environments with many bare-metal machines.

## Understanding the Core Components of PXE Boot

For a successful PXE boot, several network services must work in harmony. Think of them as a relay race, with each component passing the baton to the next until the client is fully booted.

### 1. DHCP Server (Dynamic Host Configuration Protocol)

The journey begins here. When a PXE-enabled client powers on and requests a network boot, it sends a DHCP request. The DHCP server, in addition to assigning an IP address, subnet mask, and gateway, provides two crucial pieces of information specific to PXE:

*   **`next-server` (or `siaddr`):** The IP address of the TFTP server.
*   **`filename` (or `boot-file`):** The name of the initial boot file (e.g., `pxelinux.0` or `grubnet.efi`).

Without these, the client wouldn't know where to get its bootloader or from which server.

### 2. TFTP Server (Trivial File Transfer Protocol)

Once the client receives the TFTP server's IP and the boot filename from DHCP, it uses TFTP to download that initial boot file. TFTP is a very simple, UDP-based file transfer protocol, designed for minimal overhead. It's suitable for small files like the PXE bootloader, its configuration files, and potentially the kernel/initramfs for very small boot images.

**Why not HTTP or NFS for the initial boot?** TFTP is simpler and often implemented directly into firmware, making it universally accessible at the earliest stages of a network boot. For larger files, it's inefficient and lacks features like error recovery.

### 3. HTTP/NFS/SMB Server (For ISO Content)

While TFTP handles the initial boot files, it's not practical for transferring entire ISO images or the bulk of an operating system's installation files. This is where more robust file sharing protocols come into play:

*   **NFS (Network File System):** Popular in Linux environments, NFS allows you to mount a remote directory (where your ISO content is extracted or loop-mounted) as if it were local.
*   **HTTP (Hypertext Transfer Protocol):** A web server can serve files efficiently. Many modern Linux installers and live environments support booting from an HTTP source.
*   **SMB/CIFS (Server Message Block / Common Internet File System):** Primarily used for Windows file sharing, but less common for PXE booting ISO content directly in a generic PXE setup. Microsoft's WDS (Windows Deployment Services) utilizes SMB-like functionality internally.

For booting from an ISO, you'll either "loop mount" the ISO and serve its contents via NFS/HTTP, or extract the ISO's contents to a directory and serve that.

### 4. PXE Bootloader (e.g., PXELINUX, GRUB)

This is the program that the client downloads via TFTP. It's responsible for presenting the boot menu, loading the kernel and initramfs, and passing specific boot parameters (like where to find the rest of the installation files via NFS/HTTP).

*   **PXELINUX:** Part of the SYSLINUX project, it's historically very popular for PXE booting Linux distributions.
*   **GRUB (GRand Unified Bootloader):** Increasingly common, especially with UEFI systems, GRUB offers more advanced features and a unified boot experience across various architectures and boot methods.

### 5. PXE Client

The machine you intend to boot. Its firmware (BIOS/UEFI) must support PXE, and network boot must be enabled and prioritized in the boot order.

## Prerequisites for Your PXE Server Setup

Before we dive into the steps, ensure you have:

*   **A Dedicated Server:** A physical machine or virtual machine (e.g., running Ubuntu Server, Debian, CentOS, or Fedora) to host your DHCP, TFTP, and HTTP/NFS services. This server should have a static IP address.
*   **Network Connectivity:** The PXE server and clients must be on the same network segment, or your network must be configured to relay DHCP/PXE requests across subnets (DHCP Relay Agent/IP Helper).
*   **An ISO Image:** Choose the operating system or utility ISO you wish to boot (e.g., Ubuntu Server, CentOS Stream, Debian Installer, SystemRescueCD).

**Note:** If you already have a DHCP server on your network (e.g., your router), you'll need to configure it to point to your new PXE/TFTP server, or disable its DHCP service and let your new PXE server handle DHCP for the relevant subnet. Running two DHCP servers on the same segment without careful configuration will lead to conflicts.

## Step-by-Step Guide: Setting Up a PXE Server (Linux Focus)

We'll use Ubuntu Server as our PXE server OS for this guide.

### Step 1: Install and Configure the DHCP Server (ISC DHCPD)

First, install `isc-dhcp-server`:

```bash
sudo apt update
sudo apt install isc-dhcp-server
```

Next, edit the DHCP server configuration file, typically located at `/etc/dhcp/dhcpd.conf`. You'll need to define a subnet, range, and crucially, the PXE-specific options.

```bash
# /etc/dhcp/dhcpd.conf

# Global declarations
ddns-update-style none;
default-lease-time 600;
max-lease-time 7200;
log-facility local7;

# Specify your network interface (e.g., ens160, eth0)
# This is typically set in /etc/default/isc-dhcp-server (see next step)
# INTERFACESv4="ens160"

# Define the subnet for PXE clients
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200; # IP range for clients
    option routers 192.168.1.1;       # Your network's gateway
    option domain-name-servers 8.8.8.8, 8.8.4.4; # DNS servers

    # PXE-specific options
    next-server 192.168.1.10;         # **IP address of your TFTP server** (your PXE server's IP)
    filename "pxelinux.0";            # **Initial boot file** (for BIOS/Legacy PXE)

    # For UEFI PXE boot, you might use conditional logic:
    # class "pxeclients" {
    #     match if substring(option vendor-class-identifier, 0, 9) = "PXEClient";
    #     if option arch = 00:07 or option arch = 00:09 { # UEFI 64-bit
    #         filename "grubnetx64.efi";
    #     } else { # BIOS / Legacy PXE
    #         filename "pxelinux.0";
    #     }
    # }
}
```

**Important:** Replace `192.168.1.x` with your actual network details. The `next-server` value **must be the IP address of your PXE server**.

Now, specify the network interface the DHCP server should listen on. Edit `/etc/default/isc-dhcp-server`:

```bash
# /etc/default/isc-dhcp-server
# Add your network interface here (e.g., ens160, eth0)
INTERFACESv4="ens160"
# INTERFACESv6=""
```

Finally, restart the DHCP service:

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
```

Check its status to ensure it's running without errors: `sudo systemctl status isc-dhcp-server`.

### Step 2: Install and Configure the TFTP Server (TFTP-HPA)

Install the TFTP server and SYSLINUX utilities (which contain PXELINUX):

```bash
sudo apt install tftpd-hpa syslinux-common
```

Configure `tftpd-hpa` by editing `/etc/default/tftpd-hpa`:

```bash
# /etc/default/tftpd-hpa
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/srv/tftp" # This will be your TFTP root directory
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure --create" # --create allows file creation (e.g. for uploads, not strictly needed for PXE)
```

Create the TFTP root directory and set appropriate permissions:

```bash
sudo mkdir -p /srv/tftp
sudo chown -R tftp:tftp /srv/tftp
sudo chmod -R 755 /srv/tftp
```

Copy the necessary PXELINUX boot files to the TFTP root:

```bash
sudo cp /usr/lib/syslinux/modules/bios/pxelinux.0 /srv/tftp/
sudo cp /usr/lib/syslinux/modules/bios/ldlinux.c32 /srv/tftp/
sudo cp /usr/lib/syslinux/modules/bios/libutil.c32 /srv/tftp/
sudo cp /usr/lib/syslinux/modules/bios/libmenu.c32 /srv/tftp/
sudo cp /usr/lib/syslinux/modules/bios/menu.c32 /srv/tftp/
sudo cp /usr/lib/syslinux/modules/bios/vesamenu.c32 /srv/tftp/ # For graphical menu
# Note: Paths to syslinux modules might vary slightly based on distribution version.
# Use `dpkg -L syslinux-common` to find them if needed.
```

Create the `pxelinux.cfg` directory for boot menus:

```bash
sudo mkdir /srv/tftp/pxelinux.cfg
```

Restart the TFTP service:

```bash
sudo systemctl restart tftpd-hpa
sudo systemctl enable tftpd-hpa
```

### Step 3: Set up an HTTP Server (Apache2) for ISO Content

While NFS is great for Linux, HTTP is often preferred for its simplicity and wider client support, especially for live images.

```bash
sudo apt install apache2
```

Create a directory to store your ISOs or their extracted contents:

```bash
sudo mkdir -p /srv/http/isos
sudo chown -R www-data:www-data /srv/http
```

Now, let's prepare an ISO. We'll use the Ubuntu Server 22.04 LTS ISO as an example. Download it to your PXE server, then mount it and copy its contents.

```bash
# Example: Download Ubuntu Server ISO
# wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso -P /tmp

# Create a mount point and mount the ISO
sudo mkdir -p /mnt/ubuntu_iso
sudo mount -o loop /tmp/ubuntu-22.04.3-live-server-amd64.iso /mnt/ubuntu_iso

# Copy relevant files (kernel and initrd) to TFTP for initial boot
# These paths are specific to Ubuntu live server ISOs.
sudo cp /mnt/ubuntu_iso/casper/vmlinuz /srv/tftp/ubuntu2204_vmlinuz
sudo cp /mnt/ubuntu_iso/casper/initrd /srv/tftp/ubuntu2204_initrd
# Or for some ISOs, they might be in boot/ or isolinux/

# Copy the entire ISO contents to your HTTP server's directory
# This allows the installer to fetch files via HTTP.
sudo rsync -av /mnt/ubuntu_iso/ /srv/http/isos/ubuntu2204_live/ --exclude=boot/grub/grub.efi # Exclude problematic EFI files if needed
# Note: rsync with -a (archive) preserves permissions and ownership which is important.
# This copies ALL contents, which might be large.

# Unmount the ISO
sudo umount /mnt/ubuntu_iso
```

**Alternative: Serve the ISO file directly via HTTP**
Some distributions (like SystemRescueCD or older Debian installers) can loop-mount the ISO from an HTTP server directly. In such cases, you'd just place the ISO file in `/srv/http/isos/` and reference it in your PXELINUX config. However, for most modern installers like Ubuntu Live Server, serving the extracted content is the standard approach.

### Step 4: Configure PXELINUX Boot Menu

Now, we create the `default` configuration file for PXELINUX in `/srv/tftp/pxelinux.cfg/default`. This file defines the boot menu options.

```bash
# /srv/tftp/pxelinux.cfg/default

UI vesamenu.c32        # Use graphical menu
MENU TITLE PXE Boot Menu
MENU BACKGROUND /background.png # Optional: if you have a background image in /srv/tftp

TIMEOUT 600            # Wait 60 seconds before booting default entry
DEFAULT local          # Default boot option if timeout expires

LABEL ubuntu_2204_live
    MENU LABEL ^Ubuntu 22.04 Live Server
    LINUX ubuntu2204_vmlinuz
    INITRD ubuntu2204_initrd
    APPEND root=/dev/ram0 ip=dhcp url=http://192.168.1.10/isos/ubuntu2204_live/ autoinstall ds=nocloud-net;s=http://192.168.1.10/isos/ubuntu2204_live/ --- # Note: autoinstall parameters specific to Ubuntu 22.04
    # The `url` parameter tells the installer where to find the source files via HTTP.
    # Replace 192.168.1.10 with your PXE server's IP.

# For a different distribution, e.g., Debian Installer:
# LABEL debian_installer
#     MENU LABEL ^Debian 11 Installer
#     LINUX debian11_vmlinuz
#     INITRD debian11_initrd
#     APPEND vga=788 netcfg/dhcp_hostname=debian-pxe locale=en_US keyboard-layouts=us interface=auto url=http://192.168.1.10/isos/debian11_netinst/

LABEL local
    MENU LABEL ^Boot from local disk
    LOCALBOOT 0
```

**Explanation of `APPEND` parameters for Ubuntu 22.04 Live Server:**

*   `root=/dev/ram0`: Tells the kernel to use a RAM disk as the root filesystem.
*   `ip=dhcp`: Configures the network interface using DHCP.
*   `url=http://192.168.1.10/isos/ubuntu2204_live/`: **Crucial.** This tells the Ubuntu installer where to find the installation files (the extracted ISO content) via HTTP. **Change `192.168.1.10` to your PXE server's IP address.**
*   `autoinstall ds=nocloud-net;s=http://192.168.1.10/isos/ubuntu2204_live/`: These are specific to Ubuntu's `subiquity` installer for automated installations (cloud-init style). `ds=nocloud-net` tells it to look for `user-data` and `meta-data` from the network. `s=` provides the source URL for these configuration files. You might want to omit `autoinstall` if you prefer a manual installation, or customize it for unattended installs. [Reference: Ubuntu Server Autoinstall](https://ubuntu.com/server/docs/install/autoinstall)
*   `---`: Separates kernel parameters from `initrd` parameters.

### Step 5: Configure Firewall (UFW)

Ensure your firewall allows the necessary traffic:

```bash
sudo ufw allow dhcp
sudo ufw allow tftp
sudo ufw allow http
# If you were using NFS:
# sudo ufw allow nfs
sudo ufw enable
```

Verify firewall status: `sudo ufw status`.

### Step 6: Client Configuration and Testing

Now, power on your client machine and ensure its BIOS/UEFI is configured to boot from the network (PXE/LAN boot).

1.  **Enter BIOS/UEFI Setup:** Typically by pressing `F2`, `Del`, `F10`, or `F12` during boot.
2.  **Adjust Boot Order:** Prioritize "Network Boot," "PXE Boot," or "LAN Boot" over local drives.
3.  **Save and Exit:** The client should attempt to acquire an IP address via DHCP, then download the `pxelinux.0` file from your TFTP server, and finally present the PXELINUX boot menu.
4.  **Select Your ISO:** Choose the "Ubuntu 22.04 Live Server" option from the menu. The installer should then fetch the rest of its files from your HTTP server.

## Advanced Considerations and Troubleshooting

### UEFI vs. BIOS (Legacy) PXE

The `filename` option in your DHCP configuration is critical.

*   **BIOS (Legacy):** `filename "pxelinux.0"` (from SYSLINUX)
*   **UEFI (64-bit):** `filename "grubnetx64.efi"` (from GRUB) or `filename "lpxelinux.0"` (a different SYSLINUX variant for EFI)
*   **UEFI (32-bit):** `filename "grubnetia32.efi"`

You can add conditional logic to your `dhcpd.conf` using `option arch` to serve the correct bootloader based on the client's architecture, as shown in the commented section in Step 1.

For GRUB, you'd typically install `grub-efi-amd64-bin` and copy `grubnetx64.efi` (or similar) to your TFTP root, then create a `grub.cfg` file in `/srv/tftp/grub/` (or similar) that mirrors the PXELINUX configuration structure but uses GRUB syntax.

### Windows Deployment Services (WDS)

Note: While technically possible to PXE boot a Windows PE environment or even a full Windows installer using iPXE and `wimboot`, it is significantly more complex than Linux. For mass deploying Windows, Microsoft's **Windows Deployment Services (WDS)** or **Microsoft Endpoint Configuration Manager (MECM, formerly SCCM)** are the standard, integrated solutions. These services manage the nuances of Windows imaging and PXE booting, often leveraging Active Directory and more complex file sharing (SMB/WIM format).

### Debugging PXE Issues

*   **DHCP Issues:**
    *   Check `sudo journalctl -u isc-dhcp-server` for errors.
    *   Use `tcpdump -i <your_interface> port 67 or port 68` on the server to see DHCP requests and offers.
    *   Ensure no other DHCP server is active on the network segment.
*   **TFTP Issues:**
    *   Check `sudo journalctl -u tftpd-hpa` for errors.
    *   Try to manually `tftp` a file from the server to itself: `tftp 127.0.0.1; get pxelinux.0; quit`.
    *   Use `tcpdump -i <your_interface> port 69` to monitor TFTP traffic.
    *   Verify file permissions in `/srv/tftp`. TFTP typically runs with `tftp` user permissions.
*   **HTTP/NFS Issues:**
    *   Check web server logs (`/var/log/apache2/error.log`, `/var/log/apache2/access.log`).
    *   Try accessing the HTTP URL from a browser on another machine (`http://192.168.1.10/isos/ubuntu2204_live/`).
    *   For NFS, check `sudo exportfs -v` and `sudo showmount -e <PXE_SERVER_IP>` from a client if troubleshooting.
*   **Client Boot Order:** Double-check BIOS/UEFI settings. Some older machines might have unreliable PXE implementations.
*   **Firewall:** Always remember to check firewall rules on the PXE server!

## Use Cases for Network Booting

Beyond simple OS installation, PXE opens doors to many possibilities:

*   **Bare Metal Provisioning:** Automate the entire OS installation process on new hardware.
*   **Network Diagnostics:** Boot a live Linux environment (like SystemRescueCD) to diagnose network issues, recover data, or fix corrupted systems without touching the local disk.
*   **Disk Imaging:** Deploy or capture disk images using tools like Clonezilla over the network.
*   **Centralized Utility Server:** Host various tools (memory testers, hardware diagnostics, firmware updaters) for quick access.

## Conclusion

Setting up a PXE server might seem daunting at first, with its various interacting components. However, by understanding the roles of DHCP, TFTP, and an HTTP/NFS server, and following a methodical approach, you can build a powerful infrastructure for network booting.

The ability to boot from ISOs over the network is a fundamental skill for efficient system administration, enabling faster deployments, easier troubleshooting, and centralized management of your boot environments. Experiment with different ISOs and explore advanced bootloader configurations to fully unlock the potential of PXE in your network. Happy booting!
