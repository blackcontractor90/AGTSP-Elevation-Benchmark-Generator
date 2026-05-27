# AGTSP Elevation Benchmark Generator

A modular Java framework for generating reproducible **Asymmetric Generalized Traveling Salesman Problem (AGTSP)** benchmark instances with elevation-driven cost asymmetry, intended for terrain-aware routing research in EV and UAV last-mile delivery logistics.


---

## Repository Structure

```
agtsp-elevation-benchmark/
│
├── src/
│   ├── main/java/com/agtsp/
│   │   ├── Main.java                        # Entry point (4 run modes)
│   │   ├── core/
│   │   │   ├── InstanceGenerator.java       # Main generation pipeline
│   │   │   ├── InstanceParameters.java      # Builder-pattern config
│   │   │   ├── AGTSPInstance.java           # Instance data structure
│   │   │   ├── InstanceMetrics.java         # 15 characterisation metrics
│   │   │   ├── InstanceValidator.java       # 4-stage validation pipeline
│   │   ├── spatial/
│   │   │   ├── SpatialDistributor.java      # Node placement (3 methods)
│   │   │   ├── ClusterFormation.java        # Balanced k-means clustering
│   │   ├── terrain/
│   │   │   ├── ElevationGenerator.java      # Multi-frequency terrain synthesis
│   │   │   ├── CostMatrixGenerator.java     # Asymmetric cost matrix
│   │   ├── algorithms/
│   │   │   ├── ILS.java                     # Iterated Local Search
│   │   │   ├── GeneticAlgorithm.java        # Genetic Algorithm
│   │   │   ├── SimulatedAnnealing.java      # Simulated Annealing
│   │   │   ├── TabuSearch.java              # Tabu Search
│   │   ├── analysis/
│   │   │   ├── ExperimentRunner.java        # Batch experiment execution
│   │   │   ├── MetricsCalculator.java       # Metric computation
│   │   ├── io/
│   │   │   ├── JSONExporter.java            # JSON export/import
│   │   │   ├── TSPLIBWriter.java            # TSPLIB .agtsp format export
│   │   │   ├── TSPLIBReader.java            # TSPLIB format import
│   │   └── ui/
│   │       └── AGTSPApplication.java        # Swing GUI application
│   └── test/java/com/agtsp/
│       └── AGTSPGeneratorTest.java          # JUnit test suite
│
├── benchmarks/                              # Generated instance files (.json, .agtsp)
├── results/                                 # Experiment output CSVs
└── README.md
```

---

## Requirements

- **Java** 8 or higher
- **Maven** (for building and running tests)
- Dependencies managed via Maven: `gson`, `junit`

---

## Building the Project

```bash
# Clone the repository
git clone https://github.com/[your-username]/agtsp-elevation-benchmark.git
cd agtsp-elevation-benchmark

# Compile
mvn compile

# Run tests
mvn test

# Package into a runnable JAR
mvn package
```

---

## Running the Generator

The entry point is `Main.java`. The run mode is controlled by the `MODE` constant at the top of the file. Set it to one of four values before compiling:

```java
private static final String MODE = "SINGLE"; // SINGLE | BENCHMARK | EXPERIMENT | COMPARE_ALGOS
```

### Mode 1 — SINGLE (Quick Demo)

Generates and solves one instance with ILS. Reproduces the proof-of-concept instance from the paper:

```java
MODE = "SINGLE"
```

Default parameters (as set in `runSingleDemo()`):
- n=100 nodes, k=10 clusters, seed=42
- α=0.4, terrain=hilly, spatial=gaussian_cluster

Outputs to `benchmarks/hilly_a0.4_n100_k10_s42.json` and `.agtsp`.

---

### Mode 2 — BENCHMARK (Generate Full Suite)

Generates all 78 benchmark instances and saves them to `benchmarks/` organised by category:

```java
MODE = "BENCHMARK"
```

Instances are saved in both `.json` and `.agtsp` (TSPLIB) formats under category subfolders:

```
benchmarks/
├── category_b_terrain/
├── category_c_clusters/
├── category_d_scalability/
└── category_e_stress/
```

---

### Mode 3 — EXPERIMENT (Full Benchmark Run)

Runs all 4 algorithms × 78 instances × 10 runs (7,800 total). Results saved as CSV:

```java
MODE = "EXPERIMENT"
```

Output: `results/experiment_results.csv`

> **Note:** Estimated runtime is 2–6 hours depending on hardware, due to O(n²) cost matrix computation for large instances (n=1,000).

---

### Mode 4 — COMPARE_ALGOS (Single Instance Comparison)

Compares all 4 algorithms on a configurable single instance and prints a summary table:

```java
MODE = "COMPARE_ALGOS"
```

Edit the parameters inside `runAlgorithmComparison()` to test any configuration.

---

## Configuring a Custom Instance

Use the builder pattern in `InstanceParameters`:

```java
InstanceParameters params = new InstanceParameters.Builder()
    .numNodes(200)
    .numClusters(20)
    .seed(123)
    .asymmetryStrength(0.6)          // α ∈ [0, 1]
    .terrainType("mountainous")
    .maxElevation(500.0)
    .speedModel("linear")            // "linear" | "exponential" | "piecewise"
    .spatialDistribution("uniform")  // "uniform" | "gaussian_cluster" | "grid"
    .build();

InstanceGenerator generator = new InstanceGenerator(true); // verbose=true
AGTSPInstance instance = generator.generate(params);
```

---

## Parameter Reference

### Asymmetry Strength (α)

Controls the magnitude of elevation-driven cost asymmetry:

| α value | Effect |
|---------|--------|
| 0.0 | Symmetric baseline (no asymmetry) |
| 0.1 – 0.3 | Low asymmetry |
| 0.4 – 0.6 | Moderate asymmetry |
| 0.8 | High asymmetry |
| 1.0 | Extreme asymmetry |

### Terrain Types

| Type | Characteristic |
|------|---------------|
| `flat` | Minimal elevation variation |
| `rolling` | Gentle hills |
| `hilly` | Moderate slopes |
| `mountainous` | Steep terrain |
| `valley` | Depression pattern |
| `ridge` | Linear elevation features |

### Spatial Distributions

| Type | Description |
|------|-------------|
| `uniform` | Independent uniform random placement |
| `gaussian_cluster` | Multi-center urban simulation |
| `grid` | Regular grid with Gaussian perturbation |

### Speed Models

| Model | Description |
|-------|-------------|
| `linear` | Linear speed reduction on uphill grades (default) |
| `exponential` | Exponential decay with grade magnitude |
| `piecewise` | Grade-banded speed reduction |

---

## Benchmark Suite Structure

The 78-instance suite spans five complexity categories:

| Category | Focus | Variables | Instances |
|----------|-------|-----------|-----------|
| A | Asymmetry strength | α ∈ {0.0, 0.1, 0.2, 0.3, 0.4, 0.6, 0.8} | 18 |
| B | Terrain types | 6 terrain profiles | 18 |
| C | Cluster configuration | k ∈ {5, 10, 20, 33, 50} | 18 |
| D | Scalability | n ∈ {20, 50, 100, 200, 500, 1000} | 20 |
| E | Combined stress | Extreme α + complex terrain + large k | 4 |

---

## Instance File Format

Each instance is exported in two formats:

**JSON** (`.json`) — human-readable, includes all metadata:
```json
{
  "name": "hilly_a0.4_n100_k10_s42",
  "num_nodes": 100,
  "num_clusters": 10,
  "alpha": 0.4,
  "terrain_type": "hilly",
  "asymmetry_index": 0.978,
  "difficulty_score": 47.29,
  "difficulty_class": "hard",
  ...
}
```

**TSPLIB** (`.agtsp`) — compatible with standard TSP solvers via `TSPLIBReader`.

---

## GUI Application

A Swing-based GUI is available for interactive instance generation and visualisation:

```bash
# Run the GUI directly from your IDE:
# Right-click AGTSPApplication.java → Run As → Java Application
```

The GUI provides:
- Parameter configuration panel
- Interactive instance map with cluster visualisation
- Instance metrics and difficulty score display
- Algorithm results table
- Export to JSON / TSPLIB

---

## Running the Tests

```bash
mvn test
```

The JUnit test suite (`AGTSPGeneratorTest.java`) covers the full generation pipeline including terrain generation, cluster formation, cost matrix computation, instance validation, all four algorithms, and file I/O.

---

## Proof-of-Concept Results

The following results are reported in Section 4.3 of the paper, reproducible using `MODE = "SINGLE"` with default parameters (seed=42):

| Algorithm | Best Cost | Mean Cost | Std Dev | Mean Gap (%) | Runtime (ms) | Feasible Runs |
|-----------|-----------|-----------|---------|--------------|--------------|---------------|
| ILS | 4,977.49 | 4,977.49 | 0.00 | 0.00 | 5 | 10/10 |
| Tabu Search | 5,521.76 | 7,472.00 | 1,940.82 | +50.12 | 5 | 10/10 |
| GA | — | 671.52 | 524.59 | — | 16 | 0/10 |
| SA | — | 787.87 | 66.36 | — | 204 | 0/10 |

> GA and SA returned infeasible tours (incomplete cluster coverage) on this high-asymmetry instance (AI=0.978). Their costs are excluded from gap calculations.

---

## Citation

If you use this benchmark suite or generator in your research, please cite:

```bibtex
@article{morsidi2026agtsp,
  title     = {A Systematic Framework for Generating Asymmetric Generalized
               Traveling Salesman Problem Instances with Elevation-Based Cost Asymmetry},
  author    = {Morsidi, Farid and Ariffin, Asma Hanee and Abdul Wahid, Rohaizah},
  journal   = {Journal of Advanced Manufacturing Technology (JAMT)},
  year      = {2026}
}
```

---

## Contact

**Farid Morsidi ** (First Author)
farid_m2mfan@yahoo.com.my
Computing Department, Faculty of Computing & Meta-Technology
Universiti Pendidikan Sultan Idris, 35900 Tanjong Malim, Perak, Malaysia
