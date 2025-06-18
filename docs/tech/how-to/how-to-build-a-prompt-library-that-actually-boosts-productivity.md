---
title: How to Build a Prompt Library That Actually Boosts Productivity
date: 2025-06-17T13:05:16.383Z
description: "Learn to create a simple, effective prompt library using plain text files, CLI tools, and a touch of scripting to supercharge your AI interactions and development workflow."
tags: ["Prompt Engineering", "Productivity", "CLI", "Bash", "Python", "DevOps", "AI"]
categories: ["Software Development", "Tools", "Productivity"]
comments: true
---

Let's be honest: interacting with large language models (LLMs) and AI tools has become an integral part of many developers' workflows. From generating boilerplate code and debugging obscure errors to crafting commit messages and summarizing documentation, AI is a powerful assistant. But here's the rub: if you're like me, you're constantly re-typing, searching through chat histories, or trying to remember that *perfect* prompt that gave you exactly what you needed last week.

This is where a **prompt library** comes in. It's not about complex software; it's about a pragmatic system for storing, organizing, and quickly retrieving your most effective AI prompts. Think of it as your personal knowledge base for AI conversations. Done right, it saves you time, ensures consistency, and significantly boosts your productivity.

This post will guide you through building a simple yet incredibly powerful prompt library using tools you already know: your file system, the command line, and a touch of scripting. We'll keep it lean, mean, and effective.

## The Core Idea: Simple Files, Plain Text

The best solutions are often the simplest. A prompt library doesn't need a database, a fancy UI, or a SaaS subscription. It just needs a dedicated place to store your prompts, each in its own file. Plain text files are universal, easy to version control, and instantly accessible.

Let's start by creating a dedicated directory for our prompts:

```bash
mkdir -p ~/prompts
cd ~/prompts
```

Now, let's create a couple of sample prompts. Imagine you frequently ask an AI to debug Python code or explain complex concepts.

**`~/prompts/debug-python.txt`**:

```text
You are an expert Python developer and debugger. I will provide you with a Python code snippet and a traceback. Your task is to:
1. Identify the root cause of the error.
2. Explain the error in clear, concise terms.
3. Provide a corrected version of the code, highlighting the changes.
4. Suggest best practices or potential improvements related to the issue.

Code:
```python
# [YOUR CODE HERE]
```

Traceback:
```
# [YOUR TRACEBACK HERE]
```
```

**`~/prompts/explain-concept.txt`**:

```text
Explain the following concept as if you are teaching a junior developer. Keep the explanation concise, use analogies if helpful, and provide a small, runnable code example if applicable.

Concept: [YOUR CONCEPT HERE]
```

See how we've included placeholders? This is key for templating and reusability, which we'll address later.

## Accessing Your Prompts: CLI Power

The goal isn't just to store prompts, but to access them *instantly*. The most straightforward way to get a prompt from a file into your AI tool (e.g., a web interface, a CLI AI client) is to copy its content directly to your clipboard.

We'll use `cat` to read the file and pipe its content to a clipboard utility.

*   **Linux**: `xclip` (install via `sudo apt-get install xclip` or `sudo pacman -S xclip`)
*   **macOS**: `pbcopy` (built-in)
*   **Windows (WSL/Git Bash)**: `clip.exe` (built-in)

Let's grab our Python debugging prompt:

```bash
# On Linux
cat ~/prompts/debug-python.txt | xclip -selection clipboard

# On macOS
cat ~/prompts/debug-python.txt | pbcopy

# On Windows (WSL/Git Bash)
cat ~/prompts/debug-python.txt | clip.exe
```

After running one of these commands, the entire content of `debug-python.txt` will be in your clipboard, ready to be pasted into your AI chat window.

## Organizing for Sanity: Directories and Naming Conventions

As your library grows, a flat structure will become unmanageable. Organize your prompts into subdirectories based on category, domain, or even the type of AI task.

Consider a structure like this:

```
~/prompts/
├── coding/
│   ├── python/
│   │   ├── debug-error.txt
│   │   ├── refactor-function.txt
│   │   └── explain-decorator.txt
│   ├── javascript/
│   │   └── create-react-component.txt
│   └── general/
│       └── generate-boilerplate.txt
├── writing/
│   ├── blog-post-outline.txt
│   ├── commit-message.txt
│   └── summarise-document.txt
├── devops/
│   └── dockerfile-generator.txt
└── general/
    └── brainstorm-ideas.txt
```

To create this, you'd use `mkdir -p`:

```bash
mkdir -p ~/prompts/coding/python \
         ~/prompts/coding/javascript \
         ~/prompts/writing \
         ~/prompts/devops \
         ~/prompts/general
```

**Naming Conventions:** Be consistent.
*   `[domain]-[task].txt` (e.g., `python-debug.txt`)
*   `[action]-[subject].txt` (e.g., `summarize-article.txt`)
*   Keep names descriptive but concise.

## Boosting Access with Shell Scripts and Functions

Typing `cat ~/prompts/coding/python/debug-error.txt | xclip -selection clipboard` repeatedly gets tedious. Let's create a simple wrapper script to make this much faster.

Create a file named `prompt-get` in a directory that's in your `PATH` (e.g., `~/bin` or `/usr/local/bin`).

**`~/bin/prompt-get`**:

```bash
#!/bin/bash

PROMPT_DIR="${HOME}/prompts"
CLIPBOARD_CMD=""

# Determine clipboard command based on OS
if command -v xclip &> /dev/null; then
    CLIPBOARD_CMD="xclip -selection clipboard"
elif command -v pbcopy &> /dev/null; then
    CLIPBOARD_CMD="pbcopy"
elif command -v clip.exe &> /dev/null; then
    CLIPBOARD_CMD="clip.exe"
else
    echo "Error: No clipboard utility found (xclip, pbcopy, or clip.exe)." >&2
    exit 1
fi

if [[ "$#" -eq 0 || "$1" == "--help" || "$1" == "-h" ]]; then
    echo "Usage: prompt-get <prompt_name> [--fzf]"
    echo "  Copies the content of <prompt_name>.txt from your prompt library to the clipboard."
    echo "  <prompt_name> can be a partial path, e.g., 'coding/python/debug-error' or 'commit-message'."
    echo "  Use --fzf to interactively select a prompt."
    exit 0
fi

# Check for fzf flag
USE_FZF=0
if [[ "$1" == "--fzf" ]]; then
    USE_FZF=1
    shift # Remove --fzf from arguments
fi

# FZF integration
if [[ "$USE_FZF" -eq 1 ]]; then
    # Ensure fzf is installed
    if ! command -v fzf &> /dev/null; then
        echo "Error: fzf is not installed. Please install it to use this feature." >&2
        exit 1
    fi

    # Find all prompt files and pipe them to fzf for selection
    # `sed` removes the PROMPT_DIR prefix and the .txt suffix for cleaner display in fzf
    PROMPT_PATH=$(find "${PROMPT_DIR}" -type f -name "*.txt" | \
                  sed "s|${PROMPT_DIR}/||; s|\.txt$||" | \
                  fzf --prompt="Select a prompt: " --no-sort --border --margin=1 --padding=1 \
                      --header="Use Ctrl+R to toggle fuzzy match, ESC to exit")
    
    if [ -z "$PROMPT_PATH" ]; then
        echo "No prompt selected."
        exit 0
    fi
else
    # If not using fzf, construct path from argument
    PROMPT_PATH="$1"
fi

FULL_PROMPT_PATH="${PROMPT_DIR}/${PROMPT_PATH}.txt"

if [ ! -f "$FULL_PROMPT_PATH" ]; then
    echo "Error: Prompt file not found: $FULL_PROMPT_PATH" >&2
    echo "Try 'prompt-get --fzf' to browse available prompts." >&2
    exit 1
fi

cat "$FULL_PROMPT_PATH" | ${CLIPBOARD_CMD}
echo "Copied '${PROMPT_PATH}.txt' to clipboard."
```

Make the script executable:

```bash
chmod +x ~/bin/prompt-get
```

**Note**: Ensure `~/bin` is in your `PATH` environment variable. If not, add `export PATH="$HOME/bin:$PATH"` to your `~/.bashrc`, `~/.zshrc`, or `~/.profile` file and then `source` it.

### Usage Examples:

```bash
# Get the debug-python prompt (assuming it's in ~/prompts directly or you specify full path)
prompt-get debug-python

# Output
# Copied 'debug-python.txt' to clipboard.

# Get a nested prompt
prompt-get coding/python/debug-error

# Output
# Copied 'coding/python/debug-error.txt' to clipboard.
```

### Interactive Selection with `fzf`

The `prompt-get` script includes an `--fzf` flag for interactive selection. This is incredibly powerful when you don't remember the exact name or path of a prompt.

First, install `fzf`:

*   **Linux**: `sudo apt install fzf` or `sudo pacman -S fzf`
*   **macOS**: `brew install fzf`

Then, run:

```bash
prompt-get --fzf
```

```output
Select a prompt:
> coding/python/debug-error
  coding/python/refactor-function
  coding/javascript/create-react-component
  writing/commit-message
  devops/dockerfile-generator
  general/brainstorm-ideas
(Press Tab to multi-select, Enter to select, Ctrl+R to toggle fuzzy match)
```

(The `fzf` output will be an interactive TUI, showing a list of your prompts. You can type to filter and press Enter to select.)

## Versioning Your Knowledge: Git is Your Friend

Your prompt library is a valuable asset. Treat it like code. Initialize a Git repository in your `~/prompts` directory. This gives you:

*   **History**: See how your prompts evolved.
*   **Rollbacks**: Revert to previous versions if you make a mistake.
*   **Sharing**: Easily share your library with teammates or sync across devices.

```bash
cd ~/prompts
git init
git add .
git commit -m "Initial commit of my prompt library"
```

Push it to a private (or public, if you're feeling generous!) GitHub/GitLab/Bitbucket repository.

```bash
git remote add origin git@github.com:your_username/your_prompt_library.git
git push -u origin master # Or main
```

Remember to commit changes regularly as you refine existing prompts or add new ones!

```bash
# After modifying or adding prompts
git add .
git commit -m "Refined debug-python prompt with better placeholders"
git push
```

## Advanced: A Pythonic Prompt Manager (Optional but Powerful)

While the shell script is efficient, Python offers more robust features like proper templating, handling structured data (YAML/JSON for metadata), and more complex logic.

Let's create a more advanced `prompt-mgr.py` that supports simple templating using f-strings (or a library like Jinja2 if you need more power).

First, let's create a prompt file that leverages variables.
**`~/prompts/coding/python/generate-test.yaml`**:

```yaml
prompt: |
  You are an expert Python testing engineer. I will provide a Python function.
  Your task is to generate a comprehensive pytest test suite for this function.
  The test suite should cover:
  1.  Basic functionality.
  2.  Edge cases.
  3.  Error handling (if applicable).
  4.  Input validation (if applicable).
  
  Ensure the tests are well-structured, readable, and follow pytest best practices.
  
  Function to test:
  ```python
  {function_code}
  ```
variables:
  function_code: "The Python function code to be tested."
description: "Generates a pytest suite for a given Python function."
```

Now, the Python script.
**`~/bin/prompt-mgr.py`**:

```python
#!/usr/bin/env python3

import argparse
import os
import sys
import yaml # pip install pyyaml

def get_clipboard_command():
    if sys.platform == "darwin":
        return "pbcopy"
    elif sys.platform == "linux":
        # Check if xclip is installed and available
        if os.system("command -v xclip > /dev/null") == 0:
            return "xclip -selection clipboard"
        else:
            return None
    elif sys.platform == "win32": # For Windows (e.g., Git Bash, WSL)
        return "clip.exe"
    return None

def main():
    parser = argparse.ArgumentParser(
        description="A CLI tool to manage and retrieve AI prompts from your library.",
        formatter_class=argparse.RawTextHelpFormatter
    )
    parser.add_argument(
        "prompt_name",
        nargs="?",
        help="The name/path of the prompt (e.g., coding/python/generate-test).\n"
             "Use '--fzf' to select interactively."
    )
    parser.add_argument(
        "--fzf",
        action="store_true",
        help="Use fzf for interactive prompt selection."
    )
    parser.add_argument(
        "--list",
        action="store_true",
        help="List all available prompts."
    )
    # Add an argument for each variable that might be templated
    parser.add_argument(
        "-v", "--var",
        action="append",
        nargs=2,
        metavar=("KEY", "VALUE"),
        help="Provide values for templated variables (e.g., -v function_code 'def foo(): pass')"
    )
    args = parser.parse_args()

    PROMPT_DIR = os.path.join(os.path.expanduser("~"), "prompts")

    if not os.path.exists(PROMPT_DIR):
        print(f"Error: Prompt directory '{PROMPT_DIR}' not found.", file=sys.stderr)
        sys.exit(1)

    clipboard_cmd = get_clipboard_command()
    if not clipboard_cmd:
        print("Error: No suitable clipboard utility found (pbcopy, xclip, or clip.exe).", file=sys.stderr)
        sys.exit(1)

    # Collect all available prompts
    all_prompts = {}
    for root, _, files in os.walk(PROMPT_DIR):
        for file in files:
            if file.endswith((".txt", ".yaml")):
                full_path = os.path.join(root, file)
                relative_path = os.path.relpath(full_path, PROMPT_DIR)
                # Remove extension for display/lookup
                prompt_key = os.path.splitext(relative_path)[0]
                all_prompts[prompt_key] = full_path

    if args.list:
        if not all_prompts:
            print("No prompts found in your library.")
        else:
            print("Available prompts:")
            for key in sorted(all_prompts.keys()):
                print(f"  - {key}")
        sys.exit(0)

    selected_prompt_key = None
    if args.fzf:
        if not os.system("command -v fzf > /dev/null") == 0:
            print("Error: fzf is not installed. Please install it to use this feature.", file=sys.stderr)
            sys.exit(1)
        
        prompt_list = sorted(all_prompts.keys())
        if not prompt_list:
            print("No prompts found to select.", file=sys.stderr)
            sys.exit(1)

        fzf_input = "\n".join(prompt_list)
        fzf_command = f'echo "{fzf_input}" | fzf --prompt="Select a prompt: " --no-sort --border --margin=1 --padding=1 --header="Use Ctrl+R to toggle fuzzy match, ESC to exit"'
        
        try:
            # Use os.popen to run fzf and capture its output
            with os.popen(fzf_command) as fzf_proc:
                selected_prompt_key = fzf_proc.read().strip()
        except Exception as e:
            print(f"Error running fzf: {e}", file=sys.stderr)
            sys.exit(1)
        
        if not selected_prompt_key:
            print("No prompt selected.", file=sys.stderr)
            sys.exit(0)
    elif args.prompt_name:
        selected_prompt_key = args.prompt_name
    else:
        parser.print_help()
        sys.exit(1)

    if selected_prompt_key not in all_prompts:
        print(f"Error: Prompt '{selected_prompt_key}' not found in your library.", file=sys.stderr)
        sys.exit(1)

    file_path = all_prompts[selected_prompt_key]
    
    prompt_content = ""
    if file_path.endswith(".yaml"):
        with open(file_path, 'r') as f:
            data = yaml.safe_load(f)
            prompt_content = data.get('prompt', '')
            variables_info = data.get('variables', {})
    else: # .txt file
        with open(file_path, 'r') as f:
            prompt_content = f.read()
            variables_info = {} # No explicit variables for plain text

    # Handle templating
    template_vars = {}
    if args.var:
        for key, value in args.var:
            template_vars[key] = value

    # Warn about missing variables for YAML prompts
    if file_path.endswith(".yaml"):
        missing_vars = [k for k in variables_info if k not in template_vars]
        if missing_vars:
            print(f"Warning: Missing values for variables: {', '.join(missing_vars)}", file=sys.stderr)
            print("Please provide them using -v KEY VALUE.", file=sys.stderr)
            # You might choose to exit here, or proceed with empty/default values
            # For this example, we'll proceed, allowing f-string to use literal '{var_name}'
            # if not provided. Or use a safer templating lib.

    try:
        # Use f-string formatting. Be careful with potential KeyError if variable not provided.
        # A more robust solution might use string.Template or Jinja2.
        final_prompt = prompt_content.format(**template_vars)
    except KeyError as e:
        print(f"Error: Prompt expects variable '{e}' which was not provided.", file=sys.stderr)
        print("Please provide it using -v KEY VALUE.", file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f"Error during prompt templating: {e}", file=sys.stderr)
        sys.exit(1)

    # Copy to clipboard
    try:
        os.system(f'echo "{final_prompt.replace("\"", "\\\"")}" | {clipboard_cmd}')
        print(f"Copied '{selected_prompt_key}' to clipboard. Content length: {len(final_prompt)} characters.")
    except Exception as e:
        print(f"Failed to copy to clipboard: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

Make it executable:

```bash
chmod +x ~/bin/prompt-mgr.py
```

Install `pyyaml`:

```bash
pip install pyyaml
```

### Usage Examples with `prompt-mgr.py`:

```bash
# List all available prompts
prompt-mgr.py --list

# Output
# Available prompts:
#   - coding/python/debug-error
#   - coding/python/generate-test
#   - writing/commit-message
#   - ...
```

```bash
# Get a plain text prompt (same as shell script)
prompt-mgr.py writing/commit-message

# Output
# Copied 'writing/commit-message' to clipboard. Content length: 180 characters.
```

```bash
# Get the templated prompt, providing the variable
prompt-mgr.py coding/python/generate-test -v function_code "def add(a, b):\n    return a + b"

# Output
# Copied 'coding/python/generate-test' to clipboard. Content length: 450 characters.
```

The content copied to the clipboard would look like this:

```text
You are an expert Python testing engineer. I will provide a Python function.
Your task is to generate a comprehensive pytest test suite for this function.
The test suite should cover:
1.  Basic functionality.
2.  Edge cases.
3.  Error handling (if applicable).
4.  Input validation (if applicable).

Ensure the tests are well-structured, readable, and follow pytest best practices.

Function to test:
```python
def add(a, b):
    return a + b
```
```

You can also use the `--fzf` flag with `prompt-mgr.py` for interactive selection, just like with `prompt-get`.

## Designing Effective Prompts for Your Library

The power of your library lies not just in its organization, but in the quality of the prompts it contains. Here are principles for crafting effective prompts:

*   **Specify Persona**: "You are an expert Python developer..."
*   **Define Task Clearly**: "Your task is to generate a pytest test suite..."
*   **Provide Context**: Explain the situation, the goal, and any constraints.
*   **Specify Format**: "Respond in Markdown...", "Provide output as a JSON array..."
*   **Use Examples (Few-shot)**: Sometimes, showing the AI a couple of examples of desired input/output pairs works wonders.
*   **Break Down Complex Tasks**: For multi-step tasks, guide the AI through each step.
*   **Iterate and Refine**: Prompts are rarely perfect on the first try. Experiment, observe results, and refine. Update your library with the improved versions!
*   **Use Placeholders**: Clearly mark where dynamic content should go (e.g., `[YOUR CODE HERE]`, `{variable_name}`).

## Maintenance and Evolution

A prompt library isn't a "set it and forget it" system.
*   **Regular Review**: Periodically go through your prompts. Are they still relevant? Can they be improved?
*   **Add New Prompts**: Whenever you craft a particularly effective prompt during your daily work, add it to your library.
*   **Delete Obsolete Prompts**: Don't let cruft build up. If a prompt is no longer useful, remove it.
*   **Share (Thoughtfully)**: If you work in a team, consider a shared prompt library for common tasks. This ensures consistency and leverages collective intelligence. Just be mindful of sensitive information.

## Conclusion

Building a prompt library is a low-effort, high-reward investment in your daily productivity. By simply organizing plain text files, leveraging basic CLI tools like `cat` and `xclip`/`pbcopy`, and adding a sprinkle of scripting with Bash or Python, you can transform your AI interactions from a chore into a seamless, efficient process. Start small, iterate, and watch your productivity soar. Happy prompting!
