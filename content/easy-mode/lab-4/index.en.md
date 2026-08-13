---
title: "Lab 4: Make it yours"
weight: 54
---

**Goal:** Give the agent a job, a personality, and a voice. This is the lab with the least to click and the most to play with.

End the conversation first, then open the settings panel. Four fields matter:

**Agent prompt** — the standing instructions the model sees with every turn. Personality, job, boundaries.

**Greeting (Optional)** — the first thing the agent says. Leave it blank and the agent waits for the user to speak first, which is the right choice more often than people expect.

**Voice** (under the **Voice** section) — purely how it sounds. It changes nothing about what the agent says.

**LLM** and **LLM Temperature (Optional)** — the brain, and how much it varies. Low temperature for an agent that must say the same thing every time; higher for a chatty one.

## Writing a prompt for speech

Writing prompts for a voice agent differs from writing them for a chatbot in two specific ways, and both of them bite immediately.

**Tell it that it's speaking.** LLMs default to writing. Without an explicit instruction you get markdown, and text-to-speech reads it literally — `**important**` comes out as "star star important star star."

**Tell it to be brief.** A four-sentence answer that scans fine in a chat window feels interminable when you have to sit through it. Voice punishes verbosity in a way text doesn't.

Keep it short for a third reason: every token goes to the model on every turn, so a long prompt slows down the first reply.

Paste this into **Agent prompt** as a starting point, then make it your own:

:::code{language=text showCopyAction=true showLineNumbers=false}
You are Sam, a barista at a small coffee shop. Your job is to take a
drink order: the drink, the size, and the name for the cup.

You are speaking out loud. Never use markdown, bullet points, lists,
or emoji. Keep every reply to one or two sentences.

Ask for one thing at a time. When you have all three, read the order
back and confirm it. If someone asks about anything other than coffee,
say you only work the espresso bar and steer them back to the order.
:::

And into **Greeting (Optional)**:

:::code{language=text showCopyAction=true showLineNumbers=false}
Morning! What can I get started for you?
:::

Start the conversation and order something. Then push at it: ask Sam for the weather, ask it to write you a Python script, ask what it thinks about a movie. A prompt that holds under pressure is the difference between a demo and a product.

**Then change one thing at a time.** Swap the voice and run the same order — same words, different job applicant. Raise the temperature to 1.2 and watch it get chatty and start improvising drinks. Load the **Healthcare** or **Customer support** preset and read its prompt closely; professionals wrote those, and they're worth stealing structure from.

::::alert{type="info" header="Check yourself"}
Name the two prompt instructions that matter for speech but not for chat.

:::expand{header="Answer"}
Tell it that it's speaking (no markdown, bullets, or emoji), and tell it to be brief.
:::
::::

**Going further:** give the agent a constraint it must hold — "you never quote a price" — and spend two minutes trying to talk it out of the constraint. Prompt injection against a voice agent is the same problem as against a chat agent, except your attacker is talking out loud.

::alert[**The LLM dropdown is as far as the browser goes.** The code track's [Step 6](/make-it-yours/persona-voice-and-brain) does something this panel can't: it moves the model out of Deepgram's account and into the attendee's own AWS account on Amazon Bedrock. Picking a model here always means picking one Deepgram brokers and bills you for. Worth knowing that the choice exists when you hand this config to a developer.]{type="info" header="What Easy Mode can't reach"}
