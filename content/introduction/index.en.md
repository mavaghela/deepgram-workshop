---
title: "Introduction"
weight: 10
---

Build a real-time voice agent in Python, one runnable step at a time. You speak, it listens on Flux, thinks with an LLM, answers in a natural voice, stops when you interrupt it, and calls your code when it needs information from your domain.

Every step is a complete, working program that opens a browser tab and talks to you. The agent is entirely Python, the browser is only the microphone and the speaker.

Start at Step 1 or drop in at Step 5; each folder already contains everything the previous steps built.

## What you'll learn

- Configuring speech-to-text, an LLM, and text-to-speech through a single WebSocket
- Why Flux does turn detection server-side, and what that removes from your client
- Streaming microphone audio without blocking the thread that plays the reply
- Clearing queued playback the instant a user interrupts — the difference between a demo and something people will use
- Moving the agent's brain onto Amazon Bedrock in your own AWS account
- Wiring client-side function calls into a live conversation
- Trading turn-detection latency against accuracy, and measuring it honestly

## How the workshop is laid out

| Module | Steps | You build | Time |
|---|---|---|---|
| [Introduction](/introduction) | 0 | The parts of a voice agent | 10 min |
| [Build the agent](/build-the-agent) | 1 to 5 | Setup, the WebSocket handshake, playback, microphone input, barge-in | ~90 min |
| [Make it yours](/make-it-yours) | 6 and 7, plus optional 7b | Prompt, persona, voice, Amazon Bedrock, and a phone banking agent that answers from your data | ~55 min |
| [Optimize](/optimize) | 8 | End-of-turn thresholds and latency | 20 min |

Running behind? Steps 1 to 5 are the core — finish those and you have a working voice agent. Step 8 is dials rather than code and makes the natural take-home. And because every folder is complete, skipping ahead costs you the typing, not the workshop.

Running ahead? [Step 7b](/make-it-yours/healthcare) points the same function-calling machinery at a second vertical, and is off the chain in both directions — nothing after it depends on it.

Each step has **Check yourself** questions to test your understanding as you go, with the answer in an expandable block, and **Pause** markers where a live workshop regroups.

## Where the code lives

You'll clone [deepgram-devs/deepgram-workshop-py](https://github.com/deepgram-devs/deepgram-workshop-py) and work inside it. Each `steps/NN-slug/` folder holds the finished state of the step before it: work the `TODO (Step N.x)` blocks in `main.py`, and check the next folder if you get stuck — it's the answer key.

Inside a TODO block, `#:` marks the instructions. Everything else is code, commented out at the indentation it belongs at — select those lines and press `Cmd+/` (`Ctrl+/` on Windows and Linux) to uncomment them where they sit.

The audio plumbing itself lives in [`web/`](https://github.com/deepgram-devs/deepgram-workshop-py/tree/main/web) — a small FastAPI bridge and two AudioWorklets that every step shares and you read rather than write. [`web/README.md`](https://github.com/deepgram-devs/deepgram-workshop-py/blob/main/web/README.md) explains what it does and the three things about it that are easy to get wrong.
