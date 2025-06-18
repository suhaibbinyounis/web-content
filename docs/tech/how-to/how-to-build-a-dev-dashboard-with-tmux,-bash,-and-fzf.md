---
title: How to Build a Dev Dashboard with Tmux, Bash, and fzf
date: 2025-06-17T13:05:16.383Z
description: "Learn how to craft a powerful, personalized developer dashboard directly in your terminal using Tmux for window management, Bash for scripting logic, and fzf for interactive navigation and quick actions. This guide provides practical examples for a highly efficient CLI workflow."
tags: [Linux, CLI, Bash, Tmux, fzf, DevOps, Productivity, Automation]
categories: [Productivity, DevOps, Linux]
comments: true
---

Every developer craves efficiency. We spend hours in the terminal, switching contexts, checking statuses, and digging for information. What if you could condense all that into a single, highly customizable, interactive view? That's the promise of a developer dashboard, and in this post, we're going to build one using three incredibly powerful, ubiquitous, and often underestimated tools: **Tmux**, **Bash**, and **fzf**.

This isn't about some fancy GUI dashboard. This is about harnessing the raw power of your command line to create a pragmatic, lightning-fast, and deeply personal workspace that adapts to *your* workflow.

## Why a CLI Dev Dashboard?

### The Problem
*   **Context Switching Hell:** Jumping between monitoring logs, checking git status, observing resource usage, and managing containers is fragmented.
*   **Repetitive Tasks:** Typing `docker ps`, `kubectl get pods`, `git status` over and over.
*   **Information Overload:** Important data is scattered across multiple terminals or windows.

### The Solution: Tmux + Bash + fzf
*   **Tmux (Terminal Multiplexer):** The perfect foundation. It allows you to create persistent sessions, split windows into multiple panes, and organize your workspace. Your dashboard stays alive even if you disconnect.
*   **Bash (Shell Scripting):** The glue. Bash is everywhere, and its scripting capabilities let you automate information retrieval, format data, and orchestrate complex workflows.
*   **fzf (Fuzzy Finder):** The interactive layer. fzf transforms raw lists into searchable, selectable interfaces, enabling quick navigation, command execution, and file selection.

Together, these tools let you create a dashboard that is:
*   **Persistent:** Always there, just `tmux attach`.
*   **Customizable:** Tailor it to your exact needs.
*   **Actionable:** Not just information, but direct interaction.
*   **Fast:** No GUI overhead.
*   **Resource-Light:** Pure CLI.

Let's get started.

## Prerequisites

Before we dive in, make sure you have these tools installed. They're typically available via your system's package manager.

*   **Tmux:** `sudo apt install tmux` (Debian/Ubuntu), `sudo yum install tmux` (RHEL/CentOS), `brew install tmux` (macOS)
*   **Bash:** Almost certainly already installed.
*   **fzf:** `git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && ~/.fzf/install` (Recommended way), or via package manager.

You'll also need a basic understanding of shell scripting.

## The Foundation: Tmux for Layout and Persistence

Tmux is the backbone of our dashboard. It allows us to define a fixed layout of panes, each running a specific command or script, and keeps that layout persistent across sessions.

### Basic Tmux Concepts
*   **Session:** A collection of windows. You can detach from a session and reattach later.
*   **Window:** Like a tab in your browser, each window can have multiple panes.
*   **Pane:** A distinct shell prompt within a window.

All Tmux commands are prefixed by a "prefix key", which is `Ctrl+b` by default.

### Essential Tmux Commands

| Command           | Tmux Keybind      | Description                                     |
| :---------------- | :---------------- | :---------------------------------------------- |
| `tmux new -s name` | N/A               | Create a new session named `name`.              |
| `tmux attach -t name` | N/A             | Attach to an existing session named `name`.     |
| `tmux detach`     | `Ctrl+b d`        | Detach from the current session.                |
| `tmux split-window -h` | `Ctrl+b %`     | Split current pane horizontally.                |
| `tmux split-window -v` | `Ctrl+b "`     | Split current pane vertically.                  |
| `tmux new-window` | `Ctrl+b c`        | Create a new window.                            |
| `tmux kill-pane`  | `Ctrl+b x`        | Kill the current pane.                          |
| `tmux select-pane -t <direction>` | `Ctrl+b <arrow-key>` | Move between panes.            |

### Example: A Simple Tmux Layout

Let's create a Tmux session with a few panes. We'll name our session `dev-dash`.

```bash
# Start a new Tmux session named dev-dash
tmux new -s dev-dash

# Split the main window horizontally (Ctrl+b %)
tmux split-window -h

# Move to the new pane (Ctrl+b <left-arrow>)
tmux select-pane -L

# Split this left pane vertically (Ctrl+b ")
tmux split-window -v

# Move back to the top right pane (Ctrl+b <right-arrow>)
tmux select-pane -R

# Split this top right pane vertically (Ctrl+b ")
tmux split-window -v
```

After running these commands (or executing the keybindings within a Tmux session), you'd end up with a layout like this:

```
+------------------+------------------+
|                  |                  |
|       Pane 1     |       Pane 2     |
|                  |                  |
+------------------+------------------+
|                  |                  |
|       Pane 3     |       Pane 4     |
|                  |                  |
+------------------+------------------+
```

### Automating Tmux Session Setup

Instead of manually typing keybinds, we can script the entire layout creation. This is crucial for our dashboard.

Let's create `dashboard_layout.sh`:

```bash
#!/bin/bash

SESSION_NAME="dev-dash"

# Check if the session exists, if so, attach to it.
tmux has-session -t "$SESSION_NAME" 2>/dev/null

if [ $? == 0 ]; then
    echo "Attaching to existing session: $SESSION_NAME"
    tmux attach -t "$SESSION_NAME"
else
    echo "Creating new session: $SESSION_NAME"
    # Create a new detached session
    tmux new-session -d -s "$SESSION_NAME"

    # Split the main window into four panes
    # Pane 0 (top-left) - default pane
    tmux split-window -h -t "$SESSION_NAME:0.0" # Pane 1 (top-right)
    tmux split-window -v -t "$SESSION_NAME:0.0" # Pane 2 (bottom-left)
    tmux split-window -v -t "$SESSION_NAME:0.2" # Pane 3 (bottom-right)

    # Optional: adjust pane sizes if needed
    # tmux resize-pane -t "$SESSION_NAME:0.0" -x 50 # Example: pane 0 width 50%

    # Send commands to each pane (we'll fill these later)
    tmux send-keys -t "$SESSION_NAME:0.0" "echo 'System Info Pane';" C-m
    tmux send-keys -t "$SESSION_NAME:0.1" "echo 'Git Status Pane';" C-m
    tmux send-keys -t "$SESSION_NAME:0.2" "echo 'Docker/K8s Pane';" C-m
    tmux send-keys -t "$SESSION_NAME:0.3" "echo 'Interactive Pane';" C-m

    # Attach to the newly created session
    tmux attach -t "$SESSION_NAME"
fi
```

Make it executable: `chmod +x dashboard_layout.sh`.
Now, run `./dashboard_layout.sh`.

```output
Creating new session: dev-dash
[detached (from terminal 0)]
```
(Followed by Tmux attaching and showing the layout.)

You'll see a Tmux session with four panes, each echoing its placeholder message. You can detach with `Ctrl+b d` and reattach with `./dashboard_layout.sh` again.

## The Logic: Bash Scripting for Data

Bash is our workhorse. We'll use it to gather information, format it, and run it in specific Tmux panes. The key here is to create small, focused scripts for each piece of data you want to display.

### Auto-refreshing Information

For a dynamic dashboard, data needs to refresh. We can achieve this with a simple `while true` loop and `sleep`.

#### Example: System Info Script (`sys_info.sh`)

```bash
#!/bin/bash

# Simple script to display system information
# This will run indefinitely, updating every 2 seconds.

while true; do
    clear
    echo "--- System Information ---"
    echo "Hostname: $(hostname)"
    echo "Uptime: $(uptime -p)"
    echo "--------------------------"
    echo "Memory:"
    free -h | grep "Mem:" | awk '{print "  Total: "$2", Used: "$3", Free: "$4}'
    echo "--------------------------"
    echo "Disk Usage (/):"
    df -h / | tail -n 1 | awk '{print "  Total: "$2", Used: "$3", Avail: "$4}'
    echo "--------------------------"
    echo "Network (Eth0/WLAN0):"
    ip -4 addr show eth0 2>/dev/null | grep -oP 'inet \K[\d.]+' | awk '{print "  Eth0 IP: "$1}'
    ip -4 addr show wlan0 2>/dev/null | grep -oP 'inet \K[\d.]+' | awk '{print "  WLAN0 IP: "$1}'
    echo "--------------------------"
    echo "Press Ctrl+C to stop."
    sleep 2
done
```

Make it executable: `chmod +x sys_info.sh`.

#### Sample Output (`./sys_info.sh`)

```output
--- System Information ---
Hostname: mydevmachine
Uptime: up 2 days, 15 hours, 3 minutes
--------------------------
Memory:
  Total: 15Gi, Used: 8.5Gi, Free: 5.0Gi
--------------------------
Disk Usage (/):
  Total: 250G, Used: 80G, Avail: 158G
--------------------------
Network (Eth0/WLAN0):
  Eth0 IP: 192.168.1.100
--------------------------
Press Ctrl+C to stop.
```

Now, imagine this running in one of your Tmux panes! We'll integrate it into our `dashboard_layout.sh` by replacing the `echo` command with `bash ./sys_info.sh`.

### Integrating into Tmux

Modify `dashboard_layout.sh` to run the `sys_info.sh` script:

```bash
# ... (previous setup) ...

    # Send commands to each pane
    tmux send-keys -t "$SESSION_NAME:0.0" "bash $(pwd)/sys_info.sh" C-m # System Info
    tmux send-keys -t "$SESSION_NAME:0.1" "echo 'Git Status Pane';" C-m
    tmux send-keys -t "$SESSION_NAME:0.2" "echo 'Docker/K8s Pane';" C-m
    tmux send-keys -t "$SESSION_NAME:0.3" "echo 'Interactive Pane';" C-m

# ... (rest of the script) ...
```

Note: `$(pwd)` ensures the script is found regardless of where you execute `dashboard_layout.sh`. `C-m` is the "enter" key.

## The Interaction: fzf for Quick Actions

fzf is a general-purpose command-line fuzzy finder. It takes a list of lines from stdin, lets you interactively filter them, and then prints the selected item(s) to stdout. This makes it incredibly powerful for building interactive prompts.

### Basic fzf Usage

```bash
ls -F | fzf
```

This will show an interactive list of files and directories in your current directory. As you type, it filters. When you press Enter, the selected item is printed.

### Example: Project Picker with fzf (`project_picker.sh`)

Imagine you have all your projects in `~/dev/projects`. You want to quickly `cd` into one.

```bash
#!/bin/bash

# Define your projects base directory
PROJECTS_DIR="$HOME/dev/projects"

# Check if the directory exists
if [ ! -d "$PROJECTS_DIR" ]; then
    echo "Error: Project directory '$PROJECTS_DIR' not found."
    exit 1
fi

echo "--- Project Picker ---"
echo "Select a project to 'cd' into it."
echo "----------------------"

# Use fzf to select a project directory
# -d: only show directories
# -e: enter if there's only one match
# --bind: custom keybinding to open in VS Code instead of cd
SELECTED_PROJECT=$(find "$PROJECTS_DIR" -maxdepth 1 -mindepth 1 -type d | fzf \
    --prompt="Select Project > " \
    --height=40% \
    --layout=reverse \
    --border \
    --header="Type to filter, Enter to select, Ctrl+V to open in VS Code" \
    --bind "ctrl-v:execute(code {} & exit)" \
)

if [ -n "$SELECTED_PROJECT" ]; then
    echo "Changing directory to: $SELECTED_PROJECT"
    # This 'cd' will apply to the parent shell if sourced, or just the subshell if run directly.
    # For a dashboard pane, it might be better to explicitly open a new shell or session.
    cd "$SELECTED_PROJECT" || exit 1
    # You might want to clear the screen and run a default command like `ls -F`
    clear
    echo "You are now in: $(pwd)"
    ls -F
else
    echo "No project selected."
fi
```

Make it executable: `chmod +x project_picker.sh`.

#### Sample Interaction (`./project_picker.sh`)

When you run this script, fzf will pop up:

```output
--- Project Picker ---
Select a project to 'cd' into it.
----------------------
> █
  ~/dev/projects/my-web-app
  ~/dev/projects/api-service
  ~/dev/projects/data-pipeline
  ~/dev/projects/internal-tool
  (Type 'web' to filter, Enter to select)
```

After selecting `my-web-app` and pressing Enter:

```output
Changing directory to: /home/user/dev/projects/my-web-app
You are now in: /home/user/dev/projects/my-web-app
Dockerfile
README.md
src/
...
```

To make `cd` affect your *current* shell within a Tmux pane, you would typically run this script by `source project_picker.sh` rather than `bash project_picker.sh`. When integrated into a dashboard, we'll often use `tmux send-keys` to run it, and if it's meant to change the pane's working directory, you need to ensure it's sourced.

### Integrating fzf into the Dashboard

For the interactive pane, we can send a command to kick off the `project_picker.sh` script, perhaps after sourcing `~/.bashrc` to ensure fzf's keybindings are loaded.

```bash
# ... (inside dashboard_layout.sh) ...

    # Send commands to each pane
    tmux send-keys -t "$SESSION_NAME:0.0" "bash $(pwd)/sys_info.sh" C-m
    tmux send-keys -t "$SESSION_NAME:0.1" "git status --short" C-m
    tmux send-keys -t "$SESSION_NAME:0.2" "docker ps --format 'table {{.ID}}\t{{.Names}}\t{{.Status}}'" C-m
    tmux send-keys -t "$SESSION_NAME:0.3" "source ~/.bashrc && bash $(pwd)/project_picker.sh" C-m

# ... (rest of the script) ...
```

Note: `git status --short` is a simple placeholder. We'd create a more robust git status script later. `docker ps` is also a basic example.

## Bringing It All Together: The `dashboard.sh` Script

Now we combine everything into one master script, `dashboard.sh`. This script will:
1.  Check if our `dev-dash` Tmux session exists.
2.  If it exists, attach to it.
3.  If not, create it with predefined windows and panes.
4.  Populate each pane with specific, automatically updating information or interactive tools.

Let's organize our scripts:
*   `dashboard.sh` (main script)
*   `scripts/sys_info.sh`
*   `scripts/git_status_loop.sh`
*   `scripts/docker_k8s_status.sh`
*   `scripts/interactive_tools.sh` (which sources `project_picker.sh`)
*   `scripts/project_picker.sh`

Create a `scripts` directory and move `sys_info.sh` and `project_picker.sh` into it. Update paths accordingly.

### New Scripts to Create

#### `scripts/git_status_loop.sh`
```bash
#!/bin/bash

# Displays git status for the current directory, auto-refreshing.
# Only works if you are inside a git repository.

while true; do
    clear
    echo "--- Git Status ---"
    if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
        echo "Repo: $(basename "$(git rev-parse --show-toplevel)")"
        echo "Branch: $(git rev-parse --abbrev-ref HEAD)"
        echo "------------------"
        git status --short
    else
        echo "Not in a Git repository."
    fi
    echo "------------------"
    echo "Press Ctrl+C to stop."
    sleep 5
done
```

#### `scripts/docker_k8s_status.sh`
```bash
#!/bin/bash

# Displays Docker containers and Kubernetes context/pods, auto-refreshing.

while true; do
    clear
    echo "--- Docker Status ---"
    if command -v docker >/dev/null 2>&1 && docker info >/dev/null 2>&1; then
        docker ps --format 'table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}'
    else
        echo "Docker not running or not found."
    fi
    echo "---------------------"

    echo "--- Kubernetes Status ---"
    if command -v kubectl >/dev/null 2>&1; then
        echo "Current Context: $(kubectl config current-context 2>/dev/null || echo 'N/A')"
        echo "Pods (default namespace):"
        kubectl get pods --all-namespaces --field-selector=status.phase!=Running,status.phase!=Succeeded,status.phase!=Failed --no-headers 2>/dev/null | wc -l | xargs printf "  %s non-running pods\n"
        kubectl get pods --field-selector=status.phase=Running --no-headers 2>/dev/null | head -n 5 # Show first 5 running pods
        echo "..."
    else
        echo "kubectl not found."
    fi
    echo "-------------------------"
    echo "Press Ctrl+C to stop."
    sleep 7
done
```

#### `scripts/interactive_tools.sh`
This script can be a menu for interactive tools. For simplicity, we'll just launch the `project_picker.sh` from here. In a real dashboard, you might have an fzf menu of various tools/scripts you want to run.

```bash
#!/bin/bash

# Source project_picker so 'cd' affects the current pane's shell
source "$(dirname "$0")/project_picker.sh"
```
Note: This `interactive_tools.sh` script is just an example. For `cd` to work within the dashboard pane, you must `source` the `project_picker.sh` directly within the `tmux send-keys` command for that pane, not call `bash interactive_tools.sh`. A better approach for "interactive tools" pane would be to just have a shell, and allow the user to manually run `source ~/dashboard_scripts/project_picker.sh`. Or, if it's meant to *always* be running an interactive fzf menu, the `while true` loop is needed around the fzf command. Let's make `interactive_tools.sh` simply keep prompting for `project_picker`.

Let's refine `scripts/interactive_tools.sh`:

```bash
#!/bin/bash

# Loop to continuously offer interactive tools.

while true; do
    clear
    echo "--- Interactive Tools ---"
    echo "1. Project Picker (cd into project)"
    echo "2. Git Branch Checkout (fzf)"
    echo "3. Docker Container Action (fzf)"
    echo "4. ... your custom tools"
    echo "-------------------------"
    echo "Type number and Enter, or 'q' to quit this menu."
    echo ""

    # Provide a simple prompt, then call sub-scripts.
    read -p "Select action: " CHOICE

    case "$CHOICE" in
        1)
            # Source the project picker to affect the current shell
            source "$(dirname "$0")/project_picker.sh"
            ;;
        2)
            # Example: A simple fzf for git branches
            if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
                SELECTED_BRANCH=$(git branch --all | fzf --prompt="Select Branch > " --height=20% --layout=reverse --border --header="Type to filter, Enter to checkout")
                if [ -n "$SELECTED_BRANCH" ]; then
                    BRANCH_NAME=$(echo "$SELECTED_BRANCH" | sed 's/remotes\/origin\///g' | sed 's/\* //g' | xargs) # Clean up branch name
                    echo "Checking out branch: $BRANCH_NAME"
                    git checkout "$BRANCH_NAME"
                    read -p "Press Enter to continue..." # Pause
                fi
            else
                echo "Not in a Git repository."
                read -p "Press Enter to continue..." # Pause
            fi
            ;;
        q|Q)
            echo "Exiting interactive tools menu."
            break
            ;;
        *)
            echo "Invalid choice. Please try again."
            sleep 1
            ;;
    esac
    sleep 0.5 # Small delay to avoid flickering
done
echo "Interactive tools loop exited. Press Ctrl+C to stop the pane."
```

This makes the interactive pane much more useful.

### The Main `dashboard.sh` Script

```bash
#!/bin/bash

# Define the Tmux session name
SESSION_NAME="dev-dash"
DASH_SCRIPTS_DIR="$(dirname "$0")/scripts" # Relative path to scripts directory

# Function to check if a command exists
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Check if Tmux session exists
tmux has-session -t "$SESSION_NAME" 2>/dev/null

if [ $? == 0 ]; then
    echo "Attaching to existing session: $SESSION_NAME"
    tmux attach -t "$SESSION_NAME"
else
    echo "Creating new session: $SESSION_NAME"
    # Create a new detached session
    tmux new-session -d -s "$SESSION_NAME"

    # --- Window 0: Dashboard Overview ---
    # Pane 0 (top-left): System Information
    tmux send-keys -t "$SESSION_NAME:0.0" "clear; bash ${DASH_SCRIPTS_DIR}/sys_info.sh" C-m

    # Pane 1 (top-right): Git Status
    tmux split-window -h -t "$SESSION_NAME:0.0"
    tmux send-keys -t "$SESSION_NAME:0.1" "clear; bash ${DASH_SCRIPTS_DIR}/git_status_loop.sh" C-m

    # Pane 2 (bottom-left): Docker/K8s Status
    tmux split-window -v -t "$SESSION_NAME:0.0"
    tmux send-keys -t "$SESSION_NAME:0.2" "clear; bash ${DASH_SCRIPTS_DIR}/docker_k8s_status.sh" C-m

    # Pane 3 (bottom-right): Interactive Tools
    tmux split-window -v -t "$SESSION_NAME:0.1"
    tmux send-keys -t "$SESSION_NAME:0.3" "clear; bash ${DASH_SCRIPTS_DIR}/interactive_tools.sh" C-m

    # Optional: adjust pane sizes for a balanced look
    # tmux select-layout -t "$SESSION_NAME:0" tiled # Sometimes helps with initial layout
    tmux select-pane -t "$SESSION_NAME:0.0" # Move cursor to top-left pane

    # --- Window 1: Logs/Tail ---
    tmux new-window -t "$SESSION_NAME" -n "Logs"
    tmux send-keys -t "$SESSION_NAME:1.0" "echo 'This window is for tailing logs. Example: tail -f /var/log/syslog'" C-m
    # Example: Start a simple Python web server and tail its logs in a separate pane
    # tmux split-window -v -t "$SESSION_NAME:1.0"
    # tmux send-keys -t "$SESSION_NAME:1.1" "cd ~/my_project/backend && python -m http.server 8000 > server.log 2>&1 & tail -f server.log" C-m

    # --- Window 2: Scratchpad/Shell ---
    tmux new-window -t "$SESSION_NAME" -n "Scratchpad"
    tmux send-keys -t "$SESSION_NAME:2.0" "echo 'This is a general purpose shell for quick commands.'" C-m

    # Select the first window (index 0) and attach
    tmux select-window -t "$SESSION_NAME:0"
    tmux attach -t "$SESSION_NAME"
fi
```

Make `dashboard.sh` executable: `chmod +x dashboard.sh`.

### Directory Structure

```
.
├── dashboard.sh
└── scripts/
    ├── docker_k8s_status.sh
    ├── git_status_loop.sh
    ├── interactive_tools.sh
    ├── project_picker.sh
    └── sys_info.sh
```

### Running Your Dashboard

Simply run `./dashboard.sh` from the root of your dashboard setup.

```output
Creating new session: dev-dash
[detached (from terminal 0)]
```
(Followed by Tmux attaching and showing the layout.)

You'll now see your custom dashboard!
*   `Ctrl+b 0` to go back to the main dashboard.
*   `Ctrl+b 1` to go to the Logs window.
*   `Ctrl+b 2` to go to the Scratchpad window.
*   `Ctrl+b d` to detach from the session.
*   Run `./dashboard.sh` again to reattach.

## Practical Examples & Enhancements

This is just the beginning. Your dashboard can be infinitely customized.

### 1. Conditional Displays

Use `if command_exists <tool>` or `if [ -f <file> ]` within your scripts to only display information if the relevant tool is installed or a file exists. This keeps your dashboard clean even on different machines.

Example: `docker_k8s_status.sh` already does this for `docker` and `kubectl`.

### 2. Monitoring Logs

Dedicate a pane or a full window to live log tailing.

```bash
# In dashboard.sh, for a pane, or a new window
tail -f /var/log/syslog # or a specific application log
```
You can pass arguments to scripts like this: `tmux send-keys -t "$SESSION_NAME:1.0" "tail -f /path/to/my/app.log" C-m`.

### 3. Quick Git Actions with fzf

Expand `interactive_tools.sh` with more git actions:

```bash
# ... inside interactive_tools.sh ...
        2)
            if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
                # Git Log Grep with fzf
                SELECTED_COMMIT=$(git log --oneline --grep-reflog="$SEARCH_TERM" | fzf --prompt="Search Commit > " --height=40% --layout=reverse --border --header="Type to filter, Enter to view commit")
                if [ -n "$SELECTED_COMMIT" ]; then
                    COMMIT_HASH=$(echo "$SELECTED_COMMIT" | awk '{print $1}')
                    echo "Viewing commit: $COMMIT_HASH"
                    git show "$COMMIT_HASH"
                    read -p "Press Enter to continue..."
                fi
            fi
            ;;
# ...
```

### 4. Docker/Kubernetes Interaction

Use `fzf` to select a running Docker container and then attach to it, view its logs, or stop it.

```bash
# In scripts/docker_actions.sh (called from interactive_tools.sh)
#!/bin/bash
CONTAINER_ID=$(docker ps --format '{{.ID}}\t{{.Names}}' | fzf --prompt="Select Container > " --height=40% --layout=reverse --border --header="Select container to act on")

if [ -n "$CONTAINER_ID" ]; then
    CLEAN_ID=$(echo "$CONTAINER_ID" | awk '{print $1}')
    echo "Selected container: $(echo "$CONTAINER_ID" | awk '{print $2}') ($CLEAN_ID)"
    echo "1. Attach (exec /bin/bash)"
    echo "2. View logs"
    echo "3. Stop container"
    read -p "Choose action: " ACTION

    case "$ACTION" in
        1) docker exec -it "$CLEAN_ID" /bin/bash ;;
        2) docker logs -f "$CLEAN_ID" ;;
        3) docker stop "$CLEAN_ID" ;;
        *) echo "Invalid action." ;;
    esac
    read -p "Press Enter to continue..."
fi
```

### 5. Integrating Other CLI Tools

Your dashboard panes can run anything:
*   `btop` or `glances` for advanced system monitoring.
*   `kubectl get events -w` for real-time Kubernetes events.
*   `gh pr list` (GitHub CLI) for open PRs.
*   `npm outdated` or `pip list --outdated`.
*   A custom script to fetch data from a Jira or Trello API.

## Considerations & Limitations

*   **CLI-Centric:** This dashboard is text-based. No fancy graphs, but excellent for quick, raw data.
*   **Resource Usage:** Scripts running in `while true` loops with `sleep` are generally fine, but too many complex loops can consume CPU. Monitor your system.
*   **Dependencies:** Ensure all necessary CLI tools (`docker`, `kubectl`, `git`, `python`, etc.) are installed on the system where your dashboard runs.
*   **Maintenance:** Custom scripts require maintenance. If an API changes or a command syntax updates, you'll need to update your script.
*   **Portability:** The scripts themselves are often portable Bash, but the `tmux` layout and specific commands might need minor tweaks between systems (e.g., `eth0` vs `en0`).
*   **Security:** Be mindful of what scripts you run, especially if they interact with sensitive data or perform actions. Avoid hardcoding credentials.

## Conclusion

Building a dev dashboard with Tmux, Bash, and fzf transforms your terminal into a powerhouse. It's a highly personalized, efficient, and deeply integrated workspace that centralizes vital information and actions. While it takes an initial investment to set up, the long-term productivity gains are significant.

Start small, customize incrementally, and make it *yours*. This isn't a one-size-fits-all solution; it's a framework to build the ultimate command-line companion for your daily development workflow. Experiment with different layouts, information sources, and interactive fzf flows. The possibilities are endless.

Happy hacking!