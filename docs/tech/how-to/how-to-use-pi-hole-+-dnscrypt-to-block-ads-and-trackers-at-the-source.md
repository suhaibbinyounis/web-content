---
categories:
- Networking
- DevOps
- Security
- Privacy
comments: true
cover:
  image: https://images.pexels.com/photos/5475786/pexels-photo-5475786.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Elevate your network's privacy and security by combining Pi-hole for
  ad and tracker blocking with DNSCrypt-proxy for encrypted, private DNS lookups.
  A step-by-step guide for developers.
tags:
- Linux
- CLI
- Bash
- DNS
- Security
- Privacy
- Ad-blocking
- Networking
- DevOps
- Raspberry Pi
title: How to Use Pi-hole + DNSCrypt to Block Ads and Trackers at the Source
---

![A tech-savvy individual using a laptop in a neon-lit room, symbolizing cybersecurity.](https://images.pexels.com/photos/5475786/pexels-photo-5475786.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "A tech-savvy individual using a laptop in a neon-lit room, symbolizing cybersecurity.")

## How to Use Pi-hole + DNSCrypt to Block Ads and Trackers at the Source

Ad-blocking and privacy are hot topics, especially for developers who understand the underlying mechanics of tracking. While browser extensions offer some protection, they're limited to your browser and often miss trackers or ads embedded in other applications or devices. Enter Pi-hole: a network-wide ad blocker that acts as a DNS sinkhole. But what about the privacy of your DNS queries themselves? That's where DNSCrypt-proxy comes in, encrypting your DNS traffic to prevent eavesdropping and manipulation.

This post will guide you through setting up Pi-hole alongside DNSCrypt-proxy, creating a robust, private, and ad-free network environment. We'll focus on CLI examples, because that's how we roll.

## Why Pi-hole and DNSCrypt Together?

**Pi-hole** operates as a DNS server. When a device on your network requests to resolve a domain (e.g., `google-analytics.com`), Pi-hole checks its blocklists. If the domain is malicious or an ad/tracker, Pi-hole returns an invalid or your own IP address, effectively "sinking" the request. All other legitimate queries are forwarded to an upstream DNS server.

**DNSCrypt-proxy** is a flexible DNS proxy that adds a layer of encryption between your DNS server (Pi-hole, in this case) and the actual upstream DNS resolver. It prevents man-in-the-middle attacks, DNS spoofing, and ensures your DNS queries aren't easily intercepted or logged by your ISP. It also supports DNSSEC validation.

**The Power Couple**: Pi-hole handles the ad/tracker filtering locally, while DNSCrypt-proxy ensures that Pi-hole's outbound DNS queries are private and secure. This means you get network-wide ad blocking *and* encrypted DNS lookups, protecting your browsing habits from your ISP or other intermediaries.

## Prerequisites

Before we dive in, make sure you have the following:

*   **A dedicated Linux machine**: A Raspberry Pi is perfect, but any Debian/Ubuntu-based system (VM, old PC, cloud instance) will work. Ensure it has a stable internet connection.
*   **Static IP Address**: Your Pi-hole server needs a static IP address on your local network. This is crucial as all your devices will point to it for DNS resolution.
*   **Sudo Privileges**: You'll need `sudo` access to install and configure software.
*   **Basic Linux CLI knowledge**: Familiarity with `apt`, `curl`, `systemctl`, and text editors like `nano` or `vim`.

### Setting a Static IP Address (Example: Ubuntu/Debian with `netplan`)

How you set a static IP depends on your Linux distribution. For modern Debian/Ubuntu systems, `netplan` is common.
Note: Adjust `enpXsY` to your actual network interface name (use `ip a` to find it).

```bash
# Backup original netplan config
sudo cp /etc/netplan/00-installer-config.yaml /etc/netplan/00-installer-config.yaml.bak

# Open the netplan configuration file
sudo nano /etc/netplan/00-installer-config.yaml
```

Modify the file to look something like this, replacing values with your network's specifics:

```yaml
# /etc/netplan/00-installer-config.yaml
network:
  renderer: networkd
  ethernets:
    enp0s3: # Your actual network interface name (e.g., eth0, enp3s0)
      dhcp4: no
      addresses:
        - 192.168.1.10/24 # Your desired static IP and subnet mask
      routes:
        - to: default
          via: 192.168.1.1 # Your router's IP address (gateway)
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8] # Temporary DNS servers for internet access during setup
  version: 2
```

Apply the changes:

```bash
sudo netplan try
# If no errors, apply
sudo netplan apply
```

```output
# Example output from netplan apply
Configuration accepted.
```

Verify your IP address:

```bash
ip a show enp0s3 # Replace enp0s3 with your interface name
```

```output
# Example output
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.10/24 brd 192.168.1.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fecf:xx:xx/64 scope link 
       valid_lft forever preferred_lft forever
```

## Step 1: Install Pi-hole

The Pi-hole project provides a convenient one-liner for installation.

```bash
curl -sSL https://install.pi-hole.net | sudo bash
```

```output
# Example output (interactive installer will follow)
  [i] Root user detected
  [i] Script will now continue with root privileges
  .
  .
  .
  [i] Installer will now start dialog based setup.
```

**During the interactive setup:**

1.  **Static IP Check**: Confirm your static IP.
2.  **Upstream DNS Provider**: This is important. For now, select a public, unencrypted DNS provider like **Cloudflare (1.1.1.1)** or **Google (8.8.8.8)**. We will change this to point to `dnscrypt-proxy` later.
3.  **Ad Lists**: Keep the default lists.
4.  **Protocols**: Select both IPv4 and IPv6 if you use both.
5.  **Web Admin Interface**: Highly recommended.
6.  **Lighttpd web server**: Recommended.
7.  **Log Queries**: Recommended.

Make a note of the **Admin Web Panel Password** displayed at the end of the installation.

### Test Pi-hole (Initial)

After installation, your Pi-hole is running and listening on port 53.
Access the web interface at `http://YOUR_PIHOLE_IP/admin` using the password noted earlier.

Now, configure a client (e.g., your computer, phone) to use your Pi-hole's static IP address as its sole DNS server. Then, try browsing some ad-heavy websites. You should see an increase in "Queries Blocked" on the Pi-hole dashboard.

```bash
# On your Pi-hole server, you can tail the Pi-hole log:
pihole -t
```

```output
# Example output when a client queries and Pi-hole blocks
Oct 27 10:30:00 dnsmasq[1234]: query[A] doubleclick.net from 192.168.1.15
Oct 27 10:30:00 dnsmasq[1234]: /etc/pihole/gravity.list doubleclick.net is 0.0.0.0
Oct 27 10:30:05 dnsmasq[1234]: query[A] example.com from 192.168.1.15
Oct 27 10:30:05 dnsmasq[1234]: forwarded example.com to 1.1.1.1
Oct 27 10:30:05 dnsmasq[1234]: reply example.com is <CNAME>
Oct 27 10:30:05 dnsmasq[1234]: reply example.com is 93.184.216.34
```

## Step 2: Install and Configure DNSCrypt-Proxy

Now for the encryption layer. We'll download the latest `dnscrypt-proxy` binary and set it up.

### Download and Extract

Visit the [DNSCrypt-proxy GitHub Releases page](https://github.com/DNSCrypt/dnscrypt-proxy/releases) and find the latest stable release for your architecture (e.g., `linux-arm64` for Raspberry Pi 64-bit, `linux-x86_64` for most desktops/servers).

```bash
# Example for linux-x86_64; adjust URL and filename as needed
cd /tmp
DN_VERSION="2.1.4" # Check for the latest version
DN_ARCH="linux-x86_64" # Or linux-arm, linux-arm64, etc.
wget "https://github.com/DNSCrypt/dnscrypt-proxy/releases/download/${DN_VERSION}/dnscrypt-proxy-${DN_ARCH}-${DN_VERSION}.tar.gz"

tar -xzf "dnscrypt-proxy-${DN_ARCH}-${DN_VERSION}.tar.gz"
cd "dnscrypt-proxy-${DN_ARCH}-${DN_VERSION}"

# Copy the executable and example config
sudo cp dnscrypt-proxy /usr/local/bin/
sudo cp example-dnscrypt-proxy.toml /etc/dnscrypt-proxy.toml
```

```output
# Example output for copy
'dnscrypt-proxy' -> '/usr/local/bin/dnscrypt-proxy'
'example-dnscrypt-proxy.toml' -> '/etc/dnscrypt-proxy.toml'
```

### Configure DNSCrypt-Proxy

Now we edit the `dnscrypt-proxy.toml` configuration file. This is where you specify which encrypted DNS resolvers to use and what port DNSCrypt-proxy will listen on.

**Crucial Point**: Pi-hole already listens on port 53. DNSCrypt-proxy *must* listen on a different port, e.g., `5353`.

```bash
sudo nano /etc/dnscrypt-proxy.toml
```

Find and modify the following lines:

1.  **`listen_addresses`**: Change this to `127.0.0.1:5353` (or any other unused port). This means DNSCrypt-proxy will listen *locally* on this port, and Pi-hole will send queries to it.
    ```toml
    listen_addresses = ['127.0.0.1:5353']
    ```

2.  **`server_names`**: This is a list of DNSCrypt-compatible resolvers. You can find a full list in `dnscrypt-proxy/dnscrypt-proxy.toml` (from the extracted archive) or online. Choose a few reliable, fast, and privacy-focused ones.
    ```toml
    server_names = ['cloudflare', 'google', 'quad9-ecs', 'opendns'] # Example, pick ones you trust
    ```
    Note: You can uncomment and use the provided `require_dnssec`, `require_nologs`, `require_nofiltering` options to ensure chosen servers meet specific criteria.

3.  Optionally, review and enable/disable features like `fallback_resolvers`, `keepalive`, `timeout`, `log_level`, etc., based on your needs. For most users, the defaults are fine once `listen_addresses` and `server_names` are set.

Save and exit the file.

### Create a Systemd Service

To ensure `dnscrypt-proxy` starts automatically on boot, we'll create a systemd service file.

```bash
sudo nano /etc/systemd/system/dnscrypt-proxy.service
```

Paste the following content:

```ini
[Unit]
Description=DNSCrypt-proxy client
After=network.target

[Service]
ExecStart=/usr/local/bin/dnscrypt-proxy -config /etc/dnscrypt-proxy.toml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Reload systemd daemon, enable, and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dnscrypt-proxy.service
sudo systemctl start dnscrypt-proxy.service
```

```output
# Example output
Created symlink /etc/systemd/system/multi-user.target.wants/dnscrypt-proxy.service → /etc/systemd/system/dnscrypt-proxy.service.
```

### Test DNSCrypt-Proxy

Verify that DNSCrypt-proxy is running and listening on the specified port:

```bash
sudo systemctl status dnscrypt-proxy.service
```

```output
# Example output
● dnscrypt-proxy.service - DNSCrypt-proxy client
     Loaded: loaded (/etc/systemd/system/dnscrypt-proxy.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-10-27 10:45:00 UTC; 1min 2s ago
   Main PID: 12345 (dnscrypt-proxy)
      Tasks: 6 (limit: 4915)
     Memory: 5.6M
        CPU: 0.052s
     CGroup: /system.slice/dnscrypt-proxy.service
             └─12345 /usr/local/bin/dnscrypt-proxy -config /etc/dnscrypt-proxy.toml
```

You can also test it by querying directly:

```bash
dig @127.0.0.1 -p 5353 example.com
```

```output
# Example output
; <<>> DiG 9.16.1-Ubuntu <<>> @127.0.0.1 -p 5353 example.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23456
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 1232
;; QUESTION SECTION:
;example.com.                   IN      A

;; ANSWER SECTION:
example.com.            300     IN      A       93.184.216.34

;; Query time: 15 msec
;; SERVER: 127.0.0.1#5353(127.0.0.1)
;; WHEN: Fri Oct 27 10:48:00 UTC 2023
;; MSG SIZE  rcvd: 55
```

This confirms DNSCrypt-proxy is listening and resolving queries.

## Step 3: Integrate Pi-hole with DNSCrypt-Proxy

This is the final, crucial step: telling Pi-hole to use your local DNSCrypt-proxy instance as its upstream DNS resolver.

### Configure Pi-hole Upstream DNS

Access your Pi-hole web admin interface (e.g., `http://YOUR_PIHOLE_IP/admin`).

1.  Go to **Settings** on the left sidebar.
2.  Navigate to the **DNS** tab.
3.  Under "Upstream DNS Servers", **uncheck any previously selected public DNS servers** (e.g., Cloudflare, Google).
4.  Check the **Custom 1 (IPv4)** box.
5.  In the adjacent text field, enter `127.0.0.1#5353` (replace `5353` if you used a different port for DNSCrypt-proxy).
6.  Click **Save** at the bottom.

This tells Pi-hole: "For any non-blocked queries, send them to DNSCrypt-proxy running on this same machine, listening on port 5353."

### Verify the Integration

Now, all your network's DNS queries will flow like this:

`Client -> Pi-hole (filters ads) -> DNSCrypt-proxy (encrypts) -> External Encrypted DNS Server`

Let's confirm everything is working as expected.

1.  **Check Pi-hole's DNS settings**:
    ```bash
    # On your Pi-hole server
    grep "PIHOLE_DNS_1" /etc/pihole/setupVars.conf
    ```

    ```output
    # Expected output
    PIHOLE_DNS_1=127.0.0.1#5353
    ```

2.  **Monitor DNSCrypt-proxy logs**:
    ```bash
    journalctl -u dnscrypt-proxy.service -f
    ```
    This will show you real-time logs of queries being processed by DNSCrypt-proxy. When you visit a website from a client device configured to use Pi-hole, you should see corresponding queries here.

    ```output
    # Example output from dnscrypt-proxy logs
    Oct 27 10:55:00 dnscrypt-proxy[12345]: [2023-10-27 10:55:00] [NOTICE] Querying 'google.com' (A) from 127.0.0.1:45678 with provider cloudflare-dns
    Oct 27 10:55:00 dnscrypt-proxy[12345]: [2023-10-27 10:55:00] [NOTICE] Querying 'example.org' (A) from 127.0.0.1:45679 with provider quad9-ecs
    ```

3.  **Perform a DNS Leak Test**:
    From a client device configured to use Pi-hole for DNS, visit a site like [dnsleaktest.com](https://dnsleaktest.com). Run the "Extended test".

    *   You **should not** see your ISP's DNS servers.
    *   You **should** see the IP addresses corresponding to the encrypted DNS providers you selected in `dnscrypt-proxy.toml` (e.g., Cloudflare, Google, Quad9, OpenDNS). This confirms your queries are being encrypted and routed through DNSCrypt-proxy.

## Troubleshooting Common Issues

*   **No Internet Access on Clients**:
    *   Double-check your client's DNS settings point to the Pi-hole's static IP.
    *   Ensure Pi-hole is running (`sudo systemctl status pihole-FTL.service`).
    *   Ensure DNSCrypt-proxy is running (`sudo systemctl status dnscrypt-proxy.service`).
    *   Verify Pi-hole's upstream DNS is correctly set to `127.0.0.1#5353`.
    *   Check firewall rules (`sudo ufw status` or `sudo iptables -L`). Ensure port 53 (for Pi-hole) and the port DNSCrypt-proxy is using are open, if necessary (usually not needed for `127.0.0.1` binding).

*   **DNSCrypt-proxy not starting**:
    *   Check `journalctl -u dnscrypt-proxy.service` for errors.
    *   Ensure the `ExecStart` path in the systemd unit file is correct (`/usr/local/bin/dnscrypt-proxy`).
    *   Verify the `dnscrypt-proxy.toml` file has correct syntax and no conflicts (e.g., port already in use).

*   **Pi-hole Dashboard not showing queries**:
    *   Clients are likely not using Pi-hole for DNS.
    *   Check your router's DHCP settings to ensure it's advertising Pi-hole as the DNS server.
    *   Restart Pi-hole (`sudo pihole restartdns`).

## Maintenance and Advanced Tips

*   **Updating Pi-hole**:
    ```bash
    pihole -up
    ```
*   **Updating DNSCrypt-proxy**: This is a manual process. Download the latest binary from GitHub, replace the old one in `/usr/local/bin/`, and restart the service (`sudo systemctl restart dnscrypt-proxy.service`).
*   **Adding More Blocklists**: In the Pi-hole web interface, go to **Settings > Adlists**. You can find many community-curated blocklists online (e.g., on GitHub). After adding, run `pihole -g` to update gravity.
*   **Conditional Forwarding**: If you have local hostnames (e.g., from your router's DHCP server or a local DNS server), you can enable "Conditional Forwarding" in Pi-hole's DNS settings. This tells Pi-hole to forward queries for your local domain (e.g., `local` or `lan`) to your router's IP.
*   **Set Router's DNS**: The most effective way to ensure all devices on your network use Pi-hole is to configure your router's DHCP settings to advertise your Pi-hole's IP address as the primary DNS server. This way, all devices (including IoT gadgets) automatically benefit from blocking and privacy.

## Conclusion

By combining Pi-hole's network-wide ad and tracker blocking with DNSCrypt-proxy's encrypted DNS lookups, you've built a robust, privacy-centric shield for your entire network. You're not just blocking annoying ads; you're taking control of your DNS traffic, protecting your browsing habits from surveillance, and ensuring the integrity of your DNS resolutions.

This setup empowers you with a level of network control and privacy that's often overlooked. Enjoy a cleaner, faster, and more private internet experience for all your devices.
```
```
```output
# No output for the full blog post, but each code block has an accompanying output.
