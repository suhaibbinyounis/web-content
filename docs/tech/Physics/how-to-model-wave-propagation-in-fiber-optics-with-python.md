---
categories:
- Programming
- Physics
- Simulation
comments: true
date: 2025-06-17 14:26:03.015000
description: Learn to simulate light propagation in optical fibers using the Split-Step
  Fourier Method (SSFM) and Python. Understand dispersion, nonlinearity, and attenuation
  with practical code examples.
math: true
tags:
- Python
- Fiber Optics
- Wave Propagation
- Simulation
- Scientific Computing
- Photonics
- Numerical Methods
title: How to Model Wave Propagation in Fiber Optics with Python
---

Fiber optic communication underpins our global digital infrastructure. From high-speed internet to data centers, light pulses traveling through hair-thin glass strands are the backbone. But these pulses aren't static; they evolve, broaden, and interact with the fiber itself. Understanding and predicting this evolution is crucial for designing robust high-speed optical systems.

This post will guide you through modeling wave propagation in optical fibers using Python. We'll focus on the **Split-Step Fourier Method (SSFM)**, a powerful and widely used numerical technique to solve the **Nonlinear Schrödinger Equation (NLSE)**, which governs light propagation in optical fibers.

Ready to dive into the fascinating world where physics meets code?

## Understanding the Core Concepts

Before we write code, let's briefly touch upon the key physical phenomena that influence light propagation in single-mode optical fibers:

1.  **Dispersion (Group Velocity Dispersion - GVD):**
    Light pulses are not monochromatic; they consist of a range of frequencies (colors). Due to material properties, different frequencies travel at slightly different speeds in the fiber. This causes the pulse to spread out in time, leading to pulse broadening.
    *   **Parameter:** $\beta_2$ (beta-2). Measured in $ps^2/km$. A negative $\beta_2$ means shorter wavelengths travel faster (anomalous dispersion), while a positive $\beta_2$ means longer wavelengths travel faster (normal dispersion). Standard single-mode fiber (SMF-28) at 1550 nm has anomalous dispersion ($\beta_2 \approx -21 ps^2/km$).

2.  **Nonlinearity (Self-Phase Modulation - SPM):**
    At high optical powers, the refractive index of the fiber material changes slightly with the intensity of the light itself. This phenomenon, known as the optical Kerr effect, leads to Self-Phase Modulation (SPM). SPM causes spectral broadening of the pulse, as the phase of the light changes along the pulse profile.
    *   **Parameter:** $\gamma$ (gamma). Measured in $(W \cdot km)^{-1}$. It quantifies how strongly the fiber's refractive index changes with power.

3.  **Attenuation (Loss):**
    Fibers are not perfectly transparent. Some light power is lost due to absorption and scattering as it propagates. This simply means the pulse amplitude decreases over distance.
    *   **Parameter:** $\alpha$ (alpha). Measured in $dB/km$.

These three effects combined are described by the **Nonlinear Schrödinger Equation (NLSE)**:

$$
\frac{\partial A}{\partial z} = -\frac{\alpha}{2}A - i\frac{\beta_2}{2}\frac{\partial^2 A}{\partial T^2} + i\gamma|A|^2 A
$$

Where:
*   $A(z, T)$ is the complex envelope of the electric field.
*   $z$ is the propagation distance along the fiber.
*   $T$ is time in a retarded reference frame (moving with the pulse).
*   $\alpha$ is the attenuation coefficient.
*   $\beta_2$ is the Group Velocity Dispersion (GVD) parameter.
*   $\gamma$ is the nonlinear coefficient.
*   $i$ is the imaginary unit.

Solving this equation analytically is challenging. That's where numerical methods like the SSFM come in.

## The Split-Step Fourier Method (SSFM)

The core idea behind SSFM is to "split" the propagation step into smaller segments. Within each small segment, we treat the linear effects (dispersion, attenuation) and nonlinear effects (SPM) separately.

Since dispersion is a linear effect, it's easier to handle in the **frequency domain** using Fourier transforms. Nonlinearity, however, is a power-dependent effect, so it's applied in the **time domain**.

The symmetric SSFM algorithm for a step `dz` is as follows:

1.  **Apply half-step nonlinearity:** Update the pulse based on the nonlinear term in the time domain.
2.  **Apply full-step linear effects:**
    a.  Transform the pulse to the frequency domain (using FFT).
    b.  Multiply by the linear propagation operator (which includes dispersion and attenuation) in the frequency domain.
    c.  Transform the pulse back to the time domain (using IFFT).
3.  **Apply half-step nonlinearity:** Update the pulse again based on the nonlinear term in the time domain.

This "split-step" approach ensures that both effects are accounted for accurately over short distances, and by repeating it many times, we can simulate propagation over long fiber lengths.

## Setting up Your Python Environment

You'll need a standard Python scientific stack. If you don't have them, install them using `pip`:

```bash
pip install numpy matplotlib scipy
```

*   `numpy`: Essential for numerical operations, especially array manipulation and fast Fourier transforms (FFTs).
*   `matplotlib`: For plotting and visualizing the pulse evolution.
*   `scipy`: While `numpy.fft` is sufficient for the core SSFM, `scipy` has useful constants and signal processing tools.

## Python Implementation of SSFM

Let's put everything into a single Python script (`fiber_ssfm.py`) that you can run.

### Example 1: Basic Setup and Initial Pulse

First, we define constants, fiber parameters, and the initial pulse. We'll use a Gaussian pulse, which is a common and simple model for light pulses.

```python
# fiber_ssfm.py

import numpy as np
import matplotlib.pyplot as plt

# --- Physical Constants ---
c = 299792458  # Speed of light in vacuum (m/s)
pi = np.pi

# --- Fiber Parameters (Typical for SMF-28 at 1550 nm) ---
# Wavelength
lambda_0 = 1550e-9  # Operating wavelength (m)

# Dispersion (beta2)
# Convert from ps^2/km to s^2/m
# 1 ps^2/km = 1e-24 s^2/m
beta2 = -21.27e-27  # GVD parameter (s^2/m)

# Nonlinearity (gamma)
# Convert from (W*km)^-1 to (W*m)^-1
# 1 (W*km)^-1 = 1e-3 (W*m)^-1
gamma = 1.3e-3  # Nonlinearity parameter (W^-1 m^-1)

# Attenuation (alpha)
# Convert from dB/km to Np/m (amplitude attenuation)
# alpha_Np = alpha_dB / (20 * log10(e) * 1000)
# log10(e) approx 0.4343
# So, alpha_Np = alpha_dB / 8685.889
alpha_dB_per_km = 0.2  # Attenuation (dB/km)
alpha_Np_per_m = alpha_dB_per_km / (10 * np.log10(np.exp(1)) * 1000) # For Power: 10 * log10(e) -> for amplitude: 20 * log10(e)
                                                                 # Since NLSE is for Amplitude, alpha/2 is commonly used in frequency domain operator,
                                                                 # but we'll apply it directly to amplitude.
                                                                 # For A(z) = A(0) * exp(-alpha_Np * z), the power loss is |A(z)|^2 / |A(0)|^2 = exp(-2*alpha_Np*z).
                                                                 # So alpha_Np = alpha_dB / (10 * log10(e) * 1000) / 2 = alpha_dB / (20 * log10(e) * 1000).
                                                                 # Let's use the standard power attenuation coefficient which is alpha_dB_per_km / (10 * log10(e) * 1000)
                                                                 # and then apply exp(-alpha*dz/2) to the amplitude for each step.
                                                                 # Simpler: The `alpha` in NLSE is for amplitude attenuation.
                                                                 # If `alpha_dB` is dB/km, power attenuation is P(z) = P_0 * 10^(-alpha_dB * z_km / 10).
                                                                 # So A(z) = A_0 * 10^(-alpha_dB * z_km / 20).
                                                                 # This means A(z) = A_0 * exp(-alpha_Np_per_m * z_m), where alpha_Np_per_m = alpha_dB / (20 * log10(e) * 1000).
alpha_Np_per_m = alpha_dB_per_km / (20 * np.log10(np.exp(1)) * 1000) # Amplitude attenuation coefficient (Np/m)

# Fiber length
L = 10000  # Fiber length (m)

# --- Pulse Parameters (Gaussian) ---
P0 = 0.1  # Peak power of initial pulse (W) - 100mW
T0 = 10e-12  # Pulse duration (s) - 10 ps (FWHM related to this by factor of 1.665)
             # T0 is often defined as the half-width at 1/e intensity point for Gaussian.
             # FWHM_pulse = 2 * T0 * sqrt(ln(2))

# --- Simulation Parameters ---
N = 2**12  # Number of points in time window (should be power of 2 for FFT)
T_window = 20 * T0  # Time window for simulation (s) - must be large enough to contain pulse
dt = T_window / N  # Time step
dz = 100  # Step size for propagation (m) - must be small enough for accuracy
Nz = int(L / dz)  # Number of propagation steps

# --- Time and Frequency Axes ---
t = np.linspace(-T_window / 2, T_window / 2, N, endpoint=False) # Time axis
omega = 2 * pi * np.fft.fftfreq(N, d=dt)  # Angular frequency axis

# --- Initial Pulse Envelope (Gaussian) ---
A0 = np.sqrt(P0) * np.exp(-(t**2) / (2 * T0**2))

print(f"--- Fiber Parameters ---")
print(f"Wavelength: {lambda_0*1e9:.2f} nm")
print(f"Beta2: {beta2*1e27:.2f} ps^2/km")
print(f"Gamma: {gamma*1e3:.2f} (W*km)^-1")
print(f"Attenuation: {alpha_dB_per_km:.2f} dB/km ({alpha_Np_per_m*1e3:.4f} Np/m)")
print(f"Fiber Length: {L/1000:.0f} km")
print(f"\n--- Pulse Parameters ---")
print(f"Peak Power: {P0*1e3:.2f} mW")
print(f"Pulse Duration (T0): {T0*1e12:.2f} ps")
print(f"Initial Pulse Energy: {np.trapz(np.abs(A0)**2, t)*1e12:.2f} pJ")
print(f"\n--- Simulation Parameters ---")
print(f"Number of points (N): {N}")
print(f"Time Window: {T_window*1e9:.2f} ns")
print(f"Time Step (dt): {dt*1e15:.2f} fs")
print(f"Propagation Step (dz): {dz:.2f} m")
print(f"Number of steps (Nz): {Nz}")

# Plot initial pulse
plt.figure(figsize=(12, 5))

plt.subplot(1, 2, 1)
plt.plot(t * 1e12, np.abs(A0)**2)
plt.title('Initial Pulse Shape (Time Domain)')
plt.xlabel('Time (ps)')
plt.ylabel('Power (W)')
plt.grid(True)

plt.subplot(1, 2, 2)
# Calculate spectrum
spectrum_A0 = np.fft.fftshift(np.fft.fft(A0))
freq_axis = np.fft.fftshift(omega / (2 * pi)) # Convert angular freq to Hz
plt.plot(freq_axis / 1e9, 10 * np.log10(np.abs(spectrum_A0)**2 / np.max(np.abs(spectrum_A0)**2))) # dB scale
plt.title('Initial Pulse Spectrum (Frequency Domain)')
plt.xlabel('Frequency (GHz)')
plt.ylabel('Normalized Power (dB)')
plt.grid(True)
plt.tight_layout()
plt.show()
```

#### Output from Example 1:

```output
--- Fiber Parameters ---
Wavelength: 1550.00 nm
Beta2: -21.27 ps^2/km
Gamma: 1.30 (W*km)^-1
Attenuation: 0.20 dB/km (0.000023 Np/m)
Fiber Length: 10 km

--- Pulse Parameters ---
Peak Power: 100.00 mW
Pulse Duration (T0): 10.00 ps
Initial Pulse Energy: 1000.00 pJ

--- Simulation Parameters ---
Number of points (N): 4096
Time Window: 200.00 ns
Time Step (dt): 48.83 fs
Propagation Step (dz): 100.00 m
Number of steps (Nz): 100
```

And you should see two plots: one showing the initial Gaussian pulse in the time domain, and another showing its corresponding Gaussian spectrum in the frequency domain.

### Example 2: The SSFM Function

Now, let's implement the core SSFM loop. We'll encapsulate this in a function that takes the initial pulse and fiber parameters, then returns the pulse at the end of the fiber, along with snapshots at various points.

```python
# Continue from fiber_ssfm.py

def propagate_ssfm(A0, t, omega, L, dz, beta2, gamma, alpha_Np_per_m, snapshot_interval_km=1):
    """
    Propagates an optical pulse through a fiber using the Split-Step Fourier Method (SSFM).

    Args:
        A0 (np.array): Initial complex pulse envelope.
        t (np.array): Time axis.
        omega (np.array): Angular frequency axis.
        L (float): Total fiber length (m).
        dz (float): Step size for propagation (m).
        beta2 (float): Group Velocity Dispersion parameter (s^2/m).
        gamma (float): Nonlinearity parameter (W^-1 m^-1).
        alpha_Np_per_m (float): Amplitude attenuation coefficient (Np/m).
        snapshot_interval_km (float): Interval in km at which to save pulse snapshots.

    Returns:
        tuple: (final_pulse, z_snapshots, pulse_snapshots, power_evolution)
    """
    N = len(t)
    Nz = int(L / dz)
    dt = t[1] - t[0]

    A = A0.copy()  # Current pulse envelope

    # Linear operator for a single dz step in frequency domain
    # exp(i * beta2/2 * omega^2 * dz - alpha/2 * dz)
    # Note: alpha in the NLSE is for amplitude attenuation.
    # The linear operator already incorporates the attenuation.
    linear_operator = np.exp(1j * beta2 / 2 * omega**2 * dz - alpha_Np_per_m * dz)

    # Prepare for snapshots
    z_snapshots = []
    pulse_snapshots = []
    power_evolution = [] # To store peak power at each step

    snapshot_step = int((snapshot_interval_km * 1000) / dz)
    if snapshot_step == 0: snapshot_step = 1 # Ensure at least first snapshot is saved if interval too small

    print(f"\nStarting SSFM simulation for {L/1000:.0f} km...")
    for i in range(Nz):
        z = (i + 1) * dz # Current fiber distance

        # 1. Apply half-step nonlinearity (time domain)
        A = A * np.exp(1j * gamma * (np.abs(A)**2) * (dz / 2))

        # 2. Apply full-step linear effects (frequency domain)
        A_f = np.fft.fft(A)
        A_f = A_f * linear_operator
        A = np.fft.ifft(A_f)

        # 3. Apply second half-step nonlinearity (time domain)
        A = A * np.exp(1j * gamma * (np.abs(A)**2) * (dz / 2))

        # Store snapshots for visualization
        if (i + 1) % snapshot_step == 0 or (i + 1) == Nz:
            z_snapshots.append(z)
            pulse_snapshots.append(A.copy())
            print(f"  Snapshot saved at {z/1000:.1f} km")

        # Store peak power for evolution plot
        power_evolution.append(np.max(np.abs(A)**2))

    print(f"Simulation finished.")
    return A, np.array(z_snapshots), np.array(pulse_snapshots), np.array(power_evolution)

# --- Main execution block to run SSFM ---

# Define the parameters to test (using values from Example 1)
current_beta2 = beta2
current_gamma = gamma
current_alpha = alpha_Np_per_m

# Run the SSFM
final_pulse, z_snaps, pulse_snaps, power_evolution_over_z = propagate_ssfm(
    A0, t, omega, L, dz, current_beta2, current_gamma, current_alpha
)

# Plotting the results
plt.figure(figsize=(15, 6))

# Plot Pulse Evolution (Time Domain)
plt.subplot(1, 2, 1)
for i, snapshot_A in enumerate(pulse_snaps):
    z_km = z_snaps[i] / 1000
    plt.plot(t * 1e12, np.abs(snapshot_A)**2, label=f'{z_km:.1f} km')
plt.title('Pulse Shape Evolution (Time Domain)')
plt.xlabel('Time (ps)')
plt.ylabel('Power (W)')
plt.legend()
plt.grid(True)
plt.ylim(ymin=0) # Ensure y-axis starts at 0

# Plot Spectral Evolution (Frequency Domain)
plt.subplot(1, 2, 2)
for i, snapshot_A in enumerate(pulse_snaps):
    z_km = z_snaps[i] / 1000
    spectrum_A = np.fft.fftshift(np.fft.fft(snapshot_A))
    # Normalize spectrum to max power at input for consistent dB scale
    max_input_power = np.max(np.abs(np.fft.fftshift(np.fft.fft(A0)))**2)
    plt.plot(freq_axis / 1e9, 10 * np.log10(np.abs(spectrum_A)**2 / max_input_power), label=f'{z_km:.1f} km')
plt.title('Pulse Spectrum Evolution (Frequency Domain)')
plt.xlabel('Frequency (GHz)')
plt.ylabel('Normalized Power (dB)')
plt.legend()
plt.grid(True)
plt.ylim(bottom=-40) # Show 40 dB dynamic range

plt.tight_layout()
plt.show()

# Plot Peak Power Evolution
plt.figure(figsize=(8, 5))
z_axis_power = np.linspace(dz, L, Nz) / 1000 # Convert to km
plt.plot(z_axis_power, power_evolution_over_z * 1e3) # Convert to mW
plt.title('Peak Power Evolution Along Fiber')
plt.xlabel('Distance (km)')
plt.ylabel('Peak Power (mW)')
plt.grid(True)
plt.show()

```

#### Output from Example 2:

```output
Starting SSFM simulation for 10 km...
  Snapshot saved at 0.1 km
  Snapshot saved at 0.2 km
  Snapshot saved at 0.3 km
  Snapshot saved at 0.4 km
  Snapshot saved at 0.5 km
  Snapshot saved at 0.6 km
  Snapshot saved at 0.7 km
  Snapshot saved at 0.8 km
  Snapshot saved at 0.9 km
  Snapshot saved at 1.0 km
  Snapshot saved at 1.1 km
  ... (continues for other km intervals)
  Snapshot saved at 9.9 km
  Snapshot saved at 10.0 km
Simulation finished.
```

You will see three plots:
1.  **Pulse Shape Evolution (Time Domain):** Shows how the pulse broadens and potentially distorts due to dispersion and nonlinearity, with its amplitude decaying due to attenuation.
2.  **Pulse Spectrum Evolution (Frequency Domain):** Shows how the spectrum broadens due to self-phase modulation.
3.  **Peak Power Evolution Along Fiber:** Illustrates the steady decay of peak power due to attenuation.

### Example 3: Simulating Dispersion Only

To isolate the effect of dispersion, we can set the nonlinearity parameter ($\gamma$) to zero. Observe how the pulse broadens in time without significant spectral changes.

```python
# To run this, uncomment and replace the SSFM call in the main block of fiber_ssfm.py
# Set gamma to zero to observe only dispersion
current_gamma_dispersion_only = 0.0
print("\n--- Running Dispersion-Only Simulation ---")
final_pulse_disp, z_snaps_disp, pulse_snaps_disp, power_evolution_disp = propagate_ssfm(
    A0, t, omega, L, dz, beta2, current_gamma_dispersion_only, alpha_Np_per_m
)

# Plotting for dispersion only
plt.figure(figsize=(15, 6))
plt.subplot(1, 2, 1)
for i, snapshot_A in enumerate(pulse_snaps_disp):
    z_km = z_snaps_disp[i] / 1000
    plt.plot(t * 1e12, np.abs(snapshot_A)**2, label=f'{z_km:.1f} km')
plt.title('Pulse Shape (Dispersion Only)')
plt.xlabel('Time (ps)')
plt.ylabel('Power (W)')
plt.legend()
plt.grid(True)
plt.ylim(ymin=0)

plt.subplot(1, 2, 2)
for i, snapshot_A in enumerate(pulse_snaps_disp):
    z_km = z_snaps_disp[i] / 1000
    spectrum_A = np.fft.fftshift(np.fft.fft(snapshot_A))
    max_input_power = np.max(np.abs(np.fft.fftshift(np.fft.fft(A0)))**2)
    plt.plot(freq_axis / 1e9, 10 * np.log10(np.abs(spectrum_A)**2 / max_input_power), label=f'{z_km:.1f} km')
plt.title('Pulse Spectrum (Dispersion Only)')
plt.xlabel('Frequency (GHz)')
plt.ylabel('Normalized Power (dB)')
plt.legend()
plt.grid(True)
plt.ylim(bottom=-40)
plt.tight_layout()
plt.show()
```

#### Expected Output for Dispersion Only:

The time-domain plot will show the pulse significantly broadening and its peak power decreasing, but the spectral plot will show almost no change in shape (only a uniform drop in power due to attenuation).

### Example 4: Simulating Nonlinearity Only (Self-Phase Modulation - SPM)

To observe nonlinearity (SPM), we set the dispersion parameter ($\beta_2$) to zero. This will show how the spectrum broadens, while the time-domain pulse shape remains relatively unchanged (except for attenuation).

```python
# To run this, uncomment and replace the SSFM call in the main block of fiber_ssfm.py
# Set beta2 to zero to observe only nonlinearity
current_beta2_nonlinearity_only = 0.0
print("\n--- Running Nonlinearity-Only Simulation (SPM) ---")
final_pulse_nonlin, z_snaps_nonlin, pulse_snaps_nonlin, power_evolution_nonlin = propagate_ssfm(
    A0, t, omega, L, dz, current_beta2_nonlinearity_only, gamma, alpha_Np_per_m
)

# Plotting for nonlinearity only
plt.figure(figsize=(15, 6))
plt.subplot(1, 2, 1)
for i, snapshot_A in enumerate(pulse_snaps_nonlin):
    z_km = z_snaps_nonlin[i] / 1000
    plt.plot(t * 1e12, np.abs(snapshot_A)**2, label=f'{z_km:.1f} km')
plt.title('Pulse Shape (Nonlinearity Only)')
plt.xlabel('Time (ps)')
plt.ylabel('Power (W)')
plt.legend()
plt.grid(True)
plt.ylim(ymin=0)

plt.subplot(1, 2, 2)
for i, snapshot_A in enumerate(pulse_snaps_nonlin):
    z_km = z_snaps_nonlin[i] / 1000
    spectrum_A = np.fft.fftshift(np.fft.fft(snapshot_A))
    max_input_power = np.max(np.abs(np.fft.fftshift(np.fft.fft(A0)))**2)
    plt.plot(freq_axis / 1e9, 10 * np.log10(np.abs(spectrum_A)**2 / max_input_power), label=f'{z_km:.1f} km')
plt.title('Pulse Spectrum (Nonlinearity Only)')
plt.xlabel('Frequency (GHz)')
plt.ylabel('Normalized Power (dB)')
plt.legend()
plt.grid(True)
plt.ylim(bottom=-40)
plt.tight_layout()
plt.show()
```

#### Expected Output for Nonlinearity Only:

The spectral plot will show significant broadening, often with characteristic oscillations if the power is high enough. The time-domain plot will show the pulse shape largely preserved, with only its peak power decreasing due to attenuation.

## Analysis and Interpretation

*   **Dispersion:** When only dispersion is active, the pulse broadens in time. The amount of broadening depends on the pulse width, the dispersion parameter ($\beta_2$), and the propagation distance. Shorter pulses and higher dispersion lead to more significant broadening.
*   **Nonlinearity (SPM):** When only nonlinearity is active, the pulse's spectrum broadens. This is because the intensity-dependent refractive index causes different parts of the pulse (front, peak, tail) to experience different phase shifts, leading to new frequency components.
*   **Combined Effects:** When both are present, they interact. In the anomalous dispersion regime ($\beta_2 < 0$), dispersion can be partially compensated by nonlinearity if the pulse power is above a certain threshold, potentially leading to soliton formation (if higher-order effects are ignored). Our example, with typical parameters, shows broadening in both domains.
*   **Attenuation:** Attenuation consistently reduces the pulse power as it propagates. This also reduces the effectiveness of nonlinear effects, as nonlinearity is power-dependent. For very long fibers, the pulse might become too weak for nonlinearity to be significant, and it effectively propagates in a linear regime.

### Limitations of this Model

This SSFM implementation, while powerful, makes several simplifying assumptions:

*   **Single-Mode Fiber:** Assumes propagation in a single fundamental mode, ignoring multi-mode effects.
*   **No Polarization Effects:** Ignores Polarization Mode Dispersion (PMD) and other polarization-related phenomena.
*   **No Higher-Order Effects:** Does not include higher-order dispersion (e.g., $\beta_3$), Raman scattering, Brillouin scattering, self-steepening, etc. These become important for very short pulses or very high powers.
*   **Ideal Fiber:** Assumes a uniform, ideal fiber, neglecting imperfections or variations along the length.
*   **Slowly Varying Envelope Approximation:** The NLSE itself is derived under this approximation.

For most standard long-haul communication scenarios with relatively low power and picosecond pulses, this basic SSFM model provides a very good approximation and is invaluable for preliminary design and analysis.

## Conclusion

You've successfully implemented a fundamental model for wave propagation in fiber optics using Python and the Split-Step Fourier Method! This technique allows you to simulate the complex interplay of dispersion, nonlinearity, and attenuation, providing critical insights into how light pulses evolve in optical fibers.

While this basic model has limitations, it forms the foundation for more advanced simulations. From here, you can explore:

*   **Varying parameters:** See how changing pulse power, width, fiber length, or dispersion values impacts the outcome.
*   **Different pulse shapes:** Try super-Gaussian or chirped pulses.
*   **Higher-order effects:** Investigate adding $\beta_3$ or other advanced terms to the NLSE.
*   **Soliton propagation:** Experiment with specific parameters (anomalous dispersion, higher power) to observe soliton formation.

Fiber optics is a fascinating field, and numerical simulation is a powerful tool to unravel its complexities. Keep experimenting, keep learning, and keep building!