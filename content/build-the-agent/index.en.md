---
title: "Module 1: Build the agent"
weight: 20
---

Five steps, and at the end of them you have a voice agent you can hold a conversation with and interrupt mid-sentence.

| Step | You build | Run it | Time |
|---|---|---|---|
| [1 — Setup](/build-the-agent/setup) | Verify key, browser, and microphone | `uv run steps/01-setup/main.py` | 15 min |
| [2 — Connect](/build-the-agent/connect) | The WebSocket and settings handshake | `uv run steps/02-connect/main.py` | 20 min |
| [3 — Hear the agent](/build-the-agent/hear-the-agent) | Playback — the greeting plays | `uv run steps/03-hear-the-agent/main.py` | 25 min |
| [4 — Talk to the agent](/build-the-agent/talk-to-the-agent) | Microphone input — a real conversation | `uv run steps/04-talk-to-the-agent/main.py` | 30 min |
| [5 — Barge-in](/build-the-agent/barge-in) | Interrupting the agent mid-sentence | `uv run steps/05-barge-in/main.py` | 25 min |

This module is the core of the workshop. Finish it and you have something that works; everything after it is character, capability, and feel.

Each folder holds the finished state of the step before it, so if you fall behind you can skip ahead and keep going. You lose the typing, not the workshop.
