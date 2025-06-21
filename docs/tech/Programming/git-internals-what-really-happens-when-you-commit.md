---
categories:
- Software Development
- Git
- Version Control
- Programming
comments: true
cover:
  image: https://images.pexels.com/photos/25626446/pexels-photo-25626446.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 09:02:34.262000
description: A deep dive into the `git commit` command, exploring how Git builds and
  links blob, tree, and commit objects to create an immutable history of your project.
tags:
- git
- internals
- version control
- commit
- SCM
- data structures
- blobs
- trees
- objects
- SHA-1
title: Git Internals What Really Happens When You Commit
---

![Visual representation of geometric calculations comparing bits and qubits in black and white.](https://images.pexels.com/photos/25626446/pexels-photo-25626446.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Visual representation of geometric calculations comparing bits and qubits in black and white.")

## Git Internals What Really Happens When You Commit

You've typed it countless times: `git commit -m "My insightful changes"`. It's perhaps the most fundamental command in your Git workflow, the one that immortalizes your hard work into the project's history. But have you ever paused to wonder what *really* happens when you press Enter? It's far more sophisticated than simply "saving files."

Git, at its core, is not just a version control system; it's a content-addressable filesystem with a powerful directed acyclic graph (DAG) structure. Understanding its internals, especially how `git commit` operates, demystifies many of its "magical" behaviors and empowers you to use it more effectively.

Let's peel back the layers and uncover the intricate dance of objects that culminates in a new commit.

## The Git Object Model: The Foundation of Everything

Before we commit, we need to understand the fundamental building blocks Git uses to store your project's data. Everything in Git is stored as an object, identified by its SHA-1 hash. This hash is computed from the object's content, making each object immutable and self-identifying. If the content changes, the hash changes, and it becomes a *new* object.

There are four primary object types, but for `git commit`, we're primarily concerned with three:

1.  **Blob Object (Binary Large Object)**
    *   **What it stores**: The exact content of a file. It doesn't store metadata like filename or path, just the raw binary data.
    *   **How it's named**: The SHA-1 hash of its content.
    *   **Example**: If you have a `hello.txt` file containing "Hello, world!", its blob object will contain "Hello, world!", and its SHA-1 hash will be computed from that string. If you change "world!" to "Git!", it becomes a *new* blob object with a different SHA-1.

    You can inspect a blob with `git cat-file -p <blob-sha>`.
    *Source: [Pro Git Book - Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)*

2.  **Tree Object**
    *   **What it stores**: A directory listing. It contains entries, each pointing to a SHA-1 of another object (either a blob for a file or another tree for a subdirectory). Each entry also includes the object's type, mode (permissions), and filename.
    *   **How it's named**: The SHA-1 hash of its *content*, which is the sorted list of its entries (type, mode, SHA-1, name).
    *   **Example**: A tree object for your project's root directory might contain an entry for `README.md` (pointing to a blob) and an entry for `src/` (pointing to another tree object). Changing a file's content changes its blob, which in turn changes the tree that points to it, recursively changing parent trees up to the root.

    You can inspect a tree with `git ls-tree <tree-sha>` or `git cat-file -p <tree-sha>`.
    *Source: [Pro Git Book - Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)*

3.  **Commit Object**
    *   **What it stores**: This is the "snapshot" of your project at a given point in time. It doesn't store file contents directly but rather:
        *   A pointer (SHA-1) to the root **tree object** that represents the state of your entire working directory for that commit.
        *   One or more pointers (SHA-1s) to its **parent commit(s)**. Most commits have one parent, creating a linear history. Merge commits have two or more parents.
        *   The **author** and **committer** information (name, email, timestamp).
        *   The **commit message**.
    *   **How it's named**: The SHA-1 hash of its *content*, which is all the information listed above.
    *   **Example**: A commit object ties together a specific project state (via its root tree), its history (via its parents), and metadata about who made the change and why.

    You can inspect a commit with `git cat-file -p <commit-sha>`.
    *Source: [Pro Git Book - Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)*

These objects are stored in the `.git/objects` directory within your repository, often compressed into "packfiles" for efficiency over time.

## The Staging Area (Index): Your Next Snapshot in Progress

Before you run `git commit`, you use `git add <files>` to prepare your changes. This is where the **staging area**, also known as the **index**, comes into play. The index is a crucial, often misunderstood, component.

The staging area is *not* just a list of files you've changed. It's a precise snapshot of what your *next* commit will look like. Think of it as a temporary tree object that Git is constructing for you.

When you run `git add <file>`:
1.  Git reads the content of `<file>` from your working directory.
2.  It computes the SHA-1 hash of this file's content.
3.  If a blob object with that content doesn't already exist in the Git object database (`.git/objects/`), Git creates one and stores it.
4.  Git updates the staging area's entry for `<file>` to point to this new blob object's SHA-1. If the file was already staged, its entry is updated.

Effectively, `git add` is the command that takes your working directory files and turns them into *blob objects*, making them available for inclusion in the next commit's tree. The index itself is an internal representation that Git uses to generate the root tree object for your next commit.

## `git commit`: The Orchestrator of History

Now, with our understanding of objects and the staging area, let's dissect `git commit`.

When you execute `git commit`:

1.  **Step 1: Create the Root Tree Object(s)**
    *   Git first takes the current state of your **staging area (index)**.
    *   It recursively walks through the files and directories represented in the index.
    *   For each file, it references the blob object that was created when you `git add`ed it.
    *   It then builds **tree objects** for each directory, linking to the blobs (for files) and other tree objects (for subdirectories) within them.
    *   Finally, it creates a single, top-level **root tree object** that represents the entire state of your project at the moment of the commit, as defined by what's in your staging area.
    *   This root tree object and any newly created nested tree objects are stored in the `.git/objects/` directory, identified by their SHA-1 hashes.

    This is the core idea: a commit doesn't store individual file changes (deltas) but rather a complete snapshot of the project's state as a tree object.

2.  **Step 2: Create the Commit Object**
    *   Once the root tree object is finalized, Git gathers all the necessary information for the commit object:
        *   The SHA-1 hash of the newly created **root tree object**.
        *   The SHA-1 hash of the **parent commit(s)**. Git determines this by looking at where `HEAD` currently points (usually the tip of your current branch). For the very first commit in a repository, there are no parents. For merge commits, there will be two or more parents.
        *   Your **author** name, email, and timestamp (taken from your Git configuration).
        *   Your **committer** name, email, and timestamp (usually the same as author, but can differ in some scenarios like patching).
        *   The **commit message** you provided (e.g., with `-m "..."`).
    *   Git then concatenates all this information into a string, computes its SHA-1 hash, and stores this new **commit object** in the `.git/objects/` directory.

    This new commit object is now part of your repository's object database.

3.  **Step 3: Update `HEAD` and the Current Branch Pointer**
    *   This is the final, crucial step that advances your project's history.
    *   Git updates the reference that your current branch points to (e.g., `refs/heads/main`). This reference now points to the SHA-1 of the *newly created commit object*.
    *   Since `HEAD` typically points to your current branch (e.g., `ref: refs/heads/main`), this effectively moves `HEAD` forward to the new commit.

4.  **Step 4: Clean Up the Staging Area**
    *   The staging area (index) is updated to reflect the state of the new commit. It is now "clean" and matches the root tree of the latest commit. This is why, after a `git commit`, `git status` typically reports "nothing to commit, working tree clean."

## Why This Matters: The Power of Git's Design

Understanding these internals reveals the elegance and robustness of Git:

*   **Immutability and Integrity**: Every object is content-addressed via its SHA-1 hash. If even a single bit of a file's content (a blob) changes, its SHA-1 changes, creating a new object. This propagates up to tree objects and then to commit objects. This chain of hashes makes it incredibly difficult (computationally infeasible) to silently corrupt Git's history without detection.
*   **Snapshots, Not Diffs**: Git primarily stores full snapshots (via tree objects linked to blobs) with each commit, rather than just storing diffs. This makes checking out any historical version lightning fast, as Git just needs to retrieve the relevant tree and blob objects, rather than applying a sequence of patches. Diffs are *computed* on the fly between snapshots.
*   **Efficiency**: Despite storing snapshots, Git is remarkably efficient. If a file's content hasn't changed between commits, the same blob object is reused, saving space. Furthermore, Git uses "packfiles" (`.git/objects/pack/`) to compress objects, especially by identifying and storing deltas (differences) between similar objects, which happens in the background via `git gc`.
*   **Branching and Merging as Pointers**: Because commits are simply objects pointing to parents and trees, branches are just lightweight pointers to commit objects. Creating a branch is just creating a new pointer. Merging involves creating a new commit object with multiple parents, linking divergent histories together.

## Exploring Yourself

You can use a few commands to peek into these objects in your own repository:

*   `git cat-file -t <SHA>`: Shows the type of a Git object (blob, tree, commit, tag).
*   `git cat-file -p <SHA>`: Prints the content of a Git object.
*   `git rev-parse HEAD`: Shows the SHA-1 of your current commit.
*   `git ls-tree HEAD^{tree}`: Shows the root tree of your current commit.
*   `ls -F .git/objects`: See the hashed directories where objects are stored (often compressed into packfiles).

## Conclusion

The humble `git commit` command is a powerful orchestration of Git's internal object model. It transforms your staged changes into immutable blob and tree objects, encapsulates them within a new commit object, and then updates your branch pointer to mark this new point in history.

Understanding this process not only satisfies intellectual curiosity but also provides a robust mental model for how Git operates. It helps you grasp why Git behaves the way it does during branching, merging, resetting, and even when data corruption is detected. So, the next time you type `git commit`, remember the intricate dance of hashes and pointers happening behind the scenes, diligently preserving your project's journey.