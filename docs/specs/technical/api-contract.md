# Technical Spec: Public API Contract

**Status:** Accepted  
**Version:** 1.0  
**Owner:** AI Platform Team  
**Related PRD Section:** Section 7 – Solution Overview & Functional Requirements  
**Related ADRs:** ADR-0001  

---

## 1. Overview

The PPM library exposes a clean, typed public API designed for both direct use and integration into multi-agent systems (LangGraph, AutoGen, CrewAI, etc.) and MCP servers.

The primary goals of the API are:
- Simplicity and discoverability
- Strong typing (Pydantic)
- JSON-serializable inputs/outputs
- Clear separation between high-level convenience functions and lower-level scoring primitives

---

## 2. Public API Functions

### 2.1 `evaluate_ppm()`

**Primary entry point** for most users and agents.

```python
def evaluate_ppm(
    results: Union[str, Path, list[dict], list[ParserResult]],
    profile: str = "clinical_rag",
    reliability_mode: Literal["enterprise", "healthcare"] = "enterprise",
    cost_weight: float = 0.0,
    latency_bounds: Optional[tuple[float, float]] = None,
    memory_bounds: Optional[tuple[float, float]] = None,
) -> dict[str, Any]
```

**Returns:**
```json
{
  "profile": { ... },
  "winner": "string",
  "scores": [ list of PPMScore dicts ],
  "pareto_optimal": ["list", "of", "parser", "names"],
  "summary": "string"
}
```

### 2.2 `get_tco_breakdown()`

```python
def get_tco_breakdown(
    results: Union[str, Path, list[dict]],
    volume_override: Optional[int] = None,
    failure_penalty_override: Optional[float] = None,
) -> dict[str, Any]
```

Returns detailed cost breakdown and supports sensitivity analysis via overrides.

---

## 3. Input Contracts

### 3.1 Parser Result Format

The library accepts either:
- A path to a JSON file containing a list of parser results, **or**
- A list of dictionaries / `ParserResult` objects

Each parser result must contain at minimum:
- `parser_name`
- `quality` (Tables, Charts, Content Faithfulness, Semantic Formatting, Visual Grounding)
- `operational` (latency_sec_per_doc, peak_memory_mb, optional cost/TCO fields)
- `reliability` (success_rate, coverage, consistency, failure_resilience, optional dsp)

See `models.py` for the full Pydantic schemas.

### 3.2 Weight Profiles

Profiles can be selected by name (`clinical_rag`, `edge_deployment`, etc.) or constructed programmatically via `WeightProfile.from_preset()` or direct instantiation.

---

## 4. Output Contracts

All public functions return JSON-serializable dictionaries. This design enables:
- Easy use as LangChain/LangGraph tools
- Simple MCP tool exposure
- Straightforward logging and auditing

Key output models:
- `PPMScore`
- `WeightProfile` (in result)
- TCO breakdown items

---

## 5. Error Handling & Edge Cases

- Missing or invalid data should raise clear `ValueError` or `Pydantic` validation errors.
- When `cost_weight > 0` but insufficient cost data is provided, the engine should degrade gracefully (ignore cost component).
- Single-parser evaluations are supported (normalization uses sensible defaults or provided bounds).

---

## 6. Agent / MCP Integration Guidelines

When exposing PPM as a tool:

- Prefer `evaluate_ppm()` for most decision-making tasks.
- Use `get_tco_breakdown()` when performing sensitivity analysis.
- Always pass `reliability_mode="healthcare"` for clinical/regulated document workloads.
- Return the full result dictionary so agents can reason over scores, Pareto set, and TCO numbers.

Example tool description pattern is available in `examples/langgraph_tool_example.py`.

---

## 7. Versioning & Compatibility

- The public API (functions in `ppm.api`) follows semantic versioning.
- Breaking changes to input/output schemas will be communicated via major version bumps.
- Internal implementation details in `scoring.py` may change without affecting the public contract.

---

## 8. Traceability

- API layer lives in `src/ppm/api.py`
- Backed by `src/ppm/scoring.py` and `src/ppm/models.py`
- See also: `examples/mcp_server_pattern.py` and `src/ppm/server/mcp.py`

---

## 9. Future Enhancements

- Support for streaming/large result sets
- Async versions of the main functions
- OpenAPI schema generation for the API contract
- Pluggable result loaders (beyond JSON)