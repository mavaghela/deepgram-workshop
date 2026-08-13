---
title: "Lab 2: Interrupt it"
weight: 52
---

**Goal:** Feel the difference between a demo and something people will actually use.

Ask the agent a question with a long answer — *"explain how a car engine works"* — and then talk over it about two seconds in.

It stops. That's **barge-in**, and it's the single feature that makes a voice agent feel like a conversation instead of a phone tree. The moment Deepgram hears speech that looks like a real turn, it stops generating audio and starts listening.

You get it for free here. The code track spends a whole step on it, because in code you have to throw away the audio you already have queued up locally — the agent stops talking, but your speaker keeps playing the last two seconds you handed it. Ask a neighbor on the code track to show you their [Step 5](/build-the-agent/barge-in) before and after.

**Now break it:** take your headphones off and let the agent talk into your own microphone. It interrupts itself, because it can't tell its voice from yours. This is why the room is full of headphones.

::::alert{type="info" header="Check yourself"}
Why does interrupting need work on the client at all, if Deepgram already stopped sending audio?

:::expand{header="Answer"}
Because of everything Deepgram already sent. That audio left the server before you opened your mouth and is sitting in a playback queue on the client, so it keeps playing until the client throws it away. Deepgram can stop generating; it can't reach into a buffer it already handed over.
:::
::::
