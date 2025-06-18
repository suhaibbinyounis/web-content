---
title: Linux Networking Debug Toolbox ip, ss, netstat, and curl
date: 2025-06-17T11:22:34.549Z
description: "Navigate the complexities of Linux networking with this comprehensive guide to essential debugging tools: ip, ss, netstat, and curl. Master the command line to diagnose connectivity issues, inspect routing tables, monitor socket states, and test application-layer protocols."
tags:
  - Linux
  - Networking
  - Debugging
  - Troubleshooting
  - Command Line
  - ip
  - ss
  - netstat
  - curl
  - System Administration
categories:
  - Linux
  - Networking
  - System Administration
  - Development
comments: true
---

Navigating the intricate world of Linux networking can feel like plumbing a multi-layered system with invisible pipes. When something goes wrong – be it an unreachable service, sluggish connectivity, or a misconfigured firewall – the ability to quickly diagnose the issue is paramount. Fortunately, Linux provides a robust set of command-line tools that offer deep insights into every aspect of the network stack.

This post will delve into four indispensable tools: `ip`, `ss`, `netstat`, and `curl`. While each serves a distinct purpose, together they form a powerful debugging arsenal, allowing you to move from low-level link issues to application-layer connectivity problems with precision.

## The Modern Maestro: `ip`

The `ip` command is part of the `iproute2` utility suite and has largely superseded older tools like `ifconfig` and `route`. Its design reflects a more modern approach to network configuration and information retrieval, providing a unified and extensive interface for managing almost every aspect of networking on a Linux system. It's the first tool you should reach for when you suspect issues at the network or data link layers.

### Inspecting Network Interfaces and Addresses: `ip addr show`

This command provides a wealth of information about your network interfaces, including their state, IP addresses (IPv4 and IPv6), MAC addresses, and other vital statistics.

```bash
ip addr show
```

**Common Flags:**
*   `ip addr show eth0`: Show details for a specific interface.
*   `ip -s addr show`: Include statistics like packet counts and errors for each address.
*   `ip -details addr show`: Provide even more detailed information, including protocol-specific details.

**Example Output Interpretation:**

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86241sec preferred_lft 86241sec
    inet6 fe80::a00:27ff:fexx:xxxx/64 scope link
       valid_lft forever preferred_lft forever
```

*   `<LOOPBACK,UP,LOWER_UP>`: Indicates interface capabilities and current state. `UP` means the interface is enabled, `LOWER_UP` means the physical layer is up.
*   `mtu 1500`: Maximum Transmission Unit. A common value for Ethernet. Mismatched MTUs can cause fragmentation issues.
*   `qdisc pfifo_fast`: Queuing discipline.
*   `state UP`: The interface is operational. If it's `DOWN`, check cabling or configuration.
*   `inet 192.168.1.100/24`: IPv4 address and subnet mask. `scope global dynamic` means it's a globally routable address obtained via DHCP.
*   `link/ether 08:00:27:xx:xx:xx`: MAC address.

### Examining Link Layer Information: `ip link show`

While `ip addr show` gives address specifics, `ip link show` focuses purely on the device's link layer properties and state.

```bash
ip link show
```

**Common Flags:**
*   `ip -s link show`: Provides transmit and receive statistics for each interface, including errors, dropped packets, and overruns. This is crucial for diagnosing physical layer issues.

**Example Scenario:** If you see a high number of `RX errors` or `dropped` packets with `ip -s link show`, it could indicate a faulty cable, a duplex mismatch, or a congested network card buffer.

### Understanding the Routing Table: `ip route show`

The routing table is the brain of your network, telling your system where to send packets destined for specific IP addresses. When a host is unreachable, the routing table is one of the first places to check.

```bash
ip route show
```

**Common Flags:**
*   `ip route get <IP_ADDRESS>`: Simulates a route lookup for a specific destination IP, showing which interface and gateway would be used. Excellent for debugging routing logic.

**Example Output Interpretation:**

```
default via 192.168.1.1 dev eth0 proto dhcp metric 100
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100 metric 100
```

*   `default via 192.168.1.1 dev eth0`: The default gateway. If a packet's destination isn't covered by a more specific route, it's sent to `192.168.1.1` via `eth0`.
*   `192.168.1.0/24 dev eth0`: A direct route for the local subnet. Packets for this subnet are sent directly out `eth0`.
*   `proto dhcp`/`proto kernel`: Indicates how the route was learned (DHCP, kernel automatically for connected networks).

### Checking the ARP/NDP Cache: `ip neigh show`

The neighbor table (ARP cache for IPv4, NDP cache for IPv6) maps IP addresses to MAC addresses on the local network. Issues here can prevent communication even if IP addresses and routes are correct.

```bash
ip neigh show
```

**Example Output:**

```
192.168.1.1 dev eth0 lladdr 00:50:56:xx:xx:xx REACHABLE
192.168.1.200 dev eth0 lladdr 00:0c:29:yy:yy:yy STALE
```

*   `REACHABLE`: The entry is valid and recently confirmed.
*   `STALE`: The entry is valid but needs re-validation.
*   `FAILED`: The system failed to resolve the MAC address. This indicates a problem with the device itself or network connectivity to it.

**Source**: The `ip` command and `iproute2` utilities are part of the Linux kernel networking stack. For detailed usage, refer to `man ip` or the [Linux Foundation `iproute2` documentation](https://www.linuxfoundation.org/resources/open-source-guides/iproute2-utilities-guide/).

## The Speedy Socket Stats: `ss`

The `ss` command is another powerful tool from the `iproute2` suite. It's designed to be a faster, more efficient replacement for `netstat`, especially on systems with many active network connections. `ss` excels at displaying detailed socket statistics.

### Listing All Sockets: `ss` (basic)

Running `ss` without any arguments will show all open sockets, which can be overwhelming. You'll almost always want to use flags to filter the output.

### Essential Flags for Socket Monitoring: `ss -tunlp`

This combination of flags is a common starting point for debugging socket-related issues.

*   `-t`: Display TCP sockets.
*   `-u`: Display UDP sockets.
*   `-n`: Do not resolve service names (e.g., show port 80 instead of `http`). This makes output faster and often clearer for debugging.
*   `-l`: Display listening sockets (server applications waiting for connections).
*   `-p`: Show the process (PID and name) that owns the socket. Requires root privileges.

```bash
sudo ss -tunlp
```

**Example Output Interpretation:**

```
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port
tcp    LISTEN     0      128    0.0.0.0:22                  0.0.0.0:*       users:(("sshd",pid=874,fd=3))
tcp    ESTAB      0      0      192.168.1.100:22            192.168.1.50:54321 users:(("sshd",pid=1234,fd=4))
udp    UNCONN     0      0      0.0.0.0:68                  0.0.0.0:*       users:(("dhclient",pid=567,fd=6))
```

*   `Netid`: Protocol (tcp, udp, raw, unix).
*   `State`: TCP socket state (LISTEN, ESTAB, CLOSE_WAIT, TIME_WAIT, etc.). This is crucial for understanding connection lifecycles.
*   `Recv-Q`/`Send-Q`: Bytes in the receive/send queue that are not yet processed by the application or sent on the network. Non-zero values can indicate a backlog or performance issue.
*   `Local Address:Port`: The local IP address and port. `0.0.0.0` means listening on all available interfaces.
*   `Peer Address:Port`: The remote IP address and port for established connections.
*   `users:((...))`: The process associated with the socket.

**Common TCP States:**
*   `LISTEN`: The server is waiting for a connection request.
*   `ESTABLISHED`: A connection is active and data can be exchanged.
*   `TIME_WAIT`: The connection has been closed, but the socket is waiting for old packets to clear the network.
*   `CLOSE_WAIT`: The remote end has closed the connection, but the local application hasn't yet. This can indicate an application bug.

### Summarizing Socket Statistics: `ss -s`

For a quick overview of socket usage:

```bash
ss -s
```

This provides counts of various socket types and states, which can be useful for high-level monitoring or detecting runaway connections.

### Advanced Filtering with `ss`

`ss` offers powerful filtering capabilities, allowing you to narrow down results based on state, port, address, and more.

*   **Filter by State:** `ss -t state established` (show only established TCP connections).
*   **Filter by Port:** `ss -nt sport eq :ssh` (show TCP sockets where source port is SSH). `ss -nt dport eq :http` (show TCP sockets where destination port is HTTP).
*   **Combined Filters:** `ss -o state established '( dport = :http or sport = :http )'` (show established connections where either source or destination port is HTTP, and show TCP options).

**Source**: The `ss` command is part of the `iproute2` package. Its man page (`man ss`) is an excellent resource, and numerous online guides explain its advanced filtering.

## The Legacy Workhorse: `netstat`

While `ss` is the modern successor, `netstat` (network statistics) remains widely known and used, particularly on older systems or when working with quick and dirty scripts that haven't been updated. It provides a broad range of network information, though often less detail and performance than `ss`.

Note: `netstat` is considered deprecated by many Linux distributions in favor of `ss`. On newer systems, it might even be a symbolic link to `ss` or part of the `net-tools` package that isn't installed by default.

### Common `netstat` Usage:

*   **Listening and Established Connections (TCP/UDP, Numeric, Process): `netstat -tulnp`**
    This is the direct `netstat` equivalent of `ss -tunlp`.
    *   `-t`: TCP connections.
    *   `-u`: UDP connections.
    *   `-l`: Listening sockets.
    *   `-n`: Numeric addresses (no DNS lookups).
    *   `-p`: Show PID and program name.

    ```bash
    sudo netstat -tulnp
    ```

    The output format is very similar to `ss`.

*   **Routing Table:** `netstat -rn`
    Similar to `ip route show`.
    *   `-r`: Display the routing table.
    *   `-n`: Numeric addresses.

    ```bash
    netstat -rn
    ```

    Example output:
    ```
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 eth0
    192.168.1.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0
    ```

*   **Interface Statistics:** `netstat -ie`
    Provides statistics per interface, similar to `ip -s link show`.
    *   `-i`: Display interface table.
    *   `-e`: Extended information (including hardware address).

    ```bash
    netstat -ie
    ```

    Example output:
    ```
    Kernel Interface table
    Iface   MTU    RX-OK RX-ERR RX-DRP RX-OVR TX-OK TX-ERR TX-DRP TX-OVR Flg
    eth0   1500  1234567      0      0      0 765432      0      0      0 BMRU
    lo    65536    12345      0      0      0  12345      0      0      0 LRU
    ```
    Look for non-zero values in `RX-ERR`, `RX-DRP`, `TX-ERR`, `TX-DRP` which indicate potential issues.

**Source**: `netstat` is part of the `net-tools` package. The man page (`man netstat`) details its extensive options.

## The Versatile Application-Layer Tester: `curl`

While `ip`, `ss`, and `netstat` focus on the network and transport layers, `curl` (Client URL) is your go-to tool for debugging at the application layer, particularly for HTTP/HTTPS, FTP, and other web protocols. It's a command-line tool for transferring data with URLs.

### Basic Connectivity Check: `curl <URL>`

The simplest use case is to just request a URL. If it returns content, you have basic application-layer connectivity.

```bash
curl http://example.com
```

### Verbose Output for Debugging: `curl -v`

This is perhaps the most useful flag for debugging. It shows you the entire communication process, including DNS resolution, connection attempts, SSL/TLS handshake, request headers, server response headers, and data transfer.

```bash
curl -v https://www.google.com
```

**Example Snippets from `curl -v` Output:**
*   `* Rebuilt URL to: https://www.google.com/`
*   `* Trying 142.250.186.100:443...` (DNS resolved, attempting connection)
*   `* Connected to www.google.com (142.250.186.100) port 443 (#0)` (Connection successful)
*   `* ALPN: offers h2`
*   `* TLSv1.3 (OUT), TLS handshake, Client hello (1):` (SSL/TLS handshake details)
*   `< HTTP/2 200` (Server response code)
*   `< date: Fri, 27 Oct 2023 10:30:00 GMT` (Response header)

**Debugging with `-v`:**
*   If `Trying...` hangs or fails, it might be a DNS issue, routing problem, or firewall blocking the connection.
*   If `Connected to...` shows, but then the TLS handshake fails, it could be a certificate issue, incorrect cipher suite, or an SSL proxy problem.
*   If you get an HTTP status code other than `200 OK` (e.g., `403 Forbidden`, `500 Internal Server Error`), the network path is fine, but the server application or its configuration is the problem.

### Head Requests: `curl -I`

To fetch only the HTTP headers, which is faster and less resource-intensive when you just need to check status codes, redirection, or server information.

```bash
curl -I https://www.example.com/robots.txt
```

### Measuring Performance: `curl -o /dev/null -w "%{time_total}\n"`

`curl` can also be used to measure the time taken for various stages of a connection.

```bash
curl -o /dev/null -s -w "Total: %{time_total}s, Connect: %{time_connect}s, Start Transfer: %{time_starttransfer}s\n" http://speedtest.tele2.net/1MB.zip
```

*   `-o /dev/null`: Discards the downloaded output.
*   `-s`: Silent mode (hides progress meter).
*   `-w`: Specifies a custom write-out format, using variables like `time_total`, `time_connect`, `time_starttransfer`, etc. This is incredibly useful for benchmarking.

**Source**: The `curl` command is a free and open-source project. Its official website, [curl.se](https://curl.se/), offers extensive documentation, tutorials, and a complete man page (`man curl`).

## Debugging Workflow: A Holistic Approach

Effective network debugging often involves a systematic approach, moving from the lowest layers of the network stack upwards.

1.  **Is the Interface Up?** Start with `ip addr show` or `ip link show`.
    *   Is the interface in the `UP` state?
    *   Does it have the correct IP address and subnet mask?
    *   Are `RX errors` or `dropped` packets indicating physical issues?
    *   If no IP, check DHCP client (e.g., `dhclient -v`) or static configuration.

2.  **Can I Reach the Gateway/Local Network?** Use `ping` to test basic reachability to your default gateway or another local device.
    *   If `ping` fails, check `ip route show` for correct default gateway.
    *   Check `ip neigh show` to see if the gateway's MAC address is resolved. If it's `FAILED` or empty, you might have an ARP issue or physical connectivity problem to the gateway.

3.  **Can I Reach External Services/DNS?**
    *   Try `ping 8.8.8.8` (Google's DNS) to test external connectivity without DNS.
    *   Try `ping google.com`. If 8.8.8.8 works but `google.com` doesn't, it's likely a DNS issue. Check `/etc/resolv.conf`.

4.  **Are Services Listening?** If you're trying to connect to a service (e.g., web server, SSH), is it actually listening on the expected port and IP address?
    *   Use `sudo ss -tunlp` or `sudo netstat -tulnp`.
    *   Look for the service in `LISTEN` state on `0.0.0.0:PORT` (all interfaces) or `YOUR_IP:PORT`. If it's listening only on `127.0.0.1` (localhost), it won't be reachable from other machines.
    *   Check firewall rules (`sudo ufw status` or `sudo firewall-cmd --list-all` or `sudo iptables -L -n -v`).

5.  **Is the Application Layer Working?** Once you confirm network and transport layers are good, use `curl` to test the actual application protocol.
    *   `curl -v http://your_server_ip:port/path`
    *   Observe the full `curl -v` output for HTTP status codes, redirects, or SSL/TLS errors.
    *   This helps differentiate between a networking problem and an application configuration or code bug.

## Conclusion

Mastering the `ip`, `ss`, `netstat`, and `curl` commands is fundamental for anyone working with Linux systems. While `ip` and `ss` represent the modern approach, offering speed and detailed insight, `netstat` still serves as a familiar fallback. `curl`, on the other hand, bridges the gap from raw network connectivity to application-level functionality, making it invaluable for diagnosing web-related issues.

By understanding the distinct strengths of each tool and employing them systematically, you'll be well-equipped to demystify complex network behaviors, pinpoint problems efficiently, and ensure your Linux systems remain reliably connected. Practice is key; the more you use these commands, the more intuitive their output will become, transforming you into a confident Linux networking troubleshooter.