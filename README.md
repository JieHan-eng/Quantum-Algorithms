# Shor's Algorithm: Classical vs Quantum

**Author:** Jie Han Oh
**Module:** Y3 Individual Project
**Supervisor:** Dr. Jayadev Vijayan

This is the code for my third-year project, where I took a proper look at Shor's algorithm and tried to see just how big the gap between classical and quantum period finding actually is in practice.

The quantum side is built with IBM's Qiskit. I ran the circuits on the Aer simulator first and then pushed them onto real hardware (ibm_fez) to see how things held up once noise got involved. For the classical baseline I went with a brute-force order-finding approach across semiprimes from 15 all the way up to 100,000,000, and fitted a power law so I could extrapolate out to RSA-sized numbers.
## Dependencies

| Package | Version | What it's for |
|---------|---------|---------|
| Python | 3.14+ | Runtime |
| qiskit | 2.3.0 | Building and transpiling the circuits |
| qiskit-aer | 0.17.2 | Local statevector simulator |
| qiskit-ibm-runtime | latest | Sending jobs to IBM's hardware |
| numpy | 2.4.2 | Numerical work |
| matplotlib | 3.10.8 | Circuit diagrams and histograms |
| pandas | 3.0.1 | Wrangling results and Excel export |
| openpyxl | 3.1.5 | Writing the Excel files |
| scipy | 1.17.0 | Curve fitting and general utilities |

## Getting set up

```bash
# Clone the repository
git clone https://github.com/<your-username>/shors-algorithm-y3-project.git
cd shors-algorithm-y3-project

# Create and activate a virtual environment
python -m venv .venv

# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

If you want to run the hardware notebook you'll need an IBM Quantum API token. It's free — just make an account at [quantum.ibm.com](https://quantum.ibm.com) and then save the token somewhere sensible:

```bash
# Option 1: stick it in a .env file (what I'd recommend)
echo "IBM_QUANTUM_TOKEN=your_token_here" > .env

# Option 2: let Qiskit handle it (one-time setup)
python -c "from qiskit_ibm_runtime import QiskitRuntimeService; QiskitRuntimeService.save_account(channel='ibm_quantum_platform', token='your_token_here')"
```

Whatever you do, please don't commit your token to the repo.

## What's in here

```
.
├── README.md                          # You are here
├── requirements.txt                   # Python dependencies
├── .gitignore                         # Git ignore rules
│
├── Comparison analysis circuits/      # The main analysis notebooks
│   ├── shor's algorithm.ipynb         # Generic Qiskit Shor's circuit for N=15 and N=21 (Aer)
│   ├── shors algorithm (classical analysis).ipynb  # Classical period-finding scaling
│   ├── QFT.ipynb                      # QFT circuit diagram generator
│   ├── QFT.png                        # Exported QFT diagram
│   └── Classical_Shor_Analysis_Results.xlsx  # Raw classical scaling data (2000 semiprimes)
│
├── on hardware.ipynb                  # Runs Shor's on real hardware (ibm_fez)
├── IBM shor's algorithm.ipynb         # IBM's textbook-style hand-built circuit for N=15
│
└── data/                              # Raw outputs, kept for reproducibility
    └── README.md                      # What each file contains
```

## How to run things

### Classical period-finding

Open `Comparison analysis circuits/shors algorithm (classical analysis).ipynb` and run all the cells. It works its way through 2000 semiprimes (4-bit up to 27-bit), does 50 iterations per number, fits a power law to the operation counts, and then extrapolates out to RSA key sizes. The results land in `Classical_Shor_Analysis_Results.xlsx`. Fair warning — the full run takes a few hours because the bigger semiprimes aren't kind.

### Quantum simulation (Aer)

Open `Comparison analysis circuits/shor's algorithm.ipynb` and run all cells. This builds the generic Shor's circuit for N=15 (12 qubits) and N=21 (15 qubits), runs 4096 shots on Aer, and pulls out the phases, periods, and factors for every coprime base. You don't need an API token for this one.

### Real hardware

Open `on hardware.ipynb`. You'll need an IBM Quantum token set up first (see above). It submits transpiled circuits for both N=15 and N=21 to ibm_fez, grabs the results back, and compares how the hardware did against the simulator. Spoiler: noise matters.

### QFT diagram

Open `Comparison analysis circuits/QFT.ipynb` if you want to regenerate the 4-qubit QFT circuit diagram that ended up in the report.

## Mapping the results back to the report

| Report Section | Notebook | What it produces |
|---|---|---|
| Ch. 3 — Classical scaling | `shors algorithm (classical analysis).ipynb` | Log-log scaling plot, RSA extrapolation table |
| Ch. 3 — QFT diagram | `QFT.ipynb` | QFT circuit figure |
| Ch. 4 — Simulator (N=15, N=21) | `shor's algorithm.ipynb` | Measurement histograms, phase/period tables, success rates |
| Ch. 4 — Hardware (N=15, N=21) | `on hardware.ipynb` | Hardware histograms, hardware vs simulator comparison |
| Ch. 4 — IBM textbook circuit | `IBM shor's algorithm.ipynb` | Hand-built gate circuit for N=15 |

## Licence and thanks

Released under the [MIT Licence](LICENCE).

A few things I leaned on heavily:

- **Nielsen & Chuang**, *Quantum Computation and Quantum Information* — still the best place to go for the theory behind Shor's and the QFT.
- **Qiskit** (IBM) — did basically all the heavy lifting on the circuit side.
- **IBM Quantum** — for giving students free access to real quantum hardware through the IBM Quantum Platform. Running a circuit on an actual device never really stopped feeling cool.
- **Claude Code** (Anthropic) — a huge help throughout the project. I used it for debugging, working through coding problems, drafting this README, and generally as a sounding board whenever I got stuck.
