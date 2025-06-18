---
title: Introduction To Linked List
date: 2025-06-18T02:04:08.681Z
description: A pragmatic introduction to linked lists, explaining their structure, common operations, and practical use cases with clear Python examples. Understand why this fundamental data structure is vital for any serious developer.
tags: [DataStructures, Algorithms, LinkedList, Python, ComputerScience, Fundamentals]
categories: [Programming, Fundamentals]
comments: true
---

Every serious developer, sooner or later, encounters the bedrock of computer science: data structures. Among them, the Linked List holds a peculiar, yet powerful, position. Unlike arrays, it doesn't offer the simple, contiguous memory access we often take for granted. Instead, it provides flexibility, dynamism, and a different way of thinking about connected data.

This post will cut through the academic fluff and give you a solid, example-driven understanding of linked lists. We'll build one from scratch in Python, explore its core operations, and finally, discuss where and why you'd actually use one.

### What is a Linked List?

Imagine a treasure hunt where each clue tells you where to find the *next* clue, rather than giving you a map of all clues at once. That's essentially a linked list.

At its core, a linked list is a **linear collection of data elements where each element points to the next**. Each element is called a **node**.

Unlike an array, which stores elements in contiguous memory locations and allows direct access by index, a linked list elements can be scattered anywhere in memory. The "link" is what binds them together logically.

### Why Not Just Use Arrays?

Arrays are fantastic for many tasks:
*   **Random Access**: You can directly jump to any element using its index (e.g., `my_array[5]`). This is O(1) time complexity.
*   **Cache Locality**: Elements are stored together, often leading to better performance due to CPU caches.

However, arrays have limitations:
*   **Fixed Size**: Most traditional arrays have a fixed size defined at creation. Resizing an array often means creating a new, larger array and copying all elements over, which is an O(N) operation.
*   **Costly Insertions/Deletions**: Inserting an element in the middle, or deleting one, requires shifting all subsequent elements. This is also an O(N) operation.

Linked lists shine where arrays falter: dynamic size and efficient insertions/deletions in the middle, albeit with a trade-off in random access.

### The Building Block: The Node

Before we build a linked list, we need its fundamental component: the `Node`. Each node typically contains two parts:

1.  **Data**: The actual value or information stored in the node.
2.  **Next Pointer (or Reference)**: A reference (or memory address) to the next node in the sequence. The last node's `next` pointer will typically be `None` (or `null` in other languages), signifying the end of the list.

Let's define a simple `Node` class in Python:

```python
class Node:
    def __init__(self, data):
        self.data = data  # The data the node holds
        self.next = None  # Reference to the next node in the list

# Example of creating a node
node1 = Node(10)
print(f"Node data: {node1.data}")
print(f"Node next pointer: {node1.next}")
```

<br/>

```output
Node data: 10
Node next pointer: None
```

In the example, `node1` holds the data `10`, and its `next` pointer is `None`, meaning it doesn't point to any other node yet.

### The Linked List Class

Now that we have our `Node`, we can create the `LinkedList` itself. The `LinkedList` class doesn't hold all the nodes directly; instead, it holds a reference to the **head** of the list. The `head` is simply the first node. From the head, you can traverse the entire list by following the `next` pointers.

If the list is empty, the `head` will be `None`.

```python
class LinkedList:
    def __init__(self):
        self.head = None  # Initialize the head of the list to None

# Example of creating an empty linked list
my_list = LinkedList()
print(f"Is the list empty? {my_list.head is None}")
```

<br/>

```output
Is the list empty? True
```

### Common Operations on a Linked List

Let's add some core functionalities to our `LinkedList` class.

#### 1. Traversal: Printing the List

The most basic operation is to traverse the list and print its contents. This involves starting from the `head` and following each node's `next` pointer until we reach `None`.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def print_list(self):
        current_node = self.head
        if not current_node:
            print("List is empty.")
            return

        elements = []
        while current_node:
            elements.append(str(current_node.data))
            current_node = current_node.next
        print(" -> ".join(elements))

# Demonstrate printing an empty list
ll = LinkedList()
ll.print_list()

# Demonstrate printing a list with one node (we'll add more later)
node_a = Node("A")
ll.head = node_a
ll.print_list()
```

<br/>

```output
List is empty.
A
```

Note: I've added the `print_list` method to our `LinkedList` class for easy demonstration of subsequent operations. For now, we've manually set the head, but we'll create proper insertion methods next.

#### 2. Insertion Operations

Inserting elements is a key strength of linked lists.

##### a. `prepend()`: Adding a Node to the Beginning

Prepending (or `add_first`) involves creating a new node, making its `next` pointer point to the current `head`, and then updating the list's `head` to be the new node. This is an O(1) operation.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def print_list(self):
        current_node = self.head
        if not current_node:
            print("List is empty.")
            return
        elements = []
        while current_node:
            elements.append(str(current_node.data))
            current_node = current_node.next
        print(" -> ".join(elements))

    def prepend(self, data):
        new_node = Node(data)
        new_node.next = self.head # New node points to the current head
        self.head = new_node      # New node becomes the new head

# Demonstrate prepend
ll = LinkedList()
print("After initialization:")
ll.print_list()

ll.prepend("C")
print("After prepending 'C':")
ll.print_list()

ll.prepend("B")
print("After prepending 'B':")
ll.print_list()

ll.prepend("A")
print("After prepending 'A':")
ll.print_list()
```

<br/>

```output
After initialization:
List is empty.
After prepending 'C':
C
After prepending 'B':
B -> C
After prepending 'A':
A -> B -> C
```

##### b. `append()`: Adding a Node to the End

Appending (or `add_last`) involves creating a new node, then traversing the list to find the last node (the one whose `next` pointer is `None`). Once found, set the last node's `next` pointer to the new node. If the list is empty, the new node simply becomes the `head`. This is an O(N) operation because we might have to traverse the entire list.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def print_list(self):
        current_node = self.head
        if not current_node:
            print("List is empty.")
            return
        elements = []
        while current_node:
            elements.append(str(current_node.data))
            current_node = current_node.next
        print(" -> ".join(elements))

    def append(self, data):
        new_node = Node(data)
        if self.head is None:
            self.head = new_node # If list is empty, new node becomes head
            return

        last_node = self.head
        while last_node.next: # Traverse until we find the last node
            last_node = last_node.next
        last_node.next = new_node # The last node now points to the new node

# Demonstrate append
ll = LinkedList()
print("Initial list:")
ll.print_list()

ll.append("1")
print("After appending '1':")
ll.print_list()

ll.append("2")
print("After appending '2':")
ll.print_list()

ll.append("3")
print("After appending '3':")
ll.print_list()
```

<br/>

```output
Initial list:
List is empty.
After appending '1':
1
After appending '2':
1 -> 2
After appending '3':
1 -> 2 -> 3
```

##### c. `insert_after_node()`: Adding a Node After a Specific Node

This operation involves finding a target node (e.g., by its value or by direct reference) and then inserting the new node immediately after it.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def print_list(self):
        current_node = self.head
        if not current_node:
            print("List is empty.")
            return
        elements = []
        while current_node:
            elements.append(str(current_node.data))
            current_node = current_node.next
        print(" -> ".join(elements))

    def append(self, data): # Re-using append for setup
        new_node = Node(data)
        if self.head is None:
            self.head = new_node
            return
        last_node = self.head
        while last_node.next:
            last_node = last_node.next
        last_node.next = new_node

    def insert_after_node(self, prev_node_data, new_data):
        if not self.head:
            print(f"Cannot insert. List is empty. '{prev_node_data}' not found.")
            return

        current_node = self.head
        found_prev_node = None
        while current_node:
            if current_node.data == prev_node_data:
                found_prev_node = current_node
                break
            current_node = current_node.next

        if found_prev_node is None:
            print(f"Node with data '{prev_node_data}' not found in the list.")
            return

        new_node = Node(new_data)
        new_node.next = found_prev_node.next # New node points to what the previous node pointed to
        found_prev_node.next = new_node      # Previous node now points to the new node

# Demonstrate insert_after_node
ll = LinkedList()
ll.append("Banana")
ll.append("Grape")
ll.append("Orange")
print("Original list:")
ll.print_list()

ll.insert_after_node("Banana", "Apple")
print("After inserting 'Apple' after 'Banana':")
ll.print_list()

ll.insert_after_node("Orange", "Strawberry")
print("After inserting 'Strawberry' after 'Orange':")
ll.print_list()

ll.insert_after_node("Mango", "Pineapple") # Node not found case
print("Attempting to insert after 'Mango' (not found):")
ll.print_list()
```

<br/>

```output
Original list:
Banana -> Grape -> Orange
After inserting 'Apple' after 'Banana':
Banana -> Apple -> Grape -> Orange
After inserting 'Strawberry' after 'Orange':
Banana -> Apple -> Grape -> Orange -> Strawberry
Node with data 'Mango' not found in the list.
Attempting to insert after 'Mango' (not found):
Banana -> Apple -> Grape -> Orange -> Strawberry
```

#### 3. Deletion Operations

Deleting elements also leverages the `next` pointers.

##### a. `delete_by_value()`: Removing a Node by its Value

To delete a node by its value, we need to:
1.  Find the node to be deleted and, importantly, its **predecessor** node.
2.  Once found, we simply make the predecessor node's `next` pointer skip over the target node, effectively unlinking it from the list.
3.  Special handling is needed if the `head` node itself is the one to be deleted.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def print_list(self):
        current_node = self.head
        if not current_node:
            print("List is empty.")
            return
        elements = []
        while current_node:
            elements.append(str(current_node.data))
            current_node = current_node.next
        print(" -> ".join(elements))

    def append(self, data): # Re-using append for setup
        new_node = Node(data)
        if self.head is None:
            self.head = new_node
            return
        last_node = self.head
        while last_node.next:
            last_node = last_node.next
        last_node.next = new_node

    def delete_by_value(self, key):
        current_node = self.head

        # Case 1: Head node itself holds the key to be deleted
        if current_node and current_node.data == key:
            self.head = current_node.next # Move head to the next node
            current_node = None # Dereference the old head
            return

        # Case 2: Node to be deleted is not the head
        prev_node = None
        while current_node and current_node.data != key:
            prev_node = current_node
            current_node = current_node.next

        # If key was not present in linked list
        if current_node is None:
            print(f"Value '{key}' not found in the list.")
            return

        # Unlink the node from the list
        prev_node.next = current_node.next
        current_node = None # Dereference the deleted node

# Demonstrate delete_by_value
ll = LinkedList()
ll.append(10)
ll.append(20)
ll.append(30)
ll.append(40)
ll.append(50)
print("Original list:")
ll.print_list()

ll.delete_by_value(30)
print("After deleting 30:")
ll.print_list()

ll.delete_by_value(10) # Delete head
print("After deleting 10 (head):")
ll.print_list()

ll.delete_by_value(50) # Delete tail
print("After deleting 50 (tail):")
ll.print_list()

ll.delete_by_value(100) # Not found
print("Attempting to delete 100 (not found):")
ll.print_list()

ll.delete_by_value(20) # Only 40 left
print("After deleting 20:")
ll.print_list()

ll.delete_by_value(40) # Last element
print("After deleting 40 (last element):")
ll.print_list()

ll.delete_by_value(5) # Empty list
print("Attempting to delete from empty list:")
ll.print_list()
```

<br/>

```output
Original list:
10 -> 20 -> 30 -> 40 -> 50
After deleting 30:
10 -> 20 -> 40 -> 50
After deleting 10 (head):
20 -> 40 -> 50
After deleting 50 (tail):
20 -> 40
Value '100' not found in the list.
Attempting to delete 100 (not found):
20 -> 40
After deleting 20:
40
After deleting 40 (last element):
List is empty.
Value '5' not found in the list.
Attempting to delete from empty list:
List is empty.
```

#### 4. Search: Finding a Node

Searching for a node involves traversing the list from the `head` until the desired data is found or the end of the list (`None`) is reached.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def append(self, data): # For setup
        new_node = Node(data)
        if self.head is None:
            self.head = new_node
            return
        last_node = self.head
        while last_node.next:
            last_node = last_node.next
        last_node.next = new_node

    def search(self, key):
        current_node = self.head
        while current_node:
            if current_node.data == key:
                return True # Found the key
            current_node = current_node.next
        return False # Key not found

# Demonstrate search
ll = LinkedList()
ll.append("Alice")
ll.append("Bob")
ll.append("Charlie")

print(f"Is 'Bob' in the list? {ll.search('Bob')}")
print(f"Is 'David' in the list? {ll.search('David')}")
print(f"Is 'Alice' in the list? {ll.search('Alice')}")

empty_ll = LinkedList()
print(f"Is 'X' in an empty list? {empty_ll.search('X')}")
```

<br/>

```output
Is 'Bob' in the list? True
Is 'David' in the list? False
Is 'Alice' in the list? True
Is 'X' in an empty list? False
```

#### 5. Get Length: Counting Nodes

To find the number of nodes, we simply traverse the list and count each node until the end.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def append(self, data): # For setup
        new_node = Node(data)
        if self.head is None:
            self.head = new_node
            return
        last_node = self.head
        while last_node.next:
            last_node = last_node.next
        last_node.next = new_node

    def get_length(self):
        count = 0
        current_node = self.head
        while current_node:
            count += 1
            current_node = current_node.next
        return count

# Demonstrate get_length
ll = LinkedList()
print(f"Length of empty list: {ll.get_length()}")

ll.append("One")
print(f"Length after adding 'One': {ll.get_length()}")

ll.append("Two")
ll.append("Three")
print(f"Length after adding 'Two' and 'Three': {ll.get_length()}")
```

<br/>

```output
Length of empty list: 0
Length after adding 'One': 1
Length after adding 'Two' and 'Three': 3
```

### Advantages and Disadvantages of Linked Lists

Understanding the trade-offs is crucial for choosing the right data structure.

#### Advantages:
*   **Dynamic Size**: Linked lists can grow or shrink as needed, allocating memory for nodes only when required. No need to pre-allocate a fixed size.
*   **Efficient Insertions and Deletions**: Adding or removing a node (once the insertion/deletion point is found) is an O(1) operation. This is especially beneficial in the middle of a large list, where arrays would require shifting many elements.
*   **Flexible Memory**: Nodes can be stored non-contiguously in memory, making them suitable for systems with fragmented memory.

#### Disadvantages:
*   **No Random Access**: To access the Nth element, you must traverse from the `head` up to that element. This makes indexed access an O(N) operation, unlike arrays' O(1).
*   **More Memory Overhead**: Each node requires extra memory to store the `next` pointer (and potentially a `prev` pointer in a doubly linked list) in addition to the data itself.
*   **Poor Cache Performance**: Due to non-contiguous memory allocation, linked lists can exhibit poor cache performance. When traversing, the CPU might need to fetch data from different memory locations frequently, leading to cache misses.

### Real-World Use Cases

Despite their limitations (especially random access), linked lists are fundamental and used in many real-world scenarios:

*   **Implementing Stacks and Queues**: Both abstract data types can be efficiently implemented using linked lists. `prepend` serves as `push` for a stack, and a combination of `append` and `delete_head` (similar to `pop` in a queue) works for queues.
*   **Image Viewers/Music Playlists**: "Next" and "Previous" functionality maps perfectly to traversing a linked list.
*   **Browser History**: The "back" and "forward" buttons in a web browser can be modeled using a doubly linked list.
*   **Memory Management**: Free lists in operating systems and memory allocators often use linked lists to keep track of available memory blocks.
*   **Undo/Redo Functionality**: Many applications implement undo/redo features using linked lists, where each action is a node.
*   **Hash Table Collision Handling**: Chaining, a common method to handle collisions in hash tables, often uses linked lists to store multiple elements that hash to the same bucket.

Note: While Python's built-in lists are dynamic arrays, for conceptual understanding and for situations where true linked list behavior (like efficient middle insertions/deletions at scale) is critical, implementing or understanding linked lists is invaluable. Languages like C/C++ often see more direct use of manual linked list implementations.

### Conclusion

Linked lists are a cornerstone of computer science, offering a powerful alternative to arrays for managing dynamic collections of data. While they sacrifice random access, their efficiency in insertion and deletion operations makes them indispensable for specific problems.

By building a linked list from the ground up, you gain not just theoretical knowledge but practical insight into how data can be organized and manipulated without the constraints of contiguous memory. Keep experimenting, understand the trade-offs, and you'll be well-equipped to choose the right tool for the job.

Happy coding!
