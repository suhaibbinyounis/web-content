---
categories:
- Programming
- Artificial Intelligence
- Tools
comments: true
cover:
  image: https://images.pexels.com/photos/9242852/pexels-photo-9242852.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Learn to build powerful, native-feeling command-line tools that interact
  with Large Language Models, using Python and modern CLI frameworks.
tags:
- CLI
- Python
- LLM
- AI
- DevOps
- DevTools
- Typer
- Rich
- Productivity
title: How to Create a CLI Tool That Talks to LLMs (and Feels Native)
---

![Two young engineers working on robotic equipment in a workshop setting.](https://images.pexels.com/photos/9242852/pexels-photo-9242852.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Two young engineers working on robotic equipment in a workshop setting.")

## How to Create a CLI Tool That Talks to LLMs (and Feels Native)

# How to Create a CLI Tool That Talks to LLMs (and Feels Native)

We developers live in the terminal. It's our happy place for compiling code, managing servers, and orchestrating deployments. So, why are we constantly switching to a browser tab to chat with an LLM? Imagine having an AI assistant that lives right in your command line, feels like a native tool, and integrates seamlessly into your workflow.

That's exactly what we're going to build. This post will guide you through creating a CLI tool that talks to Large Language Models (LLMs), focusing on making it feel intuitive, fast, and, well, *native*.

We'll cover everything from the basic LLM API call to building a robust, user-friendly command-line interface with modern Python tools.

## Why Build a CLI for LLMs?

Beyond the "because we can" factor, there are tangible benefits:

*   **Workflow Integration**: Stay in your terminal. No context switching.
*   **Automation**: Easily script LLM interactions into your build processes, CI/CD pipelines, or daily tasks.
*   **Speed**: Quicker access to common prompts or specialized AI functions.
*   **Customization**: Tailor the tool exactly to your needs, defining default models, temperatures, or even specific "personas" for the AI.
*   **Data Security**: For sensitive internal use cases, a local CLI might fit better into security policies than web-based tools.

## Choosing Your Stack

For this project, we're going with a set of tools that are powerful, Pythonic, and pragmatic:

### Python & Typer

Python is the obvious choice for LLM interaction due to its rich ecosystem (think `openai`, `langchain`, `transformers`). For the CLI framework, we'll use [Typer](https://typer.tiangolo.com/).

Why Typer?
*   It's built on [Click](https://click.palletsprojects.com/), a widely adopted and robust CLI library.
*   It leverages Python type hints, making your code cleaner, more readable, and self-documenting.
*   It handles argument parsing, subcommand routing, and help messages with minimal boilerplate.
*   It has excellent support for autocompletion.

### LLM API (OpenAI)

We'll use the OpenAI API because it's widely accessible and straightforward to integrate. The concepts translate easily to other LLMs like Anthropic's Claude, Google's Gemini, or open-source models hosted locally or on services like Hugging Face.

### Enhancing User Experience (Rich)

To achieve that "native" feel, we need more than just raw text output. [Rich](https://rich.readthedocs.io/en/stable/) is an incredible Python library for beautiful terminal formatting. It provides:
*   Colors and styles.
*   Markdown rendering.
*   Syntax highlighting for code.
*   Progress bars and spinners.
*   Tables, columns, and more.

## Setting Up Your Project

Let's get the foundational pieces in place.

### Initialize Your Environment

First, create a project directory and a Python virtual environment. This keeps your dependencies isolated.

```bash
mkdir llm-cli-tool
cd llm-cli-tool
python3 -m venv .venv
source .venv/bin/activate
```

Now, install the necessary libraries: `typer`, `openai`, and `python-dotenv` (for managing API keys). `rich` will be useful too.

```bash
pip install typer openai python-dotenv "rich<13" # Pin rich to <13 for markdown rendering issue on newer versions with custom lexer
```
Note: I'm pinning `rich` to `<13` here because some older OpenAI models might return content with specific markdown issues that `rich` version 13+ handles differently, potentially causing a `ValueError` for invalid tokens in the Markdown renderer. For general safety and stability with LLM output, this is a pragmatic choice.

### Configure API Key

You'll need an OpenAI API key. Get one from your [OpenAI dashboard](https://platform.openai.com/account/api-keys). Never hardcode your API key directly in your script. We'll use environment variables for security.

Create a file named `.env` in your project directory:

```ini
OPENAI_API_KEY="sk-YOUR_ACTUAL_OPENAI_KEY_HERE"
```

Replace `"sk-YOUR_ACTUAL_OPENAI_KEY_HERE"` with your actual key. This file will be loaded by `python-dotenv`.

## The Core: Interacting with an LLM

Let's start with a basic Python script to interact with the OpenAI API. This will be the foundation of our CLI.

Create a file named `simple_llm_call.py`:

```python
import os
from dotenv import load_dotenv
import openai

# Load environment variables from .env file
load_dotenv()

# Get the API key from environment variables
openai_api_key = os.getenv("OPENAI_API_KEY")
if not openai_api_key:
    print("Error: OPENAI_API_KEY environment variable not set.")
    exit(1)

# Initialize the OpenAI client
# For older versions of openai library (<1.0.0), you might use:
# openai.api_key = openai_api_key
client = openai.OpenAI(api_key=openai_api_key)

def get_llm_response(prompt: str) -> str:
    try:
        completion = client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": prompt}
            ]
        )
        # Extracting the content from the response
        return completion.choices[0].message.content
    except Exception as e:
        return f"An error occurred: {e}"

if __name__ == "__main__":
    test_prompt = "Explain the concept of recursion in programming concisely."
    print(f"Asking LLM: '{test_prompt}'")
    response = get_llm_response(test_prompt)
    print("\n--- LLM Response ---")
    print(response)
```

Now, run this script:

```bash
python simple_llm_call.py
```

```output
Asking LLM: 'Explain the concept of recursion in programming concisely.'

--- LLM Response ---
Recursion in programming is a technique where a function calls itself directly or indirectly to solve a problem. It works by breaking down a large problem into smaller, identical sub-problems until a base case is reached, which can be solved without further recursion. Each recursive call adds a new frame to the call stack, and when a base case is hit, the results unwind back up the stack. It's often used for problems that can be defined in terms of simpler versions of themselves, like traversing trees or calculating factorials.
```

This confirms our API key and basic interaction are working.

## Building a Basic CLI with Typer

Now, let's transform our script into a CLI tool using `Typer`. We'll create a single command to "ask" the LLM a question.

Create a file named `llm_cli_v1.py`:

```python
import os
import typer
from dotenv import load_dotenv
import openai

# Load environment variables from .env file
load_dotenv()

# Get the API key from environment variables
openai_api_key = os.getenv("OPENAI_API_KEY")
if not openai_api_key:
    print("Error: OPENAI_API_KEY environment variable not set.")
    exit(1)

# Initialize the OpenAI client
client = openai.OpenAI(api_key=openai_api_key)

app = typer.Typer()

@app.command()
def ask(prompt: str):
    """
    Ask the LLM a question.
    """
    typer.echo(f"Asking LLM: '{prompt}'")
    try:
        completion = client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": prompt}
            ]
        )
        response_content = completion.choices[0].message.content
        typer.echo("\n--- LLM Response ---")
        typer.echo(response_content)
    except openai.APIError as e:
        typer.echo(f"OpenAI API Error: {e}", err=True)
        raise typer.Exit(code=1)
    except Exception as e:
        typer.echo(f"An unexpected error occurred: {e}", err=True)
        raise typer.Exit(code=1)

if __name__ == "__main__":
    app()
```

Run it directly from the command line:

```bash
python llm_cli_v1.py ask "What is the capital of France?"
```

```output
Asking LLM: 'What is the capital of France?'

--- LLM Response ---
The capital of France is Paris.
```

Typer automatically generates a `--help` message:

```bash
python llm_cli_v1.py --help
```

```output
Usage: llm_cli_v1.py [OPTIONS] COMMAND [ARGS]...

Options:
  --install-completion  Install completion for the current shell.
  --show-completion     Show completion for the current shell.
  --help                Show this message and exit.

Commands:
  ask  Ask the LLM a question.
```

```bash
python llm_cli_v1.py ask --help
```

```output
Usage: llm_cli_v1.py ask [OPTIONS] PROMPT

  Ask the LLM a question.

Arguments:
  PROMPT  [required]

Options:
  --help  Show this message and exit.
```

This is already pretty good! Typer handles argument parsing and help text automatically.

## Making it "Feel Native"

This is where `rich` comes in and where we refine the user experience. A native-feeling tool provides clear feedback, handles errors gracefully, and presents information beautifully.

### Beautiful Output with Rich

Let's integrate `rich` to make the output more appealing, especially when the LLM returns formatted text like code or Markdown.

#### Adding Colors and Markdown Rendering

```python
import os
import typer
from dotenv import load_dotenv
import openai
from rich.console import Console
from rich.markdown import Markdown
from rich.text import Text

# Initialize Rich Console for beautiful output
console = Console()

# Load environment variables from .env file
load_dotenv()

# Get the API key from environment variables
openai_api_key = os.getenv("OPENAI_API_KEY")
if not openai_api_key:
    console.print("[bold red]Error:[/bold red] OPENAI_API_KEY environment variable not set.", err=True)
    raise typer.Exit(code=1)

# Initialize the OpenAI client
client = openai.OpenAI(api_key=openai_api_key)

app = typer.Typer()

@app.command()
def ask(
    prompt: str = typer.Argument(..., help="The prompt to send to the LLM."),
    model: str = typer.Option("gpt-3.5-turbo", "--model", "-m", help="The LLM model to use."),
    temp: float = typer.Option(0.7, "--temperature", "-t", min=0.0, max=2.0, help="Sampling temperature to use, between 0.0 and 2.0.")
):
    """
    Ask the LLM a question and get a formatted response.
    """
    console.print(f"[bold blue]Asking LLM ({model}, temp={temp}):[/bold blue] [italic white]{prompt}[/italic white]")
    try:
        messages = [
            {"role": "system", "content": "You are a helpful assistant. Respond concisely and use markdown where appropriate."},
            {"role": "user", "content": prompt}
        ]
        completion = client.chat.completions.create(
            model=model,
            messages=messages,
            temperature=temp
        )
        response_content = completion.choices[0].message.content

        console.print("\n[bold green]--- LLM Response ---[/bold green]")
        # Render the LLM's response as Markdown
        console.print(Markdown(response_content))

    except openai.APIError as e:
        console.print(f"[bold red]OpenAI API Error:[/bold red] {e}", err=True)
        raise typer.Exit(code=1)
    except Exception as e:
        console.print(f"[bold red]An unexpected error occurred:[/bold red] {e}", err=True)
        raise typer.Exit(code=1)

if __name__ == "__main__":
    app()
```

Let's try asking for code:

```bash
python llm_cli_v2.py ask "Write a Python function to calculate factorial using recursion."
```

```output
Asking LLM (gpt-3.5-turbo, temp=0.7): Write a Python function to calculate factorial using recursion.

--- LLM Response ---
```python
def factorial(n):
    if n == 0:  # Base case
        return 1
    else:
        return n * factorial(n - 1)  # Recursive call

# Example usage:
print(factorial(5)) # Output: 120
```
```

Notice how `rich` automatically syntax-highlighted the Python code block returned by the LLM, and added colors to our messages. This significantly improves readability.

#### Progress Indicators

For longer-running tasks (like waiting for an LLM response), a spinner or progress bar keeps the user informed that something is happening.

```python
import os
import typer
from dotenv import load_dotenv
import openai
from rich.console import Console
from rich.markdown import Markdown
from rich.status import Status
import time # For simulation

console = Console()

load_dotenv()
openai_api_key = os.getenv("OPENAI_API_KEY")
if not openai_api_key:
    console.print("[bold red]Error:[/bold red] OPENAI_API_KEY environment variable not set.", err=True)
    raise typer.Exit(code=1)

client = openai.OpenAI(api_key=openai_api_key)

app = typer.Typer()

@app.command()
def ask(
    prompt: str = typer.Argument(..., help="The prompt to send to the LLM."),
    model: str = typer.Option("gpt-3.5-turbo", "--model", "-m", help="The LLM model to use."),
    temp: float = typer.Option(0.7, "--temperature", "-t", min=0.0, max=2.0, help="Sampling temperature to use, between 0.0 and 2.0.")
):
    """
    Ask the LLM a question and get a formatted response with a progress indicator.
    """
    console.print(f"[bold blue]Asking LLM ({model}, temp={temp}):[/bold blue] [italic white]{prompt}[/italic white]")

    with Status("[bold green]Waiting for LLM response...[/bold green]", spinner="dots") as status:
        try:
            messages = [
                {"role": "system", "content": "You are a helpful assistant. Respond concisely and use markdown where appropriate."},
                {"role": "user", "content": prompt}
            ]
            completion = client.chat.completions.create(
                model=model,
                messages=messages,
                temperature=temp
            )
            response_content = completion.choices[0].message.content
            status.stop() # Stop the spinner on success
            console.print("\n[bold green]--- LLM Response ---[/bold green]")
            console.print(Markdown(response_content))

        except openai.APIError as e:
            status.stop() # Stop spinner on error
            console.print(f"[bold red]OpenAI API Error:[/bold red] {e}", err=True)
            raise typer.Exit(code=1)
        except Exception as e:
            status.stop() # Stop spinner on error
            console.print(f"[bold red]An unexpected error occurred:[/bold red] {e}", err=True)
            raise typer.Exit(code=1)

if __name__ == "__main__":
    app()
```

When you run this, you'll see a small spinning animation while the API call is in progress:

```bash
python llm_cli_v3.py ask "Give me a one-liner joke about a programmer."
```

```output
Asking LLM (gpt-3.5-turbo, temp=0.7): Give me a one-liner joke about a programmer.
Waiting for LLM response...
[Spinner animation during API call]
--- LLM Response ---
Why do programmers prefer dark mode? Because light attracts bugs!
```

### Multi-line Input for Prompts

Sometimes your prompt is long and complex. Typing it all on one line as a command-line argument is cumbersome. `typer.prompt` can handle multi-line input.

```python
import os
import typer
from dotenv import load_dotenv
import openai
from rich.console import Console
from rich.markdown import Markdown
from rich.status import Status

console = Console()

load_dotenv()
openai_api_key = os.getenv("OPENAI_API_KEY")
if not openai_api_key:
    console.print("[bold red]Error:[/bold red] OPENAI_API_KEY environment variable not set.", err=True)
    raise typer.Exit(code=1)

client = openai.OpenAI(api_key=openai_api_key)

app = typer.Typer()

@app.command()
def ask(
    prompt: str = typer.Argument(None, help="The prompt to send to the LLM. If not provided, will prompt for multi-line input."),
    model: str = typer.Option("gpt-3.5-turbo", "--model", "-m", help="The LLM model to use."),
    temp: float = typer.Option(0.7, "--temperature", "-t", min=0.0, max=2.0, help="Sampling temperature to use, between 0.0 and 2.0.")
):
    """
    Ask the LLM a question. If no prompt is provided, an interactive
    multi-line prompt will appear.
    """
    if prompt is None:
        console.print("[bold cyan]Enter your prompt (Ctrl+D to finish):[/bold cyan]")
        prompt_lines = []
        try:
            while True:
                line = input()
                prompt_lines.append(line)
        except EOFError: # Ctrl+D signals EOF
            prompt = "\n".join(prompt_lines).strip()
            if not prompt:
                console.print("[bold yellow]No prompt entered. Exiting.[/bold yellow]")
                raise typer.Exit(code=0)
        except KeyboardInterrupt: # Ctrl+C
            console.print("\n[bold yellow]Prompt entry cancelled. Exiting.[/bold yellow]")
            raise typer.Exit(code=0)

    console.print(f"[bold blue]Asking LLM ({model}, temp={temp}):[/bold blue] [italic white]{prompt[:70]}{'...' if len(prompt) > 70 else ''}[/italic white]")

    with Status("[bold green]Waiting for LLM response...[/bold green]", spinner="dots") as status:
        try:
            messages = [
                {"role": "system", "content": "You are a helpful assistant. Respond concisely and use markdown where appropriate."},
                {"role": "user", "content": prompt}
            ]
            completion = client.chat.completions.create(
                model=model,
                messages=messages,
                temperature=temp
            )
            response_content = completion.choices[0].message.content
            status.stop()
            console.print("\n[bold green]--- LLM Response ---[/bold green]")
            console.print(Markdown(response_content))

        except openai.APIError as e:
            status.stop()
            console.print(f"[bold red]OpenAI API Error:[/bold red] {e}", err=True)
            raise typer.Exit(code=1)
        except Exception as e:
            status.stop()
            console.print(f"[bold red]An unexpected error occurred:[/bold red] {e}", err=True)
            raise typer.Exit(code=1)

if __name__ == "__main__":
    app()
```

Now, if you call the command without a prompt argument, it will enter interactive mode:

```bash
python llm_cli_v4.py ask
```

```output
Enter your prompt (Ctrl+D to finish):
What is the difference between a process and a thread?
Explain it to me like I'm 5 years old.
[Ctrl+D here]
Asking LLM (gpt-3.5-turbo, temp=0.7): What is the difference between a process and a thread?
Explain it to me like I'm 5 years...
Waiting for LLM response...
[Spinner animation during API call]
--- LLM Response ---
Imagine you have a big toy box (that's a computer program!).

A **process** is like a whole kid playing with their own toy box. They have their own toys, their own space, and they don't share with anyone else unless they want to. If one kid stops playing, it doesn't bother the other kids.

A **thread** is like just one of the kid's hands inside *their own* toy box. That kid can use both hands at the same time to play with different toys from *their own* box. But if the kid stops playing (the process ends), both hands stop too!
```

This significantly improves usability for complex prompts.

### Subcommands for Organization

As your CLI grows, you might want specialized commands (e.g., `llm summarize`, `llm debug-code`). Typer makes this easy with subcommands.

Let's create a `code` subcommand that specifically asks the LLM to generate code, allowing us to specify a language.

```python
import os
import typer
from dotenv import load_dotenv
import openai
from rich.console import Console
from rich.markdown import Markdown
from rich.status import Status
from typing import Optional

console = Console()

load_dotenv()
openai_api_key = os.getenv("OPENAI_API_KEY")
if not openai_api_key:
    console.print("[bold red]Error:[/bold red] OPENAI_API_KEY environment variable not set.", err=True)
    raise typer.Exit(code=1)

client = openai.OpenAI(api_key=openai_api_key)

app = typer.Typer()
code_app = typer.Typer() # Create a Typer instance for subcommands
app.add_typer(code_app, name="code", help="Commands related to code generation.")


def _get_llm_response(prompt: str, model: str, temp: float, system_message: str = "You are a helpful assistant. Respond concisely and use markdown where appropriate.") -> str:
    """Helper function to get LLM response."""
    with Status("[bold green]Waiting for LLM response...[/bold green]", spinner="dots") as status:
        try:
            messages = [
                {"role": "system", "content": system_message},
                {"role": "user", "content": prompt}
            ]
            completion = client.chat.completions.create(
                model=model,
                messages=messages,
                temperature=temp
            )
            response_content = completion.choices[0].message.content
            status.stop()
            return response_content

        except openai.APIError as e:
            status.stop()
            console.print(f"[bold red]OpenAI API Error:[/bold red] {e}", err=True)
            raise typer.Exit(code=1)
        except Exception as e:
            status.stop()
            console.print(f"[bold red]An unexpected error occurred:[/bold red] {e}", err=True)
            raise typer.Exit(code=1)

@app.command()
def ask(
    prompt: Optional[str] = typer.Argument(None, help="The prompt to send to the LLM. If not provided, will prompt for multi-line input."),
    model: str = typer.Option("gpt-3.5-turbo", "--model", "-m", help="The LLM model to use."),
    temp: float = typer.Option(0.7, "--temperature", "-t", min=0.0, max=2.0, help="Sampling temperature to use, between 0.0 and 2.0.")
):
    """
    Ask the LLM a general question. If no prompt is provided, an interactive
    multi-line prompt will appear.
    """
    if prompt is None:
        console.print("[bold cyan]Enter your prompt (Ctrl+D to finish):[/bold cyan]")
        prompt_lines = []
        try:
            while True:
                line = input()
                prompt_lines.append(line)
        except EOFError:
            prompt = "\n".join(prompt_lines).strip()
            if not prompt:
                console.print("[bold yellow]No prompt entered. Exiting.[/bold yellow]")
                raise typer.Exit(code=0)
        except KeyboardInterrupt:
            console.print("\n[bold yellow]Prompt entry cancelled. Exiting.[/bold yellow]")
            raise typer.Exit(code=0)

    console.print(f"[bold blue]Asking LLM ({model}, temp={temp}):[/bold blue] [italic white]{prompt[:70]}{'...' if len(prompt) > 70 else ''}[/italic white]")
    response_content = _get_llm_response(prompt, model, temp)
    console.print("\n[bold green]--- LLM Response ---[/bold green]")
    console.print(Markdown(response_content))


@code_app.command("gen") # Renaming to "gen" so it's `llm code gen`
def generate_code(
    lang: str = typer.Argument(..., help="The programming language for the code (e.g., python, javascript, bash)."),
    prompt: Optional[str] = typer.Argument(None, help="The code request. If not provided, will prompt for multi-line input."),
    model: str = typer.Option("gpt-3.5-turbo", "--model", "-m", help="The LLM model to use."),
    temp: float = typer.Option(0.7, "--temperature", "-t", min=0.0, max=2.0, help="Sampling temperature to use, between 0.0 and 2.0.")
):
    """
    Generate code in a specific programming language.
    """
    if prompt is None:
        console.print(f"[bold cyan]Enter your code request for {lang} (Ctrl+D to finish):[/bold cyan]")
        prompt_lines = []
        try:
            while True:
                line = input()
                prompt_lines.append(line)
        except EOFError:
            prompt = "\n".join(prompt_lines).strip()
            if not prompt:
                console.print("[bold yellow]No code request entered. Exiting.[/bold yellow]")
                raise typer.Exit(code=0)
        except KeyboardInterrupt:
            console.print("\n[bold yellow]Code generation cancelled. Exiting.[/bold yellow]")
            raise typer.Exit(code=0)

    full_prompt = f"Generate {lang} code for the following request: {prompt}"
    system_message = f"You are a helpful assistant specializing in {lang} code generation. Provide only the code block, with minimal explanation outside of the code comments."

    console.print(f"[bold blue]Generating {lang} code ({model}, temp={temp}):[/bold blue] [italic white]{prompt[:70]}{'...' if len(prompt) > 70 else ''}[/italic white]")
    response_content = _get_llm_response(full_prompt, model, temp, system_message)
    console.print("\n[bold green]--- LLM Generated Code ---[/bold green]")
    console.print(Markdown(response_content))

if __name__ == "__main__":
    app()
```

Let's test the new subcommand:

```bash
python llm_cli_v5.py code gen python "A function to reverse a string."
```

```output
Generating python code (gpt-3.5-turbo, temp=0.7): A function to reverse a string.
Waiting for LLM response...
--- LLM Generated Code ---
```python
def reverse_string(s):
  """
  Reverses a given string.
  """
  return s[::-1]

# Example usage:
my_string = "hello"
reversed_string = reverse_string(my_string)
print(f"Original: {my_string}, Reversed: {reversed_string}") # Output: Original: hello, Reversed: olleh
```
```

You can also explore the help messages:

```bash
python llm_cli_v5.py --help
```

```output
Usage: llm_cli_v5.py [OPTIONS] COMMAND [ARGS]...

Options:
  --install-completion  Install completion for the current shell.
  --show-completion     Show completion for the current shell.
  --help                Show this message and exit.

Commands:
  ask   Ask the LLM a general question. If no prompt is provided, an
        interactive multi-line prompt will appear.
  code  Commands related to code generation.
```

```bash
python llm_cli_v5.py code --help
```

```output
Usage: llm_cli_v5.py code [OPTIONS] COMMAND [ARGS]...

  Commands related to code generation.

Options:
  --help  Show this message and exit.

Commands:
  gen  Generate code in a specific programming language.
```

This structure is robust and extensible, making it easy to add more specialized LLM interactions.

## Packaging Your CLI Tool

To make your CLI tool truly "native," you'll want to install it system-wide (or in your virtual environment) so you can run it simply by typing its name, like `llm ask` instead of `python llm_cli_v5.py ask`.

We'll use `pyproject.toml` for modern Python packaging.

### Project Structure

First, organize your project like this:

```
llm-cli-tool/
├── .env
├── pyproject.toml
└── llm_cli/
    └── __init__.py
    └── main.py
```

Move the content of `llm_cli_v5.py` into `llm_cli/main.py`. The `__init__.py` can be an empty file.

### `pyproject.toml` for Packaging

Create `pyproject.toml` in the root `llm-cli-tool/` directory:

```toml
[project]
name = "llm-assistant"
version = "0.1.0"
description = "A native CLI tool for interacting with LLMs."
authors = [{ name = "Your Name", email = "your.email@example.com" }]
license = { text = "MIT" }
readme = "README.md"
requires-python = ">=3.8"
dependencies = [
    "typer[all]", # [all] includes shell completion capabilities
    "openai",
    "python-dotenv",
    "rich<13"
]

[project.scripts]
llm = "llm_cli.main:app" # This defines your CLI entry point

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

### Making it Executable

Now, from the root `llm-cli-tool/` directory (where `pyproject.toml` is), install your package in "editable" mode. This allows you to make changes to `llm_cli/main.py` and see them reflected immediately without reinstalling.

```bash
pip install -e .
```

```output
Obtaining file:///path/to/llm-cli-tool
  Installing build dependencies ... done
  Checking if build backend supports build_editable ... done
  Getting requirements to build editable ... done
  Preparing editable metadata (pyproject.toml) ... done
Collecting typer[all] (from llm-assistant)
  Using cached typer-0.9.0-py3-none-any.whl (45 kB)
Collecting openai (from llm-assistant)
  Using cached openai-1.3.7-py3-none-any.whl (222 kB)
... (more installation output)
Installing collected packages: llm-assistant
  Running setup.py develop for llm-assistant
Successfully installed llm-assistant-0.1.0 typer-0.9.0 openai-1.3.7 rich-12.1.0 python-dotenv-1.0.0
```

Now you can run your CLI tool just by typing `llm`:

```bash
llm --help
```

```output
Usage: llm [OPTIONS] COMMAND [ARGS]...

Options:
  --install-completion  Install completion for the current shell.
  --show-completion     Show completion for the current shell.
  --help                Show this message and exit.

Commands:
  ask   Ask the LLM a general question. If no prompt is provided, an
        interactive multi-line prompt will appear.
  code  Commands related to code generation.
```

```bash
llm ask "What is the best way to learn Git?"
```

```output
Asking LLM (gpt-3.5-turbo, temp=0.7): What is the best way to learn Git?
Waiting for LLM response...
--- LLM Response ---
The best way to learn Git is by **doing**. Start a small personal project, initialize a Git repository, and practice basic commands like `git add`, `git commit`, `git push`, and `git pull`. Explore branching and merging (`git branch`, `git checkout`, `git merge`) as you progress. Resources like the official Git documentation, interactive tutorials (e.g., Git Immersion, Learn Git Branching), and online courses can supplement your hands-on experience. Focus on understanding the core concepts (staging area, commits, branches) rather than just memorizing commands.
```

Congratulations! You now have a fully functional, native-feeling CLI tool that interacts with LLMs.

## Conclusion

You've just built a powerful command-line interface that seamlessly integrates LLMs into your daily workflow. By leveraging Python with `Typer` for robust CLI scaffolding, `python-dotenv` for secure configuration, and `Rich` for a delightful user experience, you've gone from a simple script to a production-ready developer tool.

The principles covered here—clear argument parsing, rich output, progress indicators, multi-line input, and organized subcommands—are applicable to any CLI project, not just those involving AI.

Now, go forth and customize! Add more specialized commands, integrate different LLM APIs, manage conversation history, or even add intelligent file processing. The terminal is your oyster.