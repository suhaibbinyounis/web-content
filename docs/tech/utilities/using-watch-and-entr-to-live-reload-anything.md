---
categories:
- Productivity
- Development Tools
- Command Line
- Automation
comments: true
cover:
  image: https://images.pexels.com/photos/1972464/pexels-photo-1972464.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 11:22:34.549000
description: A deep dive into using the command-line tools `watch` and `entr` for
  efficient live-reloading of various files and processes, from documentation to code
  compilation and server restarts. Learn how to automate your development workflow.
tags:
- command-line
- productivity
- development
- live-reload
- watch
- entr
- shell
- automation
- linux
- macOS
- tools
title: Using `watch` and `entr` to Live-Reload Anything
---

![Close-up of colorful programming code displayed on a computer screen.](https://images.pexels.com/photos/1972464/pexels-photo-1972464.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Close-up of colorful programming code displayed on a computer screen.")

## Using `watch` and `entr` to Live-Reload Anything

As developers, writers, and anyone who interacts with text files, we often find ourselves caught in a tedious loop: edit, save, run, check. Whether it's compiling code, previewing a Markdown document, or restarting a development server, this manual iteration can quickly drain productivity and flow.

What if your tools could react instantly to your changes, automatically rebuilding, re-running, or reloading whatever you're working on? Enter `watch` and `entr` – two unassuming yet incredibly powerful command-line utilities that can transform your workflow and bring true live-reloading capabilities to nearly any task.

While often confused, `watch` and `entr` serve distinct purposes. Understanding their individual strengths and, more importantly, how they complement each other, unlocks a new level of command-line mastery.

## The Simplicity of `watch`: Continuous Monitoring

Let's start with `watch`. This utility is designed for continuous monitoring. It executes a specified command repeatedly, displaying its output on your terminal, allowing you to observe changes over time. Think of it as a persistent window into the state of your system or the output of a dynamic process.

### How `watch` Works

At its core, `watch` runs a command, waits a short interval (defaulting to 2 seconds), clears the screen, and runs the command again. It's a simple polling mechanism.

**Basic Usage:**

```bash
watch <command>
```

### Practical `watch` Examples

1.  **Monitoring Log Files:** Keep an eye on system activity or application logs in real-time.

    ```bash
    watch tail -n 20 /var/log/syslog
    ```

    This command will refresh the last 20 lines of your syslog every 2 seconds, showing new entries as they appear.

2.  **Checking Disk Space:** Periodically update your disk usage statistics.

    ```bash
    watch df -h
    ```

3.  **Observing Process Status:** Monitor a specific process or resource usage.

    ```bash
    watch 'ps aux | head -1; ps aux | grep my_application'
    ```

    This command runs two `ps aux` commands, first showing the header, then filtering for "my_application", giving you a live view of its status. The single quotes around the command are crucial to ensure the pipeline (`|`) is interpreted by the shell and not passed as arguments to `watch`.

### Limitations of `watch`

While `watch` is excellent for monitoring, it has a significant limitation for live-reloading: it's *polling-based*, not *event-driven*. It runs its command regardless of whether anything has changed. This makes it inefficient for tasks where you only want something to happen *when* a file is modified, like recompiling code. For that, we need `entr`.

## The Power of `entr`: Event-Driven Reloading

`entr` (event trigger) is the true hero for live-reloading. Unlike `watch`, `entr` doesn't poll; it waits for changes to specific files and then executes a command only when those changes occur. It leverages kernel-level mechanisms like `inotify` (Linux) or `FSEvents` (macOS) for highly efficient file monitoring.

### How `entr` Works

`entr` reads file names from its standard input (`stdin`). It then monitors these files for changes. When a change is detected in any of the specified files, `entr` executes a command provided as an argument.

### Installation

`entr` is widely available.

*   **macOS (Homebrew):**
    ```bash
    brew install entr
    ```
*   **Debian/Ubuntu:**
    ```bash
    sudo apt-get install entr
    ```
*   **Arch Linux:**
    ```bash
    sudo pacman -S entr
    ```
*   **Other systems:** Check your distribution's package manager or [build from source](https://github.com/eradman/entr#readme).

### Basic `entr` Usage Pattern

The typical pattern involves piping a list of files to `entr` using `find` or `ls`:

```bash
ls <files_to_watch> | entr <command_to_execute_on_change>
```

Or, more commonly and robustly:

```bash
find <directory> -name "<pattern>" | entr <command_to_execute_on_change>
```

### Key `entr` Options

`entr` comes with several powerful options that make it incredibly versatile:

*   `-r`: **Reload the program.** Instead of just restarting the command, this option causes `entr` to exit and re-execute itself, effectively reloading the entire pipeline. This is crucial for long-running processes or scripts.
*   `-c`: **Clear the screen** before each command execution. Keeps your terminal tidy.
*   `-d`: **Watch directories** for new files. If you add a new file to a watched directory, `entr` will automatically start watching it (and re-execute the command if necessary).
*   `-p`: **Pause `entr`** while the command is running. Prevents multiple executions if files change rapidly during a long build process.
*   `-s`: **Execute the command in a subshell.** This allows you to use shell features like pipes (`|`), redirects (`>`), and logical operators (`&&`, `||`) within your `entr` command. Without `-s`, `entr` would try to run `command1 | command2` as a single executable.

### Real-World `entr` Examples (Live-Reloading Anything!)

Let's dive into practical applications across different development scenarios.

#### 1. Live-Reloading a Markdown Previewer

If you're writing documentation or a blog post in Markdown, you want to see the rendered output instantly.

```bash
ls *.md | entr -r sh -c 'pandoc -s README.md -o preview.html && open preview.html'
```

*   `ls *.md`: Lists all Markdown files in the current directory.
*   `entr -r`: Watches these files, and if any change, it re-runs the subsequent command, and reloads `entr` itself.
*   `sh -c '...'`: Executes the enclosed command string in a subshell.
*   `pandoc -s README.md -o preview.html`: Converts `README.md` to `preview.html`. (Note: you might need to adjust `README.md` to match the specific file that changed, or watch only one file. For simplicity here, we assume one primary file or `pandoc` generating from a specific target.)
*   `&& open preview.html`: Opens the generated HTML in your default browser. On Linux, you might use `xdg-open preview.html`.

**Note:** For this specific example to auto-update correctly in the browser, `open preview.html` will open a *new* tab/window each time. A more sophisticated solution would involve a live-reloading dev server like `browser-sync` or a dedicated Markdown previewer with a watch mode. However, `entr` can still *trigger* these tools.

#### 2. Auto-Compiling LaTeX Documents

Writing scientific papers or books in LaTeX can be slow with manual compilation. Automate it!

```bash
find . -name "*.tex" | entr -r pdflatex main.tex
```

This command watches all `.tex` files in your current directory (and subdirectories) and, upon any change, recompiles your `main.tex` file, updating the PDF.

#### 3. JavaScript/TypeScript Development (Node.js Server)

Automatically restart your Node.js server whenever your application code changes.

```bash
find src -name "*.js" -o -name "*.ts" | entr -r node dist/app.js
```

This watches all `.js` and `.ts` files in your `src` directory. When a change occurs, `entr` kills the running `node` process (because of `-r`) and restarts it. If you have a build step (e.g., TypeScript compilation), you'd include that:

```bash
find src -name "*.ts" | entr -r sh -c 'npm run build && node dist/app.js'
```

*   `npm run build` compiles your TypeScript.
*   `&& node dist/app.js` runs the compiled JavaScript.

#### 4. Go Development

For Go applications, you might want to recompile and run, or re-run tests.

**Recompile and Run:**

```bash
ls *.go | entr -r go run main.go
```

**Run Tests on Change:**

```bash
find . -name "*.go" | entr -r go test ./...
```

To run tests, then run the application if tests pass:

```bash
find . -name "*.go" | entr -r sh -c 'go test ./... && go run main.go'
```

#### 5. Rust Development

Automate your `cargo build` or `cargo test` cycles.

```bash
find src -name "*.rs" | entr -r cargo build
```

Or for tests:

```bash
find src -name "*.rs" | entr -r cargo test
```

#### 6. Python Development

For local scripts or web frameworks (like Flask/Django without their own reloaders).

```bash
ls *.py | entr -r python my_script.py
```

For running tests with `pytest`:

```bash
find . -name "*.py" | entr -r pytest
```

#### 7. General Web Development (CSS/HTML/JS)

For static sites, `entr` can trigger a simple local web server reload.

```bash
find . -name "*.html" -o -name "*.css" -o -name "*.js" | entr -r python3 -m http.server 8000
```

This command will restart a simple Python web server whenever HTML, CSS, or JS files change. You'd then manually refresh your browser, or use a browser extension that connects to a live-reloading server triggered by `entr`.

#### 8. Complex Multi-Stage Operations

Combine `entr` with shell scripting for more intricate workflows.

```bash
find . -name "*.go" | entr -r sh -c 'goimports -w . && gofmt -w . && go vet ./... && go build -o myapp && ./myapp'
```

This single `entr` command will:
1.  Watch all Go files.
2.  On change, format them with `goimports` and `gofmt`.
3.  Run `go vet` for static analysis.
4.  Build the application.
5.  If all previous steps succeed, run the compiled `myapp` executable.

## `watch` vs. `entr`: When to Use Which

The distinction is clear and crucial:

*   **`watch`:** Use for **monitoring** and **displaying continuous output**. It's polling-based and will run its command repeatedly, regardless of whether anything has changed. Ideal for observing dynamic metrics, logs, or system states.
    *   *Example:* `watch uptime`, `watch netstat -an`.

*   **`entr`:** Use for **event-driven execution** and **live-reloading**. It only runs its command when specified files change. It's highly efficient and perfect for triggering builds, tests, server restarts, or any action that should react to file modifications.
    *   *Example:* `find . -name "*.js" | entr npm run build`, `ls *.md | entr pandoc -s input.md -o output.html`.

In most live-reloading scenarios, `entr` is the tool you want due to its efficiency and event-driven nature. `watch` is your go-to for persistent, real-time feedback.

## Advanced Tips & Considerations

1.  **Ignoring Files/Directories:** Be mindful of what `find` pipes to `entr`. You often want to exclude directories like `node_modules`, `dist`, `.git`, or build artifacts.

    ```bash
    find . -type f \( -name "*.js" -o -name "*.ts" \) -not -path "./node_modules/*" -not -path "./dist/*" | entr -r npm run build
    ```

    This finds `.js` or `.ts` files but excludes anything within `node_modules` or `dist` directories.

2.  **Multiple `entr` Instances:** For very complex projects with independent build steps (e.g., separate frontend and backend), you can run multiple `entr` commands in parallel in different terminal windows or using a terminal multiplexer like `tmux` or `screen`.

3.  **Performance:** `entr` is highly optimized. It uses native file system event APIs (like `inotify` on Linux and `FSEvents` on macOS), which are far more efficient than polling the file system manually.

4.  **Debugging `entr` Commands:** If your `entr` command isn't behaving as expected, first run the command directly in your shell without `entr` to ensure it works. Then, ensure `entr` is receiving the correct file list by replacing `entr` with `xargs echo` or `cat -`.

    ```bash
    find . -name "*.go" | cat -  # See what files entr would receive
    ```

5.  **Alternatives (and why `entr` still shines):**
    *   **`nodemon` (Node.js):** Excellent for Node.js projects, but specialized.
    *   **`watchexec` (Cross-language):** A good Rust-based alternative, often with more configuration options and `.gitignore` awareness.
    *   **Build tools with watch modes (Webpack, Vite, Gulp, Grunt, Parcel):** These are fantastic for their specific ecosystems and offer deep integration, but `entr` is universal.
    *   **IDE/Editor integrations:** Many IDEs have built-in watch features, but they might be less flexible for custom scripts.

    The beauty of `entr` is its **generality and simplicity**. It's a fundamental Unix tool that can be chained with anything, requiring minimal configuration and working across virtually any language or project setup.

## Conclusion

The `watch` and `entr` utilities are prime examples of the Unix philosophy at its best: simple tools that do one thing well, and can be powerfully combined. `watch` provides a quick window into changing states, while `entr` empowers you with robust, event-driven live-reloading for nearly any command-line task.

By integrating `entr` into your daily workflow, you can drastically reduce the mental overhead of repetitive tasks, allowing you to focus on creation rather than coordination. Experiment with these tools, adapt them to your specific needs, and watch your productivity soar.

---

### References & Further Reading

*   **`entr` GitHub Repository:** The official source for `entr`, including installation instructions and detailed usage examples.
    *   [https://github.com/eradman/entr](https://github.com/eradman/entr)
*   **`man entr`:** The manual page for `entr` provides comprehensive documentation on its options and behavior. Run `man entr` in your terminal.
*   **`man watch`:** The manual page for `watch`. Run `man watch` in your terminal.
*   **Watch (Unix) Wikipedia:** A brief overview of the `watch` command.
    *   [https://en.wikipedia.org/wiki/Watch_(Unix)](https://en.wikipedia.org/wiki/Watch_(Unix))
*   **Inotify (Linux kernel feature):** The underlying technology `entr` often uses on Linux for efficient file monitoring.
    *   [https://en.wikipedia.org/wiki/Inotify](https://en.wikipedia.org/wiki/Inotify)
*   **FSEvents (macOS):** The macOS equivalent to inotify, also utilized by `entr`.
    *   [https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/FSEvents_ProgGuide/Introduction/Introduction.html](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/FSEvents_ProgGuide/Introduction/Introduction.html)
