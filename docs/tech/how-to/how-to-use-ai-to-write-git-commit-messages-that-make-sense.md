---
categories:
- Development
- Tools
- AI/ML
comments: true
cover:
  image: https://images.pexels.com/photos/16587313/pexels-photo-16587313.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Learn to leverage AI, both local and cloud, to generate consistent, concise,
  and meaningful Git commit messages. Includes practical examples with Ollama and
  shell scripting.
tags:
- Git
- AI
- LLM
- DevOps
- CLI
- Bash
title: How to Use AI to Write Git Commit Messages That Make Sense
---

![A smartphone displaying the Wikipedia page for ChatGPT, illustrating its technology interface.](https://images.pexels.com/photos/16587313/pexels-photo-16587313.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "A smartphone displaying the Wikipedia page for ChatGPT, illustrating its technology interface.")

## How to Use AI to Write Git Commit Messages That Make Sense

Every developer knows the pain of writing a good Git commit message. What changed? Why? Is it concise enough? Detailed enough? Consistent with the team's conventions? Often, we rush, or we simply forget. The result: a commit history that's more a cryptic puzzle than a helpful timeline.

Enter AI. Large Language Models (LLMs) are exceptionally good at summarizing, extracting key information, and generating text in specific formats. This makes them a surprisingly powerful tool for automating one of the most tedious parts of our workflow: crafting sensible Git commit messages.

This post will guide you through integrating AI into your Git workflow, from choosing your AI backend to writing effective prompts and automating the process with simple scripts.

## Why Good Commit Messages Matter (And Why AI Can Help)

Before we dive into the "how," let's quickly recap the "why":

*   **Context for Future You (or Others):** When you're debugging an issue six months from now, a well-crafted commit message is invaluable. It explains *why* a change was made, not just *what* changed.
*   **Code Review Efficiency:** Reviewers can quickly grasp the intent of a commit without digging through every line of code.
*   **Easier `git blame` and `git log`:** Quickly understanding the history of a file or a feature.
*   **Better Release Notes & Changelogs:** Automated generation of release notes becomes much simpler when commits follow a consistent structure (e.g., Conventional Commits).

The challenge is consistency and effort. Manually ensuring every commit message adheres to best practices is tough. This is where AI shines: it can take a `git diff` as input and, with the right prompt, produce a structured, meaningful message consistently, saving you time and mental overhead.

## Choosing Your AI Tool: Local vs. Cloud

You have two primary approaches for integrating AI:

1.  **Local Models:** Run an LLM directly on your machine.
    *   **Pros:**
        *   **Privacy:** Your code never leaves your machine. Essential for sensitive projects.
        *   **Cost-Free:** Once set up, there are no ongoing API costs.
        *   **Offline Capability:** Works without an internet connection.
    *   **Cons:**
        *   **Setup Complexity:** Requires installing software (e.g., Ollama, LM Studio) and downloading models.
        *   **Performance:** Can be slower than cloud APIs and requires decent hardware (RAM, GPU) for larger models.
        *   **Model Quality:** Smaller, locally runnable models might not always match the nuance of large cloud models.
    *   **Example Tools:** Ollama, LM Studio, LiteLLM (for local API endpoints).
2.  **Cloud Models:** Use an LLM hosted by a third-party provider via an API.
    *   **Pros:**
        *   **Ease of Use:** Often just an API key and an `http` request.
        *   **Performance & Quality:** Access to the most powerful and up-to-date models (GPT-4o, Claude 3, Gemini, Llama 3 via API).
        *   **No Local Hardware Requirements:** Computations happen in the cloud.
    *   **Cons:**
        *   **Privacy Concerns:** Your code (the `diff`) is sent to a third-party server. **Do NOT use this for highly sensitive or proprietary code without explicit approval and understanding of the provider's data policies.**
        *   **Cost:** Pay-per-token usage can add up, especially with large diffs or frequent use.
        *   **Internet Dependency:** Requires an active internet connection.
    *   **Example Tools:** OpenAI API, Anthropic API, Google Cloud Vertex AI, Azure OpenAI Service.

For this guide, we'll focus on **Ollama** for local AI integration, as it's straightforward, privacy-friendly, and perfect for illustrating the concept without incurring costs or sending your code to the cloud.

## Setting Up Your Environment: Ollama (Local LLM)

First, let's get Ollama running.

### 1. Install Ollama

Ollama provides a simple installation script for Linux/macOS and a dedicated installer for Windows.

**Linux/macOS:**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Windows:**
Download the installer from [ollama.com/download](https://ollama.com/download).

### 2. Download a Model

Once installed, download a suitable model. For Git commits, you want a model that's good at summarization and following instructions, but it doesn't need to be massive. `phi3` or `llama3` are excellent choices that run well on most modern machines.

```bash
ollama pull phi3
```

```output
pulling manifest
pulling 00d4d2932373... 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████
```

Now, your environment is ready to generate messages.

## Integrating AI into Git: The Journey

Let's explore several ways to use AI for commit messages, moving from basic manual use to more integrated automation.

### 1. Manual Prompting (Basic)

This is the simplest way. You generate the `git diff` for the staged changes and paste it directly into your AI tool (or pipe it to a CLI tool like `ollama`).

First, let's create some dummy changes in a Git repository:

```bash
mkdir my-ai-project
cd my-ai-project
git init

echo "# My Project" > README.md
git add README.md
git commit -m "feat: initial project setup with README"

# Make some changes
echo "This is a simple project to demonstrate AI commit messages." >> README.md
mkdir src
echo "def hello():\n    print('Hello, AI commits!')" > src/main.py
echo "new_feature_flag = True" >> src/main.py

# Stage the changes
git add .
```

Now, get the diff of your staged changes:

```bash
git diff --staged
```

```output
diff --git a/README.md b/README.md
index 0e33ef7..843d1a3 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,2 @@
 # My Project
+This is a simple project to demonstrate AI commit messages.
diff --git a/src/main.py b/src/main.py
new file mode 100644
index 0000000..f95e5b3
--- /dev/null
+++ b/src/main.py
@@ -0,0 +1,3 @@
+def hello():
+    print('Hello, AI commits!')
+new_feature_flag = True
```

Now, pipe this diff directly to `ollama` with a good prompt. We'll aim for a [Conventional Commit](https://www.conventionalcommits.org/en/v1.0.0/) format.

```bash
git diff --staged | ollama run phi3 --template "{{ .Input }}" "Given the following git diff, generate a concise, Conventional Commit compliant message (type: subject). Focus on the core changes. If there are multiple changes, try to summarize them into one cohesive message. Do NOT include any additional text, just the commit message."
```

```output
feat: add project description and initial Python script

- Updates README.md with a description of the project.
- Adds `src/main.py` with a `hello` function and a new feature flag.
```

**Note:** The `--template "{{ .Input }}"` flag with Ollama is crucial. It tells Ollama to treat the entire piped input as the user's prompt, preventing it from adding conversational boilerplate. Different LLM clients or APIs might have different ways to handle this.

This is a good start! You can then manually copy and paste this into `git commit -m "..."`.

### 2. Using a Script (More Automated)

Typing that long `ollama` command every time is not practical. Let's wrap it in a simple shell script.

Create a file named `git-ai-commit.sh` in your project root or in your `$PATH`.

```bash
#!/bin/bash

# Define the Ollama model to use
OLLAMA_MODEL="phi3"

# Define the prompt for the AI
AI_PROMPT="Given the following git diff, generate a concise, Conventional Commit compliant message (type: subject\n\nbody). Focus on the core changes. If there are multiple changes, summarize them into one cohesive message. Do NOT include any additional text, just the commit message."

# Get the staged changes diff
DIFF_CONTENT=$(git diff --staged)

# Check if there are any staged changes
if [ -z "$DIFF_CONTENT" ]; then
  echo "No staged changes to commit."
  exit 1
fi

echo "--- Generating commit message with AI (Model: $OLLAMA_MODEL) ---"

# Pipe the diff to Ollama and capture the output
COMMIT_MESSAGE=$(echo "$DIFF_CONTENT" | ollama run "$OLLAMA_MODEL" --template "{{ .Input }}" "$AI_PROMPT")

# Check if the AI returned a message
if [ -z "$COMMIT_MESSAGE" ]; then
  echo "AI failed to generate a commit message. Please review the diff and try again manually."
  exit 1
fi

echo -e "\n--- Suggested Commit Message ---\n"
echo "$COMMIT_MESSAGE"
echo -e "\n-------------------------------\n"

# Offer to commit or let the user edit
read -p "Commit with this message? (y/n/e for edit): " -n 1 -r REPLY
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
  git commit -m "$COMMIT_MESSAGE"
  echo "Commit successful!"
elif [[ $REPLY =~ ^[Ee]$ ]]; then
  # Open message in editor for review/edit
  echo "$COMMIT_MESSAGE" > .git/COMMIT_EDITMSG
  EDITOR=$(git var GIT_EDITOR)
  if [ -z "$EDITOR" ]; then
    EDITOR=$(which vim || which nano || which vi) # Fallback editors
  fi
  "$EDITOR" .git/COMMIT_EDITMSG
  COMMIT_MESSAGE=$(cat .git/COMMIT_EDITMSG)
  git commit -F .git/COMMIT_EDITMSG
  rm .git/COMMIT_EDITMSG
  echo "Commit successful after edit!"
else
  echo "Commit cancelled. You can commit manually or re-run the script."
fi
```

Make the script executable:

```bash
chmod +x git-ai-commit.sh
```

Now, stage some new changes (e.g., modify `src/main.py`):

```bash
echo "    # New comment" >> src/main.py
echo "    print('Updated output!')" >> src/main.py
git add src/main.py
```

Run the script:

```bash
./git-ai-commit.sh
```

```output
--- Generating commit message with AI (Model: phi3) ---

--- Suggested Commit Message ---

refactor: update main.py for improved output

- Adds a new print statement to `hello` function.
- Includes a comment for clarity.

-------------------------------

Commit with this message? (y/n/e for edit): y
Commit successful!
```

This script makes the process much smoother! You get a suggestion, and you can accept, edit, or cancel.

#### Refinement: Git Alias

For even quicker use, add a Git alias to your `~/.gitconfig`:

```ini
[alias]
  aic = "!bash -c './git-ai-commit.sh' -"
```

Now, you can just type `git aic` in your project.

### 3. Git Hook (Automated Suggestion)

For the most seamless experience, you can integrate this into a Git hook. The `prepare-commit-msg` hook is ideal because it runs *before* the commit message editor is opened, allowing you to pre-populate the message.

Create or edit the file `.git/hooks/prepare-commit-msg` inside your repository.

```bash
#!/bin/bash

# This script is a Git hook that runs before the commit message editor is opened.
# It attempts to generate a commit message using an AI (Ollama) based on staged changes.

# Arguments to prepare-commit-msg hook:
# $1: Name of the file that contains the commit message (e.g., .git/COMMIT_EDITMSG)
# $2: Commit source (e.g., message, template, merge, squash, commit)
# $3: SHA1 of the commit, if --amend or --fixup

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2

# Only auto-generate if the message file is empty or default,
# and it's not a merge/squash/amend/fixup commit.
# This prevents overwriting messages from other sources or existing messages.
if [[ -s "$COMMIT_MSG_FILE" && "$COMMIT_SOURCE" != "message" && "$COMMIT_SOURCE" != "template" ]]; then
  echo "Existing commit message found or special commit type. Skipping AI generation."
  exit 0
fi

# Define the Ollama model to use
OLLAMA_MODEL="phi3" # Or "llama3", etc.

# Define the prompt for the AI
# Requesting Conventional Commit format for clarity
AI_PROMPT="Given the following git diff, generate a concise, Conventional Commit compliant message (type: subject\n\nbody). Focus on the core changes. If there are multiple changes, summarize them into one cohesive message. Do NOT include any additional text, just the commit message."

# Get the staged changes diff
DIFF_CONTENT=$(git diff --staged)

# Check if there are any staged changes
if [ -z "$DIFF_CONTENT" ]; then
  # If no staged changes, let the user write their own message.
  # Do not exit here; the editor will still open.
  echo "# No staged changes. Write your own commit message." >> "$COMMIT_MSG_FILE"
  exit 0
fi

# Attempt to generate the commit message using Ollama
echo "--- AI is generating commit message... (Model: $OLLAMA_MODEL) ---" >&2 # Output to stderr
COMMIT_MESSAGE=$(echo "$DIFF_CONTENT" | ollama run "$OLLAMA_MODEL" --template "{{ .Input }}" "$AI_PROMPT" 2>/dev/null)

if [ -n "$COMMIT_MESSAGE" ]; then
  # Prepend the AI-generated message to the commit message file
  # And add a comment noting it was AI-generated
  echo "$COMMIT_MESSAGE" > "$COMMIT_MSG_FILE"
  echo "" >> "$COMMIT_MSG_FILE"
  echo "# AI Generated Message - Please review and edit if necessary." >> "$COMMIT_MSG_FILE"
  echo "# Lines starting with '#' will be ignored." >> "$COMMIT_MSG_FILE"
  echo "#" >> "$COMMIT_MSG_FILE"
  # Add original Git comments after our content
  # This makes sure the user still sees relevant info like branch name etc.
  git_comments=$(cat "$COMMIT_MSG_FILE" | grep '^#') # Grab existing comments if any
  echo "$git_comments" >> "$COMMIT_MSG_FILE"
else
  # If AI fails, inform the user and let them write manually
  echo "# AI failed to generate a commit message. Please write your own." >> "$COMMIT_MSG_FILE"
  echo "#" >> "$COMMIT_MSG_FILE"
fi

exit 0
```

Make the hook executable:

```bash
chmod +x .git/hooks/prepare-commit-msg
```

Now, whenever you run `git commit` (and you have staged changes), your editor will pop up with an AI-generated message already filled in.

Let's make another change:

```bash
echo "print('Script started!')" > src/run.py
git add src/run.py
```

Now, just run:

```bash
git commit
```

Your default Git editor (e.g., Vim, VS Code, Nano) will open, pre-populated:

```output
--- AI is generating commit message... (Model: phi3) ---

# This is an example of what you'd see in your editor.
# The content below is what the AI generated and the hook added.

feat: add new run.py script

- Introduces `src/run.py` to print a startup message.

# AI Generated Message - Please review and edit if necessary.
# Lines starting with '#' will be ignored.
#
# On branch main
# Your branch is ahead of 'origin/main' by 1 commit.
#   (use "git push" to publish your local commits)
#
# Changes to be committed:
#       new file:   src/run.py
#
```

You can then review, modify, and save the message to complete the commit. This approach gives you the best of both worlds: automation and human oversight.

## Crafting Effective Prompts for AI

The quality of your AI-generated message directly depends on your prompt. Here are key elements for a good prompt:

*   **Role/Persona:** "You are an expert software engineer..."
*   **Task:** "Generate a concise Git commit message..."
*   **Format Constraints:** "Adhere strictly to Conventional Commits (type: subject\n\nbody).", "Max 72 characters for subject line."
*   **Content Focus:** "Focus on the core changes.", "Summarize multiple changes cohesively."
*   **Exclusion:** "Do NOT include any conversational text, pleasantries, or extra information."

**Example of a Robust Prompt (for the `ollama run` command or script):**

```
"You are an expert software engineer. Your task is to generate a concise and meaningful Git commit message based on the provided `git diff`.
The message MUST strictly follow the Conventional Commits specification:
1.  **Type:** Must be one of `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
2.  **Subject:** A concise, imperative mood summary of the change (max 72 characters, lowercase).
3.  **Body (Optional):** A brief explanation of *why* the change was made, *what* problem it solves, or *how* it was implemented. Use bullet points if listing multiple granular changes.
Do NOT include any conversational text, pleasantries, or markdown formatting (e.g., ```). Provide only the commit message, nothing else.
"
```

**Bad Prompt Example:**

```
"Hey AI, tell me about this code diff. Can you write a commit message? Make it sound good."
```

*   **Why it's bad:** Vague, no format specified, conversational. Will likely result in inconsistent or verbose output.

**Adding Context:**
For more complex changes, you might even consider adding a few lines of context to your prompt, like "The user is working on a REST API, and this diff relates to user authentication." However, for general commit messages, the diff itself usually provides sufficient context for summarization.

## Best Practices & Considerations

While AI can be incredibly helpful, it's not a magic bullet.

1.  **Always Review AI Output:** AI is a tool, not a replacement for critical thinking. Always read the suggested message. Does it accurately reflect your changes? Is it clear? Is it correctly formatted? Edit it if necessary.
2.  **Privacy is Paramount:**
    *   **Local Models (Ollama):** Generally safe as your code stays on your machine.
    *   **Cloud Models (OpenAI, etc.):** **Be extremely cautious.** Understand the provider's data retention and usage policies. For highly sensitive, proprietary, or regulated code, avoid sending diffs to cloud APIs unless explicitly approved and necessary.
3.  **Cost Management (for Cloud Models):** Generating diffs for large changes can consume a lot of tokens, leading to higher API costs. Monitor your usage. Consider techniques like truncating diffs or only sending parts of the diff if a small section is most relevant.
4.  **Prompt Engineering is Key:** Invest time in refining your prompt. A good prompt makes all the difference in the quality and consistency of the generated messages. Tailor it to your team's specific commit conventions.
5.  **Handling Large Diffs:** LLMs have context windows. Very large diffs might be truncated, leading to incomplete summaries.
    *   **Workaround:** Break down large changes into smaller, logical commits. This is good Git practice anyway!
    *   **Alternative:** For very large diffs, you might need to process them in chunks or rely on your manual skills.
6.  **Fallbacks:** What if the AI gives a terrible message, or the service is down? Your workflow should always have a graceful fallback to manual commit message writing. The scripts provided above do this.
7.  **Over-Reliance vs. Tooling:** Don't let AI make you lazy. Understand *why* a good commit message is good. AI simply automates the mechanics; the underlying understanding should still be yours.

## Conclusion

Leveraging AI to assist with Git commit messages can significantly improve your development workflow's efficiency and the quality of your project's commit history. By automating the generation of consistent, meaningful summaries from your code changes, you free up mental space for more complex tasks and ensure a cleaner, more navigable codebase.

Whether you opt for the privacy and control of local models like Ollama or the convenience and power of cloud APIs, the principles remain the same: feed the AI a clear diff, guide it with a precise prompt, and always apply human oversight. This symbiotic relationship between human intelligence and AI automation is where the true value lies.

Start experimenting, refine your prompts, and enjoy a commit history that truly makes sense!
