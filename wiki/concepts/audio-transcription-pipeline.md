---
sources: [summaries/README.md]
brief: A fully local pipeline for transcribing and summarizing clinical audio using on-device ML tools.
---

# Local Audio Transcription and Summarization Pipeline

A pipeline for converting clinical audio recordings into text transcripts and structured summaries entirely on a local machine — with no audio data ever sent to cloud services. This approach is essential in neuropsychological and clinical settings where recorded sessions, dictations, or interviews may contain sensitive patient information governed by HIPAA.

## Core Principles

- **Privacy by default**: Audio files are processed entirely on-device; no cloud ASR (Automatic Speech Recognition) services are used.
- **Integrated into clinical workflows**: The pipeline is embedded alongside PDF ingestion, RAG Q&A, and report generation in the same application interface.
- **Composable with local LLMs**: Transcription feeds directly into a local summarization model without any data leaving the machine.

See [[summaries/README]] for the full application context.

## Components

### MacWhisper CLI (`mw`)

MacWhisper is an on-device transcription tool for macOS that wraps OpenAI's Whisper model. In the Luria pipeline it is invoked via its CLI interface (`mw`) and supports common clinical audio formats:

- `.m4a` (common from iPhone/iPad voice memos)
- `.mp3`
- `.wav`

Because Whisper runs locally via MacWhisper, no audio bytes are transmitted externally. See [[concepts/local-first-architecture]] for the broader design philosophy.

### oMLX Local Summarization Server

After transcription, the raw transcript is passed to a locally running oMLX server (an OpenAI-compatible inference server built on Apple's MLX framework) for summarization. This keeps the entire pipeline on-device. See [[concepts/omlx-server]] and [[concepts/mlx-framework]] for technical details on the inference layer.

The summarization model is configured via:
```bash
OMLX_CHAT_MODEL=<model-id>  # e.g., Qwen3.6-35B-A3B-oQ4
OMLX_BASE_URL=http://127.0.0.1:8000/v1
```

## Workflow

1. User uploads an audio file (`.m4a`, `.mp3`, `.wav`) via the **Audio** tab in the Streamlit UI.
2. The app invokes `mw` (MacWhisper CLI) to transcribe the audio to plain text.
3. The transcript is passed to the local oMLX server for summarization using a configured chat model.
4. Both the transcript and summary are made available for download.

## Integration with the Luria Pipeline

This audio pipeline is one of four functional modules in the [[concepts/luria-neuropsych-pipeline]] application:

| Tab | Function |
|---|---|
| Ingest | PDF → structured clinical data |
| Ask | RAG Q&A against ingested reports |
| Knowledge Base | Browse SQLite tables |
| **Audio** | **Transcribe + summarize recordings** |

The audio pipeline is intentionally decoupled from the PDF ingestion workflow — transcripts are not automatically indexed into the RAG knowledge base, but could be fed in as additional documents in future iterations.

## Privacy & Security Considerations

Audio recordings in clinical contexts are among the most sensitive PHI artifacts:

- MacWhisper runs the Whisper model **entirely on-device** — no audio is uploaded.
- The oMLX summarization server runs **locally** on `127.0.0.1`.
- No transcript or summary is transmitted to any external API.

This aligns with the broader [[concepts/phi-data-handling]] and [[concepts/privacy-first-software]] principles of the application. See [[summaries/SECURITY]] for additional security considerations across the project.

## Related Concepts

- [[concepts/local-first-architecture]] — overarching design principle ensuring data stays on-device
- [[concepts/local-llm-inference]] — how local models handle summarization
- [[concepts/omlx-server]] — the local OpenAI-compatible inference server
- [[concepts/mlx-framework]] — Apple Silicon ML framework underpinning local inference
- [[concepts/phi-data-handling]] — PHI protection requirements driving local-only processing
- [[concepts/privacy-first-software]] — broader design philosophy
- [[concepts/openai-compatible-api]] — the API contract used by oMLX for summarization
- [[concepts/audio-transcription-pipeline]] — related pipeline concept
- [[summaries/README]] — primary source document
- [[summaries/README_PIPELINE]] — pipeline architecture details
