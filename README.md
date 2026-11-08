# Deutsch's Algorithm with Qiskit

This project demonstrates **Deutsch's algorithm** in Qiskit. The algorithm
determines whether a one-bit function is **constant** or **balanced** with
only one evaluation of the function.

- A **constant** function returns the same value for both inputs:
  `f(0) = f(1)`.
- A **balanced** function returns different values:
  `f(0) != f(1)`.

The notebook is [main.ipynb](./main.ipynb).

## Run in Google Colab

Run this setup cell first:

```python
%pip install qiskit qiskit_aer matplotlib pylatexenc
```

Then run the notebook cells from top to bottom. `pylatexenc` is required for
Qiskit's Matplotlib circuit drawer.

## What the notebook does

### `daut_oracle(qc, f_type)`

This function adds one of four possible oracle circuits to `qc`. The oracle
represents a hidden function `f`.

| `f_type` | Oracle gates | Function | Result |
|---|---|---|---|
| `1` | None | `f(x) = 0` | Constant |
| `2` | `X` on qubit 1 | `f(x) = 1` | Constant |
| `3` | `CX(0, 1)` | `f(x) = x` | Balanced |
| `4` | `X(0)`, then `CX(0, 1)` | `f(x) = NOT x` | Balanced |

The notebook uses qubit `0` as the input qubit and qubit `1` as the target
qubit. `X` flips a qubit. `CX(0, 1)` flips the target only when input qubit
`0` is `|1>`.

### `deus_algo(f_type)`

This function creates the complete Deutsch circuit:

```python
def deus_algo(f_type):
    qc = qsk.QuantumCircuit(2, 1)
    qc.x(1)
    qc.h(0)
    qc.h(1)
    daut_oracle(qc, f_type)
    qc.h(0)
    qc.measure(0, 0)
    return qc
```

The steps are:

1. Create two qubits and one classical bit.
2. Apply `X` to the target, preparing it in `|1>`.
3. Apply Hadamard gates to create superposition.
4. Apply the selected oracle.
5. Apply a final Hadamard to the input qubit.
6. Measure the input qubit.

The target is prepared as `|->`, allowing the oracle to encode the function
as a phase through **phase kickback**. Interference at the final Hadamard
reveals whether the function is constant or balanced.

## Simulating the circuit

```python
qc = deus_algo(3)
simulator = Aer.get_backend("qasm_simulator")
compiled_circuit = transpile(qc, simulator)
job = simulator.run(compiled_circuit, shots=1024)
result = job.result()
counts = result.get_counts(qc)
print(counts)
```

For `f_type=3`, the function is balanced, so the expected result is normally:

```python
{'1': 1024}
```

For constant functions (`f_type=1` or `f_type=2`), the expected result is
`{'0': 1024}`. The measurement means:

- `0` = constant
- `1` = balanced

Real quantum hardware can produce occasional unexpected results because of
noise.

## Visualizing results

The histogram displays measurement counts:

```python
plot_histogram(counts)
plt.show()
```

The circuit can be drawn with Matplotlib:

```python
figure = qc.draw(output="mpl")
plt.show()
```

## Real-world examples

These four oracle types are simplified examples of classifying an unknown
black-box rule:

### 1. Constant sensor status

If a hidden system receives either `0` or `1` but always reports `0`, it is
`f(x) = 0` (`f_type=1`). This models a sensor or service that always reports
the same status.

### 2. Constant authorization response

If a service always returns `1`, regardless of the user category, it is
`f(x) = 1` (`f_type=2`). The response does not depend on the input.

### 3. Direct input-dependent rule

```text
input 0 -> output 0
input 1 -> output 1
```

This is `f(x) = x` (`f_type=3`). It models a feature flag or classifier that
copies the selected mode and is balanced.

### 4. Inverted input-dependent rule

```text
input 0 -> output 1
input 1 -> output 0
```

This is `f(x) = NOT x` (`f_type=4`). It models an inverted status flag and is
also balanced.

These examples are educational rather than production applications. They
show how quantum interference can classify a hidden binary rule in the
algorithm's query model.

## Project structure

```text
.
├── main.ipynb   # Deutsch's algorithm, simulation, and visualizations
└── README.md    # Project documentation
```
