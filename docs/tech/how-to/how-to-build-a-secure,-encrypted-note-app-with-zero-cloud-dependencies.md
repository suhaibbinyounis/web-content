---
categories:
- Security
- Development
- CLI Tools
- Privacy
comments: true
cover:
  image: https://images.pexels.com/photos/2061168/pexels-photo-2061168.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Learn to build a simple, robust, and highly private encrypted note-taking
  system using command-line tools like GPG, OpenSSL, and Bash, with no reliance on
  cloud services.
tags:
- encryption
- security
- CLI
- bash
- python
- gpg
- openssl
- local-first
- privacy
- data-security
- devops
title: How to Build a Secure, Encrypted Note App with Zero Cloud Dependencies
---

![Detailed view of colorful programming code on a computer screen.](https://images.pexels.com/photos/2061168/pexels-photo-2061168.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Detailed view of colorful programming code on a computer screen.")

## How to Build a Secure, Encrypted Note App with Zero Cloud Dependencies

The cloud is convenient, but it comes with a significant trade-off: control over your data. For sensitive personal notes, journaling, or confidential project details, relying on third-party servers means trusting their security, their privacy policies, and their ability to withstand breaches. What if you want absolute control, maximum privacy, and zero external dependencies?

This post will guide you through building a secure, encrypted note-taking system that lives entirely on your local machine. We'll leverage time-tested command-line tools, focusing on simplicity, security, and the "Unix philosophy" of composing small, powerful utilities.

No more worries about forgotten passwords, data breaches on remote servers, or terms-of-service changes. Your notes, your rules, your encryption keys.

## Why Zero Cloud Dependencies?

Before we dive into the how, let's solidify the why:

*   **Absolute Privacy**: Your data never leaves your device. No analytics, no third-party access, no government requests.
*   **Enhanced Security**: You control the encryption keys and the security posture. You're not relying on a vendor's implementation or their operational security.
*   **Freedom from Lock-in**: No proprietary formats, no API changes, no subscription fees. Your notes are plain text, encrypted with open standards.
*   **Offline Access**: Your notes are always available, regardless of your internet connection.
*   **Longevity**: Your chosen tools (`gpg`, `openssl`, `bash`) are mature, well-maintained, and likely to be around for decades.

This isn't about shunning all cloud services, but understanding the trade-offs and providing an alternative for information that demands the highest level of privacy.

## Core Concepts & Tools

Our solution will be built upon these principles and tools:

*   **Encryption**: We'll primarily use `GnuPG` (GPG) for its robust asymmetric encryption capabilities and key management. We'll also briefly touch on `OpenSSL` for symmetric (password-based) encryption.
*   **Plain Text**: Notes will be stored as plain text files, making them easily editable and future-proof.
*   **Local Storage**: All files reside on your hard drive, within a designated directory.
*   **CLI-First**: We'll interact with our notes exclusively through the command line, using `bash` scripts for automation.
*   **Optional Enhancements**: Tools like `fzf` for fuzzy finding and `tree` for directory visualization can improve usability.

### Prerequisites

You'll need these tools installed on your system. Most Linux and macOS distributions have them pre-installed or available via their package managers.

*   **`gpg`**: GnuPG, for encryption/decryption.
*   **`openssl`**: For symmetric encryption (alternative/complementary).
*   **`bash`**: Your shell scripting environment.
*   **`python3`**: For a slightly more advanced wrapper script (optional).
*   **`mktemp`**: For creating secure temporary files (usually part of `coreutils`).
*   **`vim` or `nano` or `emacs`**: Your preferred command-line text editor.
*   **`fzf`**: (Optional) A command-line fuzzy finder.
*   **`tree`**: (Optional) A command-line utility to display directory contents in a tree-like format.

**Installation Examples**:

On Debian/Ubuntu:
```bash
sudo apt update
sudo apt install gnupg openssl bash python3-full mktemp vim fzf tree
```
```output
# Example output for apt install (will vary)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
...
```

On Fedora:
```bash
sudo dnf install gnupg openssl bash python3 mktemp vim fzf tree
```

On macOS (with Homebrew):
```bash
brew install gnupg openssl python@3.11 fzf tree
```

## Section 1: Basic Encrypted Notes with `gpg`

GnuPG (GPG) is the cornerstone of our encryption strategy. It allows you to encrypt data using public-key cryptography, where you encrypt for a specific recipient (which can be yourself). This means only the holder of the corresponding private key can decrypt the data.

### Generating a GPG Key (if you don't have one)

If you already have a GPG key, you can skip this step. Otherwise, you'll need to generate one. This will be the key used to encrypt your notes.

```bash
gpg --full-generate-key
```
```output
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
Your selection? 9
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) NIST P-256
   (3) NIST P-384
   (4) NIST P-521
   (5) Brainpool P-256
   (6) Brainpool P-384
   (7) Brainpool P-512
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: John Doe
Email address: john.doe@example.com
Comment: My Personal Note Key
You selected this USER-ID:
    "John Doe (My Personal Note Key) <john.doe@example.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the key generation; this gives the random number generator
enough entropy.
gpg: key 69A7F8B6C7D8E9F0 marked as ultimately trusted
... (key generation process) ...
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2024-10-26
pub   ed25519 2023-10-27 [C]
      69A7F8B6C7D8E9F0A1B2C3D4E5F6A7B8C9D0E1F2
uid       [ultimate] John Doe (My Personal Note Key) <john.doe@example.com>
sub   cv25519 2023-10-27 [E]
```

Remember the email address you used; you'll need it as the recipient.

### Encrypting a Note

To encrypt a note, create a plain text file, then use `gpg --encrypt`.

First, let's create a dedicated directory for our notes:
```bash
mkdir -p ~/SecureNotes
cd ~/SecureNotes
```
```output
# No output if successful.
```

Now, create a test note:
```bash
echo "My top secret shopping list:
- Milk
- Eggs
- Quantum Computer Parts (for home project)
" > secret_shopping.txt
```
```output
# No output.
```

Encrypt `secret_shopping.txt` for your GPG key. Replace `john.doe@example.com` with your own GPG email address.

```bash
gpg --encrypt --recipient "john.doe@example.com" secret_shopping.txt
```
```output
# No output if successful. A new file `secret_shopping.txt.gpg` will be created.
```

Verify the encrypted file exists:
```bash
ls -l
```
```output
total 8
-rw-r--r-- 1 user user 114 Oct 27 10:30 secret_shopping.txt
-rw-r--r-- 1 user user 387 Oct 27 10:31 secret_shopping.txt.gpg
```

You can safely delete the original `secret_shopping.txt` now:
```bash
rm secret_shopping.txt
```

### Decrypting a Note

To decrypt, simply use `gpg --decrypt`. GPG will prompt you for your GPG key's passphrase.

```bash
gpg --decrypt secret_shopping.txt.gpg
```
```output
gpg: encrypted with 1 ELG key:
      ID: 69A7F8B6C7D8E9F0A1B2C3D4E5F6A7B8C9D0E1F2
      ...
My top secret shopping list:
- Milk
- Eggs
- Quantum Computer Parts (for home project)
```

By default, `gpg --decrypt` outputs to standard output. To decrypt to a file, use the `-o` option:

```bash
gpg --output decrypted_shopping.txt --decrypt secret_shopping.txt.gpg
```
```output
gpg: encrypted with 1 ELG key:
      ID: 69A7F8B6C7D8E9F0A1B2C3D4E5F6A7B8C9D0E1F2
      ...
```
```bash
cat decrypted_shopping.txt
```
```output
My top secret shopping list:
- Milk
- Eggs
- Quantum Computer Parts (for home project)
```
After viewing, remember to delete the decrypted temporary file.

## Section 2: Symmetric Encryption with `openssl` (Alternative)

While GPG is great for key management, `openssl` offers a simpler, symmetric (password-based) encryption option. It's useful if you prefer to rely on a single passphrase per file rather than managing GPG keys. The AES-256-CBC cipher is a strong, widely accepted standard.

**Note**: For `openssl` encryption, you're relying entirely on a password. There's no key pair involved. This is simpler but means you must remember a strong, unique password for each file if you don't want to reuse one master password.

### Encrypting a Note with `openssl`

```bash
echo "Another secret: Don't tell anyone I like pineapple on pizza." > pizza_secret.txt
```

Now, encrypt it. You'll be prompted for a passphrase twice.

```bash
openssl enc -aes-256-cbc -salt -in pizza_secret.txt -out pizza_secret.txt.enc
```
```output
enter aes-256-cbc encryption password:
Verifying - enter aes-256-cbc encryption password:
# No other output if successful.
```
Check the files:
```bash
ls -l pizza_secret.*
```
```output
-rw-r--r-- 1 user user 60 Oct 27 10:45 pizza_secret.txt
-rw-r--r-- 1 user user 80 Oct 27 10:46 pizza_secret.txt.enc
```
Again, delete the plaintext file:
```bash
rm pizza_secret.txt
```

### Decrypting a Note with `openssl`

To decrypt, use the `-d` (decrypt) flag and provide the same passphrase.

```bash
openssl enc -aes-256-cbc -d -in pizza_secret.txt.enc -out pizza_secret_decrypted.txt
```
```output
enter aes-256-cbc decryption password:
# No other output if successful.
```

View the decrypted content:
```bash
cat pizza_secret_decrypted.txt
```
```output
Another secret: Don't tell anyone I like pineapple on pizza.
```
And clean up:
```bash
rm pizza_secret_decrypted.txt
```

While `openssl` is viable, for a note *system*, GPG's key management (using one key for all notes) is generally more convenient and secure for daily use. We'll proceed with GPG for our automated scripts.

## Section 3: Automating with Bash Scripts

Typing `gpg --encrypt` and `gpg --decrypt` every time gets tedious. Let's create some simple Bash scripts to automate the common tasks: creating, viewing, and searching notes.

We'll assume all encrypted notes live in `~/SecureNotes`.

### Directory Structure

It's good practice to have a clear structure. We'll use a simple flat directory for now, but you could easily extend this with subdirectories for categories.

```bash
mkdir -p ~/SecureNotes
tree -a ~/SecureNotes # This command might not show anything yet if you just created it.
```
```output
/home/user/SecureNotes [error opening dir]

0 directories, 0 files
```
*(The `[error opening dir]` is expected if the directory is empty)*

### Script 1: `note-new` (Create a new encrypted note)

This script will:
1.  Prompt for a note title.
2.  Create a temporary file for editing.
3.  Open your preferred editor (e.g., `vim`, `nano`) with the temporary file.
4.  Encrypt the temporary file using GPG.
5.  Save the encrypted file with a timestamped filename in `~/SecureNotes`.
6.  Clean up the temporary file.

Save this as `~/bin/note-new` (create `~/bin` and add it to your `PATH` if you haven't already).

```bash
#!/bin/bash

# Configuration
NOTES_DIR="$HOME/SecureNotes"
GPG_RECIPIENT="john.doe@example.com" # !!! IMPORTANT: Change this to your GPG email !!!
EDITOR="${EDITOR:-vim}" # Use system default editor or vim

# Ensure the notes directory exists
mkdir -p "$NOTES_DIR"

# Prompt for a descriptive title
read -p "Enter note title: " NOTE_TITLE

if [ -z "$NOTE_TITLE" ]; then
    echo "Note title cannot be empty. Aborting."
    exit 1
fi

# Sanitize title for filename
FILENAME_TITLE=$(echo "$NOTE_TITLE" | tr -c 'a-zA-Z0-9_' '-' | sed 's/-\{1,\}/-/g' | sed 's/^-//;s/-$//' | tr '[:upper:]' '[:lower:]')

# Create a unique temporary file
TEMP_FILE=$(mktemp)
if [ $? -ne 0 ]; then
    echo "Error creating temporary file. Aborting."
    exit 1
fi

# Ensure the temporary file is deleted on exit, even if script fails
trap "rm -f \"$TEMP_FILE\"" EXIT

echo "Opening editor for '$NOTE_TITLE'..."
"$EDITOR" "$TEMP_FILE"

# Check if content was added (file size > 0)
if [ ! -s "$TEMP_FILE" ]; then
    echo "Note is empty. Not saving."
    exit 0
fi

# Generate a timestamp for the filename
TIMESTAMP=$(date +%Y%m%d%H%M%S)

# Construct the final encrypted filename
ENCRYPTED_FILE="${NOTES_DIR}/${TIMESTAMP}-${FILENAME_TITLE}.gpg"

echo "Encrypting and saving note to '$ENCRYPTED_FILE'..."
gpg --encrypt --recipient "$GPG_RECIPIENT" --output "$ENCRYPTED_FILE" "$TEMP_FILE"

if [ $? -eq 0 ]; then
    echo "Note saved successfully."
else
    echo "Error encrypting note. Original content is still in '$TEMP_FILE' (will be deleted on exit)."
    exit 1
fi

# The trap will handle deleting TEMP_FILE
```

Make it executable:
```bash
chmod +x ~/bin/note-new
```

**Usage**:
```bash
~/bin/note-new
```
```output
Enter note title: My first secure note
Opening editor for 'My first secure note'...
# Your editor will open. Type your note, save, and exit.
# Example content typed in editor:
# This is my first secure note.
# It contains highly confidential information.
# Like my plan to take over the world.
Encrypting and saving note to '/home/user/SecureNotes/20231027110000-my-first-secure-note.gpg'...
Note saved successfully.
```
After saving and exiting your editor, you should see the new encrypted note:
```bash
ls -l ~/SecureNotes/
```
```output
total 8
-rw-r--r-- 1 user user 387 Oct 27 11:00 20231027110000-my-first-secure-note.gpg
```

### Script 2: `note-view` (View/Edit an existing encrypted note)

This script will:
1.  List all encrypted notes in `~/SecureNotes`.
2.  Optionally use `fzf` to allow fuzzy selection of a note.
3.  Decrypt the selected note to a temporary file.
4.  Open the temporary file in your editor.
5.  On editor exit, re-encrypt the (potentially modified) note.
6.  Clean up the temporary file.

Save this as `~/bin/note-view`.

```bash
#!/bin/bash

# Configuration
NOTES_DIR="$HOME/SecureNotes"
GPG_RECIPIENT="john.doe@example.com" # !!! IMPORTANT: Change this to your GPG email !!!
EDITOR="${EDITOR:-vim}" # Use system default editor or vim

# Ensure the notes directory exists
if [ ! -d "$NOTES_DIR" ]; then
    echo "Notes directory not found: $NOTES_DIR"
    echo "Create some notes first using 'note-new'."
    exit 1
fi

# List .gpg files, allow selection with fzf if available, otherwise prompt.
# Only show the base filename for clarity.
NOTES=$(find "$NOTES_DIR" -type f -name "*.gpg" -printf "%f\n" | sort -r)

if [ -z "$NOTES" ]; then
    echo "No encrypted notes found in $NOTES_DIR. Create one using 'note-new'."
    exit 0
fi

SELECTED_NOTE_BASENAME=""
if command -v fzf &>/dev/null; then
    # Use fzf for interactive selection
    SELECTED_NOTE_BASENAME=$(echo "$NOTES" | fzf --prompt="Select note to view: ")
else
    # Fallback to simple numbered list if fzf is not available
    echo "Available notes:"
    SELECTABLE_NOTES=()
    i=1
    while IFS= read -r note; do
        echo "  $i) $note"
        SELECTABLE_NOTES+=("$note")
        i=$((i+1))
    done <<< "$NOTES"

    if [ ${#SELECTABLE_NOTES[@]} -eq 0 ]; then
        echo "No notes found."
        exit 0
    fi

    read -p "Enter number to view: " selection_num
    if [[ "$selection_num" =~ ^[0-9]+$ ]] && [ "$selection_num" -ge 1 ] && [ "$selection_num" -le ${#SELECTABLE_NOTES[@]} ]; then
        SELECTED_NOTE_BASENAME="${SELECTABLE_NOTES[$((selection_num-1))]}"
    else
        echo "Invalid selection. Aborting."
        exit 1
    fi
fi

if [ -z "$SELECTED_NOTE_BASENAME" ]; then
    echo "No note selected. Exiting."
    exit 0
fi

ENCRYPTED_FILE="${NOTES_DIR}/${SELECTED_NOTE_BASENAME}"

# Create a unique temporary file for decryption
TEMP_FILE=$(mktemp)
if [ $? -ne 0 ]; then
    echo "Error creating temporary file. Aborting."
    exit 1
fi

# Ensure the temporary file is deleted on exit
trap "rm -f \"$TEMP_FILE\"" EXIT

echo "Decrypting '$SELECTED_NOTE_BASENAME'..."
# Decrypt the note to the temporary file
gpg --decrypt --output "$TEMP_FILE" "$ENCRYPTED_FILE"
if [ $? -ne 0 ]; then
    echo "Error decrypting note. Check your GPG passphrase."
    exit 1
fi

# Get the initial modification time of the decrypted file
# This helps us determine if the file was modified by the editor
INITIAL_MTIME=$(stat -c %Y "$TEMP_FILE" 2>/dev/null || stat -f %m "$TEMP_FILE" 2>/dev/null)

echo "Opening editor for '$SELECTED_NOTE_BASENAME'..."
"$EDITOR" "$TEMP_FILE"

# Get the final modification time
FINAL_MTIME=$(stat -c %Y "$TEMP_FILE" 2>/dev/null || stat -f %m "$TEMP_FILE" 2>/dev/null)

# Check if the file was modified
if [ "$INITIAL_MTIME" != "$FINAL_MTIME" ]; then
    echo "Note modified. Re-encrypting..."
    gpg --encrypt --recipient "$GPG_RECIPIENT" --output "$ENCRYPTED_FILE" "$TEMP_FILE"
    if [ $? -eq 0 ]; then
        echo "Note re-encrypted successfully."
    else
        echo "Error re-encrypting note. Original encrypted file untouched."
        exit 1
    fi
else
    echo "Note not modified. No re-encryption needed."
fi

# The trap will handle deleting TEMP_FILE
```

Make it executable:
```bash
chmod +x ~/bin/note-view
```

**Usage**:
```bash
~/bin/note-view
```
```output
Select note to view:
> 20231027110000-my-first-secure-note.gpg
  secret_shopping.txt.gpg
Decrypting '/home/user/SecureNotes/20231027110000-my-first-secure-note.gpg'...
# GPG will prompt for your passphrase here.
Opening editor for '20231027110000-my-first-secure-note.gpg'...
# Your editor will open with the decrypted content.
# Make changes, save, and exit.
Note modified. Re-encrypting...
# GPG will prompt for your passphrase again to re-encrypt.
Note re-encrypted successfully.
```
**Note**: It's crucial that your editor doesn't create unencrypted swap files or backups in accessible locations. For Vim, you can add `set noswapfile` and `set nobackup` to your `.vimrc` for sensitive editing sessions, or ensure your `tmpdir` is set to a secure, in-memory location if possible. However, the `mktemp` utility creates temporary files in secure locations by default (e.g., `/tmp`), which are usually cleared on reboot.

### Script 3: `note-search` (Search across encrypted notes)

Searching encrypted content requires decrypting it first. This script will decrypt all notes into temporary files, `grep` through them, and then clean up.

**Note**: For a large number of notes, this can be slow and temporarily expose all note content in the `/tmp` directory (which is usually `tmpfs` / in-memory on modern Linux distributions). Use with caution and ensure your `/tmp` is secure.

Save this as `~/bin/note-search`.

```bash
#!/bin/bash

# Configuration
NOTES_DIR="$HOME/SecureNotes"
SEARCH_TERM="$1" # The first argument will be our search term

# Ensure search term is provided
if [ -z "$SEARCH_TERM" ]; then
    echo "Usage: note-search <search_term>"
    exit 1
fi

# Create a temporary directory for decrypted notes
TEMP_DIR=$(mktemp -d)
if [ $? -ne 0 ]; then
    echo "Error creating temporary directory. Aborting."
    exit 1
fi

# Ensure the temporary directory is deleted on exit
trap "rm -rf \"$TEMP_DIR\"" EXIT

echo "Decrypting notes to temporary directory for search..."

# Decrypt all .gpg notes into the temporary directory
for ENCRYPTED_FILE in "$NOTES_DIR"/*.gpg; do
    if [ -f "$ENCRYPTED_FILE" ]; then
        BASENAME=$(basename "$ENCRYPTED_FILE" .gpg)
        DECRYPTED_FILE="$TEMP_DIR/$BASENAME"
        gpg --decrypt --output "$DECRYPTED_FILE" "$ENCRYPTED_FILE" &>/dev/null # Suppress GPG output
        if [ $? -ne 0 ]; then
            echo "Warning: Failed to decrypt $ENCRYPTED_FILE. Skipping."
            rm -f "$DECRYPTED_FILE" # Clean up partial decryption
        fi
    fi
done

echo "Searching for '$SEARCH_TERM'..."

# Search within the decrypted notes
grep -i -n "$SEARCH_TERM" "$TEMP_DIR"/* 2>/dev/null | sed "s|$TEMP_DIR/||"

if [ $? -ne 0 ]; then
    echo "No matches found."
fi

echo "Search complete. Temporary files cleaned up."
# The trap will handle deleting TEMP_DIR
```

Make it executable:
```bash
chmod +x ~/bin/note-search
```

**Usage**:
```bash
~/bin/note-search "shopping list"
```
```output
Decrypting notes to temporary directory for search...
# GPG will prompt for your passphrase for each unique GPG key that was used to encrypt the notes.
Searching for 'shopping list'...
secret_shopping.txt:1:My top secret shopping list:
Search complete. Temporary files cleaned up.
```
This might prompt your GPG passphrase multiple times if your GPG agent isn't configured to remember it, or if you've used different keys for different notes. For convenience, consider configuring `gpg-agent`.

## Section 4: A Simple Python Wrapper (for more robustness)

While Bash scripts are excellent for quick automation, Python offers better error handling, more complex logic, and platform independence (though our GPG/OpenSSL reliance still ties us to those binaries). Here's a conceptual outline and a simple `notes.py` script demonstrating how you might build a more robust system.

We'll use `subprocess` to interact with `gpg`.

Save this as `~/bin/notes.py`.

```python
#!/usr/bin/env python3

import os
import sys
import subprocess
import tempfile
import shutil
from datetime import datetime

# --- Configuration ---
NOTES_DIR = os.path.expanduser("~/SecureNotes")
GPG_RECIPIENT = "john.doe@example.com" # !!! IMPORTANT: Change this to your GPG email !!!
EDITOR = os.environ.get("EDITOR", "vim")

# Ensure the notes directory exists
os.makedirs(NOTES_DIR, exist_ok=True)

def run_command(cmd, input_data=None, suppress_output=False):
    """Helper to run shell commands."""
    try:
        process = subprocess.run(
            cmd,
            input=input_data,
            capture_output=suppress_output,
            text=True,
            check=True
        )
        if suppress_output:
            return process.stdout.strip()
        else:
            return process.returncode == 0
    except subprocess.CalledProcessError as e:
        print(f"Error running command: {' '.join(cmd)}", file=sys.stderr)
        print(f"Stderr: {e.stderr}", file=sys.stderr)
        print(f"Stdout: {e.stdout}", file=sys.stderr)
        return False
    except FileNotFoundError:
        print(f"Error: Command not found. Is '{cmd[0]}' installed and in your PATH?", file=sys.stderr)
        return False

def new_note():
    """Create a new encrypted note."""
    title = input("Enter note title: ").strip()
    if not title:
        print("Note title cannot be empty. Aborting.")
        return

    # Sanitize title for filename
    filename_title = "".join(c if c.isalnum() else "-" for c in title).lower()
    filename_title = "-".join(filter(None, filename_title.split('-'))) # Remove multiple hyphens and leading/trailing
    if not filename_title: # Fallback if title becomes empty after sanitization
        filename_title = "untitled"

    timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
    encrypted_filepath = os.path.join(NOTES_DIR, f"{timestamp}-{filename_title}.gpg")

    with tempfile.NamedTemporaryFile(mode='w+', delete=False, encoding='utf-8') as tmp_file:
        tmp_path = tmp_file.name

    try:
        print(f"Opening editor for '{title}'...")
        # Open editor
        cmd = [EDITOR, tmp_path]
        editor_process = subprocess.run(cmd)

        if editor_process.returncode != 0:
            print("Editor exited with an error. Aborting note creation.", file=sys.stderr)
            return

        # Check if content was added
        if os.path.getsize(tmp_path) == 0:
            print("Note is empty. Not saving.")
            return

        print(f"Encrypting and saving note to '{encrypted_filepath}'...")
        gpg_cmd = [
            "gpg", "--encrypt", "--recipient", GPG_RECIPIENT,
            "--output", encrypted_filepath, tmp_path
        ]
        if run_command(gpg_cmd):
            print("Note saved successfully.")
        else:
            print("Failed to encrypt note. Check GPG setup.", file=sys.stderr)

    finally:
        # Clean up the temporary file
        if os.path.exists(tmp_path):
            os.remove(tmp_path)

def list_notes():
    """List all encrypted notes."""
    notes = sorted([f for f in os.listdir(NOTES_DIR) if f.endswith(".gpg")], reverse=True)
    if not notes:
        print("No notes found. Create one using 'notes new'.")
        return []

    print("Available notes:")
    for i, note in enumerate(notes):
        print(f"  {i+1}) {note}")
    return notes

def view_note():
    """View/edit an existing encrypted note."""
    notes = list_notes()
    if not notes:
        return

    selection = input("Enter number or part of filename to view: ").strip()

    selected_note_basename = None
    if selection.isdigit():
        idx = int(selection) - 1
        if 0 <= idx < len(notes):
            selected_note_basename = notes[idx]
    else: # Try fuzzy match
        matching_notes = [n for n in notes if selection.lower() in n.lower()]
        if len(matching_notes) == 1:
            selected_note_basename = matching_notes[0]
        elif len(matching_notes) > 1:
            print("Multiple matches found. Please be more specific or use the number:")
            for note in matching_notes:
                print(f"  - {note}")
            return
        else:
            print(f"No note found matching '{selection}'.")
            return

    if not selected_note_basename:
        print("Invalid selection.")
        return

    encrypted_filepath = os.path.join(NOTES_DIR, selected_note_basename)

    with tempfile.NamedTemporaryFile(mode='w+', delete=False, encoding='utf-8') as tmp_file:
        tmp_path = tmp_file.name

    try:
        print(f"Decrypting '{selected_note_basename}'...")
        gpg_decrypt_cmd = ["gpg", "--decrypt", "--output", tmp_path, encrypted_filepath]
        if not run_command(gpg_decrypt_cmd):
            print("Failed to decrypt note. Check your GPG passphrase.", file=sys.stderr)
            return

        initial_mtime = os.path.getmtime(tmp_path)

        print(f"Opening editor for '{selected_note_basename}'...")
        editor_process = subprocess.run([EDITOR, tmp_path])

        if editor_process.returncode != 0:
            print("Editor exited with an error. Changes might not be saved.", file=sys.stderr)
            return

        final_mtime = os.path.getmtime(tmp_path)

        if final_mtime > initial_mtime:
            print("Note modified. Re-encrypting...")
            gpg_encrypt_cmd = [
                "gpg", "--encrypt", "--recipient", GPG_RECIPIENT,
                "--output", encrypted_filepath, tmp_path
            ]
            if run_command(gpg_encrypt_cmd):
                print("Note re-encrypted successfully.")
            else:
                print("Failed to re-encrypt note. Original encrypted file is unchanged.", file=sys.stderr)
        else:
            print("Note not modified. No re-encryption needed.")

    finally:
        if os.path.exists(tmp_path):
            os.remove(tmp_path)

def search_notes():
    """Search content across all encrypted notes."""
    search_term = input("Enter search term: ").strip()
    if not search_term:
        print("Search term cannot be empty.")
        return

    temp_search_dir = tempfile.mkdtemp()
    try:
        print("Decrypting notes for search (this may take a moment)...")
        decrypted_files = []
        for encrypted_file in os.listdir(NOTES_DIR):
            if encrypted_file.endswith(".gpg"):
                full_encrypted_path = os.path.join(NOTES_DIR, encrypted_file)
                base_name = os.path.splitext(encrypted_file)[0]
                decrypted_path = os.path.join(temp_search_dir, base_name)

                # Decrypt silently for search
                gpg_decrypt_cmd = ["gpg", "--decrypt", "--output", decrypted_path, full_encrypted_path]
                if not run_command(gpg_decrypt_cmd, suppress_output=True):
                    print(f"Warning: Failed to decrypt '{encrypted_file}'. Skipping.", file=sys.stderr)
                    if os.path.exists(decrypted_path):
                        os.remove(decrypted_path) # Clean up partial decrypt
                    continue
                decrypted_files.append(decrypted_path)

        if not decrypted_files:
            print("No decryptable notes found to search.")
            return

        print(f"\nSearching for '{search_term}'...")
        grep_cmd = ["grep", "-i", "-n", search_term] + decrypted_files
        try:
            grep_output = subprocess.run(grep_cmd, capture_output=True, text=True, check=True)
            for line in grep_output.stdout.splitlines():
                # Replace temp dir path with actual filename for cleaner output
                clean_line = line.replace(temp_search_dir + os.sep, "")
                print(clean_line)
            print("Search complete.")
        except subprocess.CalledProcessError:
            print("No matches found.")

    finally:
        # Clean up the temporary directory and its contents
        if os.path.exists(temp_search_dir):
            shutil.rmtree(temp_search_dir)

def display_help():
    print("Usage: notes <command>")
    print("\nCommands:")
    print("  new      - Create a new encrypted note.")
    print("  list     - List all existing encrypted notes.")
    print("  view     - View and optionally edit an existing encrypted note.")
    print("  search   - Search content within all encrypted notes.")
    print("  help     - Display this help message.")
    print("\nConfiguration:")
    print(f"  Notes directory: {NOTES_DIR}")
    print(f"  GPG Recipient:   {GPG_RECIPIENT}")
    print(f"  Editor:          {EDITOR}")


if __name__ == "__main__":
    if len(sys.argv) < 2:
        display_help()
        sys.exit(1)

    command = sys.argv[1].lower()

    if command == "new":
        new_note()
    elif command == "list":
        list_notes()
    elif command == "view":
        view_note()
    elif command == "search":
        search_notes()
    elif command == "help":
        display_help()
    else:
        print(f"Unknown command: '{command}'", file=sys.stderr)
        display_help()
        sys.exit(1)
```

Make it executable:
```bash
chmod +x ~/bin/notes.py
```

**Usage (assuming `~/bin` is in your `PATH`)**:

```bash
notes.py new
```
```output
Enter note title: My Python-managed note
Opening editor for 'My Python-managed note'...
# Editor opens. Add some content. Save and exit.
Encrypting and saving note to '/home/user/SecureNotes/20231027120000-my-python-managed-note.gpg'...
Note saved successfully.
```

```bash
notes.py list
```
```output
Available notes:
  1) 20231027120000-my-python-managed-note.gpg
  2) 20231027110000-my-first-secure-note.gpg
  3) secret_shopping.txt.gpg
```

```bash
notes.py view
```
```output
Available notes:
  1) 20231027120000-my-python-managed-note.gpg
  2) 20231027110000-my-first-secure-note.gpg
  3) secret_shopping.txt.gpg
Enter number or part of filename to view: 2
Decrypting '/home/user/SecureNotes/20231027110000-my-first-secure-note.gpg'...
# GPG passphrase prompt
Opening editor for '20231027110000-my-first-secure-note.gpg'...
# Editor opens. Make some changes. Save and exit.
Note modified. Re-encrypting...
# GPG passphrase prompt
Note re-encrypted successfully.
```

```bash
notes.py search "quantum"
```
```output
Decrypting notes for search (this may take a moment)...
# GPG passphrase prompts
Searching for 'quantum'...
secret_shopping.txt:3:- Quantum Computer Parts (for home project)
Search complete.
```

This Python script provides a more structured and extensible foundation compared to raw Bash, especially for error handling and user interaction.

## Security Considerations & Best Practices

Building a local, encrypted note app gives you immense control, but also means you're solely responsible for its security.

1.  **Strong GPG Passphrase**: Your GPG key's passphrase is the primary guardian of your notes. Choose a long, complex, and memorable passphrase. Consider using a password manager or a secure method for remembering it.
2.  **GPG Agent**: Configure `gpg-agent` to cache your passphrase for a period. This reduces the number of times you need to type it, improving usability without significantly compromising security (as long as your session is secure).
3.  **Physical Device Security**: If someone gains physical access to your unencrypted device while you're logged in and `gpg-agent` is caching your passphrase, they could access your notes. Use strong disk encryption (e.g., LUKS on Linux, FileVault on macOS) and strong user login passwords. Lock your screen when you step away.
4.  **Temporary Files**:
    *   Our scripts use `mktemp` and `trap` to handle temporary files securely. `mktemp` creates unique files in `/tmp` (or `TMPDIR`), which are typically readable only by the owner and often reside in RAM (`tmpfs`).
    *   `trap "rm -f \"$TEMP_FILE\"" EXIT` is crucial: it ensures the temporary decrypted content is deleted even if the script crashes or is interrupted.
    *   **Editor Swap/Backup Files**: Be aware that some editors (like Vim by default) create swap files or backup files during editing. Ensure these files are also written to secure locations or disabled for sensitive edits. For Vim, add `set noswapfile nobackup nowritebackup` to your `.vimrc` or execute them before editing sensitive files.
5.  **Backups**: Encrypted notes can be safely backed up to cloud storage or external drives. However, ensure you also have a secure backup of your GPG private key and its passphrase! Without them, your encrypted notes are useless. Export your GPG key pair (private and public) and store it securely (e.g., on an encrypted USB drive, in a physically secure location).
    ```bash
    # Export private key (recommended to add --armor for text output)
    gpg --export-secret-keys --armor "john.doe@example.com" > my_private_gpg_key.asc
    # Export public key (useful if you ever share encrypted notes with someone else)
    gpg --export --armor "john.doe@example.com" > my_public_gpg_key.asc
    ```
6.  **Audit Your Scripts**: Understand what your scripts do. The provided examples are simple, but if you extend them, review your code carefully, especially regarding file handling and `subprocess` calls.

## Limitations & Future Enhancements

While powerful, this simple setup has some inherent limitations by design:

*   **No Synchronization**: Notes are strictly local. For multi-device sync, you'd need to implement a separate, secure synchronization mechanism (e.g., `rsync` over `SSH` to a personal server, or syncing the encrypted `SecureNotes` directory via a trusted cloud sync provider like Syncthing or a self-hosted Nextcloud instance, being careful about conflict resolution on encrypted files).
*   **Plain Text Only**: No rich text formatting, images, or attachments. This keeps the system simple and robust.
*   **CLI Only**: No graphical user interface. This is often preferred by power users but might be a barrier for others.
*   **No Version Control**: The scripts don't inherently track changes. You could initialize a `git` repository *within* the `SecureNotes` directory and commit changes to the `.gpg` files. While `git diff` won't show plaintext changes, `git log` would track versions of the encrypted blobs.

**Potential Enhancements**:

*   **Note Tagging**: Implement a system for adding tags to notes (e.g., within the note content itself, or in a separate index file), and extend `note-search` to filter by tags.
*   **Markdown Support**: While notes are plain text, you could encourage Markdown syntax and use a Markdown viewer (like `mdless` or `glow`) on the decrypted temporary file.
*   **Simple GUI Wrapper**: Use Python libraries like `tkinter` or `PyQt` to create a basic desktop GUI that calls the same backend GPG/`subprocess` logic.
*   **`fzf` Integration**: As shown in `note-view`, `fzf` can significantly improve interactive selection.
*   **Encrypted Journal**: Adapt the `note-new` script to automatically name notes by date (e.g., `YYYY-MM-DD.gpg`) for a simple journaling system.

## Conclusion

You've now built the foundation of a highly secure, private, and local note-taking system. By understanding the core tools and applying simple scripting, you reclaim control over your most sensitive information, freeing yourself from reliance on external services. This project demonstrates that robust security and privacy don't always require complex software or expensive subscriptions – sometimes, the best solutions are built from simple, well-understood primitives.

Go forth and encrypt your thoughts! Your data, your rules.
---
