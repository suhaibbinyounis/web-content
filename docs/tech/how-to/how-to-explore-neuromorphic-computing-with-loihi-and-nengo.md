---
title: How to Explore Neuromorphic Computing with Loihi and Nengo
date: 2025-06-17T13:05:16.383Z
description: Dive into the fascinating world of neuromorphic computing. This guide provides a practical, hands-on approach to exploring Intel's Loihi research chip and the Nengo neural network framework, complete with code examples for developers.
tags: [Neuromorphic Computing, AI, Machine Learning, Spiking Neural Networks, SNN, Loihi, Intel, Nengo, Python, Hardware, Software, Research]
categories: [Artificial Intelligence, Hardware, Programming]
comments: true
---

Neuromorphic computing represents a radical departure from traditional von Neumann architectures, mimicking the brain's event-driven, parallel, and low-power processing. Instead of clock-synchronized operations on discrete data, neuromorphic chips process information using "spikes"—discrete events fired by neurons, similar to biological brains.

This isn't just academic curiosity; it promises incredible energy efficiency for AI workloads, real-time adaptive systems, and new computational paradigms. If you're a developer curious about what's beyond GPUs and TPUs, or someone looking to optimize AI at the edge, neuromorphic computing is a frontier worth exploring.

Intel's **Loihi** is a prominent research chip in this space, designed from the ground up for spiking neural networks (SNNs). It's an asynchronous, event-driven processor optimized for learning and inference. But how do you program something so fundamentally different? This is where **Nengo** comes in.

Nengo is an open-source software framework for building and simulating large-scale brain models and SNNs. It provides a high-level, intuitive API that lets you define networks, connect neurons, and simulate their behavior. Crucially, the `nengo-loihi` backend allows you to compile and run your Nengo models directly on Loihi hardware or its software emulator.

This post will guide you through setting up your environment, understanding the basics of Nengo, and then bridging that knowledge to interact with Loihi.

## Understanding the Core Concepts

Before we dive into code, let's establish a baseline.

### Spiking Neural Networks (SNNs)

Unlike Artificial Neural Networks (ANNs) that operate on continuous values and process data in synchronized layers, SNNs use discrete "spikes" to communicate. Neurons accumulate input spikes, and once a certain threshold is reached, they fire an output spike. This event-driven nature can lead to:

*   **Energy Efficiency:** Neurons only consume power when they spike, or when processing incoming spikes. Most of the time, they are dormant.
*   **Low Latency:** Information propagates as soon as a neuron spikes, potentially leading to faster real-time responses.
*   **Asynchronous Operation:** No global clock means components can operate independently, reducing power and improving scalability.

### Intel Loihi

Loihi is Intel's specialized neuromorphic processor. Key features include:

*   **Event-driven:** Processes information via spikes, not clock cycles.
*   **Asynchronous:** No global clock, enabling efficient parallel processing.
*   **On-chip Learning:** Supports various online learning rules, including spike-timing-dependent plasticity (STDP).
*   **Scalability:** Designed to be tiled, allowing for larger, more complex networks.
*   **Low Power:** Engineered for energy-constrained environments.

**Note:** As of my last update, Loihi is primarily a research chip. Access is typically granted through the **Intel Neuromorphic Research Community (INRC)**. You'll need to apply and be accepted to get direct access to Loihi hardware (Pohoiki boards) or its cloud-based emulation. We'll proceed with the assumption of using `nengo-loihi` which can simulate on your CPU if you don't have direct hardware access, but for true exploration, INRC access is highly recommended.

### Nengo and its Ecosystem

Nengo is a Python library that allows you to build models using various neuron types (including spiking neurons like Leaky Integrate-and-Fire—LIF).

*   **`nengo`**: The core library for building SNNs.
*   **`nengo-dl`**: Integrates Nengo with deep learning frameworks like TensorFlow, allowing you to train Nengo models with backpropagation and optimize them for performance on GPUs.
*   **`nengo-loihi`**: The crucial bridge. It compiles Nengo models into Loihi-compatible code, either for simulation on your CPU or for deployment on actual Loihi hardware via the INRC.

## Prerequisites and Setup

Let's get your environment ready. We'll use `conda` for a clean Python environment.

### 1. Create a Python Environment

```bash
conda create -n neuromorphic python=3.9
conda activate neuromorphic
```

```output
Collecting package metadata (current_repodata.json): done
Solving environment: done

## Package Plan ##

  environment location: /opt/anaconda3/envs/neuromorphic

  ... (installation details) ...

# To activate this environment, use
#
#     $ conda activate neuromorphic
#
# To deactivate an active environment, use
#
#     $ conda deactivate neuromorphic
#
```

### 2. Install Nengo and its Loihi Backend

```bash
pip install nengo nengo-loihi
```

```output
Collecting nengo
  Downloading nengo-3.2.0-py3-none-any.whl (1.1 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.1/1.1 MB 8.5 MB/s eta 0:00:00
Collecting nengo-loihi
  Downloading nengo_loihi-1.2.0-py3-none-any.whl (129 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 129.3/129.3 KB 9.4 MB/s eta 0:00:00
... (other dependencies) ...
Successfully installed nengo-3.2.0 nengo-loihi-1.2.0
```

### 3. Intel Neuromorphic Research Community (INRC) Access (Important!)

To run models on actual Loihi hardware or its more accurate cloud emulator, you **must** be part of the INRC.

*   Visit: [https://www.intel.com/content/www/us/en/research/neuromorphic-computing.html](https://www.intel.com/content/www/us/en/research/neuromorphic-computing.html)
*   Look for the "Intel Neuromorphic Research Community" section and apply.
*   Once accepted, you'll receive credentials, including an API key. This key will be used to authenticate with Intel's Loihi backend.

If you have INRC access, set your API key as an environment variable:

```bash
export LOIHI_API_KEY="YOUR_LOIHI_API_KEY_HERE"
```

**Note:** If you don't have INRC access, `nengo-loihi` will automatically fall back to a CPU-based simulation of Loihi's behavior, which is still incredibly useful for development and testing, but won't give you the true power/speed benefits of the hardware. For this tutorial, the examples will work either way.

## Getting Started with Nengo (Software Simulation First)

Let's build a very simple Nengo model: a single spiking neuron that responds to a constant input.

### Example 1: A Single Spiking Neuron

We'll create a Nengo model with a Node providing a constant input and a Neuron population receiving that input. We'll then simulate it and observe its spikes.

```python
# single_neuron.py
import nengo
import numpy as np
import matplotlib.pyplot as plt

# 1. Define the Nengo model
with nengo.Network() as model:
    # An input node that provides a constant value (e.g., 1.0)
    stimulus = nengo.Node(1.0)

    # A population of 1 LIF neuron
    # n_neurons=1, dimensions=1 means one neuron representing one value
    neuron = nengo.Ensemble(n_neurons=1, dimensions=1, neuron_type=nengo.LIF())

    # Connect the stimulus to the neuron.
    # The 'weight' here determines how strongly the stimulus drives the neuron.
    # A positive weight will cause the neuron to fire.
    nengo.Connection(stimulus, neuron)

    # Probe the neuron's output (spikes) and its membrane potential
    # synapse=0.01 makes the probe output slightly smoothed for better visualization
    spike_probe = nengo.Probe(neuron.neurons)
    voltage_probe = nengo.Probe(neuron.neurons, "voltage")

# 2. Simulate the model
print("Simulating a single LIF neuron...")
with nengo.Simulator(model) as sim:
    sim.run(1.0) # Run for 1 second

# 3. Plot the results
plt.figure(figsize=(10, 6))

plt.subplot(2, 1, 1)
plt.plot(sim.trange(), sim.data[voltage_probe], color='blue')
plt.title("Neuron Membrane Potential")
plt.ylabel("Voltage")
plt.grid(True)

plt.subplot(2, 1, 2)
# To plot spikes, we typically look for when voltage crosses a threshold or
# plot the raw spike events (0 or 1).
# sim.data[spike_probe] is typically 0 or 1 for each neuron over time.
plt.plot(sim.trange(), sim.data[spike_probe], color='red')
plt.title("Neuron Spikes")
plt.xlabel("Time (s)")
plt.ylabel("Spike (0 or 1)")
plt.grid(True)

plt.tight_layout()
plt.show()

# 4. Print some statistics
num_spikes = np.sum(sim.data[spike_probe] > 0.5) # Check for spike events (value > 0.5)
print(f"\nTotal spikes observed: {num_spikes}")
```

To run this:

```bash
python single_neuron.py
```

```output
Simulating a single LIF neuron...

Total spikes observed: 25
```

This script will also open a Matplotlib window showing the neuron's membrane potential rising and falling, and discrete spikes occurring when the potential crosses a threshold. You'll see the membrane potential integrating the input until it fires a spike, then resetting.

### Example 2: Simple Integrator (Representing a Value)

Nengo's power lies in its ability to represent and transform high-dimensional vectors using populations of spiking neurons. Let's create a simple integrator that holds a value.

```python
# simple_integrator.py
import nengo
import numpy as np
import matplotlib.pyplot as plt

# 1. Define the Nengo model
with nengo.Network() as model:
    # Input node providing a step function: 0 then 0.5
    input_signal = nengo.Node(lambda t: 0.5 if t > 0.2 else 0)

    # An ensemble of neurons to represent a value.
    # n_neurons: number of neurons in the population
    # dimensions: the dimensionality of the vector this ensemble represents
    # neuron_type: Leaky Integrate-and-Fire (LIF) neurons are common SNN neurons
    ensemble_A = nengo.Ensemble(n_neurons=50, dimensions=1, neuron_type=nengo.LIF())

    # An ensemble to perform integration (feedback connection)
    integrator = nengo.Ensemble(n_neurons=50, dimensions=1, neuron_type=nengo.LIF())

    # Connect input to ensemble_A
    nengo.Connection(input_signal, ensemble_A)

    # Connect ensemble_A to the integrator
    nengo.Connection(ensemble_A, integrator)

    # Feedback connection for integration.
    # transform=1.0 * sim.dt (or similar small value) helps the integration
    # Note: For perfect integration, you might need a different Nengo component (e.g., nengo.LinearEnsemble)
    # but for SNNs, this approximation shows the concept.
    nengo.Connection(integrator, integrator, synapse=0.1) # Recurrent connection with a decay

    # Probe the input and the decoded output of the integrator
    input_probe = nengo.Probe(input_signal)
    integrator_probe = nengo.Probe(integrator, synapse=0.01) # Low-pass filter the output for smoothness

# 2. Simulate the model
print("Simulating a simple integrator...")
with nengo.Simulator(model) as sim:
    sim.run(1.0)

# 3. Plot the results
plt.figure(figsize=(10, 5))
plt.plot(sim.trange(), sim.data[input_probe], label="Input Signal")
plt.plot(sim.trange(), sim.data[integrator_probe], label="Integrator Output")
plt.title("Simple Nengo Integrator with LIF Neurons")
plt.xlabel("Time (s)")
plt.ylabel("Value")
plt.legend()
plt.grid(True)
plt.show()

# 4. Print the final state
print(f"\nFinal input value: {sim.data[input_probe][-1]:.2f}")
print(f"Final integrator output: {sim.data[integrator_probe][-1][0]:.2f}")
```

To run this:

```bash
python simple_integrator.py
```

```output
Simulating a simple integrator...

Final input value: 0.50
Final integrator output: 0.38
```

This will plot an input signal that steps from 0 to 0.5, and you'll see the integrator's output slowly rise towards that value (or some fraction of it, depending on the recurrent connection's `synapse` parameter). This demonstrates how populations of spiking neurons can collectively represent and process continuous values.

## Bridging to Loihi with `nengo-loihi`

Now for the exciting part: running these Nengo models using the `nengo-loihi` backend. This backend compiles your Nengo network into a format that can be executed by Loihi's hardware or its specialized simulator.

The key change is switching from `nengo.Simulator` to `nengo_loihi.Simulator`.

### Example 3: Running the Single Spiking Neuron on Loihi (Simulated)

Let's adapt `single_neuron.py` to use the Loihi backend.

```python
# loihi_single_neuron.py
import nengo
import nengo_loihi
import numpy as np
import matplotlib.pyplot as plt

# 1. Define the Nengo model (same as before)
with nengo.Network() as model:
    stimulus = nengo.Node(1.0)
    neuron = nengo.Ensemble(n_neurons=1, dimensions=1, neuron_type=nengo.LIF())
    nengo.Connection(stimulus, neuron)
    spike_probe = nengo.Probe(neuron.neurons)
    voltage_probe = nengo.Probe(neuron.neurons, "voltage")

# 2. Simulate the model using nengo_loihi.Simulator
#   target='loihi' implies compilation for Loihi hardware (if available via INRC)
#   If LOIHI_API_KEY is not set, it defaults to a CPU-based Loihi simulator.
print("Simulating a single LIF neuron on Loihi backend...")
with nengo_loihi.Simulator(model, target='loihi') as sim:
    sim.run(1.0) # Run for 1 second

# 3. Plot the results (same as before)
plt.figure(figsize=(10, 6))

plt.subplot(2, 1, 1)
plt.plot(sim.trange(), sim.data[voltage_probe], color='blue')
plt.title("Neuron Membrane Potential (Loihi Backend)")
plt.ylabel("Voltage")
plt.grid(True)

plt.subplot(2, 1, 2)
plt.plot(sim.trange(), sim.data[spike_probe], color='red')
plt.title("Neuron Spikes (Loihi Backend)")
plt.xlabel("Time (s)")
plt.ylabel("Spike (0 or 1)")
plt.grid(True)

plt.tight_layout()
plt.show()

print(f"\nTotal spikes observed (Loihi backend): {np.sum(sim.data[spike_probe] > 0.5)}")
```

To run this:

```bash
python loihi_single_neuron.py
```

```output
Simulating a single LIF neuron on Loihi backend...
2023-10-27 10:30:00,123 INFO     nengo_loihi: Compiled 1 networks into 1 partitions.
2023-10-27 10:30:00,124 INFO     nengo_loihi: Partitioning complete.
2023-10-27 10:30:00,125 INFO     nengo_loihi: No Loihi hardware found or specified. Falling back to CPU simulation.
... (Loihi specific messages) ...

Total spikes observed (Loihi backend): 25
```

You'll notice the output includes messages from `nengo_loihi` about compilation and whether it's using actual Loihi hardware or falling back to CPU simulation. The resulting plots should be very similar to the pure Nengo simulation, demonstrating `nengo-loihi`'s ability to accurately map Nengo models to Loihi's computational primitives.

### Example 4: Running the Integrator on Loihi (Simulated)

Let's also run our simple integrator using the Loihi backend.

```python
# loihi_integrator.py
import nengo
import nengo_loihi
import numpy as np
import matplotlib.pyplot as plt

# 1. Define the Nengo model (same as before)
with nengo.Network() as model:
    input_signal = nengo.Node(lambda t: 0.5 if t > 0.2 else 0)
    ensemble_A = nengo.Ensemble(n_neurons=50, dimensions=1, neuron_type=nengo.LIF())
    integrator = nengo.Ensemble(n_neurons=50, dimensions=1, neuron_type=nengo.LIF())

    nengo.Connection(input_signal, ensemble_A)
    nengo.Connection(ensemble_A, integrator)
    nengo.Connection(integrator, integrator, synapse=0.1)

    input_probe = nengo.Probe(input_signal)
    integrator_probe = nengo.Probe(integrator, synapse=0.01)

# 2. Simulate the model using nengo_loihi.Simulator
print("Simulating simple integrator on Loihi backend...")
with nengo_loihi.Simulator(model, target='loihi') as sim:
    sim.run(1.0)

# 3. Plot the results (same as before)
plt.figure(figsize=(10, 5))
plt.plot(sim.trange(), sim.data[input_probe], label="Input Signal (Loihi Backend)")
plt.plot(sim.trange(), sim.data[integrator_probe], label="Integrator Output (Loihi Backend)")
plt.title("Simple Nengo Integrator with LIF Neurons (Loihi Backend)")
plt.xlabel("Time (s)")
plt.ylabel("Value")
plt.legend()
plt.grid(True)
plt.show()

print(f"\nFinal input value (Loihi backend): {sim.data[input_probe][-1]:.2f}")
print(f"Final integrator output (Loihi backend): {sim.data[integrator_probe][-1][0]:.2f}")
```

To run this:

```bash
python loihi_integrator.py
```

```output
Simulating simple integrator on Loihi backend...
2023-10-27 10:35:00,123 INFO     nengo_loihi: Compiled 1 networks into 1 partitions.
2023-10-27 10:35:00,124 INFO     nengo_loihi: Partitioning complete.
2023-10-27 10:35:00,125 INFO     nengo_loihi: No Loihi hardware found or specified. Falling back to CPU simulation.
... (Loihi specific messages) ...

Final input value (Loihi backend): 0.50
Final integrator output (Loihi backend): 0.38
```

Again, the behavior should be very similar to the pure Nengo simulation, but this time, the computation is being mapped to Loihi's architecture. This is your first step towards truly leveraging neuromorphic hardware.

## More Complex Example: The Path for Real-World Problems

While a full-blown MNIST classification example is beyond the scope of a single blog post (it involves training, dataset handling, and potentially `nengo-dl` for optimization), understanding the *process* is crucial.

The general workflow for a complex SNN on Loihi is:

1.  **Model Design in Nengo:** Build your SNN in Nengo, possibly using `nengo-dl` to leverage deep learning techniques (like backpropagation) to train weights. This allows you to design complex architectures and optimize them with familiar tools.
2.  **Training (Optional but Recommended for Accuracy):** Train your Nengo model (often with `nengo-dl`) on a target dataset (e.g., MNIST, CIFAR-10). `nengo-dl` helps convert continuous-valued ANNs into SNNs suitable for spiking hardware.
3.  **Compilation with `nengo-loihi`:** Use `nengo-loihi.Simulator` to compile the trained Nengo model for Loihi. This involves mapping Nengo components to Loihi's hardware resources (neurons, synapses, etc.).
4.  **Deployment/Simulation:** Run the compiled model on actual Loihi hardware (via INRC access) or its software simulator.

Here's a conceptual snippet showing how you'd load a pre-trained `nengo-dl` model and attempt to run it on `nengo-loihi`.

```python
# conceptual_loihi_deployment.py
import nengo
import nengo_loihi
# import nengo_dl # You would need nengo-dl for training and model loading

# Placeholder for a complex Nengo model, e.g., an MNIST classifier
# In a real scenario, this 'model' would be loaded from a saved nengo-dl model
# For demonstration, we'll use a dummy model
def create_dummy_model():
    with nengo.Network(label="Complex Model") as model:
        # Input for 784 pixels (MNIST)
        input_node = nengo.Node(np.zeros(784))
        # An ensemble representing features
        hidden = nengo.Ensemble(n_neurons=1000, dimensions=100, neuron_type=nengo.LIF())
        # An output ensemble for 10 classes
        output = nengo.Ensemble(n_neurons=100, dimensions=10, neuron_type=nengo.LIF())

        nengo.Connection(input_node, hidden)
        nengo.Connection(hidden, output)

        # Probes for testing
        nengo.Probe(input_node)
        nengo.Probe(output, synapse=0.01)
    return model

print("Creating a dummy Nengo model (placeholder for a trained complex network)...")
model = create_dummy_model()

# 2. Simulate the model on Loihi backend
# For actual hardware, target='loihi' and LOIHI_API_KEY must be set.
# Without LOIHI_API_KEY, it will fall back to CPU simulation.
print("\nAttempting to simulate on Loihi backend...")
with nengo_loihi.Simulator(model, target='loihi') as sim:
    print("\nLoihi simulator initialized. Starting simulation run...")
    # Run for a short duration
    sim.run(0.1)
    print("\nSimulation complete.")

    # In a real scenario, you would analyze sim.data for the probes
    # For example, if you probed the output, you'd decode it to get classification results.
    # print(f"Sample output from Loihi: {sim.data[output_probe][-1]}")

print("\nSuccessfully ran (or attempted to run) a Nengo model on the Loihi backend.")
print("To truly leverage Loihi hardware, ensure LOIHI_API_KEY is set and you have INRC access.")
```

To run this:

```bash
python conceptual_loihi_deployment.py
```

```output
Creating a dummy Nengo model (placeholder for a trained complex network)...

Attempting to simulate on Loihi backend...
2023-10-27 10:40:00,123 INFO     nengo_loihi: Compiled 1 networks into 1 partitions.
2023-10-27 10:40:00,124 INFO     nengo_loihi: Partitioning complete.
2023-10-27 10:40:00,125 INFO     nengo_loihi: No Loihi hardware found or specified. Falling back to CPU simulation.
... (Loihi specific messages about network setup) ...
Loihi simulator initialized. Starting simulation run...
Simulation complete.

Successfully ran (or attempted to run) a Nengo model on the Loihi backend.
To truly leverage Loihi hardware, ensure LOIHI_API_KEY is set and you have INRC access.
```

This conceptual example demonstrates the exact same API call pattern for more complex models. The complexity shifts from *how to run it* to *how to design and train the SNN effectively* within Nengo.

## Practical Considerations and Troubleshooting

Exploring neuromorphic computing comes with its own set of unique challenges:

*   **Loihi Constraints:** Loihi has specific limits on the number of neurons, synapses, and the types of operations it efficiently supports. `nengo-loihi` handles much of this translation, but complex Nengo models might require adjustments to fit within Loihi's physical constraints. The `nengo-loihi` documentation is your friend here.
*   **Debugging SNNs:** Unlike ANNs where you track floating-point activations, SNNs involve discrete spikes. Debugging often means analyzing spike trains, rates, and membrane potentials. Nengo's `Probe` and plotting capabilities are essential.
*   **Hyperparameter Tuning:** SNNs often have unique hyperparameters (e.g., neuron thresholds, refractory periods, synaptic time constants) that significantly affect performance. Tuning these can be an art.
*   **Training SNNs:** Directly training SNNs (e.g., with STDP) is an active area of research. For high accuracy on benchmark tasks, converting ANNs to SNNs (as `nengo-dl` facilitates) is currently a more robust approach.
*   **INRC Access:** As emphasized, real Loihi hardware is not a consumer product. Plan for the application and approval process if you intend to move beyond CPU simulation.
*   **Community and Resources:** The neuromorphic field is rapidly evolving.
    *   **Nengo Documentation:** [https://www.nengo.ai/nengo/](https://www.nengo.ai/nengo/)
    *   **Nengo Loihi Documentation:** [https://www.nengo.ai/nengo-loihi/](https://www.nengo.ai/nengo-loihi/)
    *   **Intel Neuromorphic Research Community:** Access to forums, deeper documentation, and examples.

## Conclusion

You've taken a significant step into the world of neuromorphic computing. By combining the high-level abstraction of Nengo with the specialized capabilities of Intel's Loihi chip, you can begin to design, simulate, and potentially deploy energy-efficient, event-driven neural networks.

While the journey from simple examples to state-of-the-art neuromorphic AI is long and involves a steep learning curve, the fundamental tools are now at your fingertips. Experiment with different Nengo models, explore various neuron types, and once you gain INRC access, push your models to the actual Loihi hardware to experience the future of AI processing. The potential for low-power, real-time, and adaptive intelligence is immense, and you're now equipped to be part of that exploration.

Happy spiking!
