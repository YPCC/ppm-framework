# Technical Spec: Scoring Engine

**Status:** Accepted  
**Version:** 1.0  
**Owner:** AI Platform Team  
**Related PRD Section:** Section 9 – Scoring Methodology  
**Related ADRs:** ADR-0001  

---

## 1. Overview

The Scoring Engine is the core computational component of the Parser Performance Matrix (PPM). It takes raw parser evaluation results (quality dimensions from ParseBench + operational metrics + reliability metrics) and produces normalized scores and rankings according to workload-specific weight profiles.

The engine must be:
- Transparent and auditable (all formulas visible)
- Reproducible
- Extensible (new profiles, new metrics)
- Suitable for both CLI usage and programmatic/agent consumption

---

## 2. Requirements

### Functional Requirements
- Compute **Quality Score (P)** using fixed ParseBench weights
- Compute **Efficiency Score (E)** with optional cost/TCO blending
- Compute **Reliability Score (R)** supporting both Enterprise and Healthcare formulas
- Compute overall **PPM Score** using workload profile weights
- Support normalization of latency and memory (lower is better)
- Support Pareto frontier identification
- Provide TCO estimation when sufficient operational data is supplied

### Non-Functional Requirements
- All calculations must be deterministic
- Scores must be normalized to `[0, 1]` where higher = better
- Must handle missing/partial data gracefully
- Must be efficient for typical workloads (hundreds of parsers × thousands of documents)

---

## 3. Detailed Design

### 3.1 Quality Score (P)

```math
P = 0.25T + 0.15C + 0.25CF + 0.15SF + 0.20VG
```

Where:
- `T`  = Tables
- `C`  = Charts
- `CF` = Content Faithfulness
- `SF` = Semantic Formatting
- `VG` = Visual Grounding

Input values are accepted in either `[0, 100]` or `[0, 1]` scale and normalized internally to `[0, 1]`.

### 3.2 Efficiency Score (E)

Base formula (when `cost_weight = 0`):

```math
E = 0.7 \times LatencyNorm + 0.3 \times MemoryNorm
```

Where normalization uses min-max with inversion (lower original value → higher normalized score):

```math
X_{Norm} = 1 - \frac{X - X_{min}}{X_{max} - X_{min}}
```

When `cost_weight > 0` and cost data is available, cost is blended in:

```math
E = (1 - w_{cost}) \times (0.7 \times L_{Norm} + 0.3 \times M_{Norm}) + w_{cost} \times CostNorm
```

### 3.3 Reliability Score (R)

See separate spec: `reliability-formulas.md`

### 3.4 Overall PPM Score

```math
PPM = w_P \times P + w_E \times E + w_R \times R
```

Weights are defined per `WeightProfile` and must sum to 1.0.

### 3.5 Pareto Frontier

A parser is considered Pareto-optimal if no other parser is strictly better in both quality and latency (or efficiency).

---

## 4. Data Models & Inputs

The engine primarily operates on `ParserResult` objects containing:
- `QualityDimensions`
- `OperationalMetrics` (including optional TCO fields)
- `ReliabilityMetrics`

See `models.py` for full Pydantic definitions.

---

## 5. Extensibility Points

- New `WeightProfile` presets can be added in `models.py`
- Custom bounds for normalization can be passed to scoring functions
- New reliability modes can be added by extending `compute_reliability_scores()`

---

## 6. Traceability & References

- Formulas are directly derived from the original PRD and "Beyond ParseBench" analysis.
- All scoring logic lives in `src/ppm/scoring.py`.
- Public entry point: `ppm.api.evaluate_ppm()`.

---

## 7. Open Questions / Future Work

- Should we support custom weight profiles loaded from YAML/JSON at runtime?
- Should energy consumption or carbon metrics be added to Efficiency in a future version?
- Formal verification of numerical stability for edge cases (all parsers having identical latency, etc.).