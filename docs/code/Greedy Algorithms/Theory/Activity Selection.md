---
title: Activity Selection - A Greedy Approach to Optimal Scheduling
date: 2025-06-18T02:04:08.681Z
description: Master the Activity Selection problem, a classic greedy algorithm for maximizing non-overlapping tasks. Understand its logic with practical Python code examples for optimal resource scheduling.
tags:
  - algorithms
  - greedy
  - python
  - data-structures
  - scheduling
  - optimization
categories:
  - Algorithms
  - Programming
comments: true
---

Every developer, at some point, faces resource allocation challenges. Whether it's scheduling builds on a CI/CD runner, allocating meeting rooms, or optimizing CPU task execution, the core problem is often about selecting the maximum number of compatible tasks from a given set. This is precisely what the "Activity Selection" problem addresses.

It's a foundational problem in computer science, particularly within the realm of greedy algorithms. Unlike complex dynamic programming solutions, Activity Selection elegantly demonstrates how a simple, locally optimal choice can lead to a globally optimal solution.

Let's break it down.

### What is the Activity Selection Problem?

Imagine you have a single resource (e.g., a meeting room, a CPU, yourself!) and a list of activities that need to use that resource. Each activity has a defined start time and finish time. Your goal is to select the largest possible number of activities that can be performed by the single resource, with the constraint that no two selected activities can overlap in time.

**Key Characteristics:**

*   **Single Resource:** Only one task can run at a time.
*   **Start and Finish Times:** Each activity `i` is defined by `(s_i, f_i)`, where `s_i` is its start time and `f_i` is its finish time.
*   **Non-overlapping:** If you select activity `i` and activity `j`, they must not overlap. This means either `f_i <= s_j` (activity `i` finishes before `j` starts) or `f_j <= s_i` (activity `j` finishes before `i` starts).
*   **Goal:** Maximize the total number of selected activities.

### The Greedy Insight: Why It Works

The brilliance of the Activity Selection problem lies in its simple, yet effective, greedy strategy. A greedy algorithm makes the locally optimal choice at each step, hoping this will lead to a globally optimal solution. For Activity Selection, this choice is:

**Always select the activity that finishes earliest among the remaining compatible activities.**

Why does this work?

Consider activities sorted by their finish times. If we pick the activity that finishes earliest, we free up the shared resource as quickly as possible. This leaves the maximum amount of time remaining for subsequent activities, thus maximizing the opportunities to schedule more tasks. Any other choice (e.g., picking an activity that starts earliest or is shortest in duration but finishes later) would potentially block more future activities.

**Note:** This problem is a perfect fit for a greedy approach because it exhibits the "greedy choice property" and "optimal substructure." The greedy choice (picking the earliest finishing activity) leaves an optimal subproblem, and combining these optimal choices yields an overall optimal solution.

### The Algorithm Steps

1.  **Sort Activities:** Sort all given activities in non-decreasing order of their **finish times**. If two activities have the same finish time, their relative order doesn't strictly matter for correctness, but sorting by start time as a secondary key can ensure deterministic output.
2.  **Select First Activity:** Choose the first activity from the sorted list. This activity, by definition, has the earliest finish time.
3.  **Iterate and Select:** Iterate through the remaining activities. For each activity, check if its start time is greater than or equal to the finish time of the **last selected activity**. If it is, select this current activity and update the "last selected activity's finish time" to its finish time. If not, skip it.

Let's see this in action with some Python code.

### Python Implementation: Generic Time Slots

We'll start with a simple example where activities are just defined by their start and finish times. To make the code clear, we'll use a `namedtuple` to represent each activity.

```python
import collections

# Define a named tuple for clarity in representing activities
Activity = collections.namedtuple('Activity', ['name', 'start', 'finish'])

def select_activities(activities_list):
    """
    Selects the maximum number of non-overlapping activities from a given list.

    Args:
        activities_list: A list of Activity named tuples, where each tuple
                         has (name, start_time, finish_time).

    Returns:
        A list of selected Activity named tuples.
    """
    if not activities_list:
        return []

    # Step 1: Sort activities by their finish times.
    # If finish times are equal, sort by start times for consistent output,
    # though it doesn't affect correctness for this problem.
    sorted_activities = sorted(activities_list, key=lambda x: (x.finish, x.start))

    # Step 2: Initialize the selected activities list with the first activity
    # (which has the earliest finish time among all activities).
    selected_activities = [sorted_activities[0]]
    last_finish_time = sorted_activities[0].finish

    # Step 3: Iterate through the remaining activities.
    for i in range(1, len(sorted_activities)):
        current_activity = sorted_activities[i]
        # If the current activity's start time is greater than or equal to
        # the last selected activity's finish time, select it.
        if current_activity.start >= last_finish_time:
            selected_activities.append(current_activity)
            last_finish_time = current_activity.finish

    return selected_activities

# --- Example 1: Generic Time Slots ---
print("--- Example 1: Generic Time Slots ---")
generic_activities = [
    Activity("Task A", 1, 2),
    Activity("Task B", 3, 4),
    Activity("Task C", 0, 6), # This one finishes later, despite starting early
    Activity("Task D", 5, 7),
    Activity("Task E", 8, 9),
    Activity("Task F", 5, 9)  # This one starts at the same time as D, but finishes later
]

print("Original activities (name, start, finish):")
for a in generic_activities:
    print(f"  {a.name}: ({a.start}, {a.finish})")

selected_generic = select_activities(generic_activities)

print("\nSelected activities (optimal schedule):")
for activity in selected_generic:
    print(f"  {activity.name}: ({activity.start}, {activity.finish})")

```

#### Output for Example 1:

```output
--- Example 1: Generic Time Slots ---
Original activities (name, start, finish):
  Task A: (1, 2)
  Task B: (3, 4)
  Task C: (0, 6)
  Task D: (5, 7)
  Task E: (8, 9)
  Task F: (5, 9)

Selected activities (optimal schedule):
  Task A: (1, 2)
  Task B: (3, 4)
  Task D: (5, 7)
  Task E: (8, 9)
```

**Walkthrough of Example 1:**

1.  **Original Activities:**
    `Task A: (1, 2)`
    `Task B: (3, 4)`
    `Task C: (0, 6)`
    `Task D: (5, 7)`
    `Task E: (8, 9)`
    `Task F: (5, 9)`

2.  **Sorted by Finish Time:**
    `Task A: (1, 2)`
    `Task B: (3, 4)`
    `Task C: (0, 6)`
    `Task D: (5, 7)`
    `Task E: (8, 9)`
    `Task F: (5, 9)` (Note: `Task E` and `Task F` have same finish, `Task E` comes first due to earlier start.)

3.  **Selection Process:**
    *   Initialize `selected_activities = []`, `last_finish_time = -infinity`.
    *   Pick `Task A (1, 2)`: It's the first in the sorted list. `selected_activities = [Task A]`, `last_finish_time = 2`.
    *   Consider `Task B (3, 4)`: `3 >= 2`. Select `Task B`. `selected_activities = [Task A, Task B]`, `last_finish_time = 4`.
    *   Consider `Task C (0, 6)`: `0 < 4`. Skip `Task C`.
    *   Consider `Task D (5, 7)`: `5 >= 4`. Select `Task D`. `selected_activities = [Task A, Task B, Task D]`, `last_finish_time = 7`.
    *   Consider `Task E (8, 9)`: `8 >= 7`. Select `Task E`. `selected_activities = [Task A, Task B, Task D, Task E]`, `last_finish_time = 9`.
    *   Consider `Task F (5, 9)`: `5 < 9`. Skip `Task F`.

4.  **Final Selected:** `Task A, Task B, Task D, Task E`. This yields 4 activities, which is the maximum possible.

### Python Implementation: Meeting Room Scheduling

Let's apply the same logic to a more relatable scenario: scheduling meetings in a single conference room.

```python
import collections

Activity = collections.namedtuple('Activity', ['name', 'start', 'finish'])

def select_activities(activities_list):
    """
    Selects the maximum number of non-overlapping activities from a given list.
    (Same function as above, included for completeness)
    """
    if not activities_list:
        return []

    sorted_activities = sorted(activities_list, key=lambda x: (x.finish, x.start))

    selected_activities = [sorted_activities[0]]
    last_finish_time = sorted_activities[0].finish

    for i in range(1, len(sorted_activities)):
        current_activity = sorted_activities[i]
        if current_activity.start >= last_finish_time:
            selected_activities.append(current_activity)
            last_finish_time = current_activity.finish

    return selected_activities

# --- Example 2: Meeting Room Scheduling ---
print("\n--- Example 2: Meeting Room Scheduling ---")
meeting_activities = [
    Activity("Meeting Zeta", 9, 11),
    Activity("Meeting Alpha", 10, 20),
    Activity("Meeting Beta", 12, 25),
    Activity("Meeting Gamma", 20, 30),
    Activity("Meeting Delta", 15, 22),
    Activity("Meeting Epsilon", 28, 35)
]

print("Original meetings (name, start, finish):")
for m in meeting_activities:
    print(f"  {m.name}: ({m.start}, {m.finish})")

selected_meetings = select_activities(meeting_activities)

print("\nSelected meetings (optimal room schedule):")
for meeting in selected_meetings:
    print(f"  {meeting.name}: ({meeting.start}, {meeting.finish})")

```

#### Output for Example 2:

```output
--- Example 2: Meeting Room Scheduling ---
Original meetings (name, start, finish):
  Meeting Zeta: (9, 11)
  Meeting Alpha: (10, 20)
  Meeting Beta: (12, 25)
  Meeting Gamma: (20, 30)
  Meeting Delta: (15, 22)
  Meeting Epsilon: (28, 35)

Selected meetings (optimal room schedule):
  Meeting Zeta: (9, 11)
  Meeting Delta: (15, 22)
  Meeting Epsilon: (28, 35)
```

### Time and Space Complexity

*   **Time Complexity:**
    *   Sorting the activities dominates the runtime: `O(N log N)`, where `N` is the number of activities.
    *   The iteration through the sorted activities is `O(N)`.
    *   Therefore, the overall time complexity is **`O(N log N)`**.

*   **Space Complexity:**
    *   Storing the sorted activities: `O(N)`.
    *   Storing the selected activities: `O(N)` in the worst case (if all activities are selected).
    *   Therefore, the overall space complexity is **`O(N)`**.

### When to Use (and When Not To)

The Activity Selection problem is a perfect example of where a greedy algorithm provides an optimal solution. It's highly efficient and straightforward to implement.

**Use it when:**

*   You have a single resource.
*   Activities have defined start and finish times.
*   The goal is to maximize the *number* of non-overlapping activities.
*   All activities are considered equally "valuable" (i.e., there are no weights or priorities associated with them).

**Do NOT use it (or modify it significantly) when:**

*   **Multiple Resources:** If you have multiple meeting rooms or CPU cores, this simple greedy approach won't work directly. You'd likely need more complex scheduling algorithms (e.g., minimum number of rooms needed for all meetings, which is a different problem solvable with a slightly modified greedy approach).
*   **Weighted Activities:** If activities have different "values" or "priorities" (e.g., a "critical bug fix" activity is more important than a "code refactor" activity), and you want to maximize the *total value* of selected activities, this greedy approach is insufficient. That variation usually requires Dynamic Programming.
*   **Dependencies:** If activities have dependencies (e.g., "build" must complete before "deploy" can start), the simple greedy choice might not respect these constraints. You'd need a topological sort or other dependency-aware scheduling.

### Conclusion

The Activity Selection problem is a clean, practical demonstration of how a well-chosen greedy strategy can lead to an optimal solution with excellent performance. It's a fundamental algorithm for anyone dealing with scheduling and resource allocation, highlighting the power of simplifying a complex problem into a series of locally optimal, easy-to-make choices. Keep it in your algorithmic toolkit for those single-resource optimization scenarios!
