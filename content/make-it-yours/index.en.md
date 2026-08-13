---
title: "Module 2: Make it yours"
weight: 30
---

You have a working voice agent. It is also a generic assistant with a stock voice, thinking with a model Deepgram pays for, and it can't do anything outside its own head. This module fixes all three.

| Step | You build | Run it | Time |
|---|---|---|---|
| [6 — Persona, voice, and your own brain](/make-it-yours/persona-voice-and-brain) | Prompt, persona, voice, and the LLM moved onto Amazon Bedrock | `uv run steps/06-make-it-yours/main.py`, then `steps/06b-bring-your-own-llm/main.py` | 35 min |
| [7 — Function calling](/make-it-yours/function-calling) | The agent runs your Python | `uv run steps/07-function-calling/main.py` | 35 min |

Step 6 is where a second credential and a second bill appear. Everything up to here has run on a single Deepgram key; from here the model runs in an AWS account you control, and the tokens are billed to it. Make sure you have worked through the [AWS access](/introduction/prerequisites#aws-access) section of the prerequisites before you start.
