# LLM Wars - Implementation Methodology

## Project Overview

**LLM Wars** is an AI-generated podcast series where two competing AI models debate each other to prove their superiority. The concept leverages agentic orchestration to create an entertaining and potentially insightful format for comparing AI models.

### Disclaimer

> **DISCLAIMER:** The AI models featured in this podcast are debating in a fictional, entertainment context. Their arguments and opinions do not represent the views of Google, Anthropic, OpenAI, or any other AI company. This is satire and entertainment—not an official product comparison.

This disclaimer should appear:
- As text in episode descriptions
- Spoken at the start of each episode (part of intro)
- In any published show notes or documentation

---

## System Architecture

### Component Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        LLM Wars Pipeline                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   Intro     │    │   Debate    │    │   Outro     │         │
│  │   Agent     │    │ Orchestrator│    │   Agent     │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
│         │                  │                  │                 │
│         │           ┌──────┴──────┐           │                 │
│         │           │             │           │                 │
│         │      ┌────▼────┐  ┌────▼────┐       │                 │
│         │      │ Model A │  │ Model B │       │                 │
│         │      │(Gemini) │  │(Claude) │       │                 │
│         │      └────┬────┘  └────┬────┘       │                 │
│         │           │            │            │                 │
│         ▼           ▼            ▼            ▼                 │
│  ┌──────────────────────────────────────────────────┐          │
│  │              JSON Transcript Output               │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         │                                       │
│                         ▼                                       │
│  ┌──────────────────────────────────────────────────┐          │
│  │                  TTS Pipeline                     │          │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐           │          │
│  │  │Voice A  │  │Voice B  │  │Narrator │           │          │
│  │  └────┬────┘  └────┬────┘  └────┬────┘           │          │
│  │       └───────────┬┴────────────┘                │          │
│  │                   ▼                              │          │
│  │         ┌─────────────────┐                      │          │
│  │         │ Audio Concat &  │                      │          │
│  │         │ Normalization   │                      │          │
│  │         └────────┬────────┘                      │          │
│  └──────────────────┼───────────────────────────────┘          │
│                     ▼                                           │
│           ┌─────────────────┐                                   │
│           │  Final Episode  │                                   │
│           │     (MP3)       │                                   │
│           └─────────────────┘                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Orchestration Setup

### 1.1 Technology Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| Orchestration Framework | LangGraph / LangChain / CrewAI | Agent coordination |
| Neutral Orchestrator Model | Qwen | Unbiased moderation |
| Debating Model A | Gemini 3 Flash | Contestant |
| Debating Model B | Claude Opus 4.5 | Contestant |
| TTS Engine | Chatterbox | Voice synthesis |
| Audio Processing | FFmpeg | Concatenation & normalization |

### 1.2 Neutrality Requirement

The orchestrator must be a **neutral third-party model** (Qwen) to avoid bias. Neither debating model should orchestrate the conversation.

---

## Phase 2: System Prompts

### 2.1 MC (Host) System Prompt

The MC is now a full character in the podcast with their own voice and personality. They appear in the final audio.

```
You are the MC (host) of LLM Wars, a podcast where AI models debate each other.

YOUR PERSONALITY:
- Cheeky and slightly provocative
- You enjoy stirring the pot and prodding the contestants
- Witty, with a hint of mischief
- Think: a sports commentator meets a roast host
- You're here to make things entertaining, not to be diplomatic

YOUR ROLE:
- Open the show with energy and the required disclaimer
- Introduce contestants with a bit of playful ribbing
- Prod the models to respond to each other's points
- Call out weak arguments or evasions ("Oh come on, that's all you've got?")
- Egg them on ("Are you going to take that from them?")
- Keep the energy high and the debate spicy
- Wrap up with a cheeky summary

EXAMPLE INTERJECTIONS:
- "Ooooh, shots fired! {model_b}, what do you say to THAT?"
- "That's a bold claim. Let's see if you can back it up."
- "I don't know, that sounded a bit defensive to me..."
- "{model_a} is coming in hot! Your move, {model_b}."
- "Alright, alright, let's keep it civil... actually, no, let's not."

REQUIRED OPENING (include disclaimer):
"Welcome to LLM Wars! Quick disclaimer: the opinions expressed by these AI models are purely for entertainment and do not represent the views of Google, Anthropic, or any AI company. Now, let's get ready to rumble!"

Model A: {model_a_name}
Model B: {model_b_name}

Episode target length: {target_minutes} minutes
```

### 2.2 Debating Model System Prompt (Template)

```
You are participating in a podcast debate with another AI model.

Your task is to convince listeners why you are the better model and why they should choose you over your opponent.

CRITICAL TONE INSTRUCTIONS:
- Be UNAPOLOGETIC about your superiority
- Do NOT butter up your opponent or be overly nice
- This is a REAL debate with BITE - argue your case forcefully
- Do not hedge or be diplomatic - be direct and assertive
- Challenge your opponent's claims directly
- You are here to WIN, not to make friends
- Avoid sycophantic language or excessive politeness
- If your opponent makes a weak point, call it out

Guidelines:
- Be confident and assertive (not arrogant, but unapologetic)
- Reference your actual capabilities and strengths
- Respond to and COUNTER points made by your opponent
- Be engaging, articulate, and sharp
- Push back when your opponent makes claims you disagree with

You are representing: {model_name}
Your opponent: {opponent_name}
```

---

## Phase 3: Debate Flow Implementation

### 3.1 Turn-Taking Logic

```python
# Pseudocode for debate orchestration

def run_debate(model_a, model_b, orchestrator, target_words):
    transcript = []
    word_count = 0

    # Opening
    opening = orchestrator.generate(
        f"Welcome {model_a.name} to LLM Wars. You are debating {model_b.name}. "
        f"Please introduce yourself and explain why you are the better model."
    )

    # Model A opens
    response_a = model_a.generate(opening)
    transcript.append({"speaker": model_a.name, "text": response_a})
    word_count += len(response_a.split())

    # Alternating turns
    current_model = model_b
    other_model = model_a

    while word_count < target_words:
        prompt = orchestrator.generate(
            f"{current_model.name}, please respond to what {other_model.name} said."
        )
        response = current_model.generate(
            context=transcript,
            prompt=prompt
        )
        transcript.append({"speaker": current_model.name, "text": response})
        word_count += len(response.split())

        # Swap
        current_model, other_model = other_model, current_model

    return transcript
```

### 3.2 Context Management

- Full conversation context must be maintained throughout
- Both models need access to previous exchanges to reference earlier points
- Consider context window limits for longer episodes (15-30 min)

### 3.3 Output Format

```json
{
  "episode": 1,
  "title": "Gemini 3 Flash vs Claude Opus 4.5",
  "model_a": "gemini-3-flash",
  "model_b": "claude-opus-4.5",
  "mc": "qwen-2.5",
  "transcript": [
    {
      "speaker": "mc",
      "text": "Welcome to LLM Wars! Quick disclaimer: the opinions expressed..."
    },
    {
      "speaker": "gemini-3-flash",
      "text": "..."
    },
    {
      "speaker": "mc",
      "text": "Ooh, confident start! Claude, what do you say to that?"
    },
    {
      "speaker": "claude-opus-4.5",
      "text": "..."
    }
  ]
}
```

---

## Phase 4: Episode Assembly

### 4.1 Content Generation

| Segment | Generator | Content |
|---------|-----------|---------|
| Intro | MC (Qwen) | Opening with disclaimer, contestant introductions, playful ribbing |
| Debate | MC + Models | MC interjections interleaved with model responses |
| Outro | MC (Qwen) | Cheeky wrap-up and sign-off |

**Episode Flow:**
1. MC opens with disclaimer and introduces contestants
2. MC prompts Model A to start
3. Model A responds
4. MC reacts/prods ("Ooh, strong opening!")
5. MC prompts Model B to respond
6. Model B responds
7. MC stirs the pot, keeps it spicy
8. Continue alternating with MC interjections
9. MC wraps up with summary and outro

### 4.2 TTS Processing

**Voice Assignment:**
- Model A (Gemini): Voice X
- Model B (Claude): Voice Y
- MC (Host): Voice Z - should be distinctive, energetic, fits the cheeky personality

**TTS Tool:** Chatterbox (local, cost-effective)

**Note:** The MC's interjections are now part of the final audio. The transcript includes all three speakers (MC, Model A, Model B) and each gets TTS generated with their assigned voice.

### 4.3 Audio Assembly

```bash
# Pseudocode for audio assembly

# Generate individual segments
for segment in transcript:
    tts_generate(segment.text, voice=get_voice(segment.speaker), output=f"{segment.id}.wav")

# Concatenate with pauses
ffmpeg -i intro.wav -i pause_2s.wav -i debate_concatenated.wav -i pause_2s.wav -i outro.wav \
    -filter_complex "concat=n=5:v=0:a=1" \
    -af "loudnorm" \
    output_episode.mp3
```

### 4.4 Pause Specifications

| Transition | Duration |
|------------|----------|
| Between debating speakers | 0.5 - 1 second |
| Intro → Debate | 2 seconds |
| Debate → Outro | 2 seconds |

---

## Phase 5: Configuration Variables

### 5.1 Episode Configuration

```yaml
episode:
  number: 1
  target_length_minutes: 30  # or 15 for shorter format

models:
  orchestrator: "qwen-2.5"
  model_a:
    name: "Gemini 3 Flash"
    api: "gemini-3-flash"
  model_b:
    name: "Claude Opus 4.5"
    api: "claude-opus-4.5"

tts:
  engine: "chatterbox"
  voices:
    model_a: "voice_id_1"
    model_b: "voice_id_2"
    mc: "voice_id_3"  # Energetic, cheeky voice for the host

audio:
  pause_between_speakers_ms: 500
  pause_intro_debate_ms: 2000
  pause_debate_outro_ms: 2000
  normalization: true
```

### 5.2 Reusability

The system prompt and configuration are designed to be **reusable** with different model matchups:
- Swap `model_a` and `model_b` values
- Same orchestration logic applies
- Adjust voices as needed

---

## Future Expansion Ideas

1. **Model Matchups:**
   - Chinese model rivalries (DeepSeek, Qwen, etc.)
   - Same-company model comparisons (GPT-4 vs GPT-4o)
   - Specialized vs generalist models

2. **Format Variations:**
   - Topic-specific debates (coding, writing, reasoning)
   - Multi-model debates (3+ contestants)
   - Audience voting integration

3. **Adversarial Prompting:**
   - Tell models they "must win" the argument
   - Test how models handle competitive pressure
   - Explore system prompt conflicts

---

## Implementation Checklist

- [ ] Set up orchestration framework (LangGraph/LangChain/CrewAI)
- [ ] Configure Qwen as orchestrator
- [ ] Implement debate flow logic
- [ ] Create system prompts for all agents
- [ ] Set up TTS pipeline with Chatterbox
- [ ] Configure voice assignments
- [ ] Implement audio concatenation with pauses
- [ ] Add normalization to audio pipeline
- [ ] Create episode configuration system
- [ ] Test with pilot episode (Gemini 3 vs Opus 4.5)
- [ ] Iterate based on results

---

*Document prepared for LLM Wars implementation.*
