# ingest-agentpack

Eve AgentPack for document ingestion — text, audio, video processing.

## Install

Add to your `.eve/manifest.yaml`:

```yaml
x-eve:
  packs:
    - source: github:eve-horizon/ingest-agentpack
      ref: <40-char-sha>
```

Sync agents:

```bash
eve agents sync --ref main --project <project-id>
```

## What's Included

| Component | Description |
|-----------|-------------|
| `doc_processor` agent | Processes ingested files using the doc-processor skill |
| `process-document` workflow | Triggers on `doc.ingest` events |
| `ingest` profile | Opus 4.6 with medium reasoning (quality) |
| `ingest-fast` profile | Sonnet 4.6 with low reasoning (speed/cost) |
| `doc-processor` skill | Full processing instructions for text, audio, video |

## Customize

Override the harness profile in your manifest:

```yaml
x-eve:
  agents:
    doc_processor:
      harness_profile: ingest-fast
```

## How It Works

1. `eve ingest <file>` uploads to S3 and fires a `doc.ingest` event
2. The `process-document` workflow matches the event and creates a job
3. The `doc_processor` agent runs with the `doc-processor` skill
4. The agent reads the file, processes it based on MIME type, and writes a structured summary

### Media Processing

- **Audio**: Transcribed via `whisper-cli` with the small English model
- **Video**: Audio extracted via `ffmpeg`, then transcribed via `whisper-cli`
- **Text/Documents**: Read directly and summarized
