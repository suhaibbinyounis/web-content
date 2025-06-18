---
title: Understanding the Basics Core Concepts Every Dev Should Know
date: 2025-06-18T02:04:08.681Z
description: Dive into fundamental developer concepts like environment variables, standard streams, process management, and file permissions with practical examples.
tags:
  - Linux
  - CLI
  - Bash
  - EnvironmentVariables
  - StandardStreams
  - ProcessManagement
  - FilePermissions
categories:
  - DevOps
  - Fundamentals
  - SystemAdministration
comments: true
---

As developers, we often jump straight into frameworks, libraries, and high-level programming languages. While that's crucial for building applications, a solid grasp of underlying "basic concepts" can elevate your debugging skills, improve your system interactions, and make you a more robust problem-solver.

This post will explore four fundamental concepts that are ubiquitous in the development world, especially if you work with Linux, the command line, or any form of system administration.

## 1. Environment Variables

**Concept:** Environment variables are dynamic named values that can affect the way running processes behave on a computer. They are part of the environment in which a process runs. Think of them as key-value pairs that the operating system makes available to any program.

**Why it Matters:**
-   **Configuration:** They allow you to configure applications without changing their code (e.g., database connection strings, API keys, feature flags).
-   **Paths:** The `PATH` variable tells your shell where to look for executable commands.
-   **Security:** Avoid hardcoding sensitive information directly into your source code.
-   **Portability:** Configure applications for different environments (development, staging, production) easily.

**Examples:**

Let's start by looking at some common environment variables.

### Viewing Environment Variables

You can view all current environment variables or a specific one.

```bash
# View a specific environment variable
echo $PATH

# View another common variable
echo $HOME

# List all environment variables
env
```

```output
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
/home/youruser
# ... (output of 'env' will be long, showing many key=value pairs)
LANG=en_US.UTF-8
PWD=/home/youruser/myproject
SHELL=/bin/bash
...
```

### Setting a Temporary Environment Variable

Variables set directly in the shell without `export` are only available within the current shell session and not to sub-processes.

```bash
# Set a temporary variable
MY_TEMP_VAR="Hello from shell"

# Access it in the current shell
echo $MY_TEMP_VAR
```

```output
Hello from shell
```

### Exporting an Environment Variable for Sub-processes

To make an environment variable available to any child processes or scripts executed from your current shell, you must `export` it.

```bash
# Export a variable
export MY_EXPORTED_VAR="This is exported"

# Access it in the current shell
echo $MY_EXPORTED_VAR

# Run a sub-process (a bash script) that tries to access it
bash -c 'echo "Inside sub-process: $MY_EXPORTED_VAR"'
```

```output
This is exported
Inside sub-process: This is exported
```

**Note:** If `MY_EXPORTED_VAR` hadn't been `export`ed, the sub-process would have output "Inside sub-process: ". This distinction is crucial for scripts and applications.

### Accessing Environment Variables in Python

Most programming languages provide a way to access environment variables. Here's how to do it in Python.

```python
# env_access.py
import os

# Access a variable that might be set (e.g., from our previous export)
my_var = os.getenv('MY_EXPORTED_VAR')
print(f"MY_EXPORTED_VAR: {my_var}")

# Access a common system variable
path_var = os.getenv('PATH')
print(f"PATH: {path_var}")

# Access a variable that doesn't exist
non_existent = os.getenv('NON_EXISTENT_VAR', 'Default Value')
print(f"NON_EXISTENT_VAR: {non_existent}")
```

```bash
# First, ensure the variable is exported
export MY_EXPORTED_VAR="Python can see me!"

# Then run the Python script
python env_access.py
```

```output
MY_EXPORTED_VAR: Python can see me!
PATH: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
NON_EXISTENT_VAR: Default Value
```

## 2. Standard Streams (stdin, stdout, stderr)

**Concept:** Every process on a Unix-like system (Linux, macOS) by default has three standard I/O (Input/Output) streams:
-   **Standard Input (stdin)**: File descriptor 0. Where a program reads its input from (e.g., keyboard, piped output from another program).
-   **Standard Output (stdout)**: File descriptor 1. Where a program writes its normal output (e.g., to the terminal, redirected to a file).
-   **Standard Error (stderr)**: File descriptor 2. Where a program writes its error messages (e.g., to the terminal, redirected to a separate error log).

**Why it Matters:**
-   **Inter-process Communication:** The `|` (pipe) operator connects `stdout` of one command to `stdin` of another, enabling powerful command chaining.
-   **Logging:** Separating normal output from error messages is critical for robust logging and debugging.
-   **Redirection:** Directing output or input to/from files allows for automated processing and persistence.

**Examples:**

### `stdout` Basics

The `echo` command writes directly to `stdout`.

```bash
echo "This message goes to standard output."
```

```output
This message goes to standard output.
```

### `stdin` Basics

The `cat` command, when used without file arguments, reads from `stdin`. You can terminate input with `Ctrl+D`.

```bash
cat
# Type some text here, then press Enter
# Type more text, then press Ctrl+D
```

```output
This is input.
It will be echoed back.
This is input.
It will be echoed back.
```

### Redirecting `stdout` to a File (`>`)

The `>` operator redirects `stdout` to a file, overwriting the file if it exists. `>>` appends.

```bash
echo "First line for the file." > my_output.txt
echo "Second line appended." >> my_output.txt
cat my_output.txt
```

```output
First line for the file.
Second line appended.
```

### Redirecting `stderr` to a File (`2>`)

Error messages often go to `stderr`. You can redirect `stderr` specifically using `2>`.

```bash
# This command tries to remove a non-existent file, generating an error
rm non_existent_file.txt 2> error_log.txt

# Check the contents of the error log
cat error_log.txt
```

```output
rm: cannot remove 'non_existent_file.txt': No such file or directory
```

**Note:** The error message itself did not appear on your terminal because it was redirected to `error_log.txt`.

### Redirecting Both `stdout` and `stderr` (`&>` or `> file 2>&1`)

Sometimes you want both regular output and errors to go to the same place.

```bash
# Create a dummy command with both stdout and stderr
(echo "Regular message"; echo "Error message" >&2; echo "Another regular message") &> combined_log.txt

# Inspect the combined log
cat combined_log.txt
```

```output
Regular message
Error message
Another regular message
```

### Piping (`|`) `stdout` to `stdin`

The pipe `|` is incredibly powerful. It takes the `stdout` of the command on the left and feeds it as `stdin` to the command on the right.

```bash
# List files, then count the lines of the output
ls -l | wc -l
```

```output
7
# (Your number will vary based on files in your current directory)
```

Here, `ls -l` outputs its directory listing to `stdout`. This `stdout` is then piped into `wc -l` (word count - lines), which treats it as its `stdin` and counts the lines.

## 3. Process Management (PID, Foreground/Background)

**Concept:** A process is an instance of a computer program that is being executed. Each process has a unique Process ID (PID). Processes can run in the foreground (interacting directly with the terminal) or in the background (running independently, allowing you to use the terminal for other tasks).

**Why it Matters:**
-   **Resource Management:** Identify and monitor resource-heavy processes.
-   **Task Control:** Start, stop, suspend, and resume long-running tasks.
-   **Debugging:** Pinpoint and terminate misbehaving applications.

**Examples:**

### Running a Process in the Foreground

When you run a command directly, it runs in the foreground, and your terminal waits for it to finish.

```bash
# This command will run for 5 seconds and block your terminal
sleep 5
echo "Sleep finished."
```

```output
# (Waits 5 seconds)
Sleep finished.
```

### Running a Process in the Background (`&`)

Appending `&` to a command runs it in the background, freeing up your terminal immediately. The shell will print a job number and the PID.

```bash
# Start a long-running process in the background
sleep 100 &
echo "Sleep command started in background."
```

```output
[1] 12345
Sleep command started in background.
```
**Note:** `[1]` is the job ID, and `12345` is the PID. Your PID will be different.

### Listing Background Jobs (`jobs`)

The `jobs` command shows processes currently running in the background for your shell session.

```bash
jobs
```

```output
[1]+  Running                 sleep 100 &
```

### Bringing a Background Process to Foreground (`fg`)

You can bring a specific background job back to the foreground using `fg` followed by its job ID.

```bash
# If sleep 100 & is job [1]
fg %1
```

```output
sleep 100
# (Terminal is now blocked by sleep 100)
```
**Note:** If you press `Ctrl+C` while it's in the foreground, it will terminate.

### Suspending a Foreground Process and Moving to Background (`Ctrl+Z`, `bg`)

If you have a foreground process and want to temporarily pause it and move it to the background:
1.  Press `Ctrl+Z` to suspend the process.
2.  Type `bg` (or `bg %<job_id>`) to resume it in the background.

```bash
# Start a process (e.g., 'top' or 'ping google.com')
ping google.com
```

```output
# (ping starts running)
PING google.com (142.250.187.78) 56(84) bytes of data.
64 bytes from ...
# ... (press Ctrl+Z here)
```

```bash
# After Ctrl+Z, you'll see something like:
^Z
[1]+  Stopped                 ping google.com
# Now, to resume it in the background:
bg
```

```output
[1]+ ping google.com &
```

The `ping` command is now running in the background.

### Finding a Process's PID (`pgrep`, `ps`)

Knowing a process's PID is essential for interacting with it, especially for killing it.

```bash
# Start a dummy process in the background
sleep 3600 &

# Find its PID using pgrep (by name)
pgrep sleep
```

```output
12345
```

```bash
# Alternatively, using ps aux (show all processes) and grep
ps aux | grep "sleep 3600" | grep -v "grep"
```

```output
youruser  12345  0.0  0.0   2644   636 pts/0    S    10:00   0:00 sleep 3600
```
**Note:** The second column in `ps aux` output is the PID. The `grep -v "grep"` part is to filter out the `grep` command itself from the output.

### Killing a Process (`kill`)

Once you have a PID, you can terminate a process using `kill`. The default signal is `SIGTERM` (15), which asks the process to shut down gracefully. `SIGKILL` (9) forces termination immediately.

```bash
# Find the PID of the sleep process you started
PID_TO_KILL=$(pgrep sleep | head -n 1) # Get the first one if multiple
echo "Killing PID: $PID_TO_KILL"

# Send SIGTERM (graceful shutdown)
kill $PID_TO_KILL
```

```output
Killing PID: 12345
# You might see 'Terminated' or 'Killed' depending on the shell and process state.
```
If the process doesn't terminate, you might need `kill -9 $PID_TO_KILL`. Use `kill -9` with caution, as it doesn't allow the process to clean up.

## 4. File Permissions

**Concept:** Unix-like systems control access to files and directories using a permission system. Each file/directory has permissions for three categories of users:
-   **User (u)**: The owner of the file.
-   **Group (g)**: Users belonging to the file's group.
-   **Others (o)**: Everyone else.

For each category, there are three types of permissions:
-   **Read (r)**: View contents (files), list directory contents (directories).
-   **Write (w)**: Modify contents (files), create/delete files in directory (directories).
-   **Execute (x)**: Run as a program (files), enter/traverse directory (directories).

These permissions are often represented in symbolic form (e.g., `rwx`) or octal (e.g., `7`).

**Why it Matters:**
-   **Security:** Prevent unauthorized access or modification of sensitive files.
-   **Stability:** Ensure only authorized users can make changes to critical system files.
-   **Functionality:** Make scripts executable, allow web servers to read web content, etc.

**Examples:**

### Viewing File Permissions (`ls -l`)

The `ls -l` command provides a detailed listing, including permissions.

```bash
touch my_file.txt
mkdir my_directory
ls -l
```

```output
-rw-rw-r-- 1 youruser youruser    0 Oct 27 14:30 my_file.txt
drwxrwxr-x 2 youruser youruser 4096 Oct 27 14:30 my_directory
```

**Breaking down the permission string (`-rw-rw-r--` or `drwxrwxr-x`):**
-   The first character: `-` for a regular file, `d` for a directory.
-   Next three (`rw-`): Permissions for the **user** (read, write, no execute).
-   Next three (`rw-`): Permissions for the **group** (read, write, no execute).
-   Last three (`r--`): Permissions for **others** (read, no write, no execute).

For `my_directory`, `drwxrwxr-x` means:
-   `d`: It's a directory.
-   `rwx`: User (owner) can read, write, execute (traverse/enter).
-   `rwx`: Group can read, write, execute.
-   `r-x`: Others can read, execute (but not write/create/delete in it).

### Changing Permissions with `chmod` (Symbolic Mode)

`chmod` (change mode) is used to modify permissions. Symbolic mode is very readable.

```bash
# Add execute permission for the user
chmod u+x my_file.txt
ls -l my_file.txt
```

```output
-rwxrw-r-- 1 youruser youruser 0 Oct 27 14:30 my_file.txt
```

```bash
# Remove write permission for others
chmod o-w my_file.txt
ls -l my_file.txt
```

```output
-rwxrwxr-- 1 youruser youruser 0 Oct 27 14:30 my_file.txt
```

### Changing Permissions with `chmod` (Octal Mode)

Octal mode assigns a numerical value to each permission:
-   `r` = 4
-   `w` = 2
-   `x` = 1
-   No permission = 0

You sum these values for each user category. For example:
-   `rwx` = 4 + 2 + 1 = 7
-   `rw-` = 4 + 2 + 0 = 6
-   `r-x` = 4 + 0 + 1 = 5
-   `r--` = 4 + 0 + 0 = 4

A common permission for executable scripts is `755`:
-   User: `rwx` (7)
-   Group: `r-x` (5)
-   Others: `r-x` (5)

```bash
# Create a simple script
echo '#!/bin/bash' > my_script.sh
echo 'echo "Hello from script!"' >> my_script.sh
ls -l my_script.sh
```

```output
-rw-rw-r-- 1 youruser youruser 34 Oct 27 14:30 my_script.sh
```

```bash
# Make the script executable for owner, readable/executable for group and others
chmod 755 my_script.sh
ls -l my_script.sh
```

```output
-rwxr-xr-x 1 youruser youruser 34 Oct 27 14:30 my_script.sh
```

```bash
# Now you can execute it
./my_script.sh
```

```output
Hello from script!
```

**Note:** If `my_script.sh` did not have execute permissions, `./my_script.sh` would result in a "Permission denied" error.

## Conclusion

Mastering these "basic concepts" will undoubtedly make you a more capable and confident developer. Understanding how environment variables, standard streams, processes, and file permissions work fundamentally changes how you interact with your operating system, debug issues, and build robust applications. Take the time to experiment with these commands in your terminal; the hands-on experience is invaluable.