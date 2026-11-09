# Three-party W-state quantum secret sharing

This folder contains the Qiskit implementation and IBM Quantum measurement data for a three-party quantum secret sharing protocol based on a three-qubit W state. The protocol distributes a predetermined sequence of classical bits. Alice's Hadamard-gate choices, together with the measurement bases independently chosen by Bob and Charlie, determine the information, test and semi-information rounds.

The notebook evaluates protocol efficiency, intercept-and-resend and entangle-and-measure attacks, ancilla information and a calibrated statistical criterion for noisy hardware measurements.

## Files

| Path | Contents |
| --- | --- |
| `w_state_qss.ipynb` | Protocol implementation, numerical experiments and figure generation |
| `requirements.txt` | Python dependencies |
| `qss_measurements/efficiency_<backend>.json` | No-attack measurements for efficiency and reconstruction-error estimates |
| `qss_measurements/ir_<backend>.json` | Intercept-and-resend measurements and their no-attack control |
| `qss_measurements/em_<backend>.json` | Entangle-and-measure measurements and their separate no-attack control |
| `qss_measurements/information_<backend>.json` | Auxiliary ancilla-information measurements |
| `qss_figures/` | Output directory, created automatically |

The notebook filename above is the release name. If your copy has a different name, open that notebook instead. Replace `<backend>` with the full processor name: `ibm_kingston`, `ibm_fez` or `ibm_marrakesh`. For example, Fez intercept-and-resend measurements must be named `ir_ibm_fez.json`. Remove download suffixes such as `(5)` from filenames.

## Installation

Use Python 3.12. The numerical experiments were validated with Python 3.12.14 and the numerical-library versions pinned in `requirements.txt`. JupyterLab and IPython kernel versions are specified as ranges because they provide the notebook interface rather than the numerical model. The requirements file specifies the main dependencies; it is not a complete lockfile for their transitive dependencies.

Open a terminal in this protocol's folder, where the notebook, `requirements.txt` and `qss_measurements/` are located. If this folder is part of a larger repository, enter the protocol folder first.

Create and activate a virtual environment.

**Windows PowerShell**

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**Linux or macOS**

```bash
python3.12 -m venv .venv
source .venv/bin/activate
```

Then install the dependencies and register the kernel:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m ipykernel install --user --name w-state-qss --display-name "Python (W-state QSS)"
```

Start JupyterLab:

```bash
jupyter lab
```

## Running with the supplied IBM measurements

The measurement collection includes data from **Kingston, Fez and Marrakesh**. The notebook analyzes one processor at a time; Kingston is the default. To choose another processor, change `IBM_BACKEND` in the first code cell before executing the experiments:

```python
IBM_BACKEND = "ibm_kingston"  # Alternatives: "ibm_fez" or "ibm_marrakesh"
```

1. Check that all four JSON files for the selected processor are present in `qss_measurements/`.
2. Open the notebook and select the **Python (W-state QSS)** kernel. This is the kernel display name created by the registration command above, not a filename.
3. Restart the kernel and run all cells in their displayed order. When changing processors, repeat the run from a fresh kernel to avoid mixing intermediate results.

All paths are relative to the notebook kernel's working directory. With all four valid measurement files for the selected processor, the notebook loads the IBM counts without connecting to IBM Quantum or submitting new hardware jobs. An IBM account is not needed for this route.

Aer simulations and the illustrative protocol run are executed again. Figures are saved as ten vector PDFs in `qss_figures/`, from `00_example_round.pdf` through `09_em_cumulative.pdf`. Rerunning figure cells replaces files with the same names. The figure filenames do not include the processor name, so copy the output directory elsewhere before running another processor if you want to retain both sets.

**If a required IBM dataset is missing, the acquisition cell automatically attempts to obtain it from IBM Quantum and saves the returned measurements.** A complete notebook run therefore requires all four files for the selected processor to avoid new jobs, even when some of that processor's datasets are already present. New acquisition uses the resources available through the configured IBM account. An invalid or incomplete dataset raises an error rather than silently replacing it.

## Obtaining new IBM Quantum measurements

This step is only needed when acquiring new hardware data. The notebook uses `QiskitRuntimeService()` to load a saved account and obtains the backend named by `IBM_BACKEND`.

Save your own credentials once in a trusted local Python session, using the same environment:

```python
from getpass import getpass
from qiskit_ibm_runtime import QiskitRuntimeService

QiskitRuntimeService.save_account(
    token=getpass("IBM Quantum API key: "),
    set_as_default=True,
)
```

An instance can also be specified when required by your account setup. See the [IBM instructions for saving credentials](https://quantum.cloud.ibm.com/docs/en/guides/save-credentials). Keep credentials outside the repository.

The selected backend must be available to your account. To acquire a dataset again, move its existing JSON file to a backup location and rerun the acquisition cell. Only missing datasets are acquired. Changing `IBM_BACKEND` also changes the expected dataset filenames; it does not reuse another processor's counts.

Default hardware shot counts for new acquisitions are:

| Experiment | Measurements per circuit |
| --- | --- |
| Efficiency | 1024 |
| Intercept-and-resend | 256 for each interception-basis configuration |
| Intercept-and-resend no-attack control | 256 |
| Entangle-and-measure | 512 for each interaction parameter |
| Entangle-and-measure no-attack control | 256 |
| Ancilla information | 2048 for each input state and interaction parameter |

The two attack families use separate no-attack datasets. For intercept-and-resend, the two interception bases are measured separately and averaged. Entangle-and-measure uses twice as many shots for each attacked circuit, matching the total number of attacked test measurements per encoded bit in intercept-and-resend at each interaction parameter.

Shot counts and other experiment settings are defined in the first code cell. **Changing a shot-count setting does not replace or extend an existing dataset.** Saved JSON files are loaded as recorded; move a dataset to a backup location if you want the notebook to acquire it again with the new settings.

## Experiments and interpretation

- **Efficiency:** estimates the mean number of attempted rounds needed to transmit the secret and the bit error rate of the reconstructed secret. These estimates do not condition on passing all noisy security tests.
- **Single-round attack detection:** compares theoretical forbidden-outcome probabilities with fresh Aer measurements and the supplied IBM counts.
- **Cumulative attack detection:** uses 1000 independent simulated protocol runs for each attack setting and secret lengths of 0, 10, 20, 40, 60, 80 and 100 bits. A run ends at the first forbidden test outcome or after all 100 bits have been transmitted. There is no fixed limit on attempted rounds. The same runs supply the estimates for different secret lengths.
- **Ancilla information:** uses a separate two-qubit experiment to measure the mutual information between the prepared input state and the ancilla measurement outcome. The protocol attack-detection circuits themselves include all three protocol qubits. Hardware mutual information characterizes the specified ancilla measurement, not an optimization over all possible noisy measurements or the adversary's total information about the secret.
- **Hardware statistical criterion:** splits each no-attack configuration's counts into equal, disjoint calibration and holdout subsets. Calibration determines a threshold; holdout data estimate the background forbidden-outcome frequency. Detection and false-alarm probabilities are then calculated using a binomial model with measured frequencies. They are not empirical frequencies from repeated complete protocol executions on IBM hardware.

The default hardware threshold plots use 10 to 200 test rounds, conditional testing levels of 0.01 and 0.05 and a total calibration failure probability of 0.05. Their horizontal axis counts test rounds, while the cumulative simulation plots use the number of secret bits transmitted.

Error bars are conservative pointwise 95% confidence bounds. They are not simultaneous confidence bands for a whole curve. The statistical guarantees assume independent measurements, stationary hardware statistics and the configuration sampling described in the article. A threshold crossing indicates excess forbidden outcomes relative to the calibrated no-attack model; it does not identify their physical cause.

## Reproducibility

No random seeds are fixed. Aer measurements, random protocol choices and calibration/holdout splits are regenerated when their cells run. Consequently, simulator estimates and hardware threshold projections can change between runs even when the IBM JSON files remain unchanged. The stored hardware counts themselves change only when replaced or newly acquired.

The supplied files allow the analysis of the recorded IBM measurements to be repeated without new processor access. Repeated randomized analyses need not reproduce every plotted value exactly. New processor measurements may also differ because of sampling fluctuations and changes in device behavior.

For the theoretical relations, attack assumptions and statistical derivations, consult the accompanying article. The implemented experiments assess the specified attack models; they do not establish security against arbitrary attacks.
