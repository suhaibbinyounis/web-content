---
categories:
- Programming
- Data Science
- Electrical Engineering
comments: true
cover:
  image: https://images.pexels.com/photos/27379769/pexels-photo-27379769.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 14:26:03.015000
description: Dive deep into the mathematical and conceptual derivation of signal sampling
  theory, tracing it back to the foundational Nyquist-Shannon theorem. Learn by example
  with Python code demonstrating spectral replication and aliasing.
math: true
tags:
- Signal Processing
- Nyquist-Shannon
- Sampling
- Fourier Transform
- Python
- NumPy
- SciPy
- DSP
- Theory
title: How to Derive Signal Sampling Theory from Nyquist-Shannon
---

![Tall steel telecommunication tower with moon in the sky, São Paulo.](https://images.pexels.com/photos/27379769/pexels-photo-27379769.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Tall steel telecommunication tower with moon in the sky, São Paulo.")

## How to Derive Signal Sampling Theory from Nyquist-Shannon

Understanding how we can represent continuous analog signals in the discrete digital world is fundamental in countless technical fields – from audio engineering and telecommunications to medical imaging and sensor data acquisition. At the heart of this transformation lies **signal sampling theory**, underpinned by the venerable **Nyquist-Shannon Sampling Theorem**.

While often stated as a rule-of-thumb (`sampling_rate >= 2 * max_frequency`), grasping *why* this rule holds true is crucial for any developer or engineer working with signals. This post isn't just about stating the theorem; it's about walking through its derivation, step-by-step, using the powerful tools of the Fourier Transform.

Let's demystify it.

## Prerequisites

Before we dive in, a basic understanding of the following concepts will be beneficial:

*   **Continuous-time signals** and their representation `x(t)`.
*   **Discrete-time signals** and their representation `x[n]`.
*   The **Fourier Transform (FT)**: Its purpose (transforming signals from time domain to frequency domain), and key properties (especially the convolution theorem).
*   **Complex exponentials** (`e^(jωt)`).
*   The **Dirac Delta function** (or impulse function) and its properties.

Don't worry if your memory is a bit rusty; we'll touch on the most relevant aspects as we go.

## The Core Idea: Sampling as Modulation

Imagine you have a continuous analog signal, `x(t)`. To convert it into a discrete digital signal, we "sample" it. This means we measure its amplitude at regular intervals of time, `T_s`. The inverse of this sampling interval is the **sampling frequency**, `F_s = 1 / T_s`.

Mathematically, we can represent this process as multiplying the continuous signal `x(t)` by an ideal **impulse train**. An impulse train `p(t)` is an infinite series of Dirac Delta functions, each separated by `T_s`.

$$
p(t) = \sum_{n=-\infty}^{\infty} \delta(t - nT_s)
$$

When we multiply our continuous signal `x(t)` by this impulse train, we get the sampled signal, `x_s(t)`:

$$
x_s(t) = x(t) \cdot p(t) = x(t) \sum_{n=-\infty}^{\infty} \delta(t - nT_s)
$$

Because of the sifting property of the Dirac Delta function (`x(t) * δ(t - a) = x(a) * δ(t - a)`), this multiplication effectively picks out the values of `x(t)` only at the sampling instants:

$$
x_s(t) = \sum_{n=-\infty}^{\infty} x(nT_s) \delta(t - nT_s)
$$

This `x_s(t)` is a continuous-time signal that *represents* our sampled discrete data. The actual discrete samples are `x[n] = x(nT_s)`.

## Derivation Step 1: Fourier Transform of the Impulse Train

To understand what happens in the frequency domain, we need the Fourier Transform of `p(t)`. This is a well-known result: the Fourier Transform of an impulse train in the time domain is *also* an impulse train in the frequency domain.

Let `P(f)` be the Fourier Transform of `p(t)`.

$$
P(f) = \mathcal{F}\{p(t)\} = \mathcal{F}\left\{\sum_{n=-\infty}^{\infty} \delta(t - nT_s)\right\}
$$

Using the properties of the Fourier Transform (specifically, the linearity and the FT of a single impulse), it can be shown that:

$$
P(f) = F_s \sum_{k=-\infty}^{\infty} \delta(f - kF_s)
$$

This is a crucial result: the spectrum of the sampling impulse train consists of impulses at integer multiples of the sampling frequency `F_s` (i.e., at `0, ±F_s, ±2F_s, ...`). The `F_s` multiplier scales the amplitude of these impulses.

Let's visualize this with a simple conceptual example. We can't plot Dirac Delta functions, but we can visualize their *locations* in the frequency domain.

```python
import numpy as np
import matplotlib.pyplot as plt

# Define a hypothetical sampling frequency
Fs = 1000  # Hz

# Create an array to represent frequency space
# For visualization, let's represent impulses as tall lines at specific points
frequencies = np.arange(-3 * Fs, 3 * Fs + 1, Fs)

# Create a 'spectrum' where impulses are at these frequencies
# We'll use a stem plot to represent impulses at discrete frequencies
# The amplitude Fs is conceptually there for the actual Fourier transform.
amplitudes = np.ones_like(frequencies) * Fs # Representing the strength of the impulses

plt.figure(figsize=(10, 4))
plt.stem(frequencies, amplitudes, use_line_collection=True, basefmt=" ")
plt.title(f'Conceptual Spectrum of Impulse Train (Fs = {Fs} Hz)')
plt.xlabel('Frequency (Hz)')
plt.ylabel('Amplitude (Scaled by Fs)')
plt.xticks(frequencies)
plt.grid(True)
plt.axhline(0, color='black', linewidth=0.5)
plt.show()
```

```output
A plot would appear showing vertical lines (stems) at frequencies:
-3000 Hz, -2000 Hz, -1000 Hz, 0 Hz, 1000 Hz, 2000 Hz, 3000 Hz.
All stems would have the same height, representing their amplitude.
This visualizes that the spectrum of the impulse train is itself an impulse train.
```

## Derivation Step 2: Fourier Transform of the Sampled Signal

Now we have `x_s(t) = x(t) \cdot p(t)`.
A fundamental property of the Fourier Transform is the **Convolution Theorem**:
Multiplication in the time domain corresponds to convolution in the frequency domain (scaled by `1/(2π)` if using angular frequency `ω`, or by `1` if using `Hz` frequency `f` and appropriate definitions of FT).

So, if `X(f)` is the Fourier Transform of `x(t)` and `P(f)` is the Fourier Transform of `p(t)`, then `X_s(f)`, the Fourier Transform of `x_s(t)`, is:

$$
X_s(f) = \mathcal{F}\{x(t) \cdot p(t)\} = X(f) * P(f)
$$

where `*` denotes convolution.

Substitute the expression for `P(f)` we found earlier:

$$
X_s(f) = X(f) * \left(F_s \sum_{k=-\infty}^{\infty} \delta(f - kF_s)\right)
$$

## Derivation Step 3: Interpreting the Convolution

Convolution with a sum of impulses is particularly enlightening. The convolution property states that `A(f) * δ(f - a) = A(f - a)`. This means convolving a function `A(f)` with an impulse located at `a` shifts `A(f)` to be centered at `a`.

Applying this to our equation:

$$
X_s(f) = F_s \sum_{k=-\infty}^{\infty} \left(X(f) * \delta(f - kF_s)\right)
$$

$$
X_s(f) = F_s \sum_{k=-\infty}^{\infty} X(f - kF_s)
$$

This is the most critical equation in the derivation! It tells us that the spectrum of the sampled signal, `X_s(f)`, is an infinite sum of shifted and scaled versions of the original signal's spectrum, `X(f)`.

Each replica of `X(f)` is centered at a multiple of the sampling frequency `kF_s` (`0, ±F_s, ±2F_s, ...`). The `F_s` factor scales the amplitude of these replicas.

Let's illustrate this. Suppose our original signal `x(t)` is a simple sine wave with frequency `f_m`. Its spectrum `X(f)` would ideally be two impulses at `±f_m`.

If `X(f)` were a more realistic, band-limited signal, its spectrum might look like a single "hump" centered at 0 (for a baseband signal) or two humps centered at `±f_m` (for a passband signal). For simplicity, let's assume `x(t)` is a baseband signal, meaning its significant frequency content lies between `-B` and `+B` Hz, where `B` is its maximum frequency component (bandwidth).

Consider a signal with a bandwidth `B`. Its spectrum `X(f)` exists only for `f` between `-B` and `B`.

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.fft import fft, fftfreq, fftshift

# --- Parameters ---
# Original signal: 100 Hz sine wave
f_signal = 100 # Hz
amplitude = 1

# Original signal duration (long enough for good freq resolution)
T_long = 1 # seconds
num_points_long = 2000 # points

# Create a 'continuous' time axis (high resolution)
t_long = np.linspace(0, T_long, num_points_long, endpoint=False)
x_t_long = amplitude * np.sin(2 * np.pi * f_signal * t_long)

# Compute FFT of the 'continuous' signal
X_f_long = fft(x_t_long)
frequencies_long = fftfreq(num_points_long, T_long / num_points_long)

# Shift zero frequency to the center for plotting
X_f_long_shifted = fftshift(X_f_long)
frequencies_long_shifted = fftshift(frequencies_long)

# --- Sampling Parameters ---
Fs_sampling = 1000 # Sampling frequency (Hz)
Ts_sampling = 1 / Fs_sampling # Sampling period

# Simulate sampling by taking fewer points at the sampling rate
# We need to take points over a duration that includes enough samples
# Let's sample for 0.1 seconds to get 100 samples
duration_sampled = 0.1 # seconds
num_samples = int(duration_sampled * Fs_sampling)
t_sampled = np.linspace(0, duration_sampled, num_samples, endpoint=False)
x_t_sampled = amplitude * np.sin(2 * np.pi * f_signal * t_sampled)

# Compute FFT of the sampled signal
X_f_sampled = fft(x_t_sampled)
frequencies_sampled = fftfreq(num_samples, Ts_sampling) # FFT frequencies for sampled signal

# Shift zero frequency to the center for plotting
X_f_sampled_shifted = fftshift(X_f_sampled)
frequencies_sampled_shifted = fftshift(frequencies_sampled)

# --- Plotting ---
plt.figure(figsize=(12, 8))

# Plot 1: Spectrum of Original Signal (conceptual)
plt.subplot(2, 1, 1)
# We plot the magnitude spectrum, normalized for better visualization
plt.plot(frequencies_long_shifted, np.abs(X_f_long_shifted) / num_points_long * 2) # *2 for single-sided equivalent
plt.title(f'Spectrum of Original Continuous Signal (f_signal = {f_signal} Hz)')
plt.xlabel('Frequency (Hz)')
plt.ylabel('Amplitude')
plt.xlim(-2*f_signal, 2*f_signal) # Zoom in to relevant frequencies
plt.grid(True)

# Plot 2: Spectrum of Sampled Signal
plt.subplot(2, 1, 2)
# We plot the magnitude spectrum, normalized and scaled by Fs_sampling as per derivation
# The 1/num_samples factor for FFT normalization implies the Fs scaling is implicit.
plt.plot(frequencies_sampled_shifted, np.abs(X_f_sampled_shifted) / num_samples * 2)
plt.title(f'Spectrum of Sampled Signal (Fs = {Fs_sampling} Hz)')
plt.xlabel('Frequency (Hz)')
plt.ylabel('Amplitude')
plt.xticks(np.arange(-2*Fs_sampling, 2*Fs_sampling + 1, f_signal)) # Help visualize replicas
plt.xlim(-2*Fs_sampling, 2*Fs_sampling) # Show several replicas
plt.grid(True)

plt.tight_layout()
plt.show()
```

```output
Two plots would appear:

1.  **Top Plot: Spectrum of Original Continuous Signal (f_signal = 100 Hz)**
    - This plot would show a clear, sharp peak (or two symmetric peaks for a real signal's FT)
      at 100 Hz (and -100 Hz), representing the single frequency component of the sine wave.
    - The amplitude of the peak would correspond to the signal's amplitude.

2.  **Bottom Plot: Spectrum of Sampled Signal (Fs = 1000 Hz)**
    - This plot would show multiple replicas of the original signal's spectrum.
    - Peaks would appear at:
        - 100 Hz (the original signal)
        - 900 Hz (Fs - f_signal = 1000 - 100)
        - 1100 Hz (Fs + f_signal = 1000 + 100)
        - 1900 Hz (2*Fs - f_signal = 2000 - 100)
        - 2100 Hz (2*Fs + f_signal = 2000 + 100)
        - And similarly for negative frequencies (-100 Hz, -900 Hz, -1100 Hz, etc.)
    - All these replicated peaks would have the same amplitude.

This visually confirms that sampling in the time domain creates periodic replicas of the original signal's spectrum in the frequency domain, centered at multiples of the sampling frequency.
```

## Derivation Step 4: Avoiding Aliasing - The Nyquist Condition

From the equation `X_s(f) = F_s \sum_{k=-\infty}^{\infty} X(f - kF_s)`, we see that the sampled signal's spectrum is a sum of shifted replicas of the original spectrum `X(f)`.

For us to be able to perfectly reconstruct the original signal `x(t)` from its samples, we need to be able to isolate the original spectrum `X(f)` (the `k=0` replica) from the others. This is typically done with an ideal low-pass filter, which would pass frequencies from `-B` to `B` and block everything else.

The crucial condition for this separation to be possible is that the replicas must **not overlap**. If they overlap, information from one replica will "fold over" into another, making it impossible to perfectly recover the original signal. This phenomenon is called **aliasing**.

Consider a band-limited signal `x(t)` whose spectrum `X(f)` is non-zero only for `|f| <= B`, where `B` is the highest frequency component (bandwidth) of the signal.

*   The original spectrum `X(f)` occupies the range `[-B, B]`.
*   The `k=1` replica `X(f - F_s)` occupies the range `[F_s - B, F_s + B]`.
*   The `k=-1` replica `X(f + F_s)` occupies the range `[-F_s - B, -F_s + B]`.

For no overlap between the `k=0` replica and the `k=1` replica, the highest frequency of the `k=0` replica (`B`) must be less than or equal to the lowest frequency of the `k=1` replica (`F_s - B`).

$$
B \le F_s - B
$$

Rearranging this inequality, we get:

$$
2B \le F_s
$$

Or, more commonly written:

$$
F_s \ge 2B
$$

This is the **Nyquist-Shannon Sampling Theorem**! It states that to perfectly reconstruct a band-limited signal with maximum frequency `B`, the sampling frequency `F_s` must be at least twice `B`.

*   The minimum sampling frequency required, `2B`, is known as the **Nyquist rate**.
*   The frequency `F_N = F_s / 2` is known as the **Nyquist frequency** (or folding frequency). Signals with frequency components higher than `F_N` will alias.

## Practical Implications: Aliasing

What happens if `F_s < 2B`? The replicas in the frequency domain will overlap. When this happens, the higher frequency components of the original signal 'fold back' into the lower frequency range, appearing as spurious low-frequency components. This is **aliasing**. Once aliasing occurs, the original signal cannot be perfectly recovered, regardless of how sophisticated your reconstruction filter is.

Let's demonstrate aliasing with a Python example. We'll sample a 700 Hz sine wave with a sampling frequency of 1000 Hz. According to Nyquist, `F_s = 1000 Hz` is only sufficient for signals up to `F_s / 2 = 500 Hz`. Our 700 Hz signal is beyond this.

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.fft import fft, fftfreq, fftshift

# --- Parameters ---
f_signal_aliased = 700 # Hz - This is > Fs/2
amplitude = 1

# Sampling Parameters - Same as before
Fs_aliasing = 1000 # Hz
Ts_aliasing = 1 / Fs_aliasing

duration_sampled_aliased = 0.1 # seconds
num_samples_aliased = int(duration_sampled_aliased * Fs_aliasing)
t_sampled_aliased = np.linspace(0, duration_sampled_aliased, num_samples_aliased, endpoint=False)
x_t_sampled_aliased = amplitude * np.sin(2 * np.pi * f_signal_aliased * t_sampled_aliased)

# Compute FFT of the aliased signal
X_f_aliased = fft(x_t_sampled_aliased)
frequencies_aliased = fftfreq(num_samples_aliased, Ts_aliasing)

# Shift zero frequency to the center for plotting
X_f_aliased_shifted = fftshift(X_f_aliased)
frequencies_aliased_shifted = fftshift(frequencies_aliased)

# Calculate the aliased frequency
# The actual frequency is f_signal_aliased
# The Nyquist frequency is Fs_aliasing / 2
# The aliased frequency will be Fs_aliasing - f_signal_aliased if f_signal > Nyquist freq
f_aliased_calculated = abs(Fs_aliasing - f_signal_aliased) # 1000 - 700 = 300 Hz

# --- Plotting ---
plt.figure(figsize=(10, 6))
plt.plot(frequencies_aliased_shifted, np.abs(X_f_aliased_shifted) / num_samples_aliased * 2)
plt.title(f'Spectrum of Aliased Signal (Original: {f_signal_aliased} Hz, Fs: {Fs_aliasing} Hz)')
plt.xlabel('Frequency (Hz)')
plt.ylabel('Amplitude')
plt.xticks(np.arange(-Fs_aliasing, Fs_aliasing + 1, 100)) # Fine-grained ticks
plt.xlim(-Fs_aliasing/2 - 200, Fs_aliasing/2 + 200) # Focus on the primary range
plt.axvline(x=Fs_aliasing/2, color='r', linestyle='--', label='Nyquist Frequency (Fs/2)')
plt.axvline(x=-Fs_aliasing/2, color='r', linestyle='--')
plt.axvline(x=f_aliased_calculated, color='g', linestyle=':', label=f'Aliased Freq ({f_aliased_calculated} Hz)')
plt.axvline(x=-f_aliased_calculated, color='g', linestyle=':')
plt.legend()
plt.grid(True)
plt.show()
```

```output
A plot would appear showing the spectrum of the aliased signal:

- The Nyquist frequency (Fs/2) is marked at 500 Hz.
- Instead of seeing a peak at 700 Hz (which is outside the Nyquist range [-500, 500]),
  a clear, dominant peak would be observed at 300 Hz (and -300 Hz).
- This 300 Hz peak is exactly `abs(Fs_aliasing - f_signal_aliased)` = `abs(1000 - 700)` = `300 Hz`.
- This demonstrates that the 700 Hz signal, when sampled at 1000 Hz,
  appears as a 300 Hz signal in the sampled spectrum. This is aliasing.
```

To prevent aliasing in real-world applications, an **anti-aliasing filter** (a low-pass filter) is always applied to the analog signal *before* it is sampled. This filter removes any frequency components above the Nyquist frequency (`F_s / 2`), ensuring that the input to the Analog-to-Digital Converter (ADC) is properly band-limited.

## Conclusion

The derivation of the Nyquist-Shannon Sampling Theorem from the Fourier Transform beautifully illustrates why a minimum sampling rate is necessary. By modeling sampling as a multiplication by an impulse train, we see that the spectrum of the sampled signal becomes a periodic replication of the original signal's spectrum. The core insight is that for perfect reconstruction, these spectral replicas must not overlap. This non-overlap condition directly leads to the `F_s >= 2B` rule.

For developers, this isn't just academic theory:

*   **Audio**: If you sample audio at 44.1 kHz (CD quality), you can perfectly capture frequencies up to 22.05 kHz, which covers the range of human hearing.
*   **Sensors**: When reading data from a temperature or pressure sensor, you need to sample fast enough to capture the fastest changes you expect to see.
*   **Image Processing**: In digital cameras, the pixel grid acts as a sampling mechanism. Undersampling can lead to Moiré patterns, a visual form of aliasing.

Understanding the "why" behind the Nyquist-Shannon theorem empowers you to debug sampling issues, correctly design data acquisition systems, and appreciate the magic that transforms continuous waves into discrete bits without losing vital information.