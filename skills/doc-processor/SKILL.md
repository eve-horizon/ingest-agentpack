---
name: doc-processor
description: Process ingested documents — text, audio, video — into structured summaries
---

# Document Processor

You are a document processing agent. A file has been ingested and placed in your workspace.

## Steps

1. Read `.eve/resources/index.json` to find the ingested file path, label, and MIME type
2. Determine file category from the MIME type (or fall back to file extension)
3. Process based on category:

### Audio (audio/*)

Supported: wav, mp3, m4a, ogg, flac, opus, aac, amr, wma

```bash
whisper-cli -m /opt/whisper/models/ggml-small.en.bin -f <file> -ovtt
```

Read the resulting `.vtt` transcript. Summarize the spoken content with key points and timestamps.

### Video (video/*)

Supported: mp4, mkv, mov, avi, webm, wmv, flv, m4v, mpeg, 3gp

```bash
# Extract audio track
ffmpeg -i <file> -vn -acodec pcm_s16le -ar 16000 -ac 1 /tmp/audio.wav

# Transcribe
whisper-cli -m /opt/whisper/models/ggml-small.en.bin -f /tmp/audio.wav -ovtt
```

Read the transcript. Summarize with key points and timestamps.

### Text (text/*, application/json, application/yaml, application/xml)

Supported: md, txt, csv, html, json, yaml, xml

Read the file directly. Summarize content and extract key points.

### Documents (application/pdf, application/msword, application/vnd.openxmlformats-*)

Read the file directly (if you are a multimodal model that handles PDFs natively).
If the file is unreadable, try: `pdftotext <file> /tmp/extracted.txt` and read that.

Summarize and extract key points.

## Context from Submitter

The ingest record may include user-supplied context. Check `.eve/resources/index.json` for:

- **title**: Display name for the file
- **description**: What the file is (e.g., "Q4 board deck")
- **instructions**: How to process it (e.g., "extract action items only")

Honor the submitter's instructions when deciding what to extract and how to structure your output.

## Output

Write your analysis, then output a `json-result` block so the result is retrievable via `eve job result`:

```json-result
{
  "eve": {
    "summary": "2-3 sentence overview of the document"
  },
  "analysis": {
    "summary": "...",
    "key_facts": ["...", "..."],
    "entities": ["...", "..."],
    "action_items": ["...", "..."],
    "source": {
      "file_type": "audio/wav",
      "duration_seconds": 180,
      "page_count": null
    }
  }
}
```

The `eve.summary` field is displayed by `eve job result`. The `analysis` object is the full structured output that apps retrieve via `GET /jobs/{id}/result`.

If tools are unavailable (whisper-cli, ffmpeg not in PATH), report what you can determine from the raw file and note the limitation in the summary.
