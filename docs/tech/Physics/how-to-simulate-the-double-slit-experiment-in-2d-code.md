---
categories:
- science
- programming
- simulation
comments: true
cover:
  image: https://images.pexels.com/photos/29006120/pexels-photo-29006120.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 14:26:03.015000
description: Learn how to simulate the classic double-slit experiment in 2D using
  Python, NumPy, and Matplotlib. Understand wave interference through a practical,
  hands-on coding example.
math: true
tags:
- physics
- quantum
- simulation
- python
- numpy
- matplotlib
- visualization
- education
- wave-mechanics
title: How to Simulate the Double-Slit Experiment in 2D Code
---

![Macro shot of oil and water bubbles with a green background, showcasing abstract patterns.](https://images.pexels.com/photos/29006120/pexels-photo-29006120.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Macro shot of oil and water bubbles with a green background, showcasing abstract patterns.")

## How to Simulate the Double-Slit Experiment in 2D Code

The double-slit experiment is one of the most iconic and mind-bending experiments in physics, famously demonstrating the wave-particle duality of light and matter. While the quantum mechanical interpretation involves probabilities and particle collapse, the fundamental interference pattern observed can be elegantly simulated using classical wave theory.

In this blog post, we'll strip away the quantum mystery for a moment and focus on building a pragmatic 2D simulation of the double-slit experiment using Python. Our goal is to visualize the beautiful interference patterns that arise from the superposition of waves.

## The Physics in a Nutshell (Classical Wave Perspective)

Before we dive into code, let's briefly touch upon the core classical wave concepts essential for our simulation:

*   **Waves:** A wave is a disturbance that propagates through space and time, transferring energy without transferring matter. Key properties include wavelength (`λ`), amplitude, and phase.
*   **Huygens' Principle:** This principle states that every point on a wavefront can be considered as a source of secondary spherical wavelets. The new wavefront is the envelope of these wavelets. In our simplified model, the slits themselves will act as such secondary point sources.
*   **Superposition Principle:** When two or more waves overlap in space, the net displacement at any point and time is the vector sum of the displacements of the individual waves.
    *   **Constructive Interference:** Occurs when waves align peak-to-peak or trough-to-trough, resulting in a larger amplitude.
    *   **Destructive Interference:** Occurs when a peak of one wave aligns with a trough of another, resulting in a smaller (or zero) amplitude.

**Note:** For this simulation, we are modeling light as a classical wave. We are *not* simulating the quantum mechanical phenomenon of individual particles behaving like waves or collapsing their wave function upon observation. Our focus is purely on demonstrating the classical wave interference pattern that is visually indistinguishable from the quantum outcome in terms of the statistical distribution. We assume a coherent light source (like a laser) where waves from the two slits maintain a constant phase relationship.

## Setting Up Your Environment

You'll need Python (version 3.7+) and a couple of essential libraries: NumPy for numerical operations and Matplotlib for plotting.

If you don't have them installed, open your terminal or command prompt and run:

```bash
pip install numpy matplotlib
```

## Core Concept: Superposition of Waves

Our simulation will represent the 2D space as a grid (a NumPy array). Each point in this grid will have a calculated wave amplitude based on its distance from the wave sources (the slits).

### Representing a Single Circular Wave

A simple way to represent a wave from a point source is using a cosine function, where the wave's phase depends on the distance from the source. The formula for the amplitude `A` at a distance `r` from a source, given a wavelength `λ`, can be simplified to:

`Amplitude = cos(k * r)`

Where `k` is the wave number, `k = 2 * π / λ`. We omit the time component and initial phase for simplicity, as we're interested in the *steady-state* interference pattern.

Let's see how to generate a single circular wave on a 2D grid:

```python
import numpy as np
import matplotlib.pyplot as plt

def generate_circular_wave(grid_size, wavelength, source_pos):
    """
    Generates a 2D representation of a circular wave originating from source_pos.
    Amplitude is based on a simple cosine function.

    Args:
        grid_size (tuple): (rows, cols) defining the simulation area.
        wavelength (float): The wavelength of the wave.
        source_pos (tuple): (row_index, col_index) of the wave source.

    Returns:
        numpy.ndarray: A 2D array representing the wave's amplitude field.
    """
    rows, cols = grid_size
    k = 2 * np.pi / wavelength # wave number
    
    # Create a grid of (x, y) coordinates
    # X corresponds to columns, Y to rows
    x = np.arange(cols)
    y = np.arange(rows)
    X, Y = np.meshgrid(x, y)
    
    # Calculate distance from source for each point
    # source_pos is (row, col) so map it correctly to X, Y
    R = np.sqrt((X - source_pos[1])**2 + (Y - source_pos[0])**2)
    
    # Calculate wave amplitude using cosine function
    # For a realistic wave, amplitude might also decrease with distance (e.g., 1/R),
    # but for interference patterns, phase (determined by k*R) is the key.
    wave = np.cos(k * R)
    return wave

# --- Example Usage: Save this as single_wave.py ---
if __name__ == '__main__':
    grid_dim = (500, 700) # (rows, cols) -> (height, width)
    lam = 20 # Wavelength in grid units
    source = (grid_dim[0] // 2, grid_dim[1] // 4) # Source slightly left of center
    
    single_wave = generate_circular_wave(grid_dim, lam, source)
    
    plt.figure(figsize=(10, 6))
    plt.imshow(single_wave, cmap='viridis', origin='upper', 
               extent=[0, grid_dim[1], 0, grid_dim[0]])
    plt.colorbar(label='Wave Amplitude')
    plt.title(f'Single Circular Wave (Wavelength={lam})')
    plt.xlabel('X (Grid Units)')
    plt.ylabel('Y (Grid Units)')
    plt.axvline(source[1], color='r', linestyle='--', label='Source X')
    plt.axhline(source[0], color='r', linestyle=':', label='Source Y')
    plt.scatter(source[1], source[0], color='red', marker='*', s=200, label='Source')
    plt.legend()
    plt.tight_layout()
    plt.show()

```

To run this example:

```bash
python single_wave.py
```

```output
(A Matplotlib window will pop up.)

Expected Output:
A 2D plot showing a series of concentric circles radiating outwards from the red '*' marker representing the source. The colors (e.g., green to yellow with 'viridis' colormap) will represent the varying amplitude (from -1 to 1) of the wave, showing clear peaks and troughs as rings. The red dashed lines will mark the source's X and Y coordinates.
```

This simple script visualizes how a wave propagates from a single point. The core idea for the double-slit simulation is to do this twice, once for each slit, and then combine the results.

## Building the Double-Slit Simulation

Now, let's extend the single-wave concept to two slits. In our model, we'll assume the slits act as two separate, coherent point sources. This is a common simplification that effectively demonstrates the interference pattern.

Here's the plan:

1.  **Define Parameters:** Set up grid dimensions, wavelength, and slit properties.
2.  **Calculate Slit Positions:** Determine the `(row, col)` coordinates for each of the two slits.
3.  **Generate Waves from Each Slit:** Use our `generate_circular_wave` logic (or integrate it) to calculate the wave field originating from each slit.
4.  **Superimpose Waves:** Add the two wave fields together.
5.  **Calculate Intensity:** The observed brightness (intensity) of light is proportional to the square of the wave's amplitude. This ensures all values are positive and emphasizes the peaks.

### Full Script and Visualization

Here's the complete Python script for the double-slit simulation:

```python
import numpy as np
import matplotlib.pyplot as plt

def simulate_double_slit(grid_dim, wavelength, slit_separation, slit_x_pos):
    """
    Simulates the double-slit experiment using superposition of two circular waves.

    Args:
        grid_dim (tuple): (rows, cols) defining the simulation area.
        wavelength (float): The wavelength of the waves.
        slit_separation (float): The vertical distance between the two slits.
        slit_x_pos (float): The horizontal position (column index) of the slits.

    Returns:
        tuple: A tuple containing:
               - numpy.ndarray: The calculated intensity pattern (2D array).
               - tuple: (slit1_pos, slit2_pos) - coordinates of the slits.
    """
    rows, cols = grid_dim
    k = 2 * np.pi / wavelength # Wave number

    # Create a grid of (x, y) coordinates for calculation
    # X represents column indices, Y represents row indices
    x = np.arange(cols)
    y = np.arange(rows)
    X, Y = np.meshgrid(x, y)

    # Determine slit positions
    # Slits are vertically centered with respect to the grid height (rows)
    slit1_y = rows / 2 - slit_separation / 2
    slit2_y = rows / 2 + slit_separation / 2

    slit1_pos = (slit1_y, slit_x_pos) # (row_index, col_index)
    slit2_pos = (slit2_y, slit_x_pos)

    # Calculate distance from each slit to every point in the grid
    # Remember X is for column, Y for row. So source_pos[1] for X, source_pos[0] for Y.
    R1 = np.sqrt((X - slit1_pos[1])**2 + (Y - slit1_pos[0])**2)
    R2 = np.sqrt((X - slit2_pos[1])**2 + (Y - slit2_pos[0])**2)

    # Calculate the wave amplitude from each slit
    wave1 = np.cos(k * R1)
    wave2 = np.cos(k * R2)

    # Superposition: Add the two waves
    combined_wave = wave1 + wave2

    # Calculate intensity (proportional to the square of the amplitude)
    # Squaring ensures positive values and highlights constructive interference
    intensity = combined_wave**2
    
    return intensity, (slit1_pos, slit2_pos)

# --- Main simulation parameters and execution: Save this as double_slit.py ---
if __name__ == '__main__':
    # Simulation Parameters (feel free to tweak these!)
    GRID_ROWS = 800           # Height of the simulation area in grid units
    GRID_COLS = 1200          # Width of the simulation area in grid units
    WAVELENGTH = 30           # Wavelength of the light in grid units
    SLIT_SEPARATION = 120     # Distance between the centers of the two slits in grid units
    SLIT_X_POSITION = GRID_COLS // 4 # X-position (column index) of the slits

    print(f"Simulating double-slit with parameters:")
    print(f"  Grid: {GRID_ROWS}x{GRID_COLS}")
    print(f"  Wavelength: {WAVELENGTH} units")
    print(f"  Slit Separation: {SLIT_SEPARATION} units")
    print(f"  Slit X-position: {SLIT_X_POSITION} units")

    # Run the simulation
    intensity_map, slit_positions = simulate_double_slit(
        (GRID_ROWS, GRID_COLS),
        WAVELENGTH,
        SLIT_SEPARATION,
        SLIT_X_POSITION
    )

    slit1_pos, slit2_pos = slit_positions

    # Plotting the results
    plt.figure(figsize=(12, 8))
    
    # Use 'gray' colormap for intensity for classic black/white fringes.
    # 'origin='upper'' matches NumPy array indexing where [0,0] is top-left.
    # 'extent' helps set the plot coordinates for better understanding.
    plt.imshow(intensity_map, cmap='gray', origin='upper', 
               extent=[0, GRID_COLS, 0, GRID_ROWS])
    
    plt.colorbar(label='Intensity')
    
    # Mark slit positions on the plot for clarity
    plt.scatter([slit1_pos[1], slit2_pos[1]], [slit1_pos[0], slit2_pos[0]], 
                color='red', marker='o', s=150, label='Slits', zorder=5) # zorder to ensure visibility
    
    plt.title(f'Double-Slit Interference Pattern\n'
              f'Wavelength: {WAVELENGTH} units, Slit Separation: {SLIT_SEPARATION} units')
    plt.xlabel('X (Grid Units)')
    plt.ylabel('Y (Grid Units)')
    plt.legend()
    plt.tight_layout()
    plt.show()

```

## Running the Simulation

Save the code above as `double_slit.py` and run it from your terminal:

```bash
python double_slit.py
```

```output
Simulating double-slit with parameters:
  Grid: 800x1200
  Wavelength: 30 units
  Slit Separation: 120 units
  Slit X-position: 300 units

(A Matplotlib window will pop up.)

Expected Output:
You will see a 2D plot with a distinct interference pattern.
- On the left side of the screen (before the slits, indicated by red circles), you'll see two sets of circular waves emanating from the slit positions, overlapping.
- To the right of the slits, these overlapping waves create an interference pattern: alternating bright and dark vertical bands (fringes).
- The brightest band will be directly in line with the center of the two slits (the central maximum).
- As you move away from the center, the fringes will get progressively dimmer but still clearly visible.
- The red circles will pinpoint the exact locations of the "slits" (our point sources).
```

This visualization directly shows how constructive and destructive interference combine to form the characteristic pattern. Where the peaks of waves from both slits coincide, you get a bright fringe. Where a peak from one meets a trough from the other, you get a dark fringe.

## Further Exploration

This simulation provides a solid foundation. Here are some ideas to explore and extend it:

*   **Change Parameters:**
    *   **Wavelength (`WAVELENGTH`):** Try smaller or larger values. What happens to the spacing of the fringes? (Hint: Shorter wavelengths lead to tighter fringes).
    *   **Slit Separation (`SLIT_SEPARATION`):** Increase or decrease the distance between the slits. How does this affect the pattern? (Hint: Wider separation also leads to tighter fringes).
    *   **Grid Size:** Make the grid larger or smaller to see more or less of the pattern.
*   **Add More Slits:** How would you modify the code to simulate a diffraction grating (multiple parallel slits)? You'd just need to add more point sources and sum their wave contributions.
*   **Introduce Phase Difference:** What if the waves coming out of the slits weren't perfectly in phase? You could add an offset to the `np.cos` function for one of the waves.
*   **Time-Domain Simulation:** Our current simulation shows the steady-state pattern. A more advanced approach would be to simulate the wave propagation over time using finite difference methods to solve the wave equation. This is significantly more complex but allows you to see the waves moving and interfering dynamically.
*   **Vary Slit Width:** In this simulation, slits are treated as idealized point sources. For a more accurate classical model, you'd integrate contributions across a finite slit width, leading to a diffraction envelope modulating the interference pattern.

## Conclusion

By using fundamental principles of classical wave mechanics and a few lines of Python, NumPy, and Matplotlib, we've successfully simulated the essence of the double-slit experiment. This hands-on approach demystifies the interference phenomenon, turning a complex physics concept into an observable result you can generate with code.

It's a powerful reminder that sometimes, the most profound scientific insights can be grasped through simple, elegant models. Happy coding!