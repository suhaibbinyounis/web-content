---
categories:
- AI
- DevOps
- Productivity
- Linux
comments: true
cover:
  image: https://images.pexels.com/photos/4816921/pexels-photo-4816921.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Learn to build a powerful, private, and customizable AI development copilot
  right in your terminal using local LLMs with Ollama, persistent sessions with Tmux,
  and smart scripting with Bash. Elevate your CLI workflow.
tags:
- Linux
- CLI
- Bash
- AI
- LLM
- Ollama
- Tmux
- Productivity
- Development
- DevOps
title: How to Build a Personal Dev Copilot with Ollama + Tmux + Bash
---

![Close-up of colorful programming code on a computer screen, showcasing digital technology.](https://images.pexels.com/photos/4816921/pexels-photo-4816921.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Close-up of colorful programming code on a computer screen, showcasing digital technology.")

## How to Build a Personal Dev Copilot with Ollama + Tmux + Bash

The world of AI copilots has revolutionized how we code and interact with complex systems. While commercial options abound, the idea of a personal, private, and highly customizable copilot, deeply integrated into your existing command-line workflow, is incredibly appealing. What if you could get AI assistance without sending your sensitive code to external servers?

This post will guide you through building exactly that: a local development copilot using a powerful trio – [Ollama](https://ollama.ai/) for running large language models (LLMs) locally, [Tmux](https://github.com/tmux/tmux/wiki) for managing your terminal sessions and context, and [Bash](https://www.gnu.org/software/bash/) for scripting and seamless integration. This isn't about fancy UIs; it's about raw, efficient, and private power at your fingertips.

Let's dive in.

## Why a Local Dev Copilot?

Before we start, why bother with a local setup when services like GitHub Copilot or ChatGPT are so readily available?

1.  **Privacy & Security:** Your code, your data, stays on your machine. No sending proprietary algorithms or sensitive project details to third-party APIs.
2.  **Offline Capability:** Work from anywhere, even without an internet connection.
3.  **Speed:** Local inference can be surprisingly fast, often faster than round-trips to cloud APIs, especially for smaller models.
4.  **Customization:** You pick the model, fine-tune its behavior, and integrate it exactly how you want into your existing toolkit.
5.  **Cost-Effective:** Once hardware is in place (a decent CPU or GPU), there are no ongoing per-token costs.

This setup leverages the power of open-source LLMs and the flexibility of the Linux command line.

## 1. Setting Up Ollama: Your Local LLM Engine

Ollama simplifies running large language models locally. It handles everything from downloading model weights to setting up the inference server.

### Installation

Ollama supports Linux, macOS, and even Windows (via WSL2). Installation is usually a single command.

For Linux (and WSL2):

```bash
curl -fsSL https://ollama.ai/install.sh | sh
```

For macOS, you can download the application from the [Ollama website](https://ollama.ai/download/mac).

Once installed, Ollama runs as a background service. You can verify its status:

```bash
systemctl status ollama
```

### Downloading a Model

Ollama has a model library that you can browse on their website. For development tasks, models like `llama2`, `mistral`, `codellama`, or `phi` are excellent starting points. `llama2` and `mistral` are general-purpose and quite capable. `codellama` is specifically fine-tuned for code.

Let's download `llama2`. This might take a while depending on your internet speed and the model size.

```bash
ollama pull llama2
```

```output
pulling manifest
pulling 0033c467640c... 100% |█████████████████████████████████████████| 3.8 GB
pulling 8a256193796d... 100% |█████████████████████████████████████████| 7.2 KB
pulling 97c67480a7bc... 100% |█████████████████████████████████████████| 4.8 KB
pulling 42dd83935266... 100% |█████████████████████████████████████████| 106 B
pulling 7a0b36ee263b... 100% |█████████████████████████████████████████| 136 B
pulling c62e2697b0a8... 100% |█████████████████████████████████████████| 136 B
pulling d461239f20e4... 100% |█████████████████████████████████████████| 136 B
pulling 2e40409a63dd... 100% |█████████████████████████████████████████| 4.8 KB
success
```

### Basic Interaction with Ollama

You can interact with a model directly from the command line using `ollama run`.

```bash
ollama run llama2 "Explain what a Linux inode is in simple terms."
```

```output
A Linux inode (index node) is a fundamental data structure in the Unix-like file system. Think of it as a small record or entry that stores all the essential metadata about a file or directory, except for its name and the actual data content itself.

Here's a breakdown of what an inode typically contains:

1.  **File Type:** Whether it's a regular file, directory, symbolic link, block device, character device, or named pipe.
2.  **Permissions:** Who can read, write, or execute the file (e.g., owner, group, others).
3.  **Owner and Group IDs:** The user ID (UID) of the file's owner and the group ID (GID) of its group.
4.  **Timestamps:**
    *   **Access Time (atime):** When the file was last accessed (read or written to).
    *   **Modification Time (mtime):** When the file's content was last modified.
    *   **Change Time (ctime):** When the file's metadata (like permissions or ownership) was last changed.
5.  **Link Count:** The number of hard links pointing to this inode. If a file has multiple names (hard links), they all point to the same inode.
6.  **Size:** The size of the file in bytes.
7.  **Block Pointers:** These are crucial. They point to the actual data blocks on the disk where the file's content is stored.
...
```

You can also have a persistent chat session:

```bash
ollama run llama2
>>> What is the capital of France?
Paris is the capital of France.
>>> What about Germany?
The capital of Germany is Berlin.
>>> /bye
```

**Note:** For development, `ollama run` is convenient as it starts the `ollama serve` process in the background if it's not already running. For more control or if you're building an application, you might explicitly run `ollama serve` in a separate session/daemon.

## 2. Leveraging Tmux for Context and Workflow

[Tmux](https://github.com/tmux/tmux/wiki) is a terminal multiplexer. It allows you to create and manage multiple terminal sessions, windows, and panes within a single terminal window. This is incredibly powerful for staying organized and keeping your copilot accessible.

### Why Tmux for a Dev Copilot?

*   **Persistent Sessions:** Your copilot conversations can live in a dedicated pane or window, even if you disconnect from your SSH session or close your terminal emulator.
*   **Dedicated Space:** You can allocate a specific pane for your copilot, allowing you to quickly switch to it without leaving your development environment.
*   **Multi-tasking:** Easily work on code in one pane while interacting with the AI in another.
*   **Context Transfer:** While not automatic, Tmux's copy mode can help you grab content from one pane and paste it into your copilot pane for analysis.

### Basic Tmux Commands (Quick Refresher)

*   `tmux new -s <session_name>`: Create a new session.
*   `tmux attach -t <session_name>`: Attach to an existing session.
*   `Ctrl+b c`: Create a new window.
*   `Ctrl+b %`: Split current pane vertically.
*   `Ctrl+b "`: Split current pane horizontally.
*   `Ctrl+b <arrow_key>`: Navigate between panes.
*   `Ctrl+b x`: Kill current pane.
*   `Ctrl+b d`: Detach from current session.

### Setting Up a Dedicated Copilot Pane

Let's set up a `dev-workflow` Tmux session with a dedicated pane for your copilot.

1.  Start a new Tmux session:

    ```bash
    tmux new -s dev-workflow
    ```

    You'll be in your primary development pane.

2.  Split the window horizontally to create a new pane below (or vertically with `Ctrl+b %`):

    ```bash
    # Press Ctrl+b then " (double quote)
    ```

    This creates a new pane, typically below your current one. You can press `Ctrl+b` and then an arrow key (`up`, `down`, `left`, `right`) to navigate between panes. For our copilot, let's keep it simple: one main work pane, one copilot pane.

    Your terminal now looks like this (conceptually):

    ```
    +-----------------------+
    |                       |
    |      Main Work        |
    |        Pane           |
    |                       |
    +-----------------------+
    |                       |
    |      Copilot          |
    |        Pane           |
    |                       |
    +-----------------------+
    ```

3.  Navigate to the copilot pane (e.g., `Ctrl+b down arrow`). Now you can interact with Ollama directly in this pane.

## 3. Bash Scripting the Copilot Interaction

This is where the magic happens. We'll create a simple Bash function that wraps `ollama run`, making it easier to invoke and allowing it to accept piped input (like code or logs) as context.

Add the following function to your `~/.bashrc` file (or a separate `~/.bash_functions` file and `source` it from `~/.bashrc`).

```bash
# Add this to your ~/.bashrc or a sourced file
copilot() {
    local model="llama2" # Default model. Change this to 'mistral', 'codellama', etc.
    local prompt=""
    local context_from_stdin=false

    # Check if we're receiving input via a pipe
    if ! [ -t 0 ]; then # -t 0 checks if stdin is a terminal (i.e., not piped)
        context_from_stdin=true
    fi

    # If prompt is provided as arguments, use it
    if [ "$#" -gt 0 ]; then
        prompt="$*"
    fi

    echo "Thinking..." # User feedback

    # Execute ollama. Ollama handles combining prompt args with stdin content.
    if "$context_from_stdin"; then
        # When stdin is present, ollama reads from it directly.
        # The prompt arguments are passed as the user's specific query.
        ollama run "$model" "$prompt"
    else
        # No stdin, just run with the prompt
        if [ -z "$prompt" ]; then
            echo "Usage: copilot \"Your query here\""
            echo "Or: cat file.py | copilot \"Explain this code\""
            return 1
        fi
        ollama run "$model" "$prompt"
    fi
}
```

After adding the function, remember to `source ~/.bashrc` or open a new terminal session for the changes to take effect.

```bash
source ~/.bashrc
```

### Examples of the `copilot` function in action:

#### Example 1: Simple Question

Ask your copilot a general development question.

```bash
copilot "What's the difference between 'git merge' and 'git rebase'?"
```

```output
Thinking...
Okay, let's break down the differences between `git merge` and `git rebase` in Git. Both commands are used to integrate changes from one branch into another, but they do so in fundamentally different ways, leading to different project histories.

**`git merge`**

*   **What it does:** `git merge` takes the changes from one branch and integrates them into your current branch by creating a *new merge commit*. This merge commit has two parent commits: the tip of your current branch and the tip of the branch you're merging in.
*   **History:** It preserves the exact history of both branches, including where they diverged and where they were merged. The history is non-linear, showing the explicit merge points.
*   **Pros:**
    *   **Non-destructive:** It doesn't rewrite history. All original commits remain as they are. This makes it generally safer for collaborative workflows.
    *   **Traceable:** You can easily see when and where integrations happened, which can be useful for auditing or understanding project flow.
...
```

#### Example 2: Explaining a Code Snippet

Pipe a small code snippet to your copilot and ask for an explanation.

First, create a dummy Python file:

```bash
cat <<EOF > my_script.py
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n-1)

if __name__ == "__main__":
    num = 5
    print(f"The factorial of {num} is {factorial(num)}")
EOF
```

Now, pipe its content to your copilot:

```bash
cat my_script.py | copilot "Explain this Python script."
```

```output
Thinking...
The provided Python script calculates the factorial of a given non-negative integer. Here's a breakdown:

1.  **`def factorial(n):`**: This line defines a function named `factorial` that takes one argument, `n`.

2.  **`if n == 0:`**: This is the base case for the recursion. The factorial of 0 is defined as 1. If `n` is 0, the function returns 1 immediately, stopping the recursive calls.

3.  **`else:`**: If `n` is not 0, the recursive step is executed.
    *   **`return n * factorial(n-1)`**: This line calculates the factorial. It multiplies `n` by the factorial of `n-1`. The `factorial(n-1)` part is a recursive call to the same function, but with a smaller argument. This process continues until `n` becomes 0 (the base case).

4.  **`if __name__ == "__main__":`**: This is a standard Python idiom. The code inside this block will only run when the script is executed directly (not when it's imported as a module into another script).

5.  **`num = 5`**: An integer variable `num` is initialized with the value 5.

6.  **`print(f"The factorial of {num} is {factorial(num)}")`**: This line calls the `factorial` function with `num` (which is 5) as the argument, and then prints the result in a formatted string.

**In essence:** The script demonstrates a recursive implementation of the factorial function, calculating 5! (5 * 4 * 3 * 2 * 1) which equals 120. When you run this script, it will output:
`The factorial of 5 is 120`
```

#### Example 3: Refactoring a Script

You can also ask for modifications or refactoring suggestions.

```bash
cat my_script.py | copilot "Refactor this Python script to use an iterative approach instead of recursion."
```

```output
Thinking...
Okay, here's the `factorial` function refactored to use an iterative approach (a loop) instead of recursion. This can be more memory-efficient for very large numbers as it avoids deep call stacks.

```python
def factorial_iterative(n):
    if n < 0:
        raise ValueError("Factorial is not defined for negative numbers.")
    if n == 0:
        return 1
    
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result

if __name__ == "__main__":
    num = 5
    print(f"The factorial of {num} (iterative) is {factorial_iterative(num)}")

    num_neg = -1
    try:
        print(f"The factorial of {num_neg} is {factorial_iterative(num_neg)}")
    except ValueError as e:
        print(e)
```

**Changes Made:**

1.  **Function Name:** Renamed `factorial` to `factorial_iterative` for clarity.
2.  **Negative Number Handling:** Added a check for `n < 0` to raise a `ValueError`, as factorials are typically defined for non-negative integers.
3.  **Iterative Loop:**
    *   Initialized `result = 1`.
    *   Used a `for` loop that iterates from 1 up to `n` (inclusive).
    *   In each iteration, `result` is multiplied by the current loop variable `i`.
4.  **`if __name__ == "__main__":` block:** Updated the `print` statement to call the new function and added a test case for a negative number to demonstrate the error handling.

This iterative version achieves the same result as the recursive one but uses a loop structure.
```

## 4. Integrating It All: The Workflow

Now, let's put it together into a typical development workflow using Tmux.

1.  **Start/Attach to your Tmux session:**

    ```bash
    tmux new -s dev-workflow || tmux attach -t dev-workflow
    ```
    This command will create a new session named `dev-workflow` if it doesn't exist, or attach you to it if it does.

2.  **Ensure you have a copilot pane:**
    If you haven't already, split your main window. For example, to split horizontally (top for code, bottom for copilot):

    ```bash
    # If in main work pane: Ctrl+b "
    ```
    Or vertically (left for code, right for copilot):

    ```bash
    # If in main work pane: Ctrl+b %
    ```
    Then navigate to your preferred copilot pane (`Ctrl+b <arrow_key>`).

3.  **Work in your primary pane:**
    Switch back to your main development pane (`Ctrl+b <arrow_key>`). Here, you'll be editing files, running tests, executing commands, etc.

    ```bash
    # In your main pane (e.g., top or left)
    vim my_app.py
    git status
    ./run_tests.sh
    ```

4.  **When you need AI assistance:**
    *   **Switch to the copilot pane:** Press `Ctrl+b <arrow_key>` to move your cursor to the copilot pane.
    *   **Use the `copilot` function:**
        *   **For general questions:** `copilot "How do I secure a REST API with OAuth2?"`
        *   **For code explanation/refactoring:** Copy the relevant code from your editor (using your editor's copy functionality or Tmux's copy mode, `Ctrl+b [` then select and `Ctrl+b ]`) and paste it into the copilot pane, then pipe it:
            ```bash
            # In copilot pane, after pasting or redirecting content from a file:
            # e.g., cat /path/to/my_app.py | copilot "Find potential security vulnerabilities in this Python Flask app."
            ```
            Or simply type out your question:
            ```bash
            # In copilot pane:
            copilot "I'm seeing a 'Permission denied' error when running my script. What are the common causes and how do I debug them?"
            ```
            The response will appear directly in that pane, maintaining your conversational history.

5.  **Switch back to your work:**
    After getting your answer, switch back to your primary development pane (`Ctrl+b <arrow_key>`) to apply the insights.

This workflow keeps your AI assistant ever-present but unobtrusive, allowing you to quickly context-switch and leverage its power without leaving the terminal.

## Advanced Tips & Future Enhancements

*   **Custom System Prompts:** For more specific behaviors (e.g., "Act as a senior DevOps engineer"), you can define custom system prompts for your Ollama models. This requires creating a `Modelfile`. Check Ollama's documentation for details.
*   **Different Models:** Experiment with `mistral`, `codellama`, `phi`, or other models from Ollama's library to find what works best for your specific tasks. Each model has its strengths.
*   **Tmux Keybindings:** For power users, you could bind a Tmux key combination to automatically copy the current selection and send it to your copilot pane with a predefined prompt. This is more advanced but highly automatable.
*   **Version Control Integration:** Consider adding a `copilot-diff` function that pipes `git diff` output to the AI to ask for code review suggestions.
*   **Logging Conversations:** You might want to log your `copilot` interactions to a file for review later. Bash redirection can achieve this: `copilot "My question" 2>&1 | tee ~/copilot_logs/$(date +%F_%H-%M-%S).log`.
*   **Pre-fill Prompts:** Create dedicated `copilot_explain_code`, `copilot_refactor`, `copilot_debug` functions that pre-fill the prompt for common tasks.

```bash
# Example of a specialized copilot function
copilot_refactor() {
    local model="llama2"
    local prompt="Refactor the following code for better readability, performance, and adherence to best practices. Provide the refactored code and an explanation of your changes:"
    
    if ! [ -t 0 ]; then
        ollama run "$model" "$prompt"
    else
        echo "Usage: cat code.py | copilot_refactor"
        return 1
    fi
}
```

## Conclusion

You've now built a powerful, private, and highly integrated development copilot directly into your command-line workflow. By combining Ollama's local LLM capabilities, Tmux's session management, and Bash's scripting power, you have an AI assistant that respects your privacy and works offline, all while staying within the familiar confines of your terminal.

This setup is highly customizable and can be expanded to suit your exact needs. Experiment with different models, refine your `copilot` functions, and explore further Tmux automations. The possibilities are vast when you control your tools. Happy coding!