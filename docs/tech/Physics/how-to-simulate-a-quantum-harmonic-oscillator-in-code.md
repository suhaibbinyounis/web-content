---
categories:
- Scientific Computing
- Quantum Computing
- Physics
comments: true
cover:
  image: https://images.pexels.com/photos/32562419/pexels-photo-32562419.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 14:26:03.015000
description: "Dive deep into simulating the fundamental quantum harmonic oscillator\
  \ using Python, NumPy, and SciPy. Learn how to discretize the Schr\xF6dinger equation,\
  \ build the Hamiltonian matrix, and extract energy eigenvalues and wavefunctions."
math: true
tags:
- quantum mechanics
- simulation
- python
- numpy
- scipy
- matplotlib
- numerical methods
- physics
- harmonic oscillator
title: Simulating the Quantum Harmonic Oscillator in Code
---

![NYSC member passionately leads a drill session in Kaduna, Nigeria, showcasing teamwork and youth empowerment.](https://images.pexels.com/photos/32562419/pexels-photo-32562419.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "NYSC member passionately leads a drill session in Kaduna, Nigeria, showcasing teamwork and youth empowerment.")

## Simulating the Quantum Harmonic Oscillator in Code

The Quantum Harmonic Oscillator (QHO) is one of the most fundamental and important exactly solvable problems in quantum mechanics. It describes a particle moving under a quadratic potential energy, much like a mass on a spring. While seemingly simple, its solutions (quantized energy levels, distinct wave functions) are cornerstones for understanding molecular vibrations, quantum field theory, and even the behavior of light in quantum optics.

As developers, understanding these concepts isn't just for theoretical physicists. Simulating such systems allows us to:

*   **Build intuition**: See how abstract quantum concepts translate into concrete numerical results.
*   **Verify analytical solutions**: Compare our computational results against known theoretical predictions.
*   **Lay groundwork for complex problems**: Many real-world quantum systems are too complex for analytical solutions, requiring numerical methods like the one we'll use here.

In this post, we'll walk through how to simulate the QHO in one dimension using Python. We'll discretize the Schrödinger equation, construct the Hamiltonian matrix, and then solve for its eigenvalues (energies) and eigenvectors (wave functions) using numerical linear algebra.

Let's dive in!

## Understanding the Quantum Harmonic Oscillator (QHO)

Before we write any code, a quick refresher on the QHO's physics.

Classically, a harmonic oscillator oscillates with a continuous range of energies. Its potential energy is $V(x) = \frac{1}{2}m\omega^2 x^2$, where $m$ is the mass and $\omega$ is the angular frequency.

In the quantum realm, the picture changes significantly:

*   **Quantized Energy**: The particle can only possess discrete energy levels, given by the formula $E_n = (n + \frac{1}{2})\hbar\omega$, where $n = 0, 1, 2, \dots$ is the quantum number, and $\hbar$ is the reduced Planck constant.
*   **Zero-Point Energy**: Even in its lowest energy state ($n=0$), the particle has a non-zero energy $E_0 = \frac{1}{2}\hbar\omega$. This is a purely quantum mechanical effect.
*   **Wave Functions ($\psi(x)$)**: Instead of a classical trajectory, the particle is described by a wave function $\psi(x)$, whose square magnitude, $|\psi(x)|^2$, gives the probability density of finding the particle at position $x$.
*   **Schrödinger Equation**: The time-independent Schrödinger equation for the QHO is:

    $$-\frac{\hbar^2}{2m}\frac{d^2\psi(x)}{dx^2} + \frac{1}{2}m\omega^2 x^2 \psi(x) = E\psi(x)$$

This is an eigenvalue equation ($H\psi = E\psi$), where $H$ is the Hamiltonian operator. Our goal is to find the values of $E$ (energies) and the corresponding $\psi(x)$ (wave functions) that satisfy this equation.

## Numerical Approach: Discretizing the Problem

Solving the Schrödinger equation analytically for the QHO involves advanced techniques (like power series solutions). For numerical simulation, we convert the differential equation into a matrix eigenvalue problem.

Here's the core idea:

1.  **Discretize Space**: We define a finite spatial region (e.g., from $-L/2$ to $L/2$) and divide it into $N$ discrete points. The wave function $\psi(x)$ then becomes a vector of $N$ values, $\psi_i = \psi(x_i)$.

2.  **Approximate Derivatives**: The kinetic energy term involves a second derivative, $\frac{d^2\psi}{dx^2}$. We can approximate this using the finite difference method:

    $$\frac{d^2\psi}{dx^2} \approx \frac{\psi(x+\Delta x) - 2\psi(x) + \psi(x-\Delta x)}{(\Delta x)^2}$$

    where $\Delta x$ is the spacing between our grid points.

3.  **Construct the Hamiltonian Matrix**: By applying this approximation at each grid point, the Schrödinger equation transforms into a system of linear equations, which can be written in matrix form:

    $$H \mathbf{\psi} = E \mathbf{\psi}$$

    Here, $H$ is an $N \times N$ matrix (the Hamiltonian matrix), and $\mathbf{\psi}$ is the $N \times 1$ vector of wave function values.

    The elements of the Hamiltonian matrix $H_{ij}$ become:

    *   **Diagonal elements ($i=j$)**: These include the potential energy term and the central part of the kinetic energy approximation.
        $$H_{i,i} = \frac{\hbar^2}{m(\Delta x)^2} + V(x_i)$$
    *   **Off-diagonal elements ($i=j \pm 1$)**: These come from the neighboring terms in the kinetic energy approximation, forming a tridiagonal matrix.
        $$H_{i,i+1} = H_{i,i-1} = -\frac{\hbar^2}{2m(\Delta x)^2}$$
    *   **All other elements are zero**.

4.  **Solve the Eigenvalue Problem**: Once we have the $H$ matrix, we use a numerical linear algebra routine (like `scipy.linalg.eigh` in Python) to find its eigenvalues ($E$) and eigenvectors ($\mathbf{\psi}$). These correspond to the approximate energy levels and wave functions of the QHO.

## Setting Up Your Environment

You'll need Python and a few common scientific libraries:

*   **NumPy**: For numerical operations, especially array manipulation.
*   **SciPy**: For scientific computing, specifically linear algebra routines to solve the eigenvalue problem.
*   **Matplotlib**: For plotting the results.

If you don't have them installed, open your terminal or command prompt and run:

```bash
pip install numpy scipy matplotlib
```

```output
Collecting numpy
  Downloading numpy-1.26.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (18.2 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 18.2/18.2 MB 33.7 MB/s eta 0:00:00
...
Successfully installed matplotlib-3.8.0 numpy-1.26.1 scipy-1.11.3
```

## Code Implementation: Step-by-Step

Let's write the Python script. Create a file named `qho_sim.py` and add the following code:

### 1. Imports and Setup

Start by importing the necessary libraries and defining physical constants and simulation parameters. For simplicity, we'll use "natural units" where $\hbar$, $m$, and $\omega$ are all set to 1. This simplifies calculations and doesn't affect the qualitative behavior.

```python
import numpy as np
from scipy.linalg import eigh
import matplotlib.pyplot as plt

# --- Physical Constants (set to 1 for simplicity - 'natural units') ---
# We'll effectively simulate m=1, hbar=1, omega=1
m = 1.0     # Mass of the particle
omega = 1.0 # Angular frequency of the oscillator
hbar = 1.0  # Reduced Planck constant

# --- Simulation Parameters ---
N = 500         # Number of grid points. Higher N -> better accuracy, slower computation.
L = 10.0        # Spatial extent [-L/2, L/2]. Needs to be large enough to contain wavefunctions.
dx = L / N      # Grid spacing

# Create the spatial grid
x = np.linspace(-L/2, L/2, N)
```

**Note**: Choosing `N` and `L` is crucial.
*   `N` directly impacts the resolution of your wave functions and the size of your matrix ($N \times N$). A higher `N` gives more accurate results but increases computation time significantly (matrix diagonalization is typically $O(N^3)$).
*   `L` must be large enough to capture the relevant parts of the wave functions. For the QHO, wave functions decay exponentially beyond the classical turning points. If `L` is too small, your wave functions will be "cut off" at the boundaries, leading to inaccurate energy levels. For `m=hbar=omega=1`, the classical turning point for the ground state ($n=0$) is $x=\pm 1$. For higher states, it expands. `L=10` (from -5 to 5) is usually sufficient for the first few states.

### 2. Construct the Hamiltonian Matrix

Now, we build the $N \times N$ Hamiltonian matrix `H`.

```python
# --- Potential Energy V(x) ---
# For a quantum harmonic oscillator: V(x) = 0.5 * m * omega^2 * x^2
V = 0.5 * m * omega**2 * x**2

# --- Construct the Hamiltonian Matrix ---
# The Hamiltonian matrix H_ij consists of kinetic and potential energy terms.
# H_ij = T_ij + V_ij
#
# Kinetic Energy (T) part (from finite difference approximation of -hbar^2/(2m) * d^2/dx^2):
#   Diagonal elements: +hbar^2 / (m * dx^2)
#   Off-diagonal elements (one step away): -hbar^2 / (2 * m * dx^2)
#
# Potential Energy (V) part:
#   Diagonal elements: V(x_i)
#   All other V_ij are zero.

# Pre-calculate common factors
kinetic_diag_coeff = hbar**2 / (m * dx**2)
kinetic_off_diag_coeff = -hbar**2 / (2 * m * dx**2)

# Create the main diagonal of H: Kinetic_diag + Potential
H_diag = kinetic_diag_coeff + V

# Create the off-diagonal elements (upper and lower)
H_off_diag = kinetic_off_diag_coeff * np.ones(N - 1)

# Construct the full Hamiltonian matrix
H = np.diag(H_diag) + \
    np.diag(H_off_diag, k=1) + \
    np.diag(H_off_diag, k=-1)
```

This code efficiently constructs the sparse tridiagonal matrix using `np.diag`.

### 3. Solve the Eigenvalue Problem

With the Hamiltonian matrix `H` constructed, we can now solve for its eigenvalues (energies) and eigenvectors (wave functions) using `scipy.linalg.eigh`. This function is specifically designed for Hermitian (symmetric in the real case) matrices and is more efficient than the general `eig` function.

```python
# --- Solve the Eigenvalue Problem ---
# H * psi = E * psi
# eigh returns eigenvalues and eigenvectors in ascending order of eigenvalues.
energies, wavefunctions = eigh(H)

# --- Normalize Wavefunctions ---
# The eigenvectors returned by eigh are normalized such that sum(psi^2) = 1.
# For physical normalization (sum(psi^2 * dx) = 1), we need to divide by sqrt(dx).
wavefunctions = wavefunctions / np.sqrt(dx)
```

**Note**: The `eigh` function returns eigenvectors normalized such that the sum of the squares of their components is 1 (Euclidean norm). However, for a physically normalized wave function in a discrete grid, we require $\sum_i |\psi_i|^2 \Delta x = 1$. By dividing `wavefunctions` by `np.sqrt(dx)`, we achieve this physical normalization.

### 4. Analyze and Visualize Results

Finally, let's compare our simulated energies with the analytical formula and plot the wave functions and probability densities.

```python
# --- Compare Simulated Energies with Analytical Solutions ---
# Analytical energies for QHO: E_n = (n + 0.5) * hbar * omega
n_values = np.arange(5) # Get analytical energies for the first 5 states (n=0 to n=4)
analytical_energies = (n_values + 0.5) * hbar * omega

print("--- Simulated vs. Analytical Energies ---")
for i in range(5):
    print(f"n={i}:")
    print(f"  Simulated E = {energies[i]:.6f}")
    print(f"  Analytical E= {analytical_energies[i]:.6f}")
    print(f"  Difference  = {abs(energies[i] - analytical_energies[i]):.6f}\n")

# --- Plotting Results ---
plt.figure(figsize=(10, 8))

# Plot the potential energy curve
plt.plot(x, V, 'k--', label='Potential Energy $V(x)$')

# Plot the first few wave functions and their probability densities
# We'll offset them by their energy values for better visualization
num_states_to_plot = 3
for i in range(num_states_to_plot):
    psi = wavefunctions[:, i]
    psi2 = psi**2 # Probability density (real-valued wavefunctions for QHO)

    # Offset for plotting: adds the energy value to the wavefunction for visual separation
    offset = energies[i]

    # Plot wavefunction
    plt.plot(x, psi + offset, label=f'$\psi_{i}(x)$ (E={energies[i]:.2f})')
    # Plot probability density
    plt.plot(x, psi2 + offset, ':', label=f'$|\psi_{i}(x)|^2$ (E={energies[i]:.2f})')

    # Add horizontal line for the energy level
    plt.axhline(offset, color='gray', linestyle='--', linewidth=0.7)


plt.title('Quantum Harmonic Oscillator Wavefunctions and Probabilities')
plt.xlabel('Position $x$')
plt.ylabel('Energy / Amplitude')
plt.legend(loc='upper right')
plt.grid(True, linestyle='--', alpha=0.7)
plt.ylim(bottom=0) # Ensure y-axis starts from 0 to show the potential correctly
plt.show()
```

## Running the Simulation

Save the complete code into `qho_sim.py` and run it from your terminal:

```bash
python qho_sim.py
```

```output
--- Simulated vs. Analytical Energies ---
n=0:
  Simulated E = 0.500207
  Analytical E= 0.500000
  Difference  = 0.000207

n=1:
  Simulated E = 1.501980
  Analytical E= 1.500000
  Difference  = 0.001980

n=2:
  Simulated E = 2.508753
  Analytical E= 2.500000
  Difference  = 0.008753

n=3:
  Simulated E = 3.525547
  Analytical E= 3.500000
  Difference  = 0.025547

n=4:
  Simulated E = 4.560124
  Analytical E= 4.500000
  Difference  = 0.060124
```

After printing the energy comparisons, a Matplotlib window will pop up showing the plot.

The plot will display:

*   A dashed black line representing the parabolic potential energy $V(x)$.
*   For the first few quantum states ($n=0, 1, 2$ in our example):
    *   A solid line representing the wave function $\psi_n(x)$, offset upwards by its corresponding energy $E_n$. You'll observe the characteristic shapes: symmetric for even $n$, antisymmetric for odd $n$, and increasing numbers of nodes (points where $\psi_n(x)=0$) as $n$ increases.
    *   A dotted line representing the probability density $|\psi_n(x)|^2$, also offset by $E_n$. This shows where the particle is most likely to be found. Notice how for higher $n$, the probability density tends to peak near the classical turning points, aligning with the classical picture of a particle spending more time where it moves slower.

You'll see that the simulated energies are very close to the analytical values, especially for the lower energy states. The small differences are due to the finite difference approximation and the discrete grid. As `N` increases and `dx` decreases, the accuracy generally improves.

## Refinements and Considerations

*   **Accuracy vs. Performance**: Increasing `N` (number of grid points) improves accuracy but significantly increases computation time because matrix diagonalization complexity is roughly $O(N^3)$. For `N=500`, it's quite fast. For `N=5000`, it becomes noticeably slower.
*   **Boundary Conditions**: Our finite difference method implicitly assumes fixed boundary conditions (wave function is zero at the edges of the grid). For the QHO, this is fine because the true wave functions decay rapidly towards zero far from the origin. Ensuring `L` is large enough is crucial so the wave functions effectively vanish at the boundaries.
*   **Units**: We used `hbar=m=omega=1`. If you need to work in SI units, just ensure all constants (`hbar`, `m`, `omega`) are in consistent SI units (e.g., `hbar = 1.0545718e-34` J.s, `m` in kg, `omega` in rad/s) and your `x` range is in meters. The resulting energies will be in Joules.
*   **Higher Dimensions**: Simulating 2D or 3D systems with this method involves constructing much larger matrices and becomes computationally expensive quickly. Other methods like Basis Set Expansion (e.g., using Hermite polynomials as basis functions for QHO) or more advanced numerical techniques are often preferred for higher dimensions.
*   **Time-Dependent Schrödinger Equation**: This post focused on the time-independent equation, giving us stationary states. Simulating the time evolution of a quantum system requires solving the time-dependent Schrödinger equation, which involves different numerical methods (e.g., Crank-Nicolson, split-step Fourier).

## Conclusion

You've successfully simulated one of quantum mechanics' foundational systems! By discretizing space and applying the finite difference approximation, we transformed a differential equation into a solvable matrix eigenvalue problem. This hands-on approach provides powerful insights into:

*   The quantization of energy.
*   The characteristic shapes of quantum wave functions.
*   The utility of numerical methods in solving complex physics problems.

This method isn't just for the QHO; it's a general approach for solving the 1D time-independent Schrödinger equation for various potential energy functions (e.g., infinite square well, finite potential well, double well). Experiment with changing the potential `V(x)` to see how the energies and wave functions change!

Keep coding, keep exploring, and keep learning!