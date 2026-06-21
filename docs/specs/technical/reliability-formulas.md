# Technical Spec: Reliability Formulas

**Status:** Accepted  
**Version:** 1.0  
**Owner:** AI Platform Team  
**Related PRD Section:** Section 8 – Functional Requirements (Reliability)  
**Related ADRs:** ADR-0001  

---

## 1. Overview

Reliability is a critical pillar of the Parser Performance Matrix. Unlike many benchmarks that treat reliability as a simple success rate, PPM uses a decomposed, weighted model that distinguishes between operational robustness and (in healthcare contexts) deterministic preservation of critical clinical structure.

Two modes are supported:
- **Enterprise mode** (default)
- **Healthcare mode** (activates DSP – Deterministic Structure Preservation)

---

## 2. Requirements

- Must support two distinct reliability calculation modes.
- Must clearly separate concerns: success rate, coverage, consistency/stability, failure resilience, and clinical structure preservation.
- Must be explainable and auditable.
- Healthcare mode must give significant weight to clinical structure preservation.

---

## 3. Enterprise Reliability Formula

```math
R = 0.40 \times SR + 0.20 \times CV + 0.20 \times CS + 0.20 \times FR
```

### Component Definitions

| Component | Abbreviation | Weight | Description | Measurement |
|---------|--------------|--------|-------------|-------------|
| Success Rate | SR | 0.40 | Documents successfully parsed without fatal errors | `successful / total` |
| Coverage | CV | 0.20 | Breadth of document types/layouts handled | `supported_types / total_types` |
| Consistency | CS | 0.20 | Stability of results across repeated runs | `1 - (std_dev / mean)` or `1 - CV_variation` |
| Failure Resilience | FR | 0.20 | Ability to recover gracefully from corrupt, huge, rotated, or malformed documents | `recovery_cases_passed / total_recovery_cases` |

---

## 4. Healthcare Reliability Formula

```math
R = 0.30 \times SR + 0.15 \times CV + 0.15 \times CS + 0.15 \times FR + 0.25 \times DSP
```

### Additional Component: DSP

**DSP (Deterministic Structure Preservation)** — Weight: **0.25**

Measures whether the parser consistently and correctly preserves critical clinical structures across runs. This is especially important for:

- Medication lists and administration records
- Lab result tables
- Pathology reports
- Procedure timelines / event sequences
- Allergy and contraindication sections

A high DSP score means the parser does not randomly drop, reorder, or hallucinate these high-stakes structures.

---

## 5. When to Use Each Mode

| Use Case | Recommended Mode | Rationale |
|----------|------------------|---------|
| General enterprise document ingestion | Enterprise | Balanced operational reliability |
| Clinical RAG, Patient 360, medication safety | **Healthcare** | DSP is critical for patient safety |
| High-throughput non-clinical pipelines | Enterprise | Operational metrics dominate |
| Regulated / compliance-heavy environments | Healthcare | Auditability of structure preservation |

---

## 6. Implementation Notes

- The mode is controlled via `WeightProfile.reliability_mode`
- `ReliabilityMetrics` model supports all fields (`success_rate`, `coverage`, `consistency`, `failure_resilience`, `dsp`)
- `dsp` is optional. If not provided in Healthcare mode, the engine falls back gracefully.

---

## 7. Traceability

- Formulas defined in this spec are implemented in `src/ppm/scoring.py` → `compute_reliability_scores()`
- Public usage exposed via `ppm.api.evaluate_ppm(reliability_mode="healthcare")`
- See also: `models.py` → `ReliabilityMetrics` and `WeightProfile`

---

## 8. Future Considerations

- More granular DSP sub-dimensions (e.g., medication table fidelity vs. lab result fidelity)
- Integration with clinical validation rules or SHACL shapes for structure checking
- Human-in-the-loop scoring contribution for DSP in high-stakes scenarios