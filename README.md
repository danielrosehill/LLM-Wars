# LLM Wars

An AI-generated podcast where competing LLMs debate each other to prove their superiority.

## Concept

Two AI models (e.g., Gemini 3 Flash vs Claude Opus 4.5) argue head-to-head in a podcast format, each trying to convince listeners why they're the better model. A neutral orchestrator (Qwen) moderates without appearing in the final audio.

## Structure

```
├── notes/           # Transcripts and source audio
│   ├── audio/       # Original voice notes
│   ├── verbatim-transcript.md
│   └── cleaned-transcript.md
└── design/          # Implementation methodology
    └── methodology.md
```

## Key Design Points

- **Neutral orchestration** via Qwen to avoid bias
- **Unapologetic tone** - models argue with bite, not politeness
- **Reusable prompts** for different model matchups
- **TTS pipeline** using Chatterbox with distinct voices per model

## Tech Stack

- Orchestration: LangGraph / LangChain / CrewAI
- TTS: Chatterbox
- Audio: FFmpeg (concatenation + normalization)

See [design/methodology.md](design/methodology.md) for full implementation details.
