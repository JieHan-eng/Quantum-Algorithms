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
git clone https://github.com/JieHan-eng/Quantum-Algorithms
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

## How it all fits together

There are basically four moving parts in this project, and they each live in their own notebook:

1. **Classical baseline** — brute-force order finding across thousands of semiprimes, fitted to a power law. This is what the quantum side gets compared against.
2. **Quantum Fourier Transform** — the heart of the period-finding step. I generate a 4-qubit QFT circuit diagram on its own so it can be referenced in the report.
3. **Shor's circuit on the simulator** — the full generic Qiskit Shor circuit for N=15 and N=21, run noiselessly on Aer so you can see what the algorithm *should* do.
4. **Shor's circuit on real hardware** — the same circuits transpiled and submitted to ibm_fez, so the simulator results have something noisy to be measured against.

The flow through the project is roughly: classical scaling → QFT → ideal quantum (Aer) → noisy quantum (hardware) → comparison. The QFT diagram (`QFT.png`) gives a visual sense of the subroutine that makes the quantum side work in the first place.

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

## Technical details

A few notes on the algorithms and design choices, in case you want to follow along without having to dig through every cell:

- **Shor's algorithm.** The whole thing rests on reducing factoring to *order finding*: given a coprime base `a < N`, find the smallest `r` such that `a^r ≡ 1 (mod N)`. Once you have `r` (and it's even, and `a^(r/2) ≢ -1`), then `gcd(a^(r/2) ± 1, N)` gives you a non-trivial factor of `N`.
- **Classical side.** Order finding is done by brute force — multiply `a` by itself mod `N` until you hit 1. That's `O(r)` per base, and `r` can be as large as `N` itself, which is exactly why it scales badly. I run 50 iterations per semiprime (different random bases) to smooth out the variance, then fit a power law to the operation counts.
- **Quantum side.** The Qiskit Shor circuit uses Quantum Phase Estimation on the modular multiplication unitary `U|y⟩ = |ay mod N⟩`. The phase you read out is `s/r` for some integer `s`, and a continued-fraction expansion recovers `r`. For N=15 the circuit needs 12 qubits (8 counting + 4 work); for N=21 it's 15 (10 counting + 5 work).
- **QFT.** The inverse QFT is what turns the phase estimation register into a readable measurement. The 4-qubit version in `QFT.ipynb` is just there to make the structure visible — Hadamards interleaved with controlled phase rotations, then a swap.
- **Shots and statistics.** All simulator and hardware runs use 4096 shots. Success rates are computed as the fraction of shots that yield a valid (non-trivial) factor after the classical post-processing step.
- **Hardware backend.** ibm_fez was chosen because it had the qubit count and connectivity to fit both circuits without too much SWAP overhead after transpilation. Optimisation level 3 in the Qiskit transpiler.

## Known issues and future improvements

Things I'd do differently or extend if I had more time:

- **Hardware noise dominates at N=21.** The 15-qubit circuit on ibm_fez has a much lower success rate than N=15, and I didn't apply any error mitigation. Adding dynamical decoupling or measurement-error mitigation would be the obvious next step.
- **Classical run is single-threaded.** The full 2000-semiprime sweep takes a few hours. It would parallelise trivially across cores but I didn't bother.
- **Only one hardware backend.** Everything on hardware was run on ibm_fez. Comparing against a different topology (heavy-hex vs. something else) would say more about how much of the noise is device-specific.
- **No comparison against more efficient classical algorithms.** The classical baseline is brute-force order finding, not the General Number Field Sieve. That's deliberate (it's a like-for-like comparison of the *same subroutine* — period finding) but it's worth being explicit that this isn't a comparison against the best classical factoring algorithm overall.
- **Phase post-processing assumes ideal continued fractions.** When the hardware spits out a noisy phase, my code rounds it to the nearest valid fraction. A more robust approach would weight by measurement counts.

## Mapping the results back to the report

| Report Section | Notebook | What it produces |
|---|---|---|
| Ch. 3 — Classical scaling | `shors algorithm (classical analysis).ipynb` | Log-log scaling plot, RSA extrapolation table |
| Ch. 3 — QFT diagram | `QFT.ipynb` | QFT circuit figure |
| Ch. 4 — Simulator (N=15, N=21) | `shor's algorithm.ipynb` | Measurement histograms, phase/period tables, success rates |
| Ch. 4 — Hardware (N=15, N=21) | `on hardware.ipynb` | Hardware histograms, hardware vs simulator comparison |
| Ch. 4 — IBM textbook circuit | `IBM shor's algorithm.ipynb` | Hand-built gate circuit for N=15 |

## Third-party code and references

In line with academic integrity requirements, here's a clear breakdown of what is mine and what came from elsewhere:

- **`IBM shor's algorithm.ipynb`** — the hand-built textbook circuit for N=15 follows the structure of IBM Qiskit's own Shor's algorithm tutorial. The gate decomposition for the modular exponentiation `7x mod 15` is adapted from that tutorial rather than written from scratch.
- **Generic Shor circuits in `shor's algorithm.ipynb`** — built using Qiskit's standard library functions (`QFT`, `PhaseEstimation`, etc.). The high-level circuit construction is mine; the underlying gate implementations come from Qiskit.
- **Continued-fraction post-processing** — the period-extraction logic uses Python's standard `fractions.Fraction.limit_denominator`, with the wrapping logic written by me.
- **Classical order-finding code** (`shors algorithm (classical analysis).ipynb`) — written from scratch.
- **QFT diagram code** (`QFT.ipynb`) — uses Qiskit's built-in `QFT` class; the diagram export is mine.
- **Hardware submission code** (`on hardware.ipynb`) — follows the standard `QiskitRuntimeService` / `SamplerV2` pattern from IBM's documentation.

**AI assistance.** I used Claude Code (Anthropic) throughout the project as a debugging and coding assistant — for working through error messages, suggesting Qiskit API calls, drafting this README, and as a general sounding board when I got stuck. All code was reviewed, tested, and integrated by me, and the project design, results, analysis, and report writing are my own work.

## Licence and thanks

Released under the [MIT Licence](LICENCE).

A few things I leaned on heavily:

- **Nielsen & Chuang**, *Quantum Computation and Quantum Information* — still the best place to go for the theory behind Shor's and the QFT.
- **Qiskit** (IBM) — did basically all the heavy lifting on the circuit side.
- **IBM Quantum** — for giving students free access to real quantum hardware through the IBM Quantum Platform. Running a circuit on an actual device never really stopped feeling cool.
- **Claude Code** (Anthropic) — see the AI assistance note above.
