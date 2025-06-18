---
title: How to Use fzf + ripgrep to Replace Your File Manager
date: 2025-06-17T13:05:16.383Z
description: Learn how to leverage the power of fzf and ripgrep to navigate, find, and manage files on the command line, effectively replacing your traditional GUI file manager for many tasks.
tags: [Linux, CLI, Bash, Zsh, fzf, ripgrep, productivity, devtools]
categories: [DevOps, Productivity, Tools]
comments: true
---

For many developers, the command line is home. It's fast, powerful, and scriptable. But even in this text-based world, we often find ourselves needing to browse files, jump between directories, or open specific documents. This is where traditional GUI file managers shine – but they often feel clunky and slow when you're already in the terminal.

What if you could have the speed and power of the command line, combined with the interactive browsing and searching capabilities of a file manager? Enter `fzf` and `ripgrep`.

This post will show you how to combine these two incredibly powerful tools to navigate your filesystem, find files and content, and perform common file operations – all without leaving your terminal.

## Why fzf and ripgrep?

Before we dive into the "how," let's quickly understand the "why."

*   **`fzf` (Fuzzy Finder)**: This is an interactive Unix filter for command-line that can be used with any list-generating command. It's blazingly fast and provides a slick fuzzy-matching interface, allowing you to quickly narrow down choices from large lists. Think of it as an interactive `grep` on steroids for *selecting* items.
*   **`ripgrep` (`rg`)**: A line-oriented search tool that recursively searches the current directory for a regex pattern. It's written in Rust, which makes it incredibly fast, often significantly faster than `grep` or `ack`. `rg` is superb at *finding* content or paths.

The magic happens when you pipe the output of `ripgrep` (or any other listing command) into `fzf`. `ripgrep` efficiently finds what you're looking for, and `fzf` lets you interactively pick from the results.

## Installation

Let's get these tools on your system.

### fzf Installation

*   **macOS (Homebrew)**:
    ```bash
    brew install fzf
    $(brew --prefix)/opt/fzf/install
    ```
    The second command sets up shell extensions (key bindings like `Ctrl+T` for files, `Alt+C` for directories, and fuzzy completion).
*   **Linux (Debian/Ubuntu)**:
    ```bash
    sudo apt install fzf
    ```
    You might need to manually run `~/.fzf/install` if you install from source or a tarball, but apt often handles key bindings.
*   **Linux (Arch Linux)**:
    ```bash
    sudo pacman -S fzf
    ```
*   **Windows (WSL/Git Bash)**: Use the Linux instructions for WSL or the `scoop` package manager for Git Bash.
    ```powershell
    # With Scoop in PowerShell
    scoop install fzf
    ```

### ripgrep Installation

*   **macOS (Homebrew)**:
    ```bash
    brew install ripgrep
    ```
*   **Linux (Debian/Ubuntu)**:
    ```bash
    sudo apt install ripgrep
    ```
*   **Linux (Arch Linux)**:
    ```bash
    sudo pacman -S ripgrep
    ```
*   **Windows (WSL/Git Bash)**:
    ```powershell
    # With Scoop in PowerShell
    scoop install ripgrep
    ```
    For WSL, use the Linux instructions.

Once installed, verify they work:
```bash
fzf --version
rg --version
```
```output
0.43.0 (ea33a59)
ripgrep 14.1.0 (rev 92a7e7195d)
```

## Core Concepts: fzf & ripgrep Basics

Before combining them, let's see what each can do on its own.

### Basic `fzf` Usage

`fzf` takes any list on `stdin` and provides an interactive selector.

**Example 1: Selecting from `ls` output**
```bash
ls -F | fzf
```
```output
# Interactive fzf prompt appears, allowing you to type and filter results:
> file
  another_file.txt
  docs/
>
# ... type 'another' ...
> another
> another_file.txt
# (Press Enter to select 'another_file.txt')
```
When you select an item and press `Enter`, it's printed to `stdout`.

**Example 2: Selecting a file to `cd` into its directory (less common, but demonstrates piping)**
```bash
cd $(find . -type f | fzf | xargs dirname)
```
This command finds all files, lets you pick one, extracts its directory, and then `cd`s into it. This is useful for getting to a file's parent directory quickly.

### Basic `ripgrep` Usage

`ripgrep` is all about searching.

**Example 1: Searching for a pattern**
```bash
rg "function"
```
```output
src/main.rs
20:fn main() {
35:fn my_function() {

README.md
10:This project defines a `function` to do X.
```
This lists matching lines with file paths and line numbers.

**Example 2: Listing *files* that contain a pattern**
The `-l` (lowercase L) flag is crucial for our file manager use case. It lists only the file paths that contain a match.

```bash
rg -l "License"
```
```output
LICENSE.md
src/utils.js
```

**Example 3: Listing all files tracked by `ripgrep`**
`rg` respects `.gitignore` by default. `--files` lists all files `rg` *would* search.
```bash
rg --files
```
```output
.gitignore
LICENSE.md
README.md
src/main.rs
src/utils.js
tests/test_foo.py
```

## Combining fzf and ripgrep: The Power Duo

Now for the main event: piping `ripgrep`'s output to `fzf`.

### Use Case 1: Interactively Selecting Any File

This is a fundamental building block. We'll use `rg --files` to get a list of all non-ignored files, then pipe that to `fzf`.

```bash
rg --files | fzf
```
```output
# fzf prompt appears, showing all files in the current directory and subdirectories (excluding gitignored files).
> project.toml
  README.md
  src/
  src/main.rs
  tests/
  tests/test_utils.py
# (Type 'main', select 'src/main.rs', press Enter)
src/main.rs
```
The selected file path (`src/main.rs` in this case) is printed to `stdout`. You can now pipe this to another command!

### Use Case 2: Opening a Selected File in Your Editor

Let's extend the previous example to open the selected file in `nvim` (or `code`, `vim`, `subl`, etc.).

```bash
rg --files | fzf | xargs nvim
```
```output
# fzf prompt appears, showing all files.
> license
  .gitignore
  LICENSE.md
  README.md
# (Type 'license', select 'LICENSE.md', press Enter)
# nvim opens LICENSE.md
```
`xargs` is a utility that builds and executes command lines from standard input. Here, it takes the selected file path from `fzf` and passes it as an argument to `nvim`.

### Use Case 3: Opening a File Containing Specific Content

This is where `ripgrep`'s searching power truly shines. You want to open a file, but you only remember a keyword or a phrase that's *inside* it.

```bash
rg -l "important_function" | fzf | xargs nvim
```
```output
# fzf prompt appears, showing only files that contain "important_function".
> main
  src/main.rs
  src/logic.js
# (Type 'main', select 'src/main.rs', press Enter)
# nvim opens src/main.rs
```
This is incredibly powerful for quickly jumping to relevant files without knowing their exact names or paths.

### Use Case 4: Navigating to a Directory Based on a File

Sometimes you know a file, and you want to `cd` to its *parent directory*.

```bash
cd "$(rg --files | fzf | xargs dirname)"
```
```output
# fzf prompt appears, showing all files.
> readme
  README.md
  docs/user_guide/README.md
# (Select 'docs/user_guide/README.md', press Enter)
# Your current directory changes to 'docs/user_guide/'
```
Here, `dirname` extracts the directory path from the selected file path. We wrap the `cd` command in `$(...)` to execute the result.

## Replacing Your File Manager: Practical Aliases & Functions

To make these commands truly useful, let's create some shell functions or aliases for them. Add these to your `~/.bashrc`, `~/.zshrc`, or equivalent shell configuration file.

### Fuzzy Change Directory (`fcd`)

Instead of just `cd`ing to a file's directory, let's create a general fuzzy `cd` function.

**Option A: Basic `fcd` (using `fzf`'s built-in `ALT-C` functionality)**
If you've installed `fzf` with its shell extensions, you already have `ALT-C` which lets you fuzzy-select directories from your recent history or subdirectories. This is often sufficient for basic navigation.

**Option B: `fcd` using `rg --files` (finds a file, then changes to its directory)**
This function lets you `cd` to the directory of *any* file you can find via `ripgrep`.

```bash
# Define this in your .bashrc/.zshrc
fcd() {
    local dir
    dir=$(rg --files | fzf --preview 'ls -F {}' | xargs dirname)
    if [[ -n "$dir" ]]; then
        cd "$dir" || return
    fi
}
```
Now, type `fcd` and press Enter. You'll get an `fzf` prompt. Select a file, and you'll `cd` into its parent directory. The `--preview` option (requires `fzf` 0.25+) shows the contents of the selected file's directory, which helps with context.

```bash
fcd
```
```output
# fzf prompt appears, showing files for selection.
# As you move down the list, 'ls -F' output for the selected file's directory shows in the preview window.
> src/utils/helper.py
  src/
  src/utils/
  src/utils/helper.py    # Selected file
# (Preview Window):
#  __init__.py
#  helper.py
#  constants.py
# (Press Enter to select `src/utils/helper.py`)
# Your prompt changes to reflect you are now in `src/utils/`
```

### Fuzzy Open File (`fo`)

This function allows you to quickly open any file found by `ripgrep` in your default editor.

```bash
# Define this in your .bashrc/.zshrc
fo() {
    local file
    file=$(rg --files | fzf --preview 'bat --color=always --style=numbers --line-range=:50 {} || head -n 50 {}')
    if [[ -n "$file" ]]; then
        # Replace nvim with your preferred editor (e.g., code, vim, subl)
        nvim "$file"
    fi
}
```
`bat` is a `cat` clone with syntax highlighting and Git integration, highly recommended for the preview. If `bat` isn't found, it falls back to `head`.

```bash
fo
```
```output
# fzf prompt appears, showing all files.
# As you move down, 'bat' previews the file content in the right pane.
> README.md
  .gitignore
  LICENSE.md
  README.md             # Selected file
# (Preview Window):
#  1  # My Awesome Project
#  2
#  3  This project does X, Y, and Z.
#  4  ...
# (Press Enter to select `README.md`)
# nvim opens README.md
```

### Fuzzy Grep and Edit (`fge` or `feg`)

This is arguably the most powerful combination. You search for a *string*, and then select the *specific line* you want to jump to in your editor. This effectively replaces `grep -r "pattern"` followed by manually opening the file and searching.

```bash
# Define this in your .bashrc/.zshrc
fge() {
    local result
    # -n: show line numbers, -i: case-insensitive, --no-heading: cleaner output for fzf
    result=$(rg -n -i --no-heading "$@" | fzf --preview 'bat --color=always {1} --highlight-line {2}')

    if [[ -n "$result" ]]; then
        # Extract file path and line number from the fzf selection
        # Format: path:line:column:match
        # We need path:line
        local file_path=$(echo "$result" | awk -F: '{print $1}')
        local line_num=$(echo "$result" | awk -F: '{print $2}')
        # Open in nvim, jumping to the specific line
        # Replace nvim with your editor, adjusting its line jump syntax if needed
        nvim "+$line_num" "$file_path"
    fi
}
```
**Usage**: `fge <your-search-pattern>`
Example: `fge "error_handler"`

```bash
fge "user_service"
```
```output
# fzf prompt appears, showing all matching lines for "user_service".
# The preview window shows the file content with the selected line highlighted.
> src/api/handlers.py:25:    user_service = UserService()
  src/api/handlers.py:25: user_service = UserService() # Selected line
# (Preview Window):
#  23
#  24 def create_user_handler():
#  25 >    user_service = UserService()
#  26     # ...
#  27
# (Press Enter to select `src/api/handlers.py:25: user_service = UserService()`)
# nvim opens src/api/handlers.py and jumps to line 25.
```

### Fuzzy Delete (`fdl`) - Use with Extreme Caution!

You can also use this combo to delete files. **Always use `-m` for multi-select in `fzf` and `rm -i` for interactive confirmation.**

```bash
# Define this in your .bashrc/.zshrc
fdl() {
    local files_to_delete
    files_to_delete=$(rg --files | fzf -m --prompt="Select files to DELETE: ")
    if [[ -n "$files_to_delete" ]]; then
        echo "$files_to_delete" | xargs -r rm -i
    else
        echo "No files selected for deletion."
    fi
}
```
The `-r` flag to `xargs` ensures that `rm -i` is not run if `files_to_delete` is empty.

```bash
fdl
```
```output
# fzf prompt appears, showing all files.
# Use Tab to multi-select.
> old_logs.txt
  .gitignore
  old_logs.txt
  temp_data.csv
  README.md
# (Select 'old_logs.txt' and 'temp_data.csv' using Tab, then Enter)
rm: remove 'old_logs.txt'? y
rm: remove 'temp_data.csv'? y
```

### Other Useful Combinations

*   **View Git history of a file**:
    ```bash
    git log --pretty=oneline --abbrev-commit --name-only | fzf --ansi --preview 'git show {1}'
    ```
*   **Find and `grep` a specific commit**:
    ```bash
    git log --oneline --graph --all | fzf --ansi --no-sort --reverse --height=50% --preview "git show {1}" | awk '{print $1}' | xargs git show
    ```

## Advanced Tips & Customizations

### `FZF_DEFAULT_COMMAND`

`fzf`'s default behavior is to use `find . -path '*/\.*' -prune -o -type f -print -o -type d -print 2> /dev/null` or similar to list files. You can override this to use `rg --files` or `fd`. `fd` is a great alternative to `rg --files` if you strictly just want file paths, as it's optimized for that.

```bash
# In your .bashrc/.zshrc
export FZF_DEFAULT_COMMAND='rg --files --hidden --no-ignore'
# Or, if you have fd installed (recommended for finding files)
# export FZF_DEFAULT_COMMAND='fd --type f --hidden --exclude .git'
```
Now, simply typing `fzf` in your terminal will open an `fzf` prompt pre-populated with files from `rg` or `fd`. This also affects `Ctrl+T` if you have fzf's key bindings installed.

### `FZF_DEFAULT_OPTS`

You can customize `fzf`'s appearance and behavior globally using this environment variable.

```bash
# In your .bashrc/.zshrc
export FZF_DEFAULT_OPTS="--height 40% --layout=reverse --border --margin=1 --padding=1 --color=bg+:#313244,bg:#1e1e2e,fg:#cdd6f4,prompt:#89b4fa,pointer:#f5e0dc,marker:#f5e0dc,hl:#f38ba8,fg+:#cdd6f4,gutter:#1e1e2e,hl+:#f38ba8 --no-info --ansi"
```
This example sets height, layout, adds a border, and customizes colors (using Catppuccin Macchiato palette). `--no-info` removes the `(n/m)` info line. `--ansi` is important if your input stream contains ANSI escape codes (e.g., from `git log --color`).

### `fd` instead of `rg --files`

For purely listing files and directories, `fd` is often faster than `rg --files` as it's specifically designed for that purpose, whereas `rg`'s `--files` is a side effect of its search capabilities.

If you have `fd` installed:
```bash
# Faster file listing for fzf
export FZF_DEFAULT_COMMAND='fd --type f --hidden --exclude .git'

# For the fo function:
fo() {
    local file
    file=$(fd --type f | fzf --preview 'bat --color=always --style=numbers --line-range=:50 {} || head -n 50 {}')
    if [[ -n "$file" ]]; then
        nvim "$file"
    fi
}
```

## Why it's not a *full* file manager replacement (and why that's okay)

While `fzf` + `ripgrep` is incredibly powerful, it's essential to understand its limitations compared to a traditional GUI file manager like Nautilus, Finder, or Explorer:

*   **No Visual Tree View**: You don't get a graphical representation of your directory structure. While `tree | fzf` can provide a text-based tree, it's not the same.
*   **No Drag-and-Drop**: Moving or copying files requires explicit `mv` or `cp` commands.
*   **No Thumbnail Previews**: You can preview text content, but not images or videos directly.
*   **No Easy Permissions/Metadata Editing**: Changing file permissions or viewing detailed metadata usually requires separate `chmod`, `chown`, or `stat` commands.
*   **Steep Learning Curve (initially)**: It takes time to internalize the commands and build your custom functions.

However, for a command-line-centric workflow, these limitations are often minor. The sheer speed, flexibility, and scriptability of `fzf` and `ripgrep` far outweigh the missing GUI features for many developers. It's a workflow enhancer, not a drop-in graphical replacement.

## Conclusion

By combining the lightning-fast search capabilities of `ripgrep` with the interactive filtering of `fzf`, you gain an incredibly powerful way to navigate your filesystem, find files by name or content, and open them in your preferred editor – all without ever leaving the terminal.

This setup fosters a more efficient and keyboard-driven workflow, allowing you to focus on coding rather than clicking. Experiment with the examples provided, customize the functions to fit your needs, and you might just find yourself saying goodbye to your traditional file manager for good. Happy hacking!
