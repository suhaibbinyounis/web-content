---
categories:
- DevOps
- System Administration
comments: true
cover:
  image: https://images.pexels.com/photos/7621141/pexels-photo-7621141.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Learn how to combine the real-time file watching power of Entr with the
  robust scheduling of Cron to create persistent, event-driven automation for your
  development and production environments. Stop polling, start reacting.
tags:
- Linux
- CLI
- Bash
- Automation
- DevOps
- Cron
- entr
- File Watcher
- System Administration
title: How to Use Entr + Cron to Auto-Run Scripts When Files Change
---

![A hand holds a payment terminal against a light background with ample copy space.](https://images.pexels.com/photos/7621141/pexels-photo-7621141.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "A hand holds a payment terminal against a light background with ample copy space.")

## How to Use Entr + Cron to Auto-Run Scripts When Files Change

As developers and system administrators, we often face the challenge of automating tasks that depend on file changes. Perhaps you need to recompile a project when source files are modified, restart a service when its configuration changes, or process data as soon as new files land in a directory.

The common approaches often fall short:

1.  **Manual execution**: Tedious, error-prone, and doesn't scale.
2.  **Polling with `watch` or `while true` loops**: Inefficient, consumes resources even when nothing changes, introduces unnecessary delays, and is generally a hack.

We need something better. We need event-driven automation that's both immediate and persistent. Enter `entr` and `cron`.

This post will show you how to combine these two powerful Unix utilities to build robust, reactive, and persistent automation workflows.

## The Problem: Event-Driven Persistence

`entr` (event-notify-then-run) is a fantastic utility for running arbitrary commands when files change. It's lean, fast, and uses kernel facilities (like `inotify` on Linux) for efficiency.

`cron` is the classic Unix job scheduler. It's rock-solid for running tasks at specific times or intervals, or even at system boot.

The challenge is that `entr` typically runs in the foreground. If your terminal closes, or the system reboots, `entr` stops. We need a way to ensure our `entr` watcher is *always* running, or at least restarted reliably. This is where `cron` comes in.

By using `cron` to manage `entr` as a background process, we achieve:
*   **Event-driven execution**: Scripts run only when needed.
*   **Persistence**: The watcher restarts automatically after reboots.
*   **Robustness**: We can implement checks to ensure it's always running.

Let's break down `entr` and `cron` individually, then combine them.

## Understanding `entr`

`entr` is designed to execute a command when any of the files piped into its standard input change.

### Installation

`entr` is often available in your distribution's package manager.

**Debian/Ubuntu:**

```bash
sudo apt update
sudo apt install entr
```

**Fedora/RHEL:**

```bash
sudo dnf install entr
```

**macOS (with Homebrew):**

```bash
brew install entr
```

### Basic `entr` Usage

The core syntax is `[list of files] | entr [command]`.

**Example 1: Watching a single file**

Let's create a simple script that `entr` will execute.

```bash
echo '#!/bin/bash
echo "--- File changed at $(date) ---"
cat file.txt
' > script_on_change.sh
chmod +x script_on_change.sh

echo "Initial content" > file.txt
```

Now, watch `file.txt` and run `script_on_change.sh` whenever it changes:

```bash
ls file.txt | entr ./script_on_change.sh
```

```output
# No output immediately, entr is waiting.
# Now, open another terminal and modify file.txt:
echo "New content after modification" > file.txt
# Back in the entr terminal, you'll see:
--- File changed at Fri Oct 27 10:30:00 AM UTC 2023 ---
New content after modification
```

**Example 2: Watching multiple files and directories (recursively)**

To watch files recursively in a directory, use `find`. We also often want `entr` to execute the command *in a shell* to allow for pipelines or complex commands, and perhaps clear the screen between runs.

```bash
# Create some dummy files
mkdir my_project
echo "console.log('hello');" > my_project/app.js
echo "body { color: blue; }" > my_project/style.css

# Create a build script
echo '#!/bin/bash
echo "--- Building project at $(date) ---"
echo "Simulating build process..."
sleep 0.5
echo "Build complete."
' > my_project/build.sh
chmod +x my_project/build.sh
```

Now, watch all `.js` and `.css` files in `my_project` recursively, clear the screen (`-c`), and execute the build script in a shell (`-s`):

```bash
find my_project -name "*.js" -o -name "*.css" | entr -c -s ./my_project/build.sh
```

```output
# No output immediately, entr is waiting.
# Now, modify a file:
echo "console.log('updated');" >> my_project/app.js
# Back in the entr terminal, you'll see (screen clears first):
--- Building project at Fri Oct 27 10:35:00 AM UTC 2023 ---
Simulating build process...
Build complete.
```

### Useful `entr` Flags

*   `-c`: Clear the screen before executing the command.
*   `-d`: Watch new files created in watched directories. Essential for projects where new files are frequently added.
*   `-p`: Pause for a short period (debounce) before executing the command. Useful to prevent multiple rapid executions during a batch of saves.
*   `-s`: Execute the command using `sh -c`. This is crucial if your command involves pipes, redirections, or other shell features. Without `-s`, `entr` executes the command directly.
*   `-z`: Do not exit if the command exits with a non-zero status. Useful if your script might fail but you want `entr` to keep watching.
*   `-r`: Reload `entr` itself if its input files change. Useful if your `find` command or the list of files `entr` is watching needs to be re-evaluated.
*   `-L <pidfile>`: Write `entr`'s PID to a file. Useful for managing `entr` processes directly, though we'll use a wrapper script for our `cron` setup.
*   `-0`: Read NUL-terminated input. Use this with `find ... -print0` for robust handling of filenames with spaces or special characters.

For our persistent setup, `-s` and potentially `-d` and `-p` will be common.

## Understanding `cron`

`cron` allows you to schedule commands or scripts to run automatically at specified intervals or times. Each user has their own crontab, and there's also a system-wide crontab.

### Basic `cron` Usage

To edit your user's crontab:

```bash
crontab -e
```

This will open your crontab in your default editor. Each line represents a job.

### Cron Job Syntax

A cron job line has six fields:

```
minute hour day_of_month month day_of_week command_to_execute
```

*   **minute**: (0-59)
*   **hour**: (0-23)
*   **day_of_month**: (1-31)
*   **month**: (1-12 or Jan-Dec)
*   **day_of_week**: (0-7, where 0 and 7 are Sunday)
*   **command_to_execute**: The command or script to run.

Special strings simplify common schedules:

*   `@reboot`: Run once at system startup.
*   `@yearly` or `@annually`: Run once a year (0 0 1 1 *).
*   `@monthly`: Run once a month (0 0 1 * *).
*   `@weekly`: Run once a week (0 0 * * 0).
*   `@daily` or `@midnight`: Run once a day (0 0 * * *).
*   `@hourly`: Run once an hour (0 * * * *).

**Important Cron Considerations:**

*   **Environment**: The cron environment is very minimal. `PATH` is often restricted. Always use **full absolute paths** to executables (e.g., `/usr/bin/entr`, `/bin/bash`, `/usr/bin/find`) and scripts, or explicitly set `PATH` at the top of your crontab.
*   **Output**: By default, cron emails `stdout` and `stderr` to the user. For long-running or frequent jobs, redirect output to log files (e.g., `command >> /path/to/log.log 2>&1`).
*   **User**: Jobs in your user's crontab run as your user. Jobs in `/etc/crontab` or `/etc/cron.d/` can specify the user. `@reboot` in `root`'s crontab is common for system services.

## The Synergy: `entr` + `cron` for Persistent File Watching

The goal is to have `entr` run reliably in the background, even after reboots. We'll use `cron`'s `@reboot` directive combined with a wrapper script to achieve this.

### Strategy: `@reboot` with a Watchdog Script

Instead of directly running `entr` in `crontab`, we'll use a shell script that checks if `entr` is already running (to prevent multiple instances) and, if not, starts it in the background, redirecting output to a log file.

**Scenario**: Automatically restart a hypothetical `web_server.py` when any `.py` or `.conf` file changes in `/var/www/my_app/config` or `/var/www/my_app/src`.

1.  **Create the script `entr` will execute (`restart_web_server.sh`)**:
    This script will contain the logic to be run when files change.

    ```bash
    #!/bin/bash
    LOG_FILE="/var/log/my_app_watcher.log"

    echo "[$(date)] Detected file change. Restarting web server..." >> $LOG_FILE

    # Simulate stopping and starting a web server
    # Replace this with your actual service restart command
    # e.g., systemctl restart my_web_server.service
    echo "Stopping existing server..." >> $LOG_FILE
    pkill -f "python my_web_server.py" # Kills any running instances of our dummy server
    sleep 1
    echo "Starting new server..." >> $LOG_FILE
    # Start the server in the background, redirecting its output
    nohup python3 /var/www/my_app/my_web_server.py > /var/log/my_app_server.log 2>&1 &
    SERVER_PID=$!
    echo "Server started with PID: $SERVER_PID" >> $LOG_FILE
    echo "--- Web server restart complete ---" >> $LOG_FILE
    ```

    Make it executable:
    ```bash
    chmod +x /usr/local/bin/restart_web_server.sh
    ```

    > **Note**: For system-level services, you'd typically use `systemctl restart my_service.service` or similar instead of direct `pkill` and `nohup`. The example uses `nohup` for simplicity in a self-contained demonstration.

2.  **Create a dummy web server script (`my_web_server.py`)**:

    ```python
    # /var/www/my_app/my_web_server.py
    import time
    import datetime
    import os

    log_path = "/var/log/my_app_server.log"

    def log(message):
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        with open(log_path, "a") as f:
            f.write(f"[{timestamp}] [PID {os.getpid()}] {message}\n")

    log("Web server started.")
    print(f"Web server running with PID {os.getpid()}") # This goes to stdout/stderr of nohup
    while True:
        log("Server active...")
        time.sleep(10)
    ```

3.  **Create the `entr` watchdog script (`start_entr_watcher.sh`)**:
    This script will be called by `cron`. It checks for an existing `entr` process and starts one if it's not running.

    ```bash
    #!/bin/bash

    # --- Configuration ---
    WATCH_DIR="/var/www/my_app"
    SCRIPT_TO_RUN_ON_CHANGE="/usr/local/bin/restart_web_server.sh"
    LOG_FILE="/var/log/my_app_watcher.log"
    PID_FILE="/var/run/my_app_entr_watcher.pid" # Use /var/run for system-wide PIDs

    # Ensure log directory exists
    mkdir -p $(dirname $LOG_FILE)
    touch $LOG_FILE
    chmod 644 $LOG_FILE # Ensure permissions are appropriate for the user cron runs as

    echo "--- Starting entr watchdog at $(date) ---" >> $LOG_FILE

    # --- Check for existing entr process ---
    if [ -f "$PID_FILE" ]; then
        PID=$(cat "$PID_FILE")
        if kill -0 "$PID" 2>/dev/null; then
            echo "[$(date)] entr watcher already running with PID $PID. Exiting." >> $LOG_FILE
            exit 0
        else
            echo "[$(date)] Stale PID file found. Removing $PID_FILE." >> $LOG_FILE
            rm -f "$PID_FILE"
        fi
    fi

    # --- Start entr watcher ---
    echo "[$(date)] Starting entr file watcher..." >> $LOG_FILE

    # Build the entr command. Using -0 with find -print0 is safest for filenames.
    # -p: Debounce changes, -s: Run command in a shell
    # nohup ... & makes it run in the background, detached from the shell
    nohup /usr/bin/find "$WATCH_DIR" -type f \( -name "*.py" -o -name "*.conf" \) -print0 \
        | /usr/bin/entr -0 -p -s "$SCRIPT_TO_RUN_ON_CHANGE" \
        >> "$LOG_FILE" 2>&1 &

    # Capture the PID of the entr process
    ENTR_PID=$!
    echo "$ENTR_PID" > "$PID_FILE"
    echo "[$(date)] entr watcher started with PID $ENTR_PID." >> "$LOG_FILE"
    echo "--- entr watchdog completed at $(date) ---" >> $LOG_FILE
    ```

    Make it executable:
    ```bash
    chmod +x /usr/local/bin/start_entr_watcher.sh
    ```

    > **Permissions Note**: If you put `PID_FILE` in `/var/run`, the cron job needs write permissions there. If running as a non-root user, you might need to use a directory like `/tmp` or `~/.local/run/` for the PID file and logs, or ensure `/var/run/my_app` exists and is writable by your user. For system-level services, `root`'s crontab is often used, and it has access to `/var/run`.

4.  **Add the cron job (`crontab -e`)**:
    We want this script to run once when the system boots up. If you're managing a system service, you'd likely add this to `root`'s crontab (`sudo crontab -e`).

    ```bash
    # Open your crontab (or root's crontab if managing system services)
    crontab -e
    ```

    Add the following line to your crontab:

    ```cron
    # Run the entr watchdog script at system reboot
    @reboot /usr/local/bin/start_entr_watcher.sh
    ```

### Testing and Verification

1.  **Manually test the `restart_web_server.sh` script**:
    ```bash
    /usr/local/bin/restart_web_server.sh
    ```
    Check `/var/log/my_app_watcher.log` and `/var/log/my_app_server.log` for output.
    Verify the Python server is running:
    ```bash
    ps aux | grep "python3 /var/www/my_app/my_web_server.py"
    ```
    ```output
    user   12345  0.1  0.1  12345  6789 ?        Sl   10:00   0:00 python3 /var/www/my_app/my_web_server.py
    ```

2.  **Manually test the `start_entr_watcher.sh` script**:
    This simulates what `@reboot` will do.
    ```bash
    /usr/local/bin/start_entr_watcher.sh
    ```
    Check `/var/log/my_app_watcher.log`. It should indicate `entr` starting.
    Check if `entr` is running:
    ```bash
    ps aux | grep entr
    ```
    ```output
    user   12346  0.0  0.0  12345  6789 ?        Ss   10:01   0:00 /usr/bin/entr -0 -p -s /usr/local/bin/restart_web_server.sh
    user   12347  0.0  0.0  12345  6789 ?        Sl   10:01   0:00 /usr/bin/find /var/www/my_app -type f ( -name *.py -o -name *.conf ) -print0
    ```
    (You might see the `find` process running as a child of `entr`.)

    Try running `start_entr_watcher.sh` again immediately. It should log that `entr` is already running and exit.

3.  **Test the full flow (modify a file)**:
    Make a change to one of the watched files (e.g., `/var/www/my_app/my_web_server.py`):
    ```bash
    echo "# New line" >> /var/www/my_app/my_web_server.py
    ```
    Check `/var/log/my_app_watcher.log` and `/var/log/my_app_server.log`. You should see messages indicating the web server was restarted by `entr`.

    ```output
    # Excerpt from /var/log/my_app_watcher.log after file change:
    [2023-10-27 10:45:01] Detected file change. Restarting web server...
    Stopping existing server...
    Starting new server...
    Server started with PID: 12348
    --- Web server restart complete ---

    # Excerpt from /var/log/my_app_server.log:
    [2023-10-27 10:45:00] [PID 12345] Server active...
    [2023-10-27 10:45:01] [PID 12345] Web server started. # Old server log ends here
    [2023-10-27 10:45:01] [PID 12348] Web server started. # New server log starts here
    [2023-10-27 10:45:11] [PID 12348] Server active...
    ```

4.  **Reboot and verify**:
    Reboot your system. After it comes back up, check `ps aux | grep entr` and your log files. `entr` should be running and watching.

## Important Considerations and Best Practices

*   **Absolute Paths**: Always, always, always use full absolute paths in cron jobs and scripts called by cron. The `PATH` environment variable in cron is often very limited.
*   **Logging**: Redirect all `stdout` and `stderr` to log files. This is your primary debugging tool when things go wrong. Without it, you'll be flying blind.
*   **Permissions**: Ensure the user running the cron job (and thus the `entr` watcher and its triggered script) has the necessary read/write permissions for watched directories, log files, PID files, and any other resources it interacts with.
*   **PID Files**: Crucial for managing long-running background processes. They prevent multiple instances from starting and help you stop or restart processes.
*   **Error Handling**: Your `restart_web_server.sh` script should ideally include robust error handling. What if `pkill` fails? What if the `python` command fails? Log errors and potentially implement retry logic or notifications.
*   **Resource Usage**: While `entr` is efficient, watching an extremely large number of files or highly active directories can still consume resources. Monitor CPU and memory if you're watching vast file trees.
*   **Systemd**: For truly robust system services on Linux, `systemd` service units are generally preferred over `@reboot` cron jobs. `systemd` offers better dependency management, logging, automatic restarts, and process control. However, the `entr` + `cron` pattern is a fantastic, simpler alternative when `systemd` setup is overkill or not immediately feasible, especially for user-specific development tasks.
*   **Alternative File Watchers**: For specific needs, `inotifywait` (part of `inotify-tools` on Linux) or `fswatch` are alternatives. `inotifywait` is lower-level and powerful but requires more scripting. `fswatch` is cross-platform. `entr` strikes a great balance of power and simplicity.

## Conclusion

Combining `entr` and `cron` unlocks a powerful pattern for persistent, event-driven automation. Whether you're a developer tired of manually rebuilding projects, a system admin needing to react to config changes, or just someone looking to make their system more reactive, this approach offers a robust and elegant solution.

By leveraging `entr`'s efficient file watching and `cron`'s reliable scheduling, you can build self-managing workflows that respond immediately to changes, without the overhead of constant polling. Give it a try, and streamline your automated tasks!