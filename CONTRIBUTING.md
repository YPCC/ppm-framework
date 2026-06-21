# Contributing to Parser Performance Matrix (PPM)

Thank you for your interest in contributing to PPM! This project follows **spec-driven development**.

## Our Development Philosophy

Specifications are the single source of truth. All significant changes should start from (or update) a specification before implementation begins.

This ensures:
- Clear requirements and design before coding
- Better traceability
- Easier reviews
- Living documentation

## Development Process

### 1. Start with a Spec

- For new features or major changes, create or update a spec in `docs/specs/`.
- Use the template in `docs/specs/templates/spec-template.md`.
- Place feature-level specs in `docs/specs/features/`.
- Place technical details in `docs/specs/technical/`.

### 2. Spec Review (Lightweight)

Before writing significant code:
- Open a draft PR or discussion with your spec.
- Use the checklist in `docs/specs/templates/spec-review-checklist.md`.
- Get feedback from at least one other contributor (or the maintainer).

### 3. Implement

Once the spec is accepted or approved:
- Implement the feature.
- Add or update tests.
- Add traceability comments linking code back to the spec (e.g., `# See spec: technical/scoring-engine.md#section-3.2`).

### 4. Update Documentation

- Update relevant docs in `docs/` (USAGE.md, QUICKSTART.md, etc.).
- If the change is architectural, consider creating or updating an ADR in `docs/adr/`.

### 5. Submit a Pull Request

- Link the PR to the spec (and vice versa).
- Ensure all tests pass (`make test`).
- Update the version in `pyproject.toml` if this is a notable change.

## Key Directories

| Directory              | Purpose |
|------------------------|---------|
| `src/ppm/`             | Core library code |
| `docs/specs/`          | Formal specifications (source of truth) |
| `docs/adr/`            | Architecture Decision Records |
| `examples/`            | Usage examples (LangGraph, MCP, notebooks) |
| `tests/`               | Automated tests |

## Code Style & Quality

- Run `make lint` before submitting PRs.
- Follow existing code style (type hints, docstrings, clear naming).
- Keep public API clean and well-documented (see `technical/api-contract.md`).

## Adding New Specs

1. Copy `docs/specs/templates/spec-template.md`
2. Fill it out thoughtfully
3. Place it in the appropriate subdirectory
4. Update `docs/specs/README.md` if adding a new category

## Questions?

Open an issue or discussion. We're happy to help shape specs before implementation begins.

---

*Spec-driven development helps us build a maintainable, well-documented, and trustworthy tool — especially important for clinical and regulated use cases.*