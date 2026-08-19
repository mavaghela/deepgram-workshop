---
title: "Step 6: Persona, voice, and your own brain"
weight: 31
---

**Goal:** Give the agent a job, a personality, and a voice — then move its brain onto Amazon Bedrock in your own AWS account.

**You'll learn**

- The two prompt instructions that separate a voice agent from a chatbot read aloud
- Where rejected settings go when they don't fail the handshake
- Which think providers Deepgram brokers for you, and which ones it doesn't
- What `think.endpoint` is for, and why it's the setting that matters beyond Bedrock
- Where your AWS secret actually travels, and how to scope it accordingly

This step runs in two parts, across two folders. Part 1 gives the agent its character. Part 2 moves the model that generates that character into your account.

---

# Part 1 — Give it a job and a voice

## Start here

:::code{language=bash showCopyAction=true showLineNumbers=false}
uv run steps/06-make-it-yours/main.py
:::

Everything works: it hears you, answers you, and pauses when you talk over it. It's also a generic assistant with a stock voice, and it will stay that way until you change it.

This part has the least code in the workshop and the most to play with.

## How it works

Three settings define the agent's character, and they're independent of each other:

**`prompt`:** the standing instructions the model sees ahead of every user turn. Personality, job, boundaries.

**`speak.provider.model`:** the voice. Purely how it sounds; it changes nothing about what the agent says.

**`greeting`:** the opening line, and the only thing the agent says before it knows anything about the user. It's doing all the work of setting expectations.

The prompt deserves the most attention, because writing prompts for *speech* differs from writing them for chat in two specific ways.

**Tell it that it's speaking.** LLMs default to writing. Without an explicit instruction you get markdown, bullet points, and asterisks read aloud, and it's genuinely jarring the first time you hear a TTS engine pronounce "asterisk asterisk important asterisk asterisk."

**Tell it to be brief.** A four-sentence answer that scans fine in a chat window feels interminable when you have to sit through it. Voice punishes verbosity in a way text doesn't.

Keep the prompt short for a second reason: every turn re-sends every token, and long prompts slow the first reply.

::::alert{type="info" header="Check yourself"}
Name the two prompt instructions that matter for speech but not for chat.

:::expand{header="Answer"}
Tell it that it's speaking (no markdown, bullets, or emoji), and tell it to be brief.
:::
::::

## Do this

**TODO 6.1: Give your agent a job.** Rewrite the prompt. Cover both rules above, then make it something specific: a barista taking an order, support for a product you know well, a dungeon master, a museum guide.

**TODO 6.2: Pick a voice.** Swap `flux-alexis-en` for another Flux voice. The full list lives in the [Deepgram TTS models documentation](https://developers.deepgram.com/docs/tts-models).

**TODO 6.2b: Write a new opening line.** Match it to the job you just assigned.

**TODO 6.3: Try a different brain.** `gpt-4o-mini` is fast and cheap, which matters more than raw capability when someone is waiting to hear a reply. Switch to `gpt-4o` and watch the latency readout on the right of the browser's activity line. You're paying for that capability in a currency your users feel directly. Step 8 is where that number becomes the whole point.

`temperature` controls variability: `0.0` for an agent that must say the same thing every time, `1.0` and up for a chatty one. Other providers work here too (Anthropic, Google, Groq, AWS Bedrock) via the matching `ThinkSettingsV1Provider_*` class. Note that you're switching models without an OpenAI account: Deepgram brokers that call. It doesn't broker all of them, and Part 2 of this step is where that distinction starts to matter.

**TODO 6.4: Surface warnings.** Add a `Warning` branch mirroring the `Error` branch above it.

Warnings are where rejected settings go. A misspelled voice or an out-of-range threshold arrives here rather than failing the handshake, so without this branch the agent silently ignores a bad setting and you're left wondering why nothing changed. It also makes Step 8's thresholds debuggable.

## Verify

Your agent greets you in character, in a different voice, and stays in role when you push at it. A deliberately misspelled voice model prints something like:

:::code{language=text showCopyAction=false showLineNumbers=false}
>> Agent warning: INVALID_VOICE - Unknown speak model 'flux-alexei-en'
:::

Ask it something that would normally get a bulleted list ("what are the main types of coffee drinks?") and confirm you hear prose, not punctuation.

::alert[Good moment to go around the room. Personas are the most fun part of the workshop and hearing four different agents makes the prompt lesson land harder than any explanation.]{type="info" header="Pause: check in with the instructor"}

## Troubleshooting

:::expand{header="It still reads markdown aloud"}
The instruction has to be explicit. "Never use markdown, bullet points, or emoji" works; "be conversational" doesn't.
:::

:::expand{header="It ignores the persona after a few turns"}
Move the important constraints to the *end* of your prompt, and cut anything that isn't load-bearing.
:::

:::expand{header="Answers got long again"}
Larger models are more verbose by default. Re-state the brevity instruction, or go back to `gpt-4o-mini`.
:::

:::expand{header="Voice didn't change and there's no warning"}
Confirm you edited `speak`, not `listen`, and that TODO 6.4 is done.
:::

:::expand{header="Noticeably slower after switching models"}
Working as intended. Check the browser's latency readout and decide whether the quality is worth it.
:::

---

# Part 2 — Move the brain to Amazon Bedrock

::alert[Part 2 has its own folder: `steps/06b-bring-your-own-llm/main.py`. It is Part 1 finished, reset to a neutral prompt and voice so everyone starts from the same place. If you want to keep the persona you just wrote, copy your prompt, voice, and greeting across.]{type="info" header="A new folder for this part"}

## Start here

:::code{language=bash showCopyAction=true showLineNumbers=false}
uv run steps/06b-bring-your-own-llm/main.py
:::

It prints which brain it's using before it opens the browser:

:::code{language=text showCopyAction=false showLineNumbers=false}
>> Thinking with: OpenAI (no AWS credentials in .env)
:::

That's the agent Part 1 started from, and it will keep working exactly like that until you put AWS credentials in `.env`. `main.py` here falls back to OpenAI when there are none, so it runs regardless — if your model access hasn't come through yet, keep going, and Step 7 continues either way.

## How it works

Every step so far has needed one credential. `.env` has held a single Deepgram key, and that key has paid for speech-to-text, the LLM, and text-to-speech alike. Part 1 let you swap `gpt-4o-mini` for `gpt-4o` without so much as an OpenAI account.

That convenience comes from a split in the provider list:

**Brokered by Deepgram:** OpenAI, Anthropic, Google, NVIDIA. Deepgram holds the account, makes the call, and bills you. You name a model and you're done.

**Bring your own:** Groq and AWS Bedrock. Deepgram makes the call *as you*, with credentials you hand it, against an endpoint you name. Your account, your model access, your bill.

The second group is the interesting one, because it's the answer to a question that comes up the moment a voice agent stops being a demo: *can the model run somewhere I control?* A regulated industry, a model you've fine-tuned, a negotiated rate you'd rather keep. All of it lands here.

Two settings carry it, and Bedrock needs **both**:

**`think.provider`:** which model, and the credentials to reach it. `ThinkSettingsV1Provider_AwsBedrock` takes either long-lived `iam` keys or short-lived `sts` ones, which additionally carry a `session_token`.

**`think.endpoint`:** the URL Deepgram sends the completion request to. For Bedrock that's `https://bedrock-runtime.{region}.amazonaws.com/`, and the Region has to match the one in your credentials.

`endpoint` is the setting worth remembering after today. It isn't Bedrock-specific: point it at anything that speaks the OpenAI Chat Completions format (a self-hosted model, a gateway in front of your own inference, a router) and the agent talks to it. Bedrock is just the case with enough structure that Deepgram gave it a provider type of its own.

::alert[**Your AWS access key and secret go into the `Settings` message, over the WebSocket, to Deepgram.** That is what "Deepgram makes the call as you" means. So don't hand it your root credentials or an admin user. Create an IAM user scoped to `bedrock:InvokeModelWithResponseStream` on the one model ARN you're using, or issue short-lived STS credentials. The blast radius of a workshop credential should be one model in one Region.]{type="warning" header="Where the credential travels"}

::::alert{type="info" header="Check yourself"}
Part 1 let you switch to `gpt-4o` with nothing but a model name. Why does Bedrock need two settings and a second credential?

:::expand{header="Answer"}
Deepgram brokers OpenAI, so a model name is the whole configuration. It doesn't broker Bedrock: it makes the call *as you*, which needs credentials to make it with (`provider.credentials`) and an address to make it to (`think.endpoint`).
:::
::::

## Do this

First, get the credentials into `.env`. `.env.example` has the block, commented and at the bottom. Copy it across and fill it in.

:::::tabs

::::tab{id="ws-account" label="Workshop Studio account"}
Open the credentials panel for your Workshop Studio event and copy the three values it shows you:

:::code{language=bash showCopyAction=true showLineNumbers=false}
AWS_REGION=us-east-2
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_SESSION_TOKEN=...
:::

Set `AWS_REGION` to the Region your event was provisioned in. These are temporary credentials, so `AWS_SESSION_TOKEN` is set — and its presence is exactly what switches the credentials type from `iam` to `sts` in the code you're about to write.

Bedrock model access is already granted in your event's account, so there is nothing to request.

The credentials expire when the event does. If the agent starts failing after a long break, re-copy them from the panel.
::::

::::tab{id="own-account" label="Your own AWS account"}
Long-lived IAM keys, with no session token:

:::code{language=bash showCopyAction=true showLineNumbers=false}
AWS_REGION=us-east-2
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
:::

Then enable model access for `zai.glm-4.7-flash` in that Region, at [Bedrock → Model access](https://console.aws.amazon.com/bedrock/home#/modelaccess). Access is per model and per Region, so enabling it in `us-east-1` does nothing for an agent pointed at `us-east-2`. Some model families also want a one-time use case form, which is not instant.

The IAM user needs `bedrock:InvokeModelWithResponseStream` on that model's ARN. `bedrock:InvokeModel` alone is not enough, because the agent streams.
::::

:::::

::alert[**If you already did the Pipecat edition of this workshop, note what does *not* carry over.** `AWS_BEARER_TOKEN_BEDROCK` is a botocore convenience, and there's no botocore here. Deepgram takes an access key and secret, or STS credentials, and nothing else.]{type="info"}

Now the code. All three TODOs sit where their code goes, and each carries its own guidance, so you shouldn't need to scroll between an instruction and the line it's about.

**TODO 6b.1: Uncomment the imports.** Three of them, already sited at their alphabetical places in the import block (`6b.1a` through `6b.1c`).

**TODO 6b.2: Guard, then build the credentials.** In `think_settings()`, above the fallback `return`. Guard on `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, then build the credentials as a plain dict.

Keep the guard. It's what lets the person next to you, whose credentials expired or whose model access never got approved, run your file.

And build the credentials as a dict rather than passing keyword arguments, so `session_token` can be *absent* for long-lived IAM keys. The SDK serializes any field you pass explicitly, so `session_token=None` would put a literal `null` on the wire next to `type="iam"`.

**TODO 6b.3: Return the Bedrock settings.** Still inside the guard: `provider`, `endpoint`, and `prompt`. Bedrock needs the first two together (miss either and the handshake fails), and the Region appears in both.

Note what `model` is doing here: Bedrock model IDs pass through to AWS untouched. The SDK's type hints list two Claude 3.5 IDs and then accept any string, so nothing catches a typo locally. A wrong model ID comes back from AWS at handshake time, not from your editor.

Once Bedrock answers, set `AWS_BEDROCK_MODEL` in `.env` to another model you enabled and listen for what a different brain does to latency and voice.

## Verify

:::code{language=text showCopyAction=false showLineNumbers=false}
>> Thinking with: AWS Bedrock
>> Settings applied
:::

Then hold a conversation. It should be indistinguishable from the agent you had a moment ago, because it is: same prompt, same voice, different account paying for the tokens.

Now check the bill you just moved: the request shows up in **AWS → Bedrock → Usage**, and nowhere in your Deepgram console. Speech-to-text and text-to-speech are still Deepgram's; only the middle of the pipeline changed hands.

::alert[Worth asking out loud how many people got a Bedrock reply on the first try. That number is the whole story of whether bring-your-own is a realistic default for a team.]{type="info" header="Pause: check in with the instructor"}

## Troubleshooting

:::expand{header=">> Agent error: ... and the agent never speaks"}
Unlike a misspelled voice, a refused brain is fatal. There's nothing to fall back to, so it errors rather than warns. Read the description; it usually names which of the failures below went wrong.
:::

:::expand{header="Model access is not enabled"}
The most common failure by a wide margin for attendees using their own account, and the slowest to fix. Enable the exact model ID, in the exact Region, at [Bedrock → Model access](https://console.aws.amazon.com/bedrock/home#/modelaccess).
:::

:::expand{header="Region mismatch"}
`AWS_REGION` feeds both the credentials and the endpoint URL, so they can't disagree in the shipped code. If you hardcoded either one while experimenting, that's the first thing to check. On a Workshop Studio account, make sure `AWS_REGION` matches the Region your event was provisioned in.
:::

:::expand{header="Credentials rejected"}
Confirm the IAM principal can actually invoke Bedrock. `bedrock:InvokeModelWithResponseStream` is the permission the agent needs; `bedrock:InvokeModel` alone is not enough, because the agent streams. On a Workshop Studio account, temporary credentials expire — re-copy them from the event panel.
:::

:::expand{header="Thinking with: OpenAI (AWS credentials are in .env, but think_settings() is not using them yet)"}
Your keys are fine; the code isn't wired up. That line reports what `think_settings()` actually returned rather than what's in your environment, so it says this until TODO 6b.2 and 6b.3 are both done.
:::

:::expand{header='It says "no AWS credentials in .env" and you are sure the keys are set'}
`.env` is read by `load_dotenv()` at import. A key set only in your shell for a previous command won't be there. Check for a stray space after the `=`.
:::

There is no next folder to check this part against, so here is the finished `think_settings()`:

:::code{language=python showCopyAction=true showLineNumbers=true}
def think_settings() -> ThinkSettingsV1:
    """Build the think settings, on Bedrock when AWS credentials are present.

    Returns:
        The think settings the agent is configured with.
    """
    if AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY:
        credentials = {
            "type": "sts" if AWS_SESSION_TOKEN else "iam",
            "region": AWS_REGION,
            "access_key_id": AWS_ACCESS_KEY_ID,
            "secret_access_key": AWS_SECRET_ACCESS_KEY,
        }
        if AWS_SESSION_TOKEN:
            credentials["session_token"] = AWS_SESSION_TOKEN

        return ThinkSettingsV1(
            provider=ThinkSettingsV1Provider_AwsBedrock(
                type="aws_bedrock",
                model=BEDROCK_MODEL,
                temperature=0.7,
                credentials=AwsBedrockThinkProviderCredentials(**credentials),
            ),
            endpoint=ThinkSettingsV1Endpoint(
                url=f"https://bedrock-runtime.{AWS_REGION}.amazonaws.com/",
            ),
            prompt=PROMPT,
        )

    return ThinkSettingsV1(
        provider=ThinkSettingsV1Provider_OpenAi(
            type="open_ai",
            model="gpt-4o-mini",
            temperature=0.7,
        ),
        prompt=PROMPT,
    )
:::

## Going further

Try the other half of the idea. Leave `provider` set to `open_ai` and point `endpoint` at an OpenAI-compatible gateway of your own: a local server, a proxy that logs every completion, a router across several models. That's `think.endpoint` doing the thing it's actually for, and it's how you'd put a Bedrock Agent, request logging, or a header rewrite in front of the model.

Back on the prompt side, give the agent a constraint it has to hold under pressure ("you only discuss coffee, and you politely redirect anything else"), then spend two minutes trying to talk it off-topic. Prompt injection resistance in a voice agent is the same problem as in a chat agent, except your attacker is talking out loud.

::alert[Part 2 moved the brain into your AWS account. The speech half can move too: Flux, Nova-3, and Aura-2 are on AWS Marketplace and [deploy to Amazon SageMaker endpoints](https://developers.deepgram.com/docs/deploy-amazon-sagemaker) in your own VPC. That's a different architecture rather than a different setting, though — SageMaker's network isolation blocks outbound LLM calls, which is exactly why the Voice Agent API can't run there. You'd call transcription, your model, and synthesis yourself instead of configuring one socket. Nothing to do here; just know the option exists when the audio is the part that can't leave.]{type="info" header="The speech models can run in your account too"}

---

Your agent has a job, a voice, and a brain running where you want it. What it still can't do is anything outside its own head.

::alert[**Step 7 continues from Part 1, not Part 2.** `steps/07-function-calling/main.py` resets to the neutral prompt, the neutral voice, and the brokered OpenAI provider, so everyone starts the next step from the same place regardless of what they built here. Carry your persona and your Bedrock `think_settings()` forward if you'd rather — it's a copy-paste of one function.]{type="info"}
