# PPM Specification Structure

This directory contains the formal specifications for the Parser Performance Matrix (PPM) framework.

Spec-driven development means that **specifications are the source of truth**. All code, tests, documentation, and architectural decisions should be traceable back to these specs.

## Directory Structure

```
docs/specs/
├── README.md                   # This file
├── product/                    # High-level product and business specifications
│   └── PRD.md                  # Product Requirements Document (root reference)
├── technical/                  # Detailed technical specifications
│   ├── scoring-engine.md       # Core scoring logic and normalization
│   ├── reliability-formulas.md # Enterprise vs Healthcare reliability models
│   └── api-contract.md         # Public API design and contracts
├── non-functional/             # Cross-cutting concerns
│   └── (performance, security, etc.)
└── features/                   # Per-feature or per-module specifications
    └── (future feature specs)
```

## How to Work With Specs

1. **New work starts with specs** — Before writing code, update or create the relevant spec.
2. **Review specs first** — Lightweight spec review should happen before implementation.
3. **Maintain traceability** — Code and tests should reference relevant spec sections.
4. **Keep specs living** — Update specs when behavior changes.

## Current Key Specs

| Spec | Purpose | Priority |
|------|---------|----------|
| `technical/scoring-engine.md` | Overall PPM scoring model | High |
| `technical/reliability-formulas.md` | Enterprise & Healthcare reliability | High |
| `technical/api-contract.md` | Public API for agents & MCP | High |

## Related Documents

- [PRD](../PRD.md)
- [Architecture Decision Records](../adr/)
- [USAGE.md](../USAGE.md)
- [QUICKSTART.md](../QUICKSTART.md)