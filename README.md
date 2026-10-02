# Vector Embedding System for Capability Composition

## Assignment 2

A C++17 implementation of the supplied assignment specification for **Vector Embedding for Capability Composition**.

### What this project implements

- Formal `State`, `Goal`, and `Capability` data structures
- A unified **92-dimensional** vector embedding
- Capability type encoding
- Operational attributes: cost, reliability, availability
- Preconditions and effects
- Input/output data fields
- Resources and constraints
- Cosine similarity
- Effect/precondition compatibility verification
- Input/output dataflow compatibility
- Sequential capability composition
- Recursive composition of capability chains
- Goal relevance scoring
- Five experiments required by the assignment
- CMake build and direct `g++` build

### Important scope

This is **not a planner**. It does not implement BFS, DFS, A*, D* Lite, LPA*, graph search, or replanning. Those are explicitly outside the scope of Assignment 2.

## Project structure

```text
CapabilityEmbedding/
├── include/
│   ├── Types.h
│   └── Embedding.h
├── src/
│   └── Embedding.cpp
├── tests/
│   └── test_embedding.cpp
├── CMakeLists.txt
└── README.md
```

## 92-dimensional representation

| Range | Subspace | Dimensions |
|---|---|---:|
| 0–8 | Capability Type | 9 |
| 9–11 | Operational | 3 |
| 12–27 | Preconditions | 16 |
| 28–43 | Effects / Desired State | 16 |
| 44–59 | Inputs | 16 |
| 60–75 | Outputs | 16 |
| 76–83 | Resources | 8 |
| 84–91 | Constraints | 8 |

The state-variable registry contains the application variables used by the assignment examples. Resource and constraint registries similarly provide deterministic indices.

## Mathematical formulation

Each capability is represented as a fixed-length vector of 92 values. The vector is organized into subspaces for capability type, operational attributes, preconditions, effects, inputs, outputs, resources, and constraints.

The embedding is structured as follows:

- capability type: 9 dimensions
- operational attributes: 3 dimensions
- preconditions: 16 dimensions
- effects: 16 dimensions
- inputs: 16 dimensions
- outputs: 16 dimensions
- resources: 8 dimensions
- constraints: 8 dimensions

The operational attributes contain normalized cost, reliability, and availability. Cost is scaled with a saturating transform so that larger values remain bounded, while reliability and availability remain in the range [0, 1].

State predicates are mapped to signed values so that true and false conditions are distinguished explicitly. Compatibility for a pipeline C1 -> C2 is checked by ensuring that no effect of C1 contradicts a precondition of C2 and that all required data dependencies of C2 are either produced by C1 or supplied externally.

Sequential composition combines two capabilities into a single capability. The later capability can override shared state variables, while the aggregated cost is the sum of the two components and the reliability/availability values are multiplied.

Goal relevance is measured by cosine similarity between the effect-only vector of a capability and the goal vector. This produces a score in the range [0, 1], where a direct match gives 1.0 and a disjoint effect set gives 0.0.

## Compatibility

For `C1 -> C2`, the implementation checks:

1. No effect of `C1` contradicts a precondition of `C2`.
2. Required data inputs of `C2` are produced by `C1` when a pipeline data dependency is declared.

The result includes an explanation string.

## Composition

For `C1 + C2`:

- external inputs = inputs of `C1` + inputs of `C2` not produced by `C1`
- outputs = union of outputs
- preconditions = preconditions of `C1` + `C2` preconditions not established by `C1`
- effects = effects of `C1`, with `C2` overriding shared state variables
- cost = sum
- reliability = product
- availability = product
- resources and constraints = set unions

## Goal relevance

Goal relevance is computed from cosine similarity between the capability's effect-only vector and the goal vector, clamped to `[0,1]`.

A direct effect match therefore produces a relevance of `1.0`, while a disjoint effect produces `0.0`.

## Build on Windows with g++

Open the VS Code terminal in the project folder:

```powershell
g++ -std=c++17 -Wall -Wextra -pedantic -Iinclude src/Embedding.cpp tests/test_embedding.cpp -o test_embedding.exe
.\test_embedding.exe
```

If `g++` is not recognized, install/use MinGW-w64 or MSYS2 and add its `bin` directory to PATH.

## Build with CMake

```powershell
mkdir build
cd build
cmake ..
cmake --build .
.\Debug\test_embedding.exe
```

Depending on the generator, the executable may instead be directly under `build/`.

## Linux/macOS

```bash
g++ -std=c++17 -Wall -Wextra -pedantic -Iinclude src/Embedding.cpp tests/test_embedding.cpp -o test_embedding
./test_embedding
```

## Expected experiment results

- Experiment 1: `CreateOrder -> MakePayment` is compatible.
- Experiment 1: `CreateOrder -> CancelCart` is incompatible because `Order.exists=true` contradicts `Order.exists=false`.
- Experiment 2: total cost = `45.0`
- Experiment 2: reliability = `0.965349`
- Experiment 2: availability = `0.984065`
- Experiment 2: direct and chain composition embeddings have cosine similarity `1.0000`
- Experiment 4: `MakePayment` has goal relevance `1.0000`
- Experiment 4: `PlayMusic` has goal relevance `0.0000`

## Results

The implementation produced the expected behavior across the assignment experiments:

- Compatibility checks correctly accept `CreateOrder -> MakePayment` and reject `CreateOrder -> CancelCart`.
- Composition metrics match the expected aggregate properties, including total cost, reliability, and availability.
- The direct composition and the chained composition yield cosine similarity of `1.0000`, confirming that they encode the same semantic effect pattern.
- Goal relevance scoring distinguishes a matching goal (`MakePayment`) from a mismatch (`PlayMusic`), with values `1.0000` and `0.0000` respectively.

## Analysis

The results show that the embedding preserves the main semantic relationships required for capability composition:

- logical contradictions are detectable through effect/precondition conflicts;
- dataflow compatibility is preserved by verifying generated outputs against required inputs;
- sequential composition behaves consistently because later effects override earlier shared state values;
- cosine similarity provides a practical measure of semantic alignment between capabilities and goals.

In other words, the design captures both structural compatibility and behavioral similarity in a compact vector representation, while remaining simple enough to inspect and explain.

## Limitations

Although the model is effective for the assignment, it has several important limitations:

- The state registry is fixed and domain-specific, so the system is not automatically extensible to arbitrary new variables.
- The embedding is designed as a symbolic semantic approximation rather than a learned representation.
- The cost normalization and compatibility checks are intentionally simplified and do not capture full real-world planning dynamics.
- Dataflow is approximated through deterministic names and domains, without a richer semantic matching mechanism.
- The model does not support optimization, search, or adaptive replanning beyond static composition checks.

## Conclusion

This project demonstrates that a deterministic, 92-dimensional embedding can model capability composition, compatibility checking, and goal relevance in a compact and explainable form. The experiments confirm that the approach is suitable for the assignment’s requirements and provides a clear foundation for further work in capability reasoning or planning systems.

## Design note

The embedding intentionally uses a deterministic domain registry instead of arbitrary hashing. This makes the numerical representation reproducible and makes the semantic subspaces interpretable for an academic experiment.
