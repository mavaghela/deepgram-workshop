---
title: "Easy Mode: the workshop without the code"
weight: 50
---

Build and tune a voice agent in your browser, with nothing installed. No Python, no terminal, no API key pasted into a file. If you have a browser and a microphone, you can do every conceptual exercise in this workshop and end up with a configuration you can hand to a developer — or take into the code track later.

This track runs on the [Deepgram Playground](https://playground.deepgram.com/voice-agent). It's the same Voice Agent API the Python track talks to. The playground just puts a form in front of it.

**This track is for you if** you came to understand voice agents rather than to write one, you're on a Chromebook or a tablet, your laptop is locked down, or your environment setup went sideways and you'd rather learn something than debug a package manager.

## What you need

- A current Chrome, Edge, or Safari. The playground uses the Web Audio API, which older browsers don't have.
- **Headphones.** Your laptop speakers will feed the agent's own voice back into the microphone, and Lab 2 is where that gets confusing.
- A free Deepgram account. The playground gates the agent behind sign-in — you'll see *"Log in or sign up to talk to the agent"* until you do. Signing up takes a minute at [console.deepgram.com/signup](https://console.deepgram.com/signup) and comes with free credit; no card.
- No microphone? [Lab 1](/easy-mode/lab-1) shows you how to talk to the agent by typing. Everything still works.
- No AWS account needed. Easy Mode never leaves the browser, so the Bedrock half of Step 6 has no equivalent here — read [Step 6 Part 2](/make-it-yours/persona-voice-and-brain) if you want to see what the code track does with it.

## What you're actually building

A voice agent is three models in a row, wired together over a single connection:

![A voice agent is three models in a row: your browser sends your voice over one connection to the Deepgram Voice Agent API, where speech-to-text (flux-general-en) hears you, an LLM thinks using your prompt and functions, and text-to-speech (aura-2-thalia-en) speaks the reply back into the same browser tab. Each model is one field of the Settings JSON document: listen, think, and speak.](/static/easy-mode-pipeline.svg)

Deepgram runs all three and manages the connection between them. Your job — in code or in this playground — is configuration: which models, what the agent is told to do, and which of your functions it's allowed to call.

The one idea worth carrying out of this room: **the playground's settings panel is a form over a JSON document.** Every control you touch writes one field of a `Settings` message the playground sends the moment a conversation opens. The developers next to you are typing that same document by hand. Here it is, trimmed to the fields you'll change today:

:::code{language=json showCopyAction=true showLineNumbers=false}
{
  "agent": {
    "listen":  { "provider": { "type": "deepgram", "model": "flux-general-en" } },
    "think":   { "provider": { "type": "open_ai", "model": "gpt-4o-mini", "temperature": 0.7 },
                 "prompt": "You are a helpful AI assistant...",
                 "functions": [ ]  },
    "speak":   { "provider": { "type": "deepgram", "model": "aura-2-thalia-en" } },
    "greeting": "Hello! What would you like to talk about?"
  }
}
:::

**Transcription model** is `listen`. **LLM** and **Agent prompt** are `think`. **Voice** is `speak`. Once you see the panel as a view of that document, the code track stops looking like a different activity.

## Following along with the room

| Workshop step | Easy Mode equivalent |
|---|---|
| 1 — Setup | [Lab 1 — Get in and say hello](/easy-mode/lab-1) |
| 2 — Connect | Lab 1 (the playground opens the connection for you) |
| 3 — Hear the agent | Lab 1 |
| 4 — Talk to the agent | Lab 1 |
| 5 — Barge-in | [Lab 2 — Interrupt it](/easy-mode/lab-2) |
| 6 — Persona, voice, and your own brain | [Lab 4 — Make it yours](/easy-mode/lab-4) covers the persona half; the Bedrock half has no browser equivalent |
| 7 — Function calling | [Lab 5 — Let it call a function](/easy-mode/lab-5) |
| 8 — Turn detection and latency | [Lab 3 — Tune turn detection](/easy-mode/lab-3) — Easy Mode reaches it earlier |
| Finished | [Lab 6 — Take the config with you](/easy-mode/lab-6) |

You'll finish each lab faster than the code track finishes its step. Spend the spare minutes on the **Going further** prompts — they're where the interesting arguments happen.

## When something goes wrong

:::expand{header='"Log in or sign up to talk to the agent"'}
The agent needs an account. Free to create, no card.
:::

:::expand{header="The settings are grayed out"}
A conversation is open. Click **End Conversation** first.
:::

:::expand{header="It interrupts itself constantly"}
Speakers, not headphones. It's hearing its own voice.
:::

:::expand{header="No microphone prompt appeared"}
Check the site permissions in your browser's address bar; a previous "block" sticks. Failing that, use **Talk for me** and **Send text**.
:::

:::expand{header='"Your free agent trial minutes have been exhausted"'}
Add credit to the account, or share a machine with a neighbor for the rest of the session.
:::

:::expand{header="It cuts you off mid-sentence"}
Working as designed, and Lab 3 is about exactly that trade-off.
:::

:::expand{header="It reads punctuation out loud"}
Your prompt hasn't told it that it's speaking. Back to Lab 4.
:::
