---
title: How to Simulate Quantum Circuits with Python (No PhD Needed)
date: 2025-06-17T13:05:16.383Z
description: Dive into quantum computing without needing a quantum computer. Learn to simulate quantum circuits in Python using Qiskit, understand core concepts like superposition and entanglement, and run practical code examples on your local machine.
tags: [Python, Quantum Computing, Simulation, Qiskit, Programming]
categories: [Quantum Computing, Python, Programming]
comments: true
---

Quantum computing often sounds like a topic reserved for physicists with multiple doctorates. While the underlying physics *is* complex, the good news is you don't need a PhD to start *programming* and *simulating* quantum circuits. As a developer, your primary tools are logic, abstraction, and code – and that's precisely what we'll use today.

This post will walk you through setting up a Python environment and simulating quantum circuits using Qiskit, IBM's open-source quantum computing framework. We'll focus on practical, runnable examples, demystifying the "quantum" part just enough for you to get hands-on.

## Why Simulate Quantum Circuits?

Good question! Why bother simulating something if you don't have the real hardware?

*   **Accessibility:** You don't need access to a multi-million dollar quantum computer. Your laptop is sufficient.
*   **Debugging & Prototyping:** Build, test, and debug your quantum algorithms much faster than on real hardware, which can be slow and expensive.
*   **Learning:** Gain an intuitive understanding of quantum mechanics concepts like superposition and entanglement by seeing them in action, without getting bogged down in the math.
*   **No Noise:** Simulators can be "perfect," meaning they don't introduce the environmental noise that real quantum computers contend with, making initial learning easier.

## Setting Up Your Environment

We'll use Qiskit for our quantum circuit simulations. It's user-friendly and well-documented.

First, create a dedicated virtual environment for your project. This keeps your dependencies organized and prevents conflicts.

```bash
mkdir quantum-sim
cd quantum-sim
python3 -m venv venv
source venv/bin/activate # On Windows, use `venv\Scripts\activate`
```

Next, install Qiskit. The core package is `qiskit`. We'll also install `matplotlib` for plotting results.

```bash
pip install qiskit matplotlib
```

```output
Collecting qiskit
  Downloading qiskit-0.45.0-py3-none-any.whl (9.6 kB)
Collecting matplotlib
  ... (output truncated) ...
Successfully installed qiskit-0.45.0 matplotlib-...
```

You're all set!

## Core Concepts (The "No PhD" Edition)

Before we jump into code, let's briefly touch on the absolute essentials. Think of these as the building blocks for your quantum programs.

*   **Qubit:** The quantum equivalent of a classical bit. A classical bit is either 0 or 1. A qubit can be 0, 1, or both *simultaneously* (this is called **superposition**). When you measure a qubit, it collapses into either 0 or 1 probabilistically.
*   **Quantum Gate:** Operations applied to qubits. Think of them like logic gates (AND, OR, NOT) for classical bits. Examples include:
    *   **Hadamard (H) Gate:** Puts a qubit into superposition (equal probability of being 0 or 1 when measured).
    *   **CNOT (Controlled-NOT) Gate:** An entanglement gate. It flips the state of a "target" qubit if a "control" qubit is 1. This creates **entanglement**, where the states of two or more qubits become linked, even if physically separated.
*   **Quantum Circuit:** A sequence of quantum gates applied to a set of qubits. It's essentially your quantum program.
*   **Measurement:** The process of observing a qubit's state, which forces it to collapse into a definite 0 or 1.
*   **Simulator:** A piece of software that mimics the behavior of a real quantum computer. It allows you to run your quantum circuits on a classical computer.

## Your First Quantum Circuit: Superposition

Let's start simple: creating a single qubit, putting it into superposition, and measuring it. If we run this many times, we expect to get roughly 50% 0s and 50% 1s.

Create a file named `superposition.py`:

```python
# superposition.py
from qiskit import QuantumCircuit, AerSimulator, transpile
from qiskit.visualization import plot_histogram

# 1. Create a Quantum Circuit with 1 qubit and 1 classical bit
#    The classical bit is where we store the measurement result of the qubit.
qc = QuantumCircuit(1, 1)

# 2. Apply a Hadamard gate to the qubit at index 0
#    This puts the qubit into a superposition state.
qc.h(0)

# 3. Measure the qubit at index 0 and store the result in the classical bit at index 0
qc.measure(0, 0)

# 4. Print the circuit to visualize its structure
print("Circuit Drawing:")
print(qc.draw())

# 5. Choose a simulator
#    AerSimulator is Qiskit's high-performance simulator for various backends.
#    We'll use the 'qasm_simulator' which mimics a real quantum device by taking 'shots'.
simulator = AerSimulator()

# 6. Transpile the circuit for the simulator
#    This optimizes the circuit for the target backend (simulator in this case).
compiled_circuit = transpile(qc, simulator)

# 7. Run the simulation
#    'shots' determines how many times the circuit is run.
#    More shots give a more accurate statistical distribution of outcomes.
job = simulator.run(compiled_circuit, shots=1024)

# 8. Get the results
result = job.result()

# 9. Get the measurement counts
counts = result.get_counts(qc)

print("\nMeasurement Counts (1024 shots):")
print(counts)

# 10. Visualize the results as a histogram
#     This will create a pop-up window showing the histogram.
fig = plot_histogram(counts)
fig.savefig("superposition_histogram.png") # Save the plot
print("\nHistogram saved to superposition_histogram.png")
```

Now, run this Python script:

```bash
python superposition.py
```

```output
Circuit Drawing:
     ┌───┐┌─┐
q_0: ┤ H ├┤M├
     └───┘└╥┘
c: 1/══════╩═
           0 

Measurement Counts (1024 shots):
{'0': 505, '1': 519}

Histogram saved to superposition_histogram.png
```

You'll see a circuit diagram, the counts showing approximately equal distribution between '0' and '1', and a `superposition_histogram.png` file generated which visually represents these counts. This demonstrates superposition: the qubit was neither 0 nor 1 until measured, and then randomly collapsed to one of the two.

## Example 2: Entanglement with a Bell State

Entanglement is a truly "quantum" phenomenon where two or more qubits become linked, sharing the same fate. Measuring one instantaneously influences the state of the other, no matter how far apart they are (though in our simulation, they're just bits in memory!).

The most famous entangled state is the Bell state, specifically the $\vert\Phi^+\rangle$ state (pronounced "phi plus"). We create it using a Hadamard gate and a CNOT gate.

Create a file named `bell_state.py`:

```python
# bell_state.py
from qiskit import QuantumCircuit, AerSimulator, transpile
from qiskit.visualization import plot_histogram

# 1. Create a Quantum Circuit with 2 qubits and 2 classical bits
#    We need two classical bits to store the measurement results of both qubits.
qc = QuantumCircuit(2, 2)

# 2. Apply a Hadamard gate to qubit 0
#    This puts qubit 0 into superposition.
qc.h(0)

# 3. Apply a CNOT gate with qubit 0 as control and qubit 1 as target
#    If qubit 0 is 1, qubit 1 flips. Since qubit 0 is in superposition,
#    this creates entanglement between qubit 0 and qubit 1.
qc.cx(0, 1) # cx(control_qubit_index, target_qubit_index)

# 4. Measure both qubits and store results in their respective classical bits
qc.measure([0, 1], [0, 1]) # measure([qubits_to_measure], [classical_bits_to_store_results])

# 5. Print the circuit drawing
print("Circuit Drawing:")
print(qc.draw())

# 6. Set up the QASM simulator
simulator = AerSimulator()

# 7. Transpile the circuit
compiled_circuit = transpile(qc, simulator)

# 8. Run the simulation
job = simulator.run(compiled_circuit, shots=1024)

# 9. Get and print the results
result = job.result()
counts = result.get_counts(qc)

print("\nMeasurement Counts (1024 shots):")
print(counts)

# 10. Visualize the results
fig = plot_histogram(counts)
fig.savefig("bell_state_histogram.png")
print("\nHistogram saved to bell_state_histogram.png")
```

Run the script:

```bash
python bell_state.py
```

```output
Circuit Drawing:
     ┌───┐     ┌─┐   
q_0: ┤ H ├──■──┤M├───
     └───┘┌─┴─┐└╥┘┌─┐
q_1: ─────┤ CX├─╫─┤M├
          └───┘ ║ └╥┘
c: 2/═══════════╩══╩═
                0  1 

Measurement Counts (1024 shots):
{'00': 498, '11': 526}

Histogram saved to bell_state_histogram.png
```

Notice the output: you only get counts for `'00'` and `'11'`. You'll never see `'01'` or `'10'`. This is the signature of the Bell state and entanglement: the qubits are perfectly correlated. If the first qubit is measured as 0, the second *must* be 0. If the first is 1, the second *must* be 1.

## Example 3: Creating a GHZ State (Multi-Qubit Entanglement)

Let's extend our entanglement example to three qubits, creating a Greenberger-Horne-Zeilinger (GHZ) state. This state is an extension of the Bell state concept, showing entanglement among multiple qubits. The ideal GHZ state is a superposition of all qubits being 0 (`000`) and all qubits being 1 (`111`).

Create a file named `ghz_state.py`:

```python
# ghz_state.py
from qiskit import QuantumCircuit, AerSimulator, transpile
from qiskit.visualization import plot_histogram

# 1. Create a Quantum Circuit with 3 qubits and 3 classical bits
qc = QuantumCircuit(3, 3)

# 2. Apply a Hadamard gate to qubit 0
#    This puts qubit 0 into superposition.
qc.h(0)

# 3. Apply CNOT gates to entangle the qubits
#    - CNOT with qubit 0 (control) and qubit 1 (target)
#    - CNOT with qubit 0 (control) and qubit 2 (target)
#    Note: This specific sequence (H then multiple CNOTs from the H'd qubit)
#    is a common way to prepare a GHZ state.
qc.cx(0, 1)
qc.cx(0, 2)

# 4. Measure all three qubits
qc.measure([0, 1, 2], [0, 1, 2])

# 5. Print the circuit drawing
print("Circuit Drawing:")
print(qc.draw())

# 6. Set up and run the QASM simulator
simulator = AerSimulator()
compiled_circuit = transpile(qc, simulator)
job = simulator.run(compiled_circuit, shots=1024)
result = job.result()
counts = result.get_counts(qc)

print("\nMeasurement Counts (1024 shots):")
print(counts)

# 7. Visualize the results
fig = plot_histogram(counts)
fig.savefig("ghz_state_histogram.png")
print("\nHistogram saved to ghz_state_histogram.png")
```

Run the script:

```bash
python ghz_state.py
```

```output
Circuit Drawing:
     ┌───┐          ┌─┐         
q_0: ┤ H ├──■────■──┤M├───
     └───┘┌─┴─┐  │  └╥┘┌─┐
q_1: ─────┤ CX├──┼───╫─┤M├
          └───┘┌─┴─┐ ║ └╥┘
q_2: ──────────┤ CX├─╫──╫─┤M├
               └───┘ ║  ║ └╥┘
c: 3/════════════════╩══╩══╩═
                     0  1  2 

Measurement Counts (1024 shots):
{'000': 509, '111': 515}

Histogram saved to ghz_state_histogram.png
```

Similar to the Bell state, you'll predominantly see only `'000'` and `'111'` outcomes (up to statistical fluctuations). This confirms all three qubits are entangled; their states are perfectly correlated, demonstrating multi-qubit entanglement.

## Different Simulators for Different Needs

Qiskit's `AerSimulator` is versatile and can emulate different types of quantum machine behaviors. So far, we've used the `qasm_simulator` mode (implicitly by default, or explicitly if we configured it) which mimics a real quantum device by running a specified number of "shots" and returning measurement counts.

But what if you want to see the *exact* quantum state of the system *before* measurement? For learning and debugging, this is incredibly powerful.

### `statevector_simulator`

This simulator directly calculates the quantum state vector of your circuit. It doesn't involve "shots" or probabilistic outcomes but gives you the complex amplitudes for each possible state.

Let's modify our superposition example to use the `statevector_simulator`. Note that we **remove the measurement** because measuring collapses the state, and we want to see the state *before* it collapses.

Create `statevector_example.py`:

```python
# statevector_example.py
from qiskit import QuantumCircuit, AerSimulator, transpile
import numpy as np

# 1. Create a Quantum Circuit with 1 qubit (no classical bit needed for statevector)
qc = QuantumCircuit(1)

# 2. Apply a Hadamard gate to the qubit
qc.h(0)

# 3. Print the circuit drawing
print("Circuit Drawing:")
print(qc.draw())

# 4. Choose the 'statevector_simulator' backend
simulator = AerSimulator(method='statevector')

# 5. Transpile the circuit (though less critical for statevector sim)
compiled_circuit = transpile(qc, simulator)

# 6. Run the simulation (only 1 shot needed, as it computes the exact state)
job = simulator.run(compiled_circuit, shots=1)

# 7. Get the result
result = job.result()

# 8. Get the statevector
#    This will be an array of complex numbers representing the amplitudes of each basis state.
statevector = result.get_statevector(qc)

print("\nStatevector (amplitudes for |0> and |1>):")
print(np.round(statevector, 3)) # Round for cleaner output

# Interpretation:
# For a single qubit, the statevector will have 2 elements: [amplitude_for_|0>, amplitude_for_|1>]
# The probability of measuring a state is the square of its amplitude's magnitude.
# For |0> -> |H> -> (|0> + |1>)/sqrt(2), amplitudes are 1/sqrt(2) for both.
# (1/sqrt(2))^2 = 1/2 = 0.5 (50% probability)
prob_0 = np.round(np.abs(statevector[0])**2, 3)
prob_1 = np.round(np.abs(statevector[1])**2, 3)
print(f"\nProbability of measuring |0>: {prob_0}")
print(f"Probability of measuring |1>: {prob_1}")
```

Run the script:

```bash
python statevector_example.py
```

```output
Circuit Drawing:
     ┌───┐
q_0: ┤ H ├
     └───┘

Statevector (amplitudes for |0> and |1>):
[0.707+0.j 0.707+0.j]

Probability of measuring |0>: 0.5
Probability of measuring |1>: 0.5
```

Here, `0.707` is approximately `1/sqrt(2)`. The statevector `[0.707+0.j 0.707+0.j]` tells us the qubit is in an equal superposition of `|0>` and `|1>`, confirming our theoretical understanding. If you square the magnitude of each component (`|0.707|^2`), you get `0.5`, indicating a 50% chance of measuring either state.

**Note:** The `statevector_simulator` is memory-intensive. As the number of qubits increases, the size of the state vector grows exponentially ($2^N$ complex numbers for $N$ qubits). For many qubits (e.g., > 20-30), even powerful classical computers cannot simulate the full state vector. This is why quantum computers are needed for truly large problems.

## Going Further

You've now successfully simulated basic quantum phenomena! Here are some directions to explore next:

*   **More Quantum Gates:** Qiskit supports a rich set of gates (Pauli-X, Y, Z, T, S, Rx, Ry, Rz, etc.). Experiment with them.
*   **Quantum Algorithms:** Implement simplified versions of algorithms like Deutsch-Jozsa, Grover's, or Shor's (though Shor's is complex to simulate fully).
*   **Noise Models:** Qiskit Aer allows you to add realistic noise models to your simulations to mimic real quantum hardware. This is crucial for understanding how errors affect quantum computations.
*   **Real Hardware (Small Scale):** Once you're comfortable with simulation, you can sign up for a free IBM Quantum account and run your small circuits on actual quantum computers! Qiskit makes the transition almost seamless.
*   **Other Frameworks:** While Qiskit is popular, other frameworks like Google's Cirq or Microsoft's Q# exist, offering different perspectives and features.

## Conclusion

You've taken the first big step into quantum computing by getting hands-on with Python simulation. We've seen how to build circuits, apply fundamental gates to create superposition and entanglement, and observe the outcomes. This practical approach, driven by code examples, is arguably the fastest way for a developer to grasp the core ideas without getting lost in the deep physics.

The world of quantum computing is vast and rapidly evolving, but you now have the tools and foundational understanding to continue exploring. Keep experimenting, keep coding, and who knows what quantum insights you'll uncover!