# Parser Performance Matrix (PPM)

**A Decision Framework for Document Parser Selection**

Extends ParseBench quality benchmarks with operational performance (latency, memory, throughput), reliability, and workload-specific weighting to answer: *"Which parser is best for MY production workload?"*

This is the reference Python implementation of the framework described in the PRD and the "Beyond ParseBench" infographic.

## The Core Idea

ParseBench tells you **how well** a parser preserves content (quality across 5 dimensions).

PPM tells you **how well it will perform in your environment** by combining:

- **Quality (P)**: ParseBench-weighted score (Tables, Charts, Content Faithfulness, Semantic Formatting, Visual Grounding)
- **Efficiency (E)**: Normalized latency + memory (lower is better → higher score)
- **Reliability (R)**: Success rate, coverage, consistency
- **Workload Weights**: e.g., Clinical RAG prioritizes Quality 70%, Edge Deployment prioritizes Efficiency

Plus **Pareto Frontier** visualization so you can see optimal trade-offs instead of a single "best" score.

## Quick Start — End-to-End in 5 Minutes

See the dedicated **[QuickStart Guide](docs/QUICKSTART.md)** for a complete walkthrough including:

- One-command install + demo
- Clinical RAG evaluation with healthcare reliability (DSP) + cost weighting
- TCO sensitivity analysis
- Interactive Pareto visualization (bubble = TCO)
- Python API usage for agents

**Fastest way to try it now:**

```bash
make install
make demo
make demo-pareto
```

Or directly:

```bash
ppm evaluate --profile clinical_rag --reliability-mode healthcare --cost-weight 0.20
ppm pareto --bubble tco --open
```

## Project Status & Implementation Plan

This is being built iteratively. Current focus: **Core scoring engine + sample data + CLI foundation**.

### Completed / In Progress
- [x] Project skeleton (pyproject.toml, structure)
- [ ] Pydantic data models (ParserMetrics, WeightProfile, etc.)
- [ ] Scoring & normalization logic matching infographic formulas exactly
- [ ] Predefined workload profiles (Clinical RAG, High Throughput, Edge, Enterprise)
- [ ] Sample benchmark data seeded from ParseBench leaderboard + infographic hypotheticals (EdgeParse, etc.)
- [ ] Basic CLI with `evaluate` and `pareto` commands (rich tables + plotly HTML)
- [ ] Documentation & examples

### Planned Phases

**Phase 1: Core (Current)**
- Full scoring engine with robust normalization (handles single-parser, missing values, custom bounds)
- Weight profiles as Pydantic model with presets + custom YAML/JSON loading
- Decision matrix generation (ranked table with P/E/R/PPM)
- Pareto computation (non-dominated sort) + interactive Plotly viz (Quality vs Latency/Efficiency, with hover details, frontier line)
- Export: CSV, JSON, Markdown report, self-contained HTML dashboard

**Phase 2: Benchmark Harness & Adapters**
- `ParserAdapter` protocol / ABC
  - `LocalPDFAdapter` (pdfplumber + pypdf for text/tables extraction — baseline for perf testing)
  - `MockAdapter` for testing/scenarios
  - `ExternalResultsAdapter` (load pre-computed quality + perf from CSV/JSON, e.g. from ParseBench runs)
- `BenchmarkRunner`: measures latency (wall time), peak RSS memory (psutil subprocess monitor), success rate, throughput on a corpus
- Consistency via repeated runs (variance)
- Optional: integration hook for ParseBench evaluation code (if user has the repo + keys set up) — PPM focuses on *extending* not duplicating

**Phase 3: Advanced Analysis & Reporting**
- Sensitivity analysis: what-if weight changes
- Cost integration (if ¢/page from ParseBench)
- Regression testing mode (compare parser versions over time)
- SHACL-like validation? (future, for output structure)
- Full leadership-ready PDF/Markdown report generation (using your existing docx/pdf skills)

**Phase 4: Usability & Ecosystem**
- Jupyter-friendly API (ppm.api module)
- Streamlit/Gradio optional UI (if deps added)
- GitHub Actions example for CI benchmark regression
- Public leaderboard contribution helper (export in ParseBench format)
- Healthcare-specific extensions (e.g., FHIR document profiles, clinical trial protocol parsing weights)

## Why This Matters (from your PRD)

Traditional leaderboards optimize for one thing (quality). Production systems live in a **multi-objective** world:
- Clinical RAG on 10k PDFs/day in cloud → quality + some reliability
- On-prem hospital edge with no GPU → latency + memory + reliability win
- High-throughput ingestion pipeline → efficiency dominant

PPM turns parser selection from "pick the leaderboard #1" into a transparent, reproducible **architectural decision process** with Pareto-optimal choices visible.

## Alignment with Your Stack & Preferences

- Minimal core deps (pydantic, pandas, plotly, click, rich, psutil, numpy)
- Type-safe, reproducible, auditable (all formulas visible, no hidden magic)
- CLI-first + library for agents/multi-agent orchestration (LangGraph tool?)
- Output artifacts perfect for leadership (HTML viz, Markdown tables, CSV for further analysis)
- Extensible via adapters (easy to add your internal parsers or new SOTA ones)
- Designed for iterative refinement (your preferred style)

## Sample Data Included

Seeded with real ParseBench May/June 2026 leaderboard values (LlamaParse Agentic #1, Gemini 3 Flash, etc.) + realistic operational metrics for:
- High-quality cloud parsers (higher latency/mem)
- Edge/local parsers (fast, low-mem, slightly lower quality on complex docs)
- Hypotheticals matching your infographic examples (Parser A Llama vs Parser B EdgeParse)

This lets you immediately run `ppm evaluate --profile clinical_rag` and see LlamaParse win, then switch to `--profile edge_deployment` and see trade-off.

## Contributing / Next

Run `python -m ppm.cli --help` once core is in (or after `pip install -e .`).

I (Grok) am building this with you step-by-step. Tell me:
1. Prioritize which command/feature first? (e.g., scoring engine + CLI evaluate, or Pareto viz, or live benchmark harness)
2. Any specific parsers or document types to support early (clinical PDFs, trial protocols, etc.)?
3. Do you want integration points with your existing skills (e.g., output decision matrix as infographic via infographic-generator, or RDF provenance for benchmark runs)?
4. Target output formats for leadership (executive one-pager Markdown, full HTML dashboard, C4 diagram of the framework itself)?

Let's build this into a tool you can use for your RAG/clinical AI / multi-agent work and share with leadership.

---

## Spec-Driven Development

This project uses **spec-driven development**. All major features and changes should start from specifications located in `docs/specs/`.

Key specs:
- `technical/scoring-engine.md`
- `technical/reliability-formulas.md` (Enterprise vs Healthcare + DSP)
- `technical/api-contract.md` (for agents and MCP)
- `technical/tco-model.md`
- `technical/pareto-analysis.md`

See `docs/specs/README.md` for the full structure and process.

*This framework does not replace ParseBench. It makes ParseBench actionable for production decisions.*