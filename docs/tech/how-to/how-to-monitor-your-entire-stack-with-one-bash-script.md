---
categories:
- DevOps
- Systems
- Scripting
comments: true
cover:
  image: https://images.pexels.com/photos/577585/pexels-photo-577585.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: "Learn to build a simple, effective Bash script to monitor core components\
  \ of your stack \u2013 from CPU and memory to services and network connectivity.\
  \ Understand its power and crucial limitations for real-world use."
tags:
- Linux
- CLI
- Bash
- Scripting
- Monitoring
- DevOps
- SysAdmin
title: How to Monitor Your Entire Stack with One Bash Script
---

![Eyeglasses reflecting computer code on a monitor, ideal for technology and programming themes.](https://images.pexels.com/photos/577585/pexels-photo-577585.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Eyeglasses reflecting computer code on a monitor, ideal for technology and programming themes.")

## How to Monitor Your Entire Stack with One Bash Script

Let's be honest. When you hear "monitor your entire stack with one Bash script," your first thought might be: "That's either brilliant or incredibly naive." And you'd be right on both counts.

While a single Bash script won't replace sophisticated monitoring solutions like Prometheus, Grafana, Zabbix, or ELK, it *can* be an incredibly powerful, pragmatic tool for quick checks, understanding system health, and getting basic alerts on a single server or a small, tightly coupled stack.

This post will show you how to build such a script, highlighting its capabilities and, more importantly, its significant limitations. It's about empowering you with command-line tools and understanding what you can achieve with just Bash, not advocating it as an enterprise-grade solution.

## Why Bash? The Pragmatic Edge

Before we dive into the code, let's establish why Bash is a surprisingly good candidate for basic monitoring:

*   **Ubiquity**: Bash is everywhere. If you have a Linux server, you have Bash. No extra agents or dependencies needed.
*   **Simplicity**: It's text-based. You can `ssh` in, paste a few commands, and get immediate results.
*   **Speed**: Bash scripts execute quickly. They're lightweight and don't consume many resources themselves.
*   **Composability**: Unix philosophy at its best. Bash excels at chaining existing utilities (`grep`, `awk`, `sed`, `cut`, `curl`, `ping`, `systemctl`, `ps`, `df`, `free`, `netstat`, `tail`, etc.) to get exactly the data you need.
*   **Immediate Feedback**: You can run parts of your script directly in the terminal to debug and see output instantly.

The goal here isn't to build a beautiful dashboard, but a raw, actionable set of checks.

## Core Monitoring Concepts: What Are We Looking For?

A "stack" can mean many things. For a basic server, monitoring usually boils down to:

1.  **System Resources**: CPU, Memory, Disk I/O, Disk Space, Load Average.
2.  **Network Connectivity**: Internal and external reachability, open ports.
3.  **Process Health**: Are critical processes running? Are they consuming too many resources?
4.  **Service Status**: Is Nginx running? Is PostgreSQL active?
5.  **Application Health**: Is your web app responding? Is its API healthy?
6.  **Log Analysis**: Basic error/warning detection.

Let's break down how Bash can check each of these.

## Building Blocks: Individual Checks

Each check will be a simple command or a small pipeline. We'll capture its output and interpret it.

### 1. System Resource Monitoring

#### CPU Usage

We'll grab the current CPU usage using `top`. The `-b 1` option runs `top` in batch mode once, and `grep "Cpu(s)"` filters the output.

```bash
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
echo "Current CPU Usage: ${CPU_USAGE}%"
```

```output
Current CPU Usage: 5.7%
```

**Explanation**:
*   `top -bn1`: Run `top` in batch mode, 1 iteration.
*   `grep "Cpu(s)"`: Filter the line showing CPU summary.
*   `sed "s/.*, *\([0-9.]*\)%* id.*/\1/"`: Extract the idle percentage.
*   `awk '{print 100 - $1}'`: Calculate active CPU usage (100 - idle).

#### Memory Usage

`free -h` provides human-readable memory statistics.

```bash
MEM_INFO=$(free -h | awk '/^Mem:/ {print "Total: " $2 ", Used: " $3 ", Free: " $4}')
echo "Memory Info: ${MEM_INFO}"
```

```output
Memory Info: Total: 3.8Gi, Used: 1.5Gi, Free: 1.3Gi
```

**Explanation**:
*   `free -h`: Display memory usage in human-readable format.
*   `awk '/^Mem:/ {print "Total: " $2 ", Used: " $3 ", Free: " $4}'`: Extract total, used, and free memory from the "Mem:" line.

#### Disk Space

`df -h` shows disk space usage. We'll target the root partition.

```bash
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5 " used on " $1}')
echo "Disk Usage: ${DISK_USAGE}"
```

```output
Disk Usage: 45% used on /dev/sda1
```

**Explanation**:
*   `df -h /`: Show disk usage for the root partition in human-readable format.
*   `awk 'NR==2 {print $5 " used on " $1}'`: Select the second line (actual data) and print the usage percentage and filesystem name.

#### Load Average

`uptime` gives the system's load average over 1, 5, and 15 minutes.

```bash
LOAD_AVG=$(uptime | awk -F'load average: ' '{print $2}')
echo "Load Average (1, 5, 15 min): ${LOAD_AVG}"
```

```output
Load Average (1, 5, 15 min): 0.05, 0.08, 0.10
```

**Explanation**:
*   `uptime`: Displays how long the system has been running, number of users, and load averages.
*   `awk -F'load average: ' '{print $2}'`: Uses "load average: " as a field separator and prints the second field, which contains the load averages.

### 2. Network Connectivity Monitoring

#### Ping External Host

Check internet connectivity by pinging a well-known external host.

```bash
PING_TARGET="google.com"
if ping -c 3 -W 1 "${PING_TARGET}" > /dev/null 2>&1; then
    echo "Network Connectivity to ${PING_TARGET}: OK"
else
    echo "Network Connectivity to ${PING_TARGET}: FAILED"
fi
```

```output
Network Connectivity to google.com: OK
```

**Explanation**:
*   `ping -c 3 -W 1`: Send 3 pings, wait 1 second for each response.
*   `> /dev/null 2>&1`: Redirect stdout and stderr to `/dev/null` to suppress output.
*   `if ...; then ... else ... fi`: Standard Bash conditional check based on the `ping` command's exit status.

#### Open Ports Check

Verify a specific port is listening on the server. Useful for services like web servers or databases.

```bash
PORT_TO_CHECK="80" # Example: HTTP port
if netstat -tulnp | grep ":${PORT_TO_CHECK}" | grep -q "LISTEN"; then
    echo "Port ${PORT_TO_CHECK}: LISTENING"
else
    echo "Port ${PORT_TO_CHECK}: NOT LISTENING"
fi
```

```output
Port 80: LISTENING
```

**Explanation**:
*   `netstat -tulnp`: List TCP and UDP connections, numeric output, show listening sockets, show process IDs/names.
*   `grep ":${PORT_TO_CHECK}"`: Filter for lines containing the port number.
*   `grep -q "LISTEN"`: Quietly check if "LISTEN" is present (returns 0 if found, 1 otherwise).

### 3. Process Monitoring

Check if a specific process is running.

```bash
PROCESS_NAME="nginx" # Example: Nginx web server
if pgrep "${PROCESS_NAME}" > /dev/null; then
    echo "Process ${PROCESS_NAME}: RUNNING"
else
    echo "Process ${PROCESS_NAME}: NOT RUNNING"
fi
```

```output
Process nginx: RUNNING
```

**Explanation**:
*   `pgrep "${PROCESS_NAME}"`: Finds process IDs by name. Returns 0 if found, 1 if not.
*   `> /dev/null`: Suppress the PID output.

### 4. Service Status Monitoring

For systems using `systemd` (most modern Linux distributions), `systemctl` is your go-to.

```bash
SERVICE_NAME="mysql" # Example: MySQL database
SERVICE_STATUS=$(systemctl is-active "${SERVICE_NAME}")
if [[ "${SERVICE_STATUS}" == "active" ]]; then
    echo "Service ${SERVICE_NAME}: Active"
else
    echo "Service ${SERVICE_NAME}: ${SERVICE_STATUS}"
fi
```

```output
Service mysql: Active
```

**Explanation**:
*   `systemctl is-active "${SERVICE_NAME}"`: Returns `active`, `inactive`, `failed`, `activating`, etc.
*   `[[ "${SERVICE_STATUS}" == "active" ]]`: Bash's preferred way for string comparison in conditionals.

### 5. Application Health Check (HTTP)

If your application exposes an HTTP endpoint, `curl` is perfect for a quick health check.

```bash
APP_URL="http://localhost:8080/health" # Replace with your app's health endpoint
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "${APP_URL}")

if [[ "${HTTP_CODE}" == "200" ]]; then
    echo "Application Health Check (${APP_URL}): OK (HTTP ${HTTP_CODE})"
else
    echo "Application Health Check (${APP_URL}): FAILED (HTTP ${HTTP_CODE})"
fi
```

```output
Application Health Check (http://localhost:8080/health): OK (HTTP 200)
```

**Explanation**:
*   `curl -s`: Silent mode (don't show progress or error messages).
*   `-o /dev/null`: Discard the response body.
*   `-w "%{http_code}"`: Write the HTTP status code to stdout after the transfer.

### 6. Basic Log Analysis

Quickly check for "error" or "critical" messages in a log file.

```bash
LOG_FILE="/var/log/nginx/error.log" # Example: Nginx error log
ERROR_COUNT=$(grep -i "error" "${LOG_FILE}" | wc -l) # Count lines containing 'error' (case-insensitive)

if [[ ${ERROR_COUNT} -gt 0 ]]; then
    echo "Log Analysis (${LOG_FILE}): ${ERROR_COUNT} errors found in last 24h" # (assuming logrotate)
    # Optional: Display the last few errors
    grep -i "error" "${LOG_FILE}" | tail -n 5
else
    echo "Log Analysis (${LOG_FILE}): No errors found."
fi
```

```output
Log Analysis (/var/log/nginx/error.log): No errors found.
```

**Explanation**:
*   `grep -i "error"`: Search for "error" (case-insensitive).
*   `wc -l`: Word count, lines.
*   `tail -n 5`: Show the last 5 lines.
*   **Note**: This is very basic. For real log analysis, you'd integrate with tools like `logrotate` to ensure you're only checking recent logs, or use more sophisticated parsing.

## Structuring the Script: Putting It All Together

Now, let's combine these checks into a single, cohesive Bash script. We'll use functions for modularity and a clear output format.

```bash
#!/bin/bash

# monitoring_script.sh
# A simple Bash script for basic stack monitoring.

# --- Configuration ---
LOG_FILE="/var/log/monitor.log"
DATE_FORMAT="+%Y-%m-%d %H:%M:%S"
ALERT_THRESHOLD_CPU=80 # %
ALERT_THRESHOLD_DISK=90 # %
ALERT_THRESHOLD_LOAD=2.0 # 1-min load average per CPU core (adjust for your CPU count)

# Services and processes to monitor
SERVICES_TO_MONITOR=("nginx" "mysql" "redis")
PROCESSES_TO_MONITOR=("sshd" "php-fpm")
HTTP_APPS_TO_MONITOR=(
    "Web App:http://localhost:8080/health"
    "API Service:http://localhost:3000/api/status"
)
PING_TARGETS=("google.com" "8.8.8.8")
MONITOR_LOG_FILES=("/var/log/syslog" "/var/log/auth.log")

# --- Helper Functions ---
log_message() {
    echo "$(date "$DATE_FORMAT") - $1" | tee -a "${LOG_FILE}"
}

print_section_header() {
    log_message ""
    log_message "--- $1 ---"
}

# --- Monitoring Functions ---

monitor_cpu() {
    print_section_header "CPU Usage"
    CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
    log_message "Current CPU Usage: ${CPU_USAGE}%"
    if (( $(echo "${CPU_USAGE} > ${ALERT_THRESHOLD_CPU}" | bc -l) )); then
        log_message "ALERT: High CPU usage detected: ${CPU_USAGE}%"
    fi
}

monitor_memory() {
    print_section_header "Memory Usage"
    MEM_TOTAL=$(free -h | awk '/^Mem:/ {print $2}')
    MEM_USED=$(free -h | awk '/^Mem:/ {print $3}')
    MEM_FREE=$(free -h | awk '/^Mem:/ {print $4}')
    log_message "Total: ${MEM_TOTAL}, Used: ${MEM_USED}, Free: ${MEM_FREE}"
}

monitor_disk() {
    print_section_header "Disk Usage"
    DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')
    DISK_PATH=$(df -h / | awk 'NR==2 {print $1}')
    log_message "Root Disk Usage: ${DISK_USAGE}% on ${DISK_PATH}"
    if [[ ${DISK_USAGE} -gt ${ALERT_THRESHOLD_DISK} ]]; then
        log_message "ALERT: High Disk usage detected: ${DISK_USAGE}%"
    fi
}

monitor_load_average() {
    print_section_header "Load Average"
    LOAD_AVG_1MIN=$(uptime | awk -F'load average: ' '{print $2}' | cut -d',' -f1 | tr -d ' ')
    NUM_CPUS=$(grep -c ^processor /proc/cpuinfo)
    NORMALIZED_LOAD=$(echo "${LOAD_AVG_1MIN} / ${NUM_CPUS}" | bc -l)

    log_message "Load Average (1, 5, 15 min): $(uptime | awk -F'load average: ' '{print $2}')"
    log_message "Normalized 1-min Load Avg (per core): ${NORMALIZED_LOAD} (CPUs: ${NUM_CPUS})"
    if (( $(echo "${NORMALIZED_LOAD} > ${ALERT_THRESHOLD_LOAD}" | bc -l) )); then
        log_message "ALERT: High Load Average detected: ${NORMALIZED_LOAD} per core (Threshold: ${ALERT_THRESHOLD_LOAD})"
    fi
}

monitor_network_connectivity() {
    print_section_header "Network Connectivity"
    for target in "${PING_TARGETS[@]}"; do
        if ping -c 3 -W 1 "${target}" > /dev/null 2>&1; then
            log_message "Connectivity to ${target}: OK"
        else
            log_message "Connectivity to ${target}: FAILED"
        fi
    done
}

monitor_open_ports() {
    print_section_header "Open Ports"
    # Example: Check common ports if not linked to a service
    PORTS_TO_CHECK=("22" "80" "443" "3306" "5432")
    for port in "${PORTS_TO_CHECK[@]}"; do
        if netstat -tulnp | grep ":${port}" | grep -q "LISTEN"; then
            log_message "Port ${port}: LISTENING"
        else
            log_message "Port ${port}: NOT LISTENING"
        fi
    done
}

monitor_processes() {
    print_section_header "Process Status"
    for process in "${PROCESSES_TO_MONITOR[@]}"; do
        if pgrep "${process}" > /dev/null; then
            log_message "Process ${process}: RUNNING"
        else
            log_message "Process ${process}: NOT RUNNING"
        fi
    done
}

monitor_services() {
    print_section_header "Service Status (systemd)"
    for service in "${SERVICES_TO_MONITOR[@]}"; do
        SERVICE_STATUS=$(systemctl is-active "${service}" 2>/dev/null)
        if [[ -z "${SERVICE_STATUS}" ]]; then
            log_message "Service ${service}: UNKNOWN (service not found or systemctl error)"
        elif [[ "${SERVICE_STATUS}" == "active" ]]; then
            log_message "Service ${service}: Active"
        else
            log_message "Service ${service}: ${SERVICE_STATUS}"
        fi
    done
}

monitor_http_applications() {
    print_section_header "HTTP Application Health"
    for app in "${HTTP_APPS_TO_MONITOR[@]}"; do
        NAME=$(echo "${app}" | cut -d':' -f1)
        URL=$(echo "${app}" | cut -d':' -f2-) # Handles URLs with colons
        HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "${URL}" 2>/dev/null)

        if [[ -z "${HTTP_CODE}" ]]; then # Curl failed entirely
            log_message "App ${NAME} (${URL}): FAILED (Curl error or no response)"
        elif [[ "${HTTP_CODE}" == "200" ]]; then
            log_message "App ${NAME} (${URL}): OK (HTTP ${HTTP_CODE})"
        else
            log_message "App ${NAME} (${URL}): FAILED (HTTP ${HTTP_CODE})"
        fi
    done
}

monitor_log_files() {
    print_section_header "Log File Analysis"
    for log_file in "${MONITOR_LOG_FILES[@]}"; do
        if [[ ! -f "${log_file}" ]]; then
            log_message "Log file ${log_file}: Not found."
            continue
        fi
        # Check last 100 lines for errors/warnings
        ERROR_COUNT=$(tail -n 100 "${log_file}" | grep -Ei "error|fail|crit|warn" | wc -l)
        if [[ ${ERROR_COUNT} -gt 0 ]]; then
            log_message "Log File ${log_file}: ${ERROR_COUNT} potential issues found in last 100 lines."
            tail -n 50 "${log_file}" | grep -Ei "error|fail|crit|warn" | head -n 5 # Show up to 5 example lines
        else
            log_message "Log File ${log_file}: No common issues found in last 100 lines."
        fi
    done
}

# --- Main Execution ---
log_message "--- Starting Stack Monitoring Script ---"

monitor_cpu
monitor_memory
monitor_disk
monitor_load_average
monitor_network_connectivity
monitor_open_ports
monitor_processes
monitor_services
monitor_http_applications
monitor_log_files

log_message "--- Stack Monitoring Script Finished ---"
```

To make this script executable:
```bash
chmod +x monitoring_script.sh
```

To run it:
```bash
./monitoring_script.sh
```

**Sample Output (truncated, as it's logged to file and stdout):**

```output
2023-10-27 10:30:01 - --- Starting Stack Monitoring Script ---
2023-10-27 10:30:01 - 
2023-10-27 10:30:01 - --- CPU Usage ---
2023-10-27 10:30:01 - Current CPU Usage: 4.2%
2023-10-27 10:30:01 - 
2023-10-27 10:30:01 - --- Memory Usage ---
2023-10-27 10:30:01 - Total: 3.8Gi, Used: 1.6Gi, Free: 1.2Gi
2023-10-27 10:30:01 - 
2023-10-27 10:30:01 - --- Disk Usage ---
2023-10-27 10:30:01 - Root Disk Usage: 45% on /dev/sda1
2023-10-27 10:30:01 - 
2023-10-27 10:30:01 - --- Load Average ---
2023-10-27 10:30:01 - Load Average (1, 5, 15 min): 0.12, 0.15, 0.09
2023-10-27 10:30:01 - Normalized 1-min Load Avg (per core): 0.0300000000000000 (CPUs: 4)
2023-10-27 10:30:01 - 
2023-10-27 10:30:01 - --- Network Connectivity ---
2023-10-27 10:30:01 - Connectivity to google.com: OK
2023-10-27 10:30:01 - Connectivity to 8.8.8.8: OK
2023-10-27 10:30:01 - 
2023-10-27 10:30:01 - --- Open Ports ---
2023-10-27 10:30:01 - Port 22: LISTENING
2023-10-27 10:30:01 - Port 80: LISTENING
2023-10-27 10:30:01 - Port 443: NOT LISTENING
2023-10-27 10:30:01 - ... (and so on for all checks)
```

You can then view the log file:
```bash
cat /var/log/monitor.log
```

**Note on alerting**: The script includes basic `ALERT:` messages directly in the log. For true alerting (email, SMS, Slack), you'd need to integrate with external tools or services. For example, you could pipe these alert messages to a Python script that uses a library to send email, or `curl` to a Slack webhook.

## When and How to Use This Script

*   **Quick Sanity Checks**: Run it manually after a deployment or a configuration change.
*   **Simple Cron Jobs**: Schedule it to run every few minutes to collect basic metrics and log them.
    ```bash
    # Add to your crontab (crontab -e) to run every 5 minutes
    */5 * * * * /path/to/your/monitoring_script.sh > /dev/null 2>&1
    ```
    **Note**: If you add this to cron, ensure the script's path is absolute (`/home/user/monitoring_script.sh`).
*   **Learning Tool**: Understand the underlying commands for various monitoring aspects.

## Crucial Limitations and When NOT to Use This

This is where honesty comes in. While powerful for its simplicity, a Bash monitoring script is fundamentally limited compared to dedicated monitoring systems.

1.  **No Centralized Dashboard or UI**: All output is text-based logs. You won't get graphs, historical trends, or a nice visual overview.
2.  **Limited Historical Data**: You're only capturing snapshots. Analyzing performance over time requires parsing log files or integrating with a time-series database.
3.  **Basic Alerting**: The script logs alerts, but doesn't *send* them to you without external integrations (e.g., mail clients, webhooks). Setting up sophisticated alert routing, escalation policies, or deduplication is complex.
4.  **No Distributed Monitoring**: This script is designed for a single server. Monitoring multiple servers effectively requires a central agent and collection system.
5.  **Complexity at Scale**: As your stack grows, maintaining a massive Bash script becomes unwieldy. Adding new checks, managing thresholds, and debugging will become a nightmare.
6.  **No Anomaly Detection**: It only checks against static thresholds. It can't learn normal behavior and alert on deviations.
7.  **No Root Cause Analysis**: It tells you *what* is wrong (e.g., "CPU is high"), but not *why* (e.g., "CPU is high because process X is spinning").
8.  **Security**: Hardcoding sensitive information (though not done here) or running with elevated privileges without careful thought is a risk.
9.  **No Self-Healing**: It identifies problems but doesn't automatically restart services or perform other remediation actions.

### When to use proper monitoring solutions:

*   **Production Environments**: Any critical application or infrastructure.
*   **Distributed Systems**: Microservices, containers, cloud-native architectures.
*   **Need for Historical Data & Trends**: Performance analysis, capacity planning.
*   **Sophisticated Alerting**: On-call rotations, PagerDuty, Slack/Teams integrations.
*   **Centralized Visibility**: Dashboards, aggregated metrics across many services.
*   **Complex Root Cause Analysis**: Tracing, logging, metrics all correlated.

**Tools to consider instead for real-world scenarios**:

*   **Metrics & Dashboards**: Prometheus + Grafana, InfluxDB, Datadog, New Relic.
*   **Log Management**: ELK Stack (Elasticsearch, Logstash, Kibana), Splunk, Graylog.
*   **Distributed Tracing**: Jaeger, Zipkin.
*   **APM (Application Performance Monitoring)**: Dynatrace, AppDynamics.
*   **Traditional Monitoring**: Nagios, Zabbix.

## Conclusion

You now have a powerful, albeit simple, Bash script that can monitor key aspects of your server's health. It's a fantastic way to learn about system metrics, command-line utilities, and the fundamentals of monitoring. It's perfect for your personal server, a small side project, or as a rapid diagnostic tool.

However, remember its limitations. As your infrastructure scales and your monitoring needs become more complex, you'll inevitably graduate to more specialized, robust, and feature-rich monitoring solutions. Think of this script as your foundational knowledge – the sturdy, hand-built workbench before you invest in a fully automated factory.