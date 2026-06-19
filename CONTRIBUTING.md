# Contributing to Slate

Thanks for your interest. Slate is MIT-licensed and welcomes external contributions.

## Quick setup

```bash
cd slate
python -m venv .venv && . .venv/bin/activate    # or .venv\Scripts\activate on Windows
pip install -e ".[dev]"
pytest -v
```

Use a local checkout until the public Core repo URL is verified for launch. If
all tests pass, you're set.

## Where to contribute

Highest-value places to help:

- **New VLM providers** — implement `slate.providers.base.VLMProvider` for another vision model (open-source local LLMs, Mistral, etc.). Add tests using the existing mocked-HTTP pattern in `tests/test_providers/test_gemma.py`.
- **Manifest schema additions** — propose new fields on `slate.manifest.Manifest` that meaningfully tighten the verdict. Open an issue first.
- **Signal refinement** — improve the prompt in `slate.prompts.build_frame_analysis_prompt` to reduce false positives / negatives. PRs should include a before / after comparison on known-bad and known-good fixtures.
- **Frame samplers** — alternative sampling strategies in `slate.frames` (e.g. motion-keyframe-aware sampling, scene-boundary detection).
- **Panel and evidence bundles** — improve persona review, local provider support, bundle metadata, and redaction behavior with tests.
- **Bug fixes** — anything labeled `good first issue` on GitHub.

## What NOT to contribute

- Anything that requires live production infrastructure or secrets.
- Any change that weakens the response-quality contract, hides uncertainty, or makes unsupported claims sound verified.
- Any provider integration that cannot be tested with mocks or a documented local fixture.

## PR checklist

- [ ] `pytest -v` passes.
- [ ] `ruff check src tests` is clean.
- [ ] `mypy src` is clean.
- [ ] If you added a new public API, you also added a test for it.
- [ ] If you changed a signal name, you updated `docs/signal-reference.md`.
- [ ] You included a one-sentence description in `CHANGELOG.md` under `## [Unreleased]`.

## Code style

We follow the conventions in `pyproject.toml`:

- `ruff` lints with the default `E, F, I, N, UP, B, SIM, RUF` set.
- `black`-compatible line length (100).
- Type hints on every public function.
- Pydantic 2 for all schemas; `ConfigDict(extra="forbid")` on input models so unknown fields are rejected loudly.

## Questions?

Use the verified public repository discussion board after launch. Before that,
open issues or questions through the current private project channel.

For project questions, use the verified public repository discussion board after launch.
