---
title: How to Use Obsidian + Git to Create a Local-First Knowledge Base
date: 2025-06-17T13:05:16.383Z
description: "Master a robust, future-proof knowledge management system using Obsidian for note-taking and Git for version control and synchronization. Learn to build a local-first, plain-text knowledge base that's entirely yours, with full control over history and data."
tags:
  - Obsidian
  - Git
  - Knowledge Management
  - Local-First
  - CLI
  - Markdown
  - Productivity
categories:
  - DevOps
  - Productivity
  - Tools
comments: true
---

As developers, we're constantly learning. New languages, frameworks, architectural patterns, obscure CLI flags – the sheer volume of information is immense. Sticking all that hard-won knowledge into ephemeral memory or scattering it across various SaaS platforms feels inefficient and fragile.

You need a system that's:
1.  **Local-First**: Your data, on your disk, always accessible.
2.  **Future-Proof**: Plain text files that don't depend on proprietary formats.
3.  **Version Controlled**: The ability to rewind, branch, and see changes over time.
4.  **Highly Linked**: Connect ideas effortlessly, reflecting the network of your thoughts.
5.  **Synchronization-Friendly**: Work from any device, without vendor lock-in.

This is where [Obsidian](https://obsidian.md/) meets [Git](https://git-scm.com/). Obsidian gives us the powerful, local-first note-taking experience with excellent linking capabilities. Git gives us the robust version control and synchronization backbone. Together, they form an almost perfect local-first knowledge base.

Let's dive in.

## Why Local-First and Git?

Before we set things up, let's briefly unpack *why* this combination is so powerful for a developer's knowledge base:

*   **Control and Ownership**: Your data is yours. No vendor can hold it hostage, change terms, or disappear overnight. It lives as plain Markdown files on your hard drive.
*   **Version Control Magic**:
    *   **History**: Every change is tracked. Wonder why you wrote something a certain way last year? `git blame` or `git log` will tell you.
    *   **Rollback**: Accidentally delete a crucial paragraph? `git revert` or `git restore` can bring it back instantly.
    *   **Branching**: Experiment with a new organizational structure or a massive rewrite of a complex topic without affecting your main knowledge base. Merge it back when ready.
*   **Portability**: Markdown is universal. Your notes are accessible with any text editor, even if Obsidian ceases to exist. This is the ultimate future-proofing.
*   **Offline Access**: Your entire knowledge base is local. No internet? No problem.
*   **Synchronization Flexibility**: While Obsidian offers its own sync service, Git allows you to use *any* Git remote (GitHub, GitLab, self-hosted Gitea, etc.) for synchronization across devices. This keeps costs down and control high.

## Prerequisites

Before we start, ensure you have the following installed:

*   **Git**: The command-line tool. Most Linux distributions have it pre-installed or easily available via package managers. macOS users can install it via Homebrew or by installing Xcode Command Line Tools. Windows users can download Git Bash.
*   **Obsidian**: Download and install it from the official website.
*   **Basic Git Knowledge**: You should be comfortable with `git init`, `git add`, `git commit`, `git push`, and `git pull`.

## Setting Up Your Knowledge Base

We'll start by creating an Obsidian vault and then transforming it into a Git repository.

### Step 1: Create Your Obsidian Vault

First, open Obsidian and create a new vault. Choose a location on your local filesystem that you'll manage with Git.

Let's assume you create a vault named `MyKnowledge` in your home directory.

```bash
# Navigate to your home directory or desired parent directory
cd ~
ls -F | grep MyKnowledge/
```

```output
MyKnowledge/
```

### Step 2: Initialize Your Vault as a Git Repository

Now, navigate *into* your new Obsidian vault directory and initialize it as a Git repository.

```bash
cd ~/MyKnowledge
git init
```

```output
Initialized empty Git repository in /home/user/MyKnowledge/.git/
```

This creates a hidden `.git` directory inside your vault, turning it into a Git repository.

### Step 3: Add a `.gitignore` File

Obsidian creates various configuration files (e.g., `workspace.json`, `app.json`, cache files) and potentially hidden directories for plugins (`.obsidian/plugins/`) that you often don't want to track in Git. These files tend to change frequently and can lead to unnecessary commits or merge conflicts.

Let's create a `.gitignore` file to tell Git to ignore these.

```bash
# Create the .gitignore file
touch .gitignore

# Add common Obsidian-specific patterns to .gitignore
cat <<EOF > .gitignore
# Obsidian configuration and internal files
.obsidian/workspace.json
.obsidian/app.json
.obsidian/cache/
.obsidian/themes/
.obsidian/plugins/*/data.json
.obsidian/plugins/*/main.js
.obsidian/plugins/*/manifest.json
# If you want to track plugins, remove the above plugin lines.
# Often it's better to manage them through Obsidian's UI on each device.

# OS-specific files
.DS_Store
Thumbs.db

# Other common temporary or ignored files
*.bak
*.tmp
*.log
EOF

# Verify the content
cat .gitignore
```

```output
# Obsidian configuration and internal files
.obsidian/workspace.json
.obsidian/app.json
.obsidian/cache/
.obsidian/themes/
.obsidian/plugins/*/data.json
.obsidian/plugins/*/main.js
.obsidian/plugins/*/manifest.json
# If you want to track plugins, remove the above plugin lines.
# Often it's better to manage them through Obsidian's UI on each device.

# OS-specific files
.DS_Store
Thumbs.db

# Other common temporary or ignored files
*.bak
*.tmp
*.log
```

**Note**: You might choose to track certain plugin data (`data.json`) if you rely heavily on specific plugin configurations being identical across devices. For most users, it's simpler to let Obsidian manage plugins and their data separately on each device. The above `.gitignore` excludes them.

### Step 4: Make Your First Commit

Now that Git is initialized and we've specified what to ignore, let's stage and commit the initial state of your vault. This typically includes the `.gitignore` file and any default files Obsidian might have created (e.g., `Untitled.md` or `Welcome.md`).

```bash
# Add all untracked files (except those ignored) to the staging area
git add .

# Commit the changes with a descriptive message
git commit -m "Initial vault setup and .gitignore"
```

```output
[main (root-commit) d4a6a5e] Initial vault setup and .gitignore
 2 files changed, 25 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 Welcome.md
```

Congratulations! Your Obsidian vault is now a Git repository.

## Workflow: Daily Knowledge Management with Git

The core principle here is to make frequent, small, and descriptive commits. Think of each commit as a snapshot of your knowledge base at a specific moment.

### Adding New Notes

When you create a new note in Obsidian, Git sees it as an untracked file.

Let's create a new note named `My First Tech Note.md` inside Obsidian and add some content.

```bash
# After creating the note in Obsidian and saving it
# Check Git status to see the new file
git status
```

```output
On branch main
No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	My First Tech Note.md

nothing added to commit but untracked files present (use "git add" to track)
```

Now, stage and commit the new note:

```bash
git add "My First Tech Note.md"
git commit -m "Add new note on initial tech setup"
```

```output
[main d6a7b8c] Add new note on initial tech setup
 1 file changed, 5 insertions(+)
 create mode 100644 My First Tech Note.md
```

### Modifying Existing Notes

As you refine your thoughts, you'll constantly be editing existing notes.

Let's edit `My First Tech Note.md` to add more details.

```bash
# After modifying the note in Obsidian
# Check what changed
git diff "My First Tech Note.md"
```

```output
diff --git a/My First Tech Note.md b/My First Tech Note.md
index ad21f2d..3f5e1c0 100644
--- a/My First Tech Note.md
+++ b/My First Tech Note.md
@@ -1,3 +1,6 @@
 # My First Tech Note
 
-This is where I'll document my first tech setup.
+This is where I'll document my first tech setup.
+
+## Section 1: Initial Thoughts
+
+This is a new line of text.
```

Now, commit the changes:

```bash
git add "My First Tech Note.md"
git commit -m "Elaborate on initial tech setup in 'My First Tech Note'"
```

```output
[main d2e3f4a] Elaborate on initial tech setup in 'My First Tech Note'
 1 file changed, 3 insertions(+), 0 deletions(-)
```

### Regular Commits (Your Daily Habit)

Make committing a regular habit. Before closing Obsidian for the day, or after a significant writing session, make a commit.

```bash
# Always check status first to see what's changed
git status

# Stage all changes (new, modified, deleted files)
git add .

# Commit with a summary of your work
git commit -m "Daily knowledge sync: added thoughts on networking, refactored project ideas"
```

```output
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   My First Tech Note.md
	new file:   New Networking Concepts.md

[main e9f0a1b] Daily knowledge sync: added thoughts on networking, refactored project ideas
 2 files changed, 10 insertions(+), 2 deletions(-)
 create mode 100644 New Networking Concepts.md
```

### Viewing History and Reverting

This is where Git truly shines. You have a full history of your knowledge base.

To see a concise log of your commits:

```bash
git log --oneline
```

```output
e9f0a1b (HEAD -> main) Daily knowledge sync: added thoughts on networking, refactored project ideas
d2e3f4a Elaborate on initial tech setup in 'My First Tech Note'
d6a7b8c Add new note on initial tech setup
d4a6a5e Initial vault setup and .gitignore
```

To view the exact changes introduced by a specific commit:

```bash
# Replace 'e9f0a1b' with the actual commit hash you want to inspect
git show e9f0a1b
```

```output
commit e9f0a1ba2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e
Author: Your Name <your.email@example.com>
Date:   Fri Oct 27 15:30:00 2023 -0700

    Daily knowledge sync: added thoughts on networking, refactored project ideas

diff --git a/My First Tech Note.md b/My First Tech Note.md
index 3f5e1c0..a1b2c3d 100644
--- a/My First Tech Note.md
+++ b/My First Tech Note.md
@@ -3,4 +3,5 @@
 ## Section 1: Initial Thoughts
 
 This is a new line of text.
+Adding another point here.
diff --git a/New Networking Concepts.md b/New Networking Concepts.md
new file mode 100644
index 0000000..e4f5g6h
--- /dev/null
+++ b/New Networking Concepts.md
@@ -0,0 +1,5 @@
+# Networking Concepts
+
+## OSI Model
+
+- Layer 1: Physical
```

To revert uncommitted changes in a file (i.e., you messed up and want to go back to the last committed version):

```bash
# Modify 'My First Tech Note.md' in Obsidian, then run:
git restore "My First Tech Note.md"
```

```output
# No explicit output on success, just reverts the file.
# You can check 'git status' or 'git diff' to confirm.
```

To revert a file to a specific commit's state (if it was committed):

```bash
# Let's say you want to go back to the state of 'My First Tech Note.md' from commit 'd6a7b8c'
git checkout d6a7b8c -- "My First Tech Note.md"
```

```output
Updated 1 path from 1 commit
```

**Note**: `git checkout` can be confusing for new users as it also handles switching branches. For simply discarding changes to a file, `git restore` (introduced in Git 2.23) is often clearer.

## Synchronizing Across Devices

The real power of Git emerges when you want to access and update your knowledge base from multiple devices (e.g., desktop and laptop). We'll use a remote Git repository for this. GitHub, GitLab, and Bitbucket are popular choices.

### Step 1: Create a Remote Repository

Go to your preferred Git hosting service (e.g., GitHub.com) and create a *new, empty* private repository. Do **not** initialize it with a `README.md` or `.gitignore` as your local repository already has these.

Let's assume your new repository's URL is `https://github.com/your-username/MyKnowledge.git`.

### Step 2: Link Local to Remote and Push

Back in your local vault directory, add the remote origin and push your existing commits to it.

```bash
cd ~/MyKnowledge
git remote add origin https://github.com/your-username/MyKnowledge.git
git branch -M main # Renames your default branch to 'main' (common practice)
git push -u origin main
```

```output
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 8 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (12/12), 1.05 KiB | 1.05 MiB/s, done.
Total 12 (delta 2), reused 0 (delta 0), pack-reused 0
remote:
remote: Create a pull request for 'main' on GitHub by visiting:
remote:      https://github.com/your-username/MyKnowledge/pull/new/main
remote:
To https://github.com/your-username/MyKnowledge.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

Your knowledge base is now backed up and accessible on your remote!

### Step 3: Clone on Another Device

On your second device (e.g., laptop), clone the repository.

```bash
# Navigate to where you want the vault to reside (e.g., your Documents folder)
cd ~/Documents

# Clone the repository
git clone https://github.com/your-username/MyKnowledge.git
```

```output
Cloning into 'MyKnowledge'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 12 (delta 2), reused 12 (delta 2), pack-reused 0
Receiving objects: 100% (12/12), done.
Resolving deltas: 100% (2/2), done.
```

Now, open Obsidian on this second device and choose "Open an existing vault". Navigate to the `MyKnowledge` folder that Git just cloned. Obsidian will recognize it as a vault.

### Daily Synchronization Routine

This is crucial for keeping your vaults in sync across devices and avoiding conflicts.

1.  **Before starting work on a device**: Always `git pull` to fetch the latest changes from the remote.
    ```bash
    cd ~/MyKnowledge # (or wherever your vault is)
    git pull
    ```
    ```output
    Already up to date. # Or it will show files being updated.
    ```
    This ensures you're working on the most recent version of your notes.

2.  **After finishing work or making significant changes**: Stage, commit, and `git push` your changes to the remote.
    ```bash
    git add .
    git commit -m "Sync: Latest updates from laptop"
    git push
    ```
    ```output
    [main c7d8e9f] Sync: Latest updates from laptop
     3 files changed, 15 insertions(+), 1 deletion(-)
    Counting objects: 7, done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (4/4), done.
    Writing objects: 100% (4/4), 488 bytes | 488.00 KiB/s, done.
    Total 4 (delta 2), reused 0 (delta 0), pack-reused 0
    To https://github.com/your-username/MyKnowledge.git
       e9f0a1b..c7d8e9f  main -> main
    ```

**Note on Conflicts**: If you make conflicting changes on two different devices before pushing, `git pull` will inform you of a merge conflict. You'll need to resolve these manually using your preferred merge tool or by editing the conflicted files (they will contain Git's conflict markers like `<<<<<<<`, `=======`, `>>>>>>>`). Resolving conflicts is an essential Git skill. Often, for simple text files like Markdown, Obsidian's live preview can help you see which version you prefer.

## Obsidian Git Plugin (Brief Mention)

For those who prefer a more integrated solution, the [Obsidian Git plugin](https://github.com/denolehov/obsidian-git) can automate much of this. It can automatically commit changes, push, and pull on a schedule or on Obsidian startup/shutdown.

While powerful, it abstracts away the underlying Git commands. For this post, we focused on the manual CLI approach to ensure a foundational understanding of how Git is managing your files. If you find the manual process cumbersome after getting comfortable, the plugin is a great next step.

## Advanced Tips

*   **Branches**: For major refactoring of your knowledge base (e.g., reorganizing entire sections, experimenting with new linking strategies), create a Git branch (`git branch my-refactor` then `git checkout my-refactor`). Work on the branch, and only merge it back to `main` when you're satisfied (`git checkout main` then `git merge my-refactor`).
*   **Tags**: Mark significant milestones in your knowledge base (e.g., `git tag v1.0-initial-setup`, `git tag v2.0-major-refactor-complete`).
*   **Custom Scripts**: Automate your daily `git add . && git commit -m "Daily sync" && git push` with a simple shell script or a cron job.

## Pitfalls and Considerations

*   **Large Binary Files**: Git is optimized for text files. Storing many large images, PDFs, or other binary files directly in your vault will bloat your repository and slow down Git operations. Consider using Git LFS (Large File Storage) or keeping large assets outside your Git-managed vault, linking to them from your notes.
*   **Commit Frequency**: While frequent commits are good, avoid committing every single character change. Group related changes into logical commits.
*   **Conflict Resolution**: This is the biggest hurdle for multi-device sync. Get familiar with `git status`, `git diff`, and `git mergetool`. Practice makes perfect.
*   **Obsidian Sync vs. Git**: Obsidian offers its own paid sync service. It's often simpler to set up, but it doesn't offer the full version history, branching, and open-source control that Git does. For a developer who values complete ownership and robust version control, Git is the superior choice.

## Conclusion

By combining Obsidian's powerful, local-first markdown note-taking with Git's robust version control and synchronization capabilities, you build a resilient, future-proof, and entirely-yours knowledge base. You gain unparalleled control over your intellectual property, peace of mind regarding data integrity, and the flexibility to evolve your system as your needs change.

Embrace the command line, make Git a habit, and watch your personal knowledge flourish.