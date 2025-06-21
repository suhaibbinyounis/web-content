---
categories:
- Networking
- DevOps
comments: true
cover:
  image: https://images.pexels.com/photos/577585/pexels-photo-577585.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Master the art of troubleshooting network problems. This guide breaks
  down internet debugging into a systematic, OSI-layer-by-layer approach, equipping
  you with essential CLI tools and the mindset of a seasoned network engineer.
tags:
- Networking
- CLI
- Linux
- Debugging
- Troubleshooting
- System Administration
title: How to Debug Internet Issues Like a Network Engineer
---

![Eyeglasses reflecting computer code on a monitor, ideal for technology and programming themes.](https://images.pexels.com/photos/577585/pexels-photo-577585.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Eyeglasses reflecting computer code on a monitor, ideal for technology and programming themes.")

## How to Debug Internet Issues Like a Network Engineer

Every developer, at some point, faces the dreaded "internet not working" scenario. It might be your local dev environment failing to pull a Docker image, your CI/CD pipeline timing out on an API call, or simply your browser refusing to load a page. Panic sets in, followed by the inevitable "turn it off and on again" ritual.

While that often works for trivial issues, it won't teach you *why* something broke or how to fix it when it's more complex. This post is for developers who want to move beyond guesswork and debug network issues like a pro—systematically, logically, and with the right tools. We'll approach this like a network engineer, ascending the OSI model layer by layer, understanding what could go wrong at each step, and how to use command-line tools to diagnose it.

### The Network Engineer's Debugging Mindset

Before we dive into commands, let's cultivate the right mindset:

1.  **Don't Panic, Be Systematic**: Haphazardly trying solutions wastes time. Follow a structured approach.
2.  **Isolate the Problem**: Is it just your machine? Just your application? Your entire network? Specific websites?
3.  **Start from the Bottom (OSI Layer 1)**: Think of the OSI model. The problem is usually at the lowest layer possible. Don't check DNS if your Ethernet cable is unplugged.
4.  **Gather Information**: What changed? When did it stop working? Are there any error messages?
5.  **Hypothesize and Test**: Formulate a theory (`"I think it's a DNS issue"`), then use a command to prove or disprove it.
6.  **Document**: Note down what you tried and what the results were. This helps prevent repeating steps and aids future debugging.

Let's begin our ascent.

---

### Layer 1: Physical - "Is It Plugged In?"

This is the most fundamental layer. If there's no physical connection, nothing else will work.

**What to Check**:
*   **Cables**: Are Ethernet cables firmly plugged in? No visible damage?
*   **Lights**: Are the link lights on your network interface card (NIC), modem, and router illuminated and blinking as expected?
*   **Wireless**: Is Wi-Fi enabled on your device? Are you connected to the correct SSID?

**CLI Tool**: `ip link show` (Linux) / `ipconfig` (Windows)

While you can physically inspect cables and lights, your operating system can tell you the *status* of your network interfaces.

#### Example: Checking Link Status on Linux

```bash
ip link show
```

#### Sample Output: `ip link show`
```output
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DORMANT group default qlen 1000
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
```

**Interpretation**:
*   `lo`: The loopback interface, always `UP`.
*   `eth0`: An Ethernet interface. `state UP` and `LOWER_UP` indicate a physical link is established. This is what you want to see for wired connections.
*   `wlan0`: A Wi-Fi interface. `state DOWN` indicates it's not currently connected or enabled.

If your primary interface shows `state DOWN` and it should be `UP`, you've likely found a Layer 1 problem. This could mean a disconnected cable, a disabled Wi-Fi adapter, or a faulty NIC/router port.

#### Example: Checking Link Status on Windows

```bash
ipconfig /all
```

#### Sample Output: `ipconfig /all` (snippet)
```output
Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Realtek PCIe GbE Family Controller
   Physical Address. . . . . . . . . : 00-1A-2B-3C-4D-5E
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::abcd:efgh:ijkl:mnop%10(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.1.100(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.1
   DNS Servers . . . . . . . . . . . : 192.168.1.1
                                       8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled

Wireless LAN adapter Wi-Fi:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Intel(R) Dual Band Wireless-AC 7260
   Physical Address. . . . . . . . . : 00-11-22-33-44-55
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
```

**Interpretation**:
*   `Ethernet adapter Ethernet`: Shows an active connection with IP details.
*   `Wireless LAN adapter Wi-Fi`: `Media State . . . . . . . . . . . : Media disconnected` indicates no wireless link.

---

### Layer 2: Data Link - Local Network Connectivity

Once the physical link is up, the next step is to ensure your device can communicate with other devices on your local network (LAN) using MAC addresses. This is where the Address Resolution Protocol (ARP) comes into play, mapping IP addresses to MAC addresses.

**What to Check**:
*   Can you see your default gateway's MAC address?
*   Are there duplicate IP addresses on the network? (Less common for home users, but a nightmare in larger networks).

**CLI Tools**: `ip neighbour show` (Linux) / `arp -a` (Windows)

These commands display your ARP cache, which contains recently resolved IP-to-MAC mappings.

#### Example: Checking ARP Cache on Linux

```bash
ip neighbour show
```

#### Sample Output: `ip neighbour show`
```output
192.168.1.1 dev eth0 lladdr 00:1a:2b:3c:4d:ff REACHABLE
192.168.1.10 dev eth0 lladdr 00:0a:1b:2c:3d:ee STALE
```

**Interpretation**:
*   `192.168.1.1`: This is likely your router (default gateway). `REACHABLE` means your system has successfully resolved its MAC address and can communicate with it at Layer 2.
*   If your gateway's entry is missing or shows `FAILED`, it indicates a Layer 2 problem, even if Layer 1 looks good.

#### Example: Checking ARP Cache on Windows

```bash
arp -a
```

#### Sample Output: `arp -a`
```output
Interface: 192.168.1.100 --- 0xa
  Internet Address      Physical Address      Type
  192.168.1.1           00-1a-2b-3c-4d-ff     dynamic
  192.168.1.255         ff-ff-ff-ff-ff-ff     static
```

**Interpretation**:
*   `192.168.1.1`: Your gateway's IP, with its corresponding MAC address. This shows successful Layer 2 communication.

---

### Layer 3: Network - IP Addresses, Routing & Hops

This is where your device gets an IP address, knows about its subnet, and learns how to route traffic to networks beyond its immediate LAN. This is crucial for reaching the internet.

**What to Check**:
*   **Your IP Address**: Does your device have a valid IP address from your router?
*   **Default Gateway**: Is your default gateway correctly configured and reachable? This is the "door" to the internet.
*   **External Reachability**: Can you reach external IP addresses (bypassing DNS)?
*   **Routing Table**: Is the routing table configured correctly to send unknown traffic to your default gateway?

**CLI Tools**: `ip a show` (Linux) / `ipconfig` (Windows), `ping`, `traceroute` / `tracert`, `ip route show` (Linux) / `route print` (Windows).

#### Example: Checking Your IP Address

```bash
ip a show eth0 # For a specific interface on Linux
ip a # For all interfaces on Linux

ipconfig # For Windows
```

#### Sample Output: `ip a show eth0`
```output
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86259sec preferred_lft 86259sec
    inet6 fe80::dead:beef:cafe:fedc/64 scope link 
       valid_lft forever preferred_lft forever
```

**Interpretation**:
*   `inet 192.168.1.100/24`: Your device has a valid IPv4 address. The `/24` indicates a subnet mask of 255.255.255.0. If you see an APIPA address (e.g., 169.254.x.x), it means your device couldn't get an IP from a DHCP server.

#### Example: Pinging Your Default Gateway

Find your default gateway from the `ip a` or `ipconfig` output (e.g., `192.168.1.1`).

```bash
ping 192.168.1.1
```

#### Sample Output: `ping 192.168.1.1`
```output
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.871 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.789 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=0.912 ms
^C
--- 192.168.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.789/0.857/0.912/0.053 ms
```

**Interpretation**:
*   `0% packet loss`: This means you can successfully reach your default gateway at Layer 3. If you see `Request timed out` or `Destination Host Unreachable`, there's a problem getting past your local network.

#### Example: Pinging an External IP (Bypassing DNS)

This tests if you can reach the *internet* at a basic level, without relying on DNS resolution. Google's public DNS server (8.8.8.8) is a common choice.

```bash
ping 8.8.8.8
```

#### Sample Output: `ping 8.8.8.8`
```output
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=15.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.9 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=15.1 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 14.992/15.111/15.230/0.100 ms
```

**Interpretation**:
*   If this works but pinging `google.com` (next section) doesn't, you likely have a DNS issue.
*   If this fails, but you can ping your gateway, the problem is likely with your router's connection to the ISP or the ISP's network itself.

#### Example: Tracing the Route to a Destination

`traceroute` (Linux/macOS) or `tracert` (Windows) shows the path (hops) packets take to reach a destination. This helps identify where connectivity breaks.

```bash
traceroute google.com
```

#### Sample Output: `traceroute google.com`
```output
traceroute to google.com (142.250.72.14), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  0.789 ms  0.778 ms  0.767 ms
 2  some-isp-router (10.0.0.1)  12.345 ms  13.111 ms  11.987 ms
 3  another-isp-router (172.16.0.2)  15.678 ms  16.001 ms  15.890 ms
 4  * * *
 5  google-peering-router (142.250.X.Y)  18.123 ms  17.990 ms  17.888 ms
 ...
10  lhr25s30-in-f14.1e100.net (142.250.72.14)  19.567 ms  19.601 ms  19.543 ms
```

**Interpretation**:
*   Each numbered line is a "hop" (router) the packet passes through.
*   If you see `* * *` for multiple consecutive hops, it means packets are being dropped or blocked at that point, indicating where the problem might be.
*   Hop 1 should be your default gateway. If it fails there, your problem is local. If it fails later, it's either your ISP or further along the internet.

#### Example: Checking Your Routing Table

Your routing table tells your operating system how to send traffic to different destinations. The `default` route is crucial—it points to your gateway for all traffic not destined for your local network.

```bash
ip route show # Linux
route print   # Windows (look for "Default Gateway")
```

#### Sample Output: `ip route show`
```output
default via 192.168.1.1 dev eth0 proto dhcp metric 100 
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100 metric 100 
```

**Interpretation**:
*   `default via 192.168.1.1 dev eth0`: This is your default route. It says any traffic not explicitly routed (i.e., internet traffic) should go via `192.168.1.1` (your gateway) out of the `eth0` interface. If this entry is missing or points to the wrong address, you won't reach the internet.

---

### Layer 4: Transport - Ports & Protocols

Assuming you can reach external IP addresses, the next layer deals with how applications communicate using specific ports and protocols (TCP/UDP). This is where firewalls often cause issues.

**What to Check**:
*   Are local or network firewalls blocking outgoing or incoming connections on specific ports?
*   Is a remote service listening on the expected port?

**CLI Tools**: `nc` (netcat), `telnet`, `ss` / `netstat`, firewall commands.

#### Example: Checking Firewall Status (Linux)

If you're using `ufw` (Uncomplicated Firewall) on Ubuntu/Debian:

```bash
sudo ufw status
```

#### Sample Output: `sudo ufw status`
```output
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
```

**Interpretation**:
*   `Status: active` means the firewall is running. Ensure it's not blocking essential outbound connections (or inbound, if you're hosting a service).

#### Example: Testing Remote Port Connectivity with `nc` (Netcat)

`nc` is an incredibly versatile tool, often called the "TCP/IP Swiss Army Knife." Use `nc -zv <host> <port>` to check if a port is open and reachable. `-z` scans for listening daemons, `-v` provides verbose output.

```bash
nc -zv google.com 80
nc -zv example.com 443
nc -zv your_backend_ip 8080 # If trying to reach a specific service
```

#### Sample Output: `nc -zv google.com 80`
```output
Connection to google.com 80 port [tcp/http] succeeded!
```

**Interpretation**:
*   `succeeded!`: The remote server is listening on that port, and your machine can reach it.
*   `Connection refused`: The remote server received your connection attempt but actively refused it (e.g., a firewall on the server side, or the service isn't running).
*   `No route to host` / `timed out`: A firewall is silently dropping your packets, or there's a routing issue further upstream.

#### Example: Checking Listening Ports on Your Machine (`ss` / `netstat`)

This helps if you're trying to host a service and can't connect to it *locally*.

```bash
ss -tuln # For TCP/UDP listening ports (Linux)
netstat -an | findstr "LISTENING" # Windows equivalent
```

#### Sample Output: `ss -tuln`
```output
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port
tcp    LISTEN     0      128    127.0.0.1:6379                 0.0.0.0:*
tcp    LISTEN     0      128    0.0.0.0:22                     0.0.0.0:*
tcp    LISTEN     0      128    0.0.0.0:80                     0.0.0.0:*
tcp    LISTEN     0      128    [::]:80                        [::]:*
```

**Interpretation**:
*   This shows which services are listening on which ports. E.g., `0.0.0.0:80` means a service is listening on port 80, accessible from any IP address on your machine. `127.0.0.1:6379` means Redis is listening, but only for local connections.

#### Example: Viewing Active Connections (`ss` / `netstat`)

If you're initiating a connection that's failing, you can see its state.

```bash
ss -anp # For all active connections with process names (Linux, requires root for process names)
netstat -anb # Windows (requires admin for process names)
```

#### Sample Output: `ss -anp`
```output
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              Process                                       
tcp    ESTAB      0      0      192.168.1.100:44810            142.250.72.14:443              users:(("chrome",pid=12345,fd=67))             
tcp    ESTAB      0      0      192.168.1.100:22               192.168.1.10:54321             users:(("ssh",pid=5432,fd=3))
```

**Interpretation**:
*   `ESTAB`: An established connection. This means everything up to Layer 4 is working for this particular connection.
*   `SYN-SENT`: Your machine sent a SYN packet but didn't get a SYN-ACK back. Could be a firewall, a service not listening, or a network issue.
*   `TIME-WAIT`: The connection is in the process of closing.

---

### Layer 7: Application - DNS, HTTP, Proxies

Even if all lower layers are working, issues at the application layer can prevent internet access. This often involves DNS, HTTP/HTTPS, or proxy configurations.

**What to Check**:
*   **DNS Resolution**: Can you resolve domain names to IP addresses?
*   **HTTP/HTTPS**: Are web requests succeeding? Are you getting correct status codes?
*   **Proxies**: Are you behind a proxy, and is it configured correctly?

**CLI Tools**: `dig`, `nslookup`, `host`, `curl`, `wget`, environment variables.

#### Example: Checking DNS Resolution with `dig`

`dig` (Domain Information Groper) is the gold standard for DNS queries. It shows much more detail than `nslookup`.

```bash
dig google.com
```

#### Sample Output: `dig google.com` (snippet)
```output
; <<>> DiG 9.16.1-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59174
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             234     IN      A       142.250.72.14

;; Query time: 18 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) # This is likely systemd-resolved
;; WHEN: Fri Oct 27 10:30:00 UTC 2023
;; MSG SIZE  rcvd: 55
```

**Interpretation**:
*   `status: NOERROR`: DNS resolution was successful.
*   `ANSWER SECTION`: Shows the resolved IP address (e.g., `142.250.72.14`).
*   `SERVER: 127.0.0.53#53`: This indicates your system is using a local stub resolver (like `systemd-resolved` on Linux) which then forwards queries to your configured DNS servers (often from DHCP).
*   If `status` is `NXDOMAIN` (Non-Existent Domain) or you get no answer, your DNS isn't working. This means your computer can't translate `google.com` into an IP address.

#### Example: Testing DNS Resolution with a Specific DNS Server

If `dig google.com` fails, try querying a public DNS server directly to see if the issue is with your local DNS configuration or the DNS server your system is using.

```bash
dig @8.8.8.8 google.com
```

**Interpretation**:
*   If `dig google.com` fails but `dig @8.8.8.8 google.com` succeeds, your local DNS configuration (`/etc/resolv.conf` on Linux, network adapter settings on Windows) is likely pointing to a non-functional DNS server.

#### Example: Checking HTTP/HTTPS Connectivity with `curl`

`curl` is indispensable for testing web services. Use `-I` to fetch only headers, `-v` for verbose output.

```bash
curl -I https://www.google.com
```

#### Sample Output: `curl -I https://www.google.com`
```output
HTTP/2 200 
date: Fri, 27 Oct 2023 10:35:00 GMT
expires: -1
cache-control: private, max-age=0
content-type: text/html; charset=ISO-8859-1
...
```

**Interpretation**:
*   `HTTP/2 200`: Success! This indicates a healthy HTTP/HTTPS connection.
*   Other status codes (e.g., `404 Not Found`, `500 Internal Server Error`) mean the network connection is fine, but the *application* or *server* has an issue.
*   If it hangs or returns an error like `Could not resolve host` (DNS problem) or `Failed to connect` (Layer 3/4 problem), then the issue is lower down.

#### Example: Checking Proxy Settings

If you're in a corporate environment, you might be behind a proxy server. Incorrect proxy settings can block all internet access.
Proxy settings are often set via environment variables.

```bash
echo $http_proxy    # Linux/macOS
echo $https_proxy   # Linux/macOS
echo $no_proxy      # Linux/macOS

# Windows equivalent (check system environment variables or browser/system proxy settings)
```

**Interpretation**:
*   If these variables are set, ensure the proxy address and port are correct. If you don't need a proxy, ensure they are unset.

---

### Advanced Tools & Techniques

Sometimes, the basic checks aren't enough. These tools give you a deeper look.

#### Packet Sniffing with `tcpdump`

`tcpdump` captures network traffic, allowing you to see the actual packets flowing in and out of your machine. This is invaluable for understanding exactly what's happening on the wire.

**Note**: `tcpdump` can generate a lot of data. Use filters to focus on relevant traffic. Requires `sudo` privileges.

#### Example: Capture HTTPS traffic to google.com

```bash
sudo tcpdump -i eth0 -n -c 10 'host google.com and port 443'
```
*   `-i eth0`: Listen on the `eth0` interface.
*   `-n`: Don't resolve hostnames or port numbers (faster, easier to read IPs).
*   `-c 10`: Capture only 10 packets and then exit.
*   `'host google.com and port 443'`: Filter for traffic specifically to/from `google.com` on port 443.

#### Sample Output: `tcpdump` (snippet)
```output
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
10:45:00.123456 IP 192.168.1.100.54321 > 142.250.72.14.443: Flags [S], seq 12345, win 29200, options [mss 1460,sackOK,TS val 12345678 ecr 0,nop,wscale 7], length 0
10:45:00.123500 IP 142.250.72.14.443 > 192.168.1.100.54321: Flags [S.], seq 67890, ack 12346, win 29200, options [mss 1460,sackOK,TS val 87654321 ecr 12345678,nop,wscale 7], length 0
10:45:00.123600 IP 192.168.1.100.54321 > 142.250.72.14.443: Flags [.], ack 67891, win 229, length 0
...
```

**Interpretation**:
*   This output shows the TCP 3-way handshake (`[S]` for SYN, `[S.]` for SYN-ACK, `[.]` for ACK). If you only see `[S]` packets from your machine but no `[S.]` from the target, your packets aren't reaching the destination or the response is being dropped. This points to a firewall or routing issue.
*   If you see the full handshake, the connection is established at Layer 4, and the issue is likely higher up (e.g., application protocol error).

#### Checking System Logs

Your operating system and applications log network-related events.

*   **Linux**: `journalctl -f -u NetworkManager` (for NetworkManager logs), `dmesg` (kernel messages), `/var/log/syslog` or `/var/log/messages`.
*   **Windows**: Event Viewer (specifically "System" and "Application" logs).

#### Example: Tailoring NetworkManager Logs (Linux)

```bash
journalctl -f -u NetworkManager
```

#### Sample Output: `journalctl -f -u NetworkManager` (snippet)
```output
Oct 27 10:48:00 host NetworkManager[876]: <info>  [1698394080.1234] device (eth0): state change: activated -> deactivating (reason 'user-requested', sys-iface-state: 'managed')
Oct 27 10:48:00 host NetworkManager[876]: <info>  [1698394080.5678] device (eth0): state change: deactivating -> disconnected (reason 'user-requested', sys-iface-state: 'managed')
Oct 27 10:48:01 host NetworkManager[876]: <info>  [1698394081.9012] device (eth0): state change: disconnected -> prepare (reason 'none', sys-iface-state: 'managed')
Oct 27 10:48:01 host NetworkManager[876]: <info>  [1698394081.9999] dhcp4 (eth0): trying dhcp client 'dhclient'
```

**Interpretation**:
*   Look for error messages related to DHCP, DNS, interface state changes, or disconnections.

---

### Structured Debugging Recap

1.  **Is Layer 1 good?** (Cables, lights, `ip link show`/`ipconfig`)
    *   No? Fix physical connection.
2.  **Is Layer 2 good?** (Gateway reachable on LAN, `ip neighbour show`/`arp -a`)
    *   No? Check gateway status, cabling, duplicate IPs.
3.  **Is Layer 3 good?** (IP address, gateway pingable, external IP pingable, `traceroute`, `ip route show`)
    *   No? Check DHCP, router status, ISP connectivity, routing table.
4.  **Is Layer 4 good?** (Firewalls, port reachability, `nc`, `ss`/`netstat`)
    *   No? Check local firewall, remote firewall, service listening.
5.  **Is Layer 7 good?** (DNS, HTTP/HTTPS, proxies, `dig`, `curl`)
    *   No? Check DNS server configuration, proxy settings, web server status.
6.  **Still Stuck?** Use `tcpdump` for deeper packet inspection, check system logs for clues, or contact your ISP (if you've ruled out everything on your end).

### When to Call Your ISP

You've done your due diligence. You can ping your default gateway, but `ping 8.8.8.8` fails, and `traceroute` shows a break immediately after your gateway. You've checked your router's lights and status pages, and everything looks normal on your side. This is when it's appropriate to contact your Internet Service Provider. You'll be able to provide them with precise, technical information about where you think the break is, making their job—and your resolution—much faster.

### Conclusion

Debugging network issues can feel like chasing ghosts, but by adopting a methodical, layer-by-layer approach, you transform it from a frustrating ordeal into a solvable puzzle. The command-line tools covered here are your eyes and ears into the network, giving you the insights needed to diagnose problems efficiently. Practice these commands, understand their outputs, and you'll not only fix your internet faster but also gain a deeper appreciation for how networks truly work. Happy debugging!
