# VQE for the H₂O Ground-State Energy

A Variational Quantum Eigensolver (VQE) implementation in [PennyLane](https://pennylane.ai) for computing the ground-state energy of the water molecule, comparing different circuit ansätze, optimizers, and the effect of gate noise.

## Overview

The molecular Hamiltonian for H₂O is built with an active space of **2 electrons in 3 orbitals** (STO-3G basis), mapping the problem onto **6 qubits** via `qml.qchem.molecular_hamiltonian`. The exact ground-state energy is obtained by direct diagonalization of the Hamiltonian matrix and used as a reference to evaluate every VQE run.

## Ansätze tested

| Ansatz | Description | Qubits | Notes |
|---|---|---|---|
| **DoubleExcitation** | Chemistry-inspired native gate, single variational parameter | 4 | Physically motivated, minimal parameter count |
| **StronglyEntanglingLayers** | PennyLane's general-purpose hardware-efficient template | 6 | Initialized from the Hartree-Fock state |
| **RY + CZ (custom)** | Hand-built layered circuit: RY rotations + CZ entangling chain | 6 | 3 layers, initialized from the Hartree-Fock state |
| **RY + CZ (noisy)** | Same structure as above, with a `DepolarizingChannel` after every gate | 6 | Run on `default.mixed` to test noise resilience |

## Optimization

Each ansatz is trained for 100 iterations, comparing two optimizers:
- **Adam** (`qml.AdamOptimizer`, stepsize 0.1)
- **Gradient Descent** (`qml.GradientDescentOptimizer`, stepsize 0.4)

The VQE energy at each iteration is plotted against the exact (diagonalized) ground-state energy to visualize convergence.

## Results

The notebook reports, for each ansatz/optimizer combination:
- Final VQE energy (Hartree)
- Absolute error with respect to the exact ground-state energy

See `VQE_H2O.ipynb` for the full convergence plots and final numerical results.

## Requirements

```
pennylane
numpy
matplotlib
```

Install with:
```bash
pip install pennylane numpy matplotlib
```

## Reproducing the results

```bash
git clone https://github.com/L1me26/vqe-h2o-pennylane.git
cd vqe-h2o-pennylane
pip install -r requirements.txt
jupyter notebook VQE_H2O.ipynb
```

## Notes / possible extensions

- The `DoubleExcitation` ansatz currently uses a fixed 4-wire subset rather than the full active space and is not initialized from the Hartree-Fock state — a natural next step is to align it with the other ansätze for a fairer comparison.
- Noise is only tested on the RY+CZ ansatz; extending the noise study to the other ansätze would help isolate whether robustness comes from the ansatz structure or the amount of entanglement.
- A natural next step would be benchmarking `QNGOptimizer` (quantum natural gradient) against Adam and standard gradient descent.