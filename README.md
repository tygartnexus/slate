# Slate

**Don't ship broken AI animation.**

Slate is a configurable VLM verdict service for rendered animation. Give it a frame sequence and a manifest of what should be in it. Get a PASS/FAIL verdict backed by whichever local and cloud vision-language providers you configure, with a structured signal report.

Built for indie animators, virtual production teams, AI-cinematic creators, and ML training-data curators who can't keep eyeballing 200 shots.

## What Slate catches that AI render tools don't

- Character lying horizontal, floating mid-air, or stuck in T-pose
- Character orientation doesn't match movement direction
- Ground contact missing — character hovering above the floor
- Wrong character identity (the manifest said hero A but hero B rendered)
- Debug-pink checker materials, missing textures, all-black frames
- Lighting / composition / atmosphere quality below a configurable threshold
- Missing landmarks the manifest required to be visible

## Quick start

```bash
# Pre-release/local checkout
pip install -e ".[dev]"

# Make sure Ollama is running locally with gemma4:latest, or set
# NVIDIA_API_KEY for cloud verdicts.

slate verify \
  --frames ./my_render \
  --manifest ./shot.json \
  --panel \
  --bundle evidence.tar.gz
```

After the PyPI release is verified from a clean environment, the install command
becomes `pip install slate-ai`.

Output (truncated):

```json
{
  "status": "FAIL",
  "shot_id": "village_walk_001",
  "providers_consulted": ["gemma", "nvidia-primary"],
  "failures": [
    {"signal": "character_orientation", "value": "lying_horizontal", "frame": "frame_0360.png", "provider": "gemma", "model": "gemma4:latest"},
    {"signal": "ground_contact_visible", "value": false, "frame": "frame_0001.png", "provider": "nvidia-primary", "model": "nvidia/nemotron-nano-12b-v2-vl"}
  ],
  "quality_scores_aggregated": {"lighting_quality": 5, "composition_quality": 4},
  "response_quality": {
    "mode": "evidence_based",
    "facts": ["Slate status is FAIL."],
    "assumptions": ["Sampled frames represent the shot."],
    "unknowns": ["Unsampled frames were not inspected."],
    "confidence_score": 0.82,
    "evidence": ["frame_0360.png provider response", "frame_0001.png provider response"],
    "risks": ["Provider judgments can be wrong on ambiguous stylized frames."],
    "counterarguments": ["A flagged pose could be intentional staging."],
    "recommendation": "Do not publish until hard-fail findings are resolved.",
    "tradeoffs": ["More samples improve evidence but increase runtime."],
    "what_would_change_recommendation": ["Human review confirms the flagged pose is intentional."]
  }
}
```

See [docs/quickstart.md](docs/quickstart.md) for a full walkthrough.

## How it works

Slate samples representative frames from your render (first/middle/last by default; configurable per manifest), asks each configured VLM provider a structured set of questions, optionally runs the four-persona Panel, and writes a JSON report. A single configured provider produces a single-provider verdict; using both local and cloud lanes produces a stronger cross-check.

```
frames + manifest
       |
       v
+---------------+    +-------------------+    +-----------------------+    +----------------+
| frame sampler | -> | VLM provider(s)   | -> | signal fusion + verdict| -> | optional Panel |
+---------------+    +-------------------+    +-----------------------+    +----------------+
                          |
                          +-- gemma (local, Ollama)
                          +-- nvidia (BYO API key)
                          +-- anthropic/local panel provider when --panel is enabled
```

Slate never uploads frames anywhere you did not configure. Local Ollama keeps sampled frames on your hardware; NVIDIA and Anthropic cloud lanes send sampled frames to those providers through your own account.

## One Slate

Slate is one free MIT-licensed project:

- `slate verify` runs configured-provider checks and can add Panel review with `--panel`.
- `slate bundle` builds an evidence tarball with frame hashes, manifest JSON, verdict JSON, optional thumbnails, and raw-output redaction. Remove sensitive project/customer metadata from manifests before sharing bundles externally.
- Slate Cloud is an optional dashboard for uploaded verdict JSON, history, and detail review.
- `slate-pro` is deprecated as a compatibility alias. New scripts should call `slate`.

## Installing the VLM backends

Slate ships drivers for local and cloud backends. You configure the providers you want to use.

### Local Gemma (free, no API key)

```bash
# Install Ollama (https://ollama.com)
ollama pull gemma4:latest
ollama serve  # listens on localhost:11434
```

### NVIDIA NIM (cloud, BYO key)

```bash
export NVIDIA_API_KEY=replace-with-nvidia-api-key
# Slate uses nvidia/nemotron-nano-12b-v2-vl as primary and
# meta/llama-3.2-90b-vision-instruct as cross-check by default.
```

You pay NVIDIA directly. No Slate-hosted service receives your key or frames;
your local Slate process sends sampled frames directly to the configured
provider.

### Anthropic Panel lane (cloud, BYO key)

```bash
export ANTHROPIC_API_KEY=replace-with-anthropic-api-key
slate verify --frames ./my_render --manifest ./shot.json --panel
```

For stricter local review, use `--panel-provider local` with Ollama. Treat local Panel mode as lower assurance until you benchmark it against your own renders.

## Project status

Pre-release. v0.1 packages validation patterns from internal animation QA work
for general use. See [CHANGELOG.md](CHANGELOG.md).

## Contributing

Issues and pull requests welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) once it exists.

## License

MIT — see [LICENSE](LICENSE).
