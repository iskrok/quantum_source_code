# Quantum Secret Sharing Protocol Implementations

This repository contains implementations of Quantum Secret Sharing (QSS) protocols developed during research in quantum cryptography and quantum information theory.

The implementations use Qiskit for quantum circuit simulations and execution on IBM Quantum processors.

## Projects

### 1. GHZ-based Quantum Secret Sharing Protocol

Notebook: `qss_ghz_state.ipynb`

Implementation of a Modification of Quantum Secret Sharing protocol based on GHZ entangled states.

This implementation was developed as part of my specialist's thesis and presented at:

**E.V. Armensky Annual Interuniversity Scientific-Technical Conference of Young Researchers (2026)**

---

### 2. Bell-state-based Quantum Secret Sharing Protocol

Notebook: `qss_bell_state.ipynb`

Implementation of a Quantum Secret Sharing protocol based on Bell states.

This work was developed as part of my specialist's thesis and explores an alternative approach to quantum secret sharing using entangled quantum states.

---

### 3. W-state-based Quantum Secret Sharing Protocol

Notebook: [w_state_qss.ipynb](w_state_qss/w_state_qss.ipynb)

Implementation of a new three-party QSS protocol based on a three-qubit W state for distributing a predetermined sequence of classical bits.

Includes Qiskit Aer simulations, IBM Quantum measurements from Kingston, Fez and Marrakesh and analysis of intercept-and-resend and entangle-and-measure attacks.

---

## License

This repository is licensed under the [MIT License](LICENSE).
