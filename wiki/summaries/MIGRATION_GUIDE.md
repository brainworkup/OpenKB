---
doc_type: short
full_text: sources/MIGRATION_GUIDE.md
---

# Migration Guide: Single File → Modular Core
This document maps how callers should transition after `neuro_report_soul_agent.py` was refactored into 8 modules under `core/`. The original file now exists as a shim re-exporting everything, so old imports still resolve but new code is directed to the real locations.

## What Changed
- **Original**: A single 1,374-line script containing text processing, vector ops, database logic, profile training, and report writing all in one file.
- **New**: Split into `core/` with modules sized under ~600 lines each — following [[concepts/architecture-decision-records]] ADR 0003 to improve readability and testability.

## Import Changes (The most important update)
Update existing codebases to import from the modular API instead of the legacy shim. Using the shim indefinitely will hide the real architecture and complicate future changes. The old imports still resolve because the shim re-exports everything — a deliberate measure so that nothing breaks immediately, including existing scripts and CLI commands.

### Recommended New Imports
```python
from core.text_processing import extract_text, chunk_text, map_heading_to_section\
from core.text_processing import ChunkRecord, SectionText, CANONICAL_SECTIONS\
from core.vector_ops import omlx_embed, omlx_generate, cosine_similarity\
from core.vector_ops import embed_with_fallback, generate_with_fallback\
from core.database import build_index, retrieve_top_k, init_db\
from core.soul_profile import train_soul\
from core.report_generation import write_report, write_section\
from core.validation import validate
```

## Fix Broken Mock Paths in Tests
Unit tests patching at the old location will silently fail or mock nothing because `neuro_report_soul_agent` is now a shim. **Always patch at the real module path:**
```python
# Old (broken — these don't exist anymore):
with patch("neuro_report_soul_agent.omlx_embed", ...)

# New (correct):
with patch("core.vector_ops.omlx_embed", ...)
with patch("core.vector_ops.omlx_generate", ...)
```
If tests fail with mock errors after the migration, this is almost certainly a stale import path in a test file rather than a regression in the logic itself.

## CLI Reference
The entry point remains `main.py`, which delegates to `cli.py` and calls into core modules via standard subcommands. The shim ensures existing shell scripts that call `neuro_report_soul_agent.build-index` or `train-soul` still run without modification, but new pipelines should reference the explicit module imports above.

**Core commands:**
- **Build embedding index**: \`python main.py build-index --reports-dir ./reports/neuropsych_report_pdf_bucket --db-path ./soul_db/report_soul_index.sqlite --aliases-path ./section_aliases.json --chunk-size 1200 --overlap 150\` — builds the FAISS index with chunking and heading mapping.
- **Train global soul profile**: \`python main.py train-soul --db-path ./soul_db/report_soul_index.sqlite --profile-path ./soul_db/report_soul_profile.json --soul-examples 25\` — learns the writing voice from a batch of examples.
- **Train section-specific profile**: \`python main.py train-soul ... --section RECOMMENDATIONS --soul-examples 25\` — produces a finer-grained profile tuned specifically to that report section's voice and formatting.
- **Write report** (per section): \`python main.py write-report ... --profile-path ./soul_db/report_soul_profile.json --prompt "Draft recommendations..." --section RECOMMENDATIONS --output ./reports/...\` — generates the draft text using retrieval over the index.
- **Validate pipeline**: \`python main.py validate ...\` — runs an end-to-end health check across all core modules.

**New CLI options:**
- `--use-vector-index / --vector-index-path`: toggles FAISS retrieval (default on). If removed, it falls back to pure cosine similarity over the chunk set.
- `--omlx-url`, `--omlx-embed-model`, `--omlx-gen-model`: configures the model server and specific models used for embeddings and generation at runtime.

## Section-Specific Voice Workflow
Training a dedicated profile per section captures that voice better than one global profile, because each section has different stylistic conventions (NSE is drier; RECOMMENDATIONS more advisory). Follow this workflow:
1. **Train all four profiles** (NSE, TESTING, SUMMARY, RECOMMENDATIONS) using either `train_section_profiles.sh` or by running the manual training commands for each --section with 25 examples.
2. **Use the matching profile when writing**: For a recommendations draft, call \`python main.py write-report ... --profile-path ./soul_db/report_soul_profile.recommendations.json --section RECOMMENDATIONS\` — this ensures the voice is tuned to that section's previous examples rather than averaged across all sections.

## API for Orchestrators (Luria / AI Agents)
For code that calls this pipeline programmatically:
```python
from core.report_generation import write_section\
draft = write_section(
    prompt="Draft a Recommendations section...",\
    db_path="./soul_db/report_soul_index.sqlite",\
    profile_path="./soul_db/report_soul_profile.recommendations.json",\
    section="RECOMMENDATIONS",\
    top_k=6,\
    temperature=0.2,\
    omlx_url="http://localhost:8000/v1",\
    omlx_embed_model="Qwen3-Embedding-8B-4bit-DWQ",\
    omlx_gen_model="MLX-Qwen3.5-35B-A3B-Claude-4.6-Opus-Reasoning-Distilled-4bit",
)\
print(draft)
```
This uses the Qwen2.5/Qwen3 family on Apple Silicon via MLX, backed by an olmx server instance and indexed with a [[concepts/vector-search]] over the report chunks.

## Directory Paths & Infrastructure
- **Input reports**: `REPORTS_DIR_IN = "./reports/neuropsych_report_pdf_bucket/"` — where raw PDF source material is staged.
- **Output drafts**: `REPORTS_DIR_OUT = "./reports/soulful_reports/"` — where generated sections are written for review.
- **SQLite database**: `./soul_db/report_soul_index.sqlite` — the single file that stores chunk embeddings and learned profile weights; rebuilding it reloads all profiles from scratch.

## Troubleshooting & Fallbacks
- **Import errors after switching branches** — likely stale Python cache with old module resolution: \`find . -name "__pycache__" -type d -exec rm -rf {} + && find . -name "*.pyc" -delete\`\
- **FAISS not available**: The system falls back to pure in-memory cosine similarity automatically; no action is required, but install with `uv add faiss-cpu` for retrieval speed on larger datasets.
- **docling crashes on macOS** — automatic fallback to PyMuPDF is built-in; if that's missing or you want the parser switched: \`uv add pymupdf\`\
- **validate command exits non-zero**: check each module output in the terminal summary; it points directly at which of text_processing, vector_ops, database, soul_profile, report_generation, validation failed its health checks.

## Related Concepts
- [[concepts/model-routing]]
- [[concepts/voice-profile-per-section]]
- [[concepts/codebase-reorganization]]
- [[concepts/qwen35-model-capabilities]]
- [[concepts/uxm_client-and-api]]
- [[concepts/write-report---section-profiles-sh]]
