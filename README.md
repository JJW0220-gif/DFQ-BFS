# DFQ-BFS

**A Decentralized Frontier Queue for Improving Scalability of Breadth-First-Search on GPUs**

DFQ-BFS is a research implementation and benchmark suite for GPU breadth-first search (BFS), based on the 2023 DATE paper cited below. BFS visits a graph level by level and maintains a *frontier* of vertices to process at each level. On a GPU, many threads may try to add newly discovered vertices to the frontier at once. A single shared queue can become a contention bottleneck; DFQ-BFS explores decentralized frontier-queue designs to improve scalability.

The repository includes multiple implementations and design variants. Their directory names describe different queue, synchronization, and frontier-management choices; see the CUDA source in each directory for the precise behavior.

## Repository layout

- `design/`: CUDA source implementations organized by design choice.
  - `global-q/`: implementations using a global-queue design.
  - `shared-q/`: implementations using a shared-queue design.
- `bin/`: compiled executables. `plot.py` runs every file in this directory against every dataset in `data/`.
- `data/`: graph datasets in the input format expected by these implementations.
- `plot.py`: repeats each executable/dataset run, collects elapsed times, and generates `result.csv` and `result.png`.

## Environments

### Original paper/repository environment

The original README reports the following tested environment:

- OS: Ubuntu 18.04
- CUDA Toolkit / `nvcc`: 11.6
- CPU: x86_64; tested with Intel Core i9-10900K at 3.70 GHz
- GPU: NVIDIA GPU with compute capability 8.6 or higher and at least 12 GB of device memory; RTX 3080 supported, with most evaluations performed on an RTX 3090

### Local reproduction environment

The following setup has been used to compile and run an implementation in this repository:

- OS: Ubuntu under WSL; exact Ubuntu release not recorded
- GPU: NVIDIA GeForce RTX 5060 Ti, 16 GB (compute capability 12.0)
- CUDA Toolkit / `nvcc`: 12.8 / 12.8.93
- CPU: x86_64; model not recorded

The `sm_120` build command below targets the RTX 5060 Ti's native architecture. CUDA version, driver, GPU, and power settings can affect performance, so timing results are not directly comparable to the paper's results without matching its hardware and environment.

### Initial local validation

On `Amazon0302.dat`, the reported run from source vertex 1 reached all 262,111 vertices, had a maximum BFS distance of 70, reported zero CPU/GPU distance mismatches, and took 18.6748 ms. In the five-cycle `result.csv`, `scan-free-non-sep-lower-bound` averaged 17.82358 ms, compared with 19.86888 ms for `bfs`. These are results on the local setup, not the paper's published measurements. The target architecture used for the reported run was not recorded; use the `-arch=sm_120` build command below for a reproducible native-architecture build.

## Build and run

From the repository root, choose an implementation directory and compile it. For example:

```bash
cd design/global-q/scan-free/non-sep/lower-bound
nvcc -O3 --compiler-options -Wall -Xptxas -v -arch=sm_120 bfs.cu -o bfs
```

Copy the executable into `bin/` from the repository root:

```bash
cd "$(git rev-parse --show-toplevel)"
cp design/global-q/scan-free/non-sep/lower-bound/bfs bin/scan-free-non-sep-lower-bound
```

Run one implementation on one dataset:

```bash
./bin/scan-free-non-sep-lower-bound data/Amazon0302.dat 8
```

The program prints BFS statistics and elapsed GPU time to the terminal. The trailing `8` is an argument passed by the benchmark convention; it is **not** the number of repetitions. The source/dataset determines the starting vertex.

To run all executables in `bin/` on all datasets in `data/`, install the Python plotting dependencies and pass the number of repetitions to `plot.py`:

```bash
python3 -m pip install numpy pandas matplotlib
python3 plot.py 5
```

This runs each executable/dataset pair five times and writes `result.csv` and `result.png` in the repository root. Keep only the executables you intend to compare in `bin/`.

## Dataset format

The BFS implementations in this repository read the prepared graph format used by the `.dat` datasets, not an arbitrary SNAP edge list. For example, `Amazon0302.dat` begins with:

```text
262111 1234877 1

0 5
5 5
10 5
...
```

The first line contains the number of vertices, number of edges, and source vertex. It is followed by per-vertex adjacency offsets/lengths and the edge data. Use the repository's prepared datasets or convert other graphs to the exact format expected by the selected implementation before running it.

## Citation

```bibtex
@inproceedings{hsieh2023decentralized,
  title={A Decentralized Frontier Queue for Improving Scalability of Breadth-First-Search on GPUs},
  author={Hsieh, Chou-Ying and Cheng, Po-Hsiu and Chang, Chia-Ming and Kuo, Sy-Yen},
  booktitle={2023 Design, Automation \& Test in Europe Conference \& Exhibition (DATE)},
  pages={1--6},
  year={2023},
  organization={IEEE}
}
```
