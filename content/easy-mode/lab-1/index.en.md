---
title: "Lab 1: Get in and say hello"
weight: 51
---

**Goal:** A real conversation with an agent, in about three minutes.

1. Open [playground.deepgram.com/voice-agent](https://playground.deepgram.com/voice-agent) and log in.
1. Under **Try a use case:** pick **General**. The other presets — Healthcare, Customer support, Sales, Financial services — load a prompt and settings tuned for that scenario, and you'll come back to them in Lab 4.
1. Click **Talk To Your Agent**. The browser asks for microphone permission; allow it.
1. Talk. Ask it what it does, then interrupt yourself, then trail off mid-sentence and see what it does with the silence.
1. Click **End Conversation** when you're done.

Watch the message list while you talk. Every turn appears there, and **Expand all** opens the raw events underneath — this is the same stream of messages the Python code is reacting to. Turn on **Show client audio** to see your own audio going out alongside the agent's coming back.

::alert[**While a conversation is open, the playground locks the settings.** The playground tells you so — *"While the connection is open, you cannot update the agent in the Playground."* Click **End Conversation**, change your setting, then start again. That's a playground rule, not an API rule; over the API you can update a live agent mid-conversation.]{type="warning" header="This will trip you up"}

::::alert{type="info" header="Check yourself"}
Three models run behind that conversation. Which one decided *when you were finished speaking*?

:::expand{header="Answer"}
Speech-to-text. Flux does turn detection inside the model, so the same model that transcribes you also decides when your turn ended. The LLM never sees your audio, and it only gets your words once Flux has called the turn over.
:::
::::

## Talking without a microphone

Under the message list, the **Talk for me** box takes typed input and sends it to the agent as if you'd said it — click **Send text** or press Ctrl+Enter. The agent still replies out loud. Use this on a tablet with no mic access, in a room too loud to talk in, or any time you want to send the exact same sentence twice to compare two configurations.
