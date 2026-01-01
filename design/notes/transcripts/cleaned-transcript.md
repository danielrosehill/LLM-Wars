# LLM Wars - Experiment Design Notes

**Title:** AI Model Debate Podcast
**Date:** 1 Jan 2026

---

## The Concept

One AI experiment that I have to try came to mind. I've been working on my AI podcast and trying an ambitious second pass tweak to the pipeline. The process of creating a debate podcast has been really interesting, and I've really enjoyed listening to it. So I was thinking of what models I could use in a revised or expanded pipeline, and then this idea popped into my head—it would actually be really fun to try.

## The Core Idea

Models are emerging all the time between major AI labs, and each touts its superiority. Of course, for serious evaluation, there's benchmarks and testing. But I thought it would be interesting to create a podcast in which two warring models debate each other.

For my initial matchup: **Gemini 3 Flash** vs **Anthropic Opus 4.5**. This is the head-to-head.

## Format Overview

The format is a podcast intended to be about 30 minutes long, in which the two models argue with one another about who's better. It's a back-and-forth debate where each model tries to prove its superiority over the other. It might not be the most enjoyable thing to listen to, but it would be a clever thing to try.

## Orchestration Requirements

The orchestration has to be done carefully because the potential for bias is enormous. There has to be a neutral party working as the orchestrator that isn't either model.

**Orchestration options:** LangGraph, LangChain, CrewAI, Python

**Neutral model candidate:** Qwen (for the glue/orchestration layer)

## Text Generation Target

To get this to work, we need to generate about 30 minutes worth of text. Whatever the words-per-minute calculation comes out to, that's the target.

## Agentic Architecture

The format that makes sense is an agent-to-text orchestration framework.

### The Orchestrator Agent (Qwen)

**System prompt:** "You are a friendly diplomatic podcast host facilitating episode one of LLM Wars."

The system prompt will be reusable so I can try this with different models:
- Model One = Variable 1
- Model Two = Variable 2

### The Debating Models

Each model is told to prove their "betterness"—that they're the best model out there, specifically better than the other model.

**System prompt for Model A (e.g., Gemini 3 Flash):**
"You are participating in a podcast debate with another AI model. You are blinded initially. Your task is to convince anyone listening why you are the better model and why people should choose you over the other model you'll be debating."

**System prompt for Model B (e.g., Anthropic Opus 4.5):**
Same instruction: "You are representing X. You are here on this podcast to prove that you are the best. You will respond to the other model."

## Debate Flow Logic

1. The moderator initiates: "Gemini, welcome to the show. You are debating Anthropic Opus 4.5 today. Can you begin by telling us about yourself and why you are better?"
2. Gemini generates text response
3. Moderator says: "Anthropic, please respond"
4. Anthropic responds
5. This continues back and forth

**Context window consideration:** The context needs to be maintained so models can refer to previous points in the debate. Could also work as a 15-minute episode if context becomes a concern.

## Output Format

The debate is captured as a JSON file, ready for TTS processing.

## TTS Pipeline Design

### Moderator Exclusion

The moderating agent will NOT be included in the TTS. Its purpose is just to orchestrate the debate. Its messages to the sub-agents won't be included—they can be terse and direct. This way, when we concatenate the audio with short pauses between each speaker, it will appear as a direct dialogue between the two models.

### Episode Structure

Since the raw debate won't have natural narrative flow, there will be:

1. **Intro Agent:** Generates a short intro script: "Welcome to episode one of LLM Wars. We're going to hear from Gemini 3 and Anthropic 4.5."
2. **Main Debate:** Concatenated audio from the debate
3. **Outro Agent:** Generates a closing message

### Audio Processing

- **Normalization** applied to final output
- **Pauses between segments:**
  - 0.5-1 second pause between debating agents
  - 2 second pause between intro and debate start
  - 2 second pause between debate end and outro

## Voice Assignment

Each model needs a distinct voice:
- Voice for Gemini
- Voice for Anthropic

**TTS Tool:** Chatterbox (cost-effective)

No voice clones—just find appropriate voices for each model.

## Future Possibilities

If the concept is successful and entertaining, it could be fun to run with different model matchups:
- Rival Chinese models (the new ones coming out)
- Models prompted to "win this argument" (might work against their own system prompts, but worth trying)

---

## Additional Note: Tone Requirements

**Important:** The system prompts must instruct each agent to be **unapologetic** and not overly "nice." They're there to really argue their superiority, not to butter up the ego of the other agent. The episode needs to have a bit of **bite** - a real debate, not a polite exchange of compliments.

---

*These are the design notes for LLM Wars Episode 1.*
