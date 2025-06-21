---
categories:
- Productivity
- Software
- Developer Tools
comments: true
cover:
  image: https://images.pexels.com/photos/6932280/pexels-photo-6932280.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 11:22:34.549000
description: Unlock powerful command-line task and time management with Taskwarrior
  and Timewarrior. This deep dive covers installation, core usage, their potent synergy,
  and advanced tips for maintaining productivity and gaining profound insights into
  your work.
tags:
- productivity
- task management
- time tracking
- command line
- open source
- Linux
- macOS
- privacy
title: Using Taskwarrior + Timewarrior for Task Tracking That Sticks
---

![A subtle shadow of a flower cast on a textured wall, creating a calm and artistic ambiance.](https://images.pexels.com/photos/6932280/pexels-photo-6932280.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "A subtle shadow of a flower cast on a textured wall, creating a calm and artistic ambiance.")

## Using Taskwarrior + Timewarrior for Task Tracking That Sticks

The quest for the perfect task management system is a journey many of us embark on, often hopping from one shiny new app to the next. We try a new methodology, download a sophisticated GUI, or subscribe to a feature-rich SaaS, only to find our meticulously organized lists slowly but surely gathering digital dust. The tasks don't stick. The tracking falters.

What if the secret to a task system that truly "sticks" isn't more features, a prettier interface, or an AI-powered assistant, but rather a stripped-down, privacy-centric, and incredibly powerful approach that lives right in your terminal?

Enter **Taskwarrior** and **Timewarrior**.

These are not your average productivity apps. They are robust, open-source command-line tools designed for those who value efficiency, control, and deep insights into their work. Taskwarrior manages your to-do list, while Timewarrior precisely tracks the time you spend on tasks. Used together, they form an unparalleled system for managing your professional and personal life that, once mastered, becomes an indispensable part of your workflow.

## Taskwarrior: Your Command-Line Task Commander

Taskwarrior is a free, open-source command-line task list manager. It's designed for people who want to filter, annotate, modify, and track their tasks with speed and precision, without ever leaving the terminal.

### Why Taskwarrior?

1.  **Speed & Efficiency**: No mouse clicks, no context switching. Add, complete, or modify tasks with a few keystrokes.
2.  **Power & Flexibility**: Extensive filtering, tagging, project management, recurring tasks, dependencies, and customizable reports.
3.  **Privacy**: Your data stays on your machine, in plain text files. No cloud servers, no data mining.
4.  **Extensibility**: Hooks allow you to automate workflows, and its plain-text data format makes scripting easy.
5.  **Ubiquity**: Available on Linux, macOS, FreeBSD, Cygwin, and Windows (via WSL).

### Installation

Installation is straightforward on most systems.

**Linux (Debian/Ubuntu):**
```bash
sudo apt install taskwarrior
```

**macOS (Homebrew):**
```bash
brew install taskwarrior
```

For other systems, refer to the [official Taskwarrior installation guide](https://taskwarrior.org/download/).

### Core Concepts & Basic Usage

Taskwarrior stores your tasks in plain text files in `~/.task/` (or a location specified in your `TASKDATA` environment variable). The primary configuration file is `~/.taskrc`.

Let's dive into some essential commands:

*   **Adding a task:**
    ```bash
    task add "Write blog post about Taskwarrior + Timewarrior"
    ```
    You'll get a UUID (Universally Unique Identifier) for the task. You can refer to tasks by their UUID or their shorter ID (e.g., `1`, `2`, `3`).

*   **Adding with properties:**
    ```bash
    task add "Plan next sprint" project:development due:tomorrow priority:H +agile +meeting
    ```
    Here, `project:` assigns it to a project, `due:` sets a due date, `priority:` sets urgency, and `+tag` adds tags.

*   **Listing tasks:**
    ```bash
    task list                  # List all pending tasks
    task ls +agile             # List tasks tagged with 'agile'
    task project:development   # List tasks in 'development' project
    task 1-3,5                 # List tasks with IDs 1, 2, 3, and 5
    task next                  # List tasks that are most urgent according to its urgency calculation
    ```
    Taskwarrior's filtering is incredibly powerful. You can combine multiple criteria, use logical operators (`and`, `or`), and wildcards.

*   **Viewing task details:**
    ```bash
    task 1 info                # Get detailed info about task ID 1
    ```

*   **Modifying a task:**
    ```bash
    task 1 modify "Refine outline" priority:M -agile
    ```
    This changes the description, sets priority to medium, and removes the `agile` tag.

*   **Starting/Stopping a task (marking active):**
    ```bash
    task 1 start
    task 1 stop
    ```
    `start` marks a task as active. This is useful for tracking what you're currently working on, though it doesn't *track time* itself (that's Timewarrior's job).

*   **Completing a task:**
    ```bash
    task 1 done
    ```
    This marks the task as completed and removes it from the pending list.

*   **Deleting a task:**
    ```bash
    task 1 delete
    ```
    Use with caution! It will ask for confirmation.

### Why it "Sticks"

Taskwarrior often succeeds where others fail because of its **low friction** and **omnipresence**. It's always just a `task add` away. The lack of a distracting GUI means you focus on the task itself, not the tool. Its powerful filtering and reporting capabilities ensure you can always find exactly what you need, making the system reliable and trustworthy. The data's local nature also fosters a sense of ownership and control, free from the whims of cloud service providers.

## Timewarrior: Precise Time Tracking for Deeper Insights

Timewarrior is a free, open-source command-line time tracker. It's designed to be simple, efficient, and accurate. While it can be used standalone, its true power shines when integrated with Taskwarrior.

### Why Timewarrior?

1.  **Granular Tracking**: Record exact start and end times for activities.
2.  **Insightful Reports**: Understand where your time goes, identify distractions, and improve estimations.
3.  **Privacy**: Like Taskwarrior, your data stays local and private.
4.  **Flexible Tagging**: Attach tags to your time entries for categorization.
5.  **Integration**: Designed to work seamlessly with Taskwarrior.


Timewarrior often has similar installation methods to Taskwarrior.

**Linux (Debian/Ubuntu):**
```bash
sudo apt install timewarrior
```

**macOS (Homebrew):**
```bash
brew install timewarrior
```

For other systems, check the [official Timewarrior download page](https://timewarrior.net/download/).


Timewarrior stores its data in `~/.timewarrior/`. Its primary configuration file is `~/.timewarrior/timewarrior.cfg`.

*   **Starting a time-tracking session:**
    ```bash
    timew start "Writing blog post" +writing +blogging
    ```
    This starts a new time interval, tagged with `writing` and `blogging`.

*   **Stopping the current session:**
    ```bash
    timew stop
    ```

*   **Adding a past session:**
    ```bash
    timew track 8am - 9am "Daily standup" +meeting
    timew track 2023-10-26T14:00 - 2023-10-26T16:30 "Refactoring code" project:dev
    ```
    You can specify absolute or relative times.

*   **Viewing time reports:**
    ```bash
    timew summary                # Summary of today's tracked time
    timew today                  # Alias for summary
    timew yesterday summary      # Summary for yesterday
    timew week summary           # Summary for the current week
    timew month summary          # Summary for the current month
    timew total                  # Total tracked time
    timew tags                   # List all tags used
    ```
    You can also filter reports:
    ```bash
    timew today :ids             # Show IDs for today's entries
    timew today +writing summary # Summary of time spent on 'writing' today
    ```

*   **Manipulating entries (using IDs from `:ids` report):**
    ```bash
    timew 1 annotate "Added more details"    # Add an annotation to entry ID 1
    timew 2 tag +urgent                      # Add a tag to entry ID 2
    timew 3 untag -urgent                    # Remove a tag from entry ID 3
    timew 4 lengthen 10min                   # Extend entry ID 4 by 10 minutes
    timew 5 shorten 30min                    # Shorten entry ID 5 by 30 minutes
    timew 6 modify 9am - 10:30am             # Change start/end times
    timew 7 delete                           # Delete entry ID 7
    ```

## The Synergy: Taskwarrior + Timewarrior = Productivity Powerhouse

This is where the magic truly happens. While `task start` marks a task as active, it's `timewarrior` that *records the time*. The optimal workflow involves linking the two.

**The simplest way to integrate is manually:**

1.  **Identify the task:** Use `task next` or `task list` to pick your next item. Let's say it's task ID `1` for "Write blog post".
2.  **Start the task (optional, but good habit for Taskwarrior's internal state):**
    ```bash
    task 1 start
    ```
3.  **Start time tracking for that task with Timewarrior:**
    ```bash
    timew start :task 1 "Write blog post"
    ```
    Using `:task <id>` is a convention often used in combined scripts or just as a clear way to link. You can also simply copy the task description or relevant tags:
    ```bash
    timew start "Write blog post about Taskwarrior + Timewarrior" +writing +blogging
    ```
4.  **When done or switching, stop Timewarrior:**
    ```bash
    timew stop
    ```
5.  **Mark the Taskwarrior task as done:**
    ```bash
    task 1 done
    ```

### Achieving True Automation: The Taskwarrior Timewarrior Hook

Manually remembering to start and stop `timew` and `task` can be cumbersome. This is where **hooks** come in. Taskwarrior supports external scripts that run when certain events occur (e.g., `on-modify`, `on-add`, `on-delete`, `on-start`, `on-stop`, `on-done`).

The most powerful integration is a dedicated [Taskwarrior Timewarrior hook script](https://github.com/GothenburgBitFactory/taskwarrior-timewarrior-hook). This script automates the time tracking:

*   When you run `task <id> start`, the hook automatically calls `timew start` with the task's description and tags.
*   When you run `task <id> stop` or `task <id> done`, the hook automatically calls `timew stop`.

**To install this hook:**

1.  Clone the repository:
    ```bash
    git clone https://github.com/GothenburgBitFactory/taskwarrior-timewarrior-hook.git
    ```
2.  Copy the `task-timewarrior.py` script into your Taskwarrior hooks directory. By default, this is `~/.task/hooks/`.
    ```bash
    cp taskwarrior-timewarrior-hook/task-timewarrior.py ~/.task/hooks/
    ```
3.  Make sure the script is executable:
    ```bash
    chmod +x ~/.task/hooks/task-timewarrior.py
    ```

Now, your workflow becomes significantly simpler:

```bash
task add "Review pull request" +code +review
task 2 start             # This will automatically start timewarrior for task 2
# ... work on the pull request ...
task 2 done              # This will automatically stop timewarrior and mark task 2 as done
```

This automation is a game-changer for consistency, making the tracking truly "stick" without extra cognitive load.

## Advanced Tips & Tricks

### 1. Customizing Taskwarrior Reports

Taskwarrior's `taskrc` file (`~/.taskrc`) is incredibly powerful. You can define custom reports, modify columns, and set up aliases.

Example custom report for a weekly summary (add to `~/.taskrc`):
```ini
report.weekly.columns=id,uuid,start,end,description,project,tags
report.weekly.labels=ID,UUID,Started,Ended,Description,Project,Tags
report.weekly.sort=start+
report.weekly.filter=status:completed and end.after:now-7d and end.before:now+1d
report.weekly.description=Completed tasks in the last 7 days
```
Then run: `task weekly`

### 2. Timewarrior Extensions

Timewarrior supports extensions for more advanced reporting and visualizations. Check out the [Timewarrior Extensions repository](https://timewarrior.net/docs/extensions/) for community-contributed scripts. You can find things like Gantt charts, JSON exporters, and more.

### 3. Synchronization with Taskserver

For multi-device synchronization, Taskwarrior offers `taskd`, a dedicated server component. This allows your tasks to be synced securely across all your machines. It requires a bit more setup, but ensures your task list is always up-to-date wherever you are. [Learn more about Taskserver](https://taskwarrior.org/docs/taskserver/).

### 4. Aliases and Shell Functions

To speed up common commands, leverage your shell's alias or function capabilities.

Example `~/.bashrc` or `~/.zshrc`:
```bash
alias t="task"
alias tw="timew"
alias td="task done"
alias ts="task start"
alias tl="task list"

# Function to start a task and immediately list what's active
tstart() {
    task "$@" start && task active
}
```

### 5. Visualizations

While CLI-focused, some tools offer a more visual representation:
*   `vit` (Visual Interactive Taskwarrior) provides a ncurses-based UI in your terminal.
*   Custom scripts using `task export` (JSON output) can be parsed and visualized with tools like Python or JavaScript.

## Considerations and Learning Curve

It's important to be honest: Taskwarrior and Timewarrior have a learning curve.
*   **Initial Setup**: Getting everything installed and configured, especially the hook script, can take a little time.
*   **Command Recall**: Memorizing commands and filters takes practice. However, the consistent syntax makes it intuitive once you get the hang of it.
*   **Discipline**: Like any system, it requires discipline to use consistently. The automation from the hook helps immensely, but you still need to remember to `task start` and `task done`.
*   **No GUI**: For some, the lack of a graphical interface is a barrier. For others, it's the main attraction. There are no fancy notifications, drag-and-drop, or collaborative features built-in. This is a personal productivity tool, first and foremost.

**Note:** While Taskwarrior and Timewarrior are incredibly stable and reliable, ensure you back up your `~/.task/` and `~/.timewarrior/` directories periodically, especially if you're experimenting with hooks or advanced configurations.

## Conclusion

Taskwarrior and Timewarrior offer a refreshing alternative to the ever-growing ecosystem of complex, cloud-dependent productivity apps. They empower you with unparalleled control over your tasks and time, all while respecting your privacy. The initial investment in learning these command-line tools pays dividends in efficiency, focus, and genuine insight into your productivity.

When your task list becomes a natural extension of your terminal, always accessible and effortlessly managed, that's when task tracking truly starts to stick. Ditch the endless app-hopping, embrace the command line, and reclaim your productivity with this potent duo. Your future self (and your completed tasks) will thank you.
