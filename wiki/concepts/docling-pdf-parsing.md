---
sources: [summaries/AGE_OVERRIDE_GUIDE.md, summaries/extract_pdf.md, summaries/DEPENDENCIES.md, summaries/README.md]
brief: Local-first PDF parsing library that extracts text and layout from neuropsych reports before PHI redaction.
---

# Docling PDF Parsing

Docling is a local-first PDF parsing library used as the first stage of the [[concepts/neuropsychological-assessment-pipeline]] to extract text and layout information from neuropsychological PDF reports. It runs entirely on-device, ensuring that raw document content — including protected health information (PHI) — never leaves the local machine before redaction.

## Role in the Pipeline

In the [[concepts/luria-neuropsych-pipeline]], Docling serves as the **Parse** node within the LangGraph ingest graph (defined in `neuropsych_agent/graph.py`). Its responsibilities are:

1. Accept a raw PDF file path as input (uploaded to `data/uploads/` via the Streamlit UI).
2. Extract structured text and layout (headings, tables, paragraphs) from the document.
3. Trigger local PHI redaction before any extracted content is forwarded to a cloud API.

Only after Docling completes parsing and PHI is stripped locally does the pipeline proceed to call Anthropic's Claude Sonnet for structured data extraction. This sequence is a core [[concepts/privacy-first-software]] design decision and is central to the [[concepts/local-first-architecture]] of the application.

See [[summaries/README]] for the full architectural context.

## Why Local Parsing Matters

Neuropsychological reports contain highly sensitive patient data — diagnoses, test scores, demographic details — all of which constitute PHI under HIPAA. By keeping the parse step local:

- No raw PHI ever transits the network.
- [[concepts/pii-redaction-pipelines]] can operate on the extracted text before any cloud call.
- The application remains suitable for solo clinicians without institutional IT infrastructure.
- Audio transcription (MacWhisper) and summarization (oMLX) similarly remain local, reinforcing an end-to-end local-first posture.

This is directly contrasted with approaches that send raw PDFs to cloud OCR services. See also [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]].

## Relationship to OCR

Docling handles both native-text PDFs (where text is embedded in the PDF structure) and scanned documents requiring optical character recognition. For fully scanned reports, Docling's OCR capabilities overlap with the broader [[concepts/ocr-pipeline]] concerns documented elsewhere in the project. On first run, Docling downloads its underlying models automatically, which can cause the parse stage to appear to hang — users can pre-download models using the `docling` CLI to avoid this during live workflows. See the Troubleshooting section in [[summaries/README]].

## Integration Points

- **Input**: Raw PDF file path, supplied by the Streamlit UI after upload to `data/uploads/`.
- **Output**: Extracted text and layout structure, passed as the `parsed` field in `PipelineState` to the Extract node (Claude Sonnet).
- **Implementation**: Wrapped in `neuropsych_agent/tools/pdf_parser.py`.
- **State field**: `parsed` in `PipelineState` (`neuropsych_agent/state.py`).
- **Downstream stores**: Parsed narrative chunks flow into [[concepts/lancedb-vector-store]] (`data/vectors/`) for semantic retrieval, and structured extractions go into SQLite (`data/neuropsych.db`).

## Position in the Four-Stage Ingest Graph

The README confirms the four-stage LangGraph pipeline built by `build_ingest_graph()`:

1. **Parse** — Docling (this stage)
2. **Extract** — Claude Sonnet structures narrative into JSON (test scores, clinical summaries)
3. **Index** — SQLite + LanceDB store structured and vector data locally
4. **Report** — Generates markdown narrative; optional Typst or Quarto rendering

Docling's parse output feeds directly into the Extract node. The convenience wrapper `ingest_pdf(path, mode="ingest", **voice_kw)` in `neuropsych_agent/graph.py` orchestrates this full flow from the Streamlit UI.

## Related Concepts

- [[concepts/clinical-nlp-pipelines]] — downstream processing of parsed text
- [[concepts/pii-redaction-pipelines]] — redaction applied to Docling output
- [[concepts/phi-data-handling]] — HIPAA-conscious data handling strategy
- [[concepts/retrieval-augmented-generation]] — the RAG system that consumes indexed parsed content
- [[concepts/lancedb-vector-store]] — vector storage for parsed narrative chunks
- [[concepts/neuropsychological-assessment-pipeline]] — the full pipeline Docling initiates
- [[concepts/ocr-pipeline]] — overlapping OCR concerns for scanned PDFs
- [[concepts/pdf-score-extraction]] — structured score extraction that follows parsing
- [[concepts/local-first-architecture]] — the broader design philosophy Docling embodies
- [[concepts/langgraph-agent-workflows]] — the StateGraph that hosts the parse node
- [[summaries/neuropsych-pdf-parser]] — summary of the PDF parser subagent
- [[summaries/DEPENDENCIES]] — dependency context
- [[summaries/extract_pdf]] — PDF extraction tooling details


See also: [[summaries/AGE_OVERRIDE_GUIDE]]