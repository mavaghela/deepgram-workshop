---
title: "Prerequisites"
weight: 11
---

**Goal:** Have everything installed and every credential in hand before Step 1.

## What you need

| You need | Why you need it |
|---|---|
| [uv](https://docs.astral.sh/uv/getting-started/installation/) | The only thing to install. It fetches Python 3.13 and every dependency itself, so you don't need Python already. |
| A free [Deepgram API key](https://console.deepgram.com/signup?jump=keys) | Signup takes a minute, and the free credit covers this workshop with room to spare. |
| An AWS account with Bedrock access | Step 6 runs the agent's LLM on Amazon Bedrock. See [AWS access](#aws-access) below. |
| A current browser | Chrome, Firefox, or Safari. The browser is the microphone and the speaker — it needs `getUserMedia` and `AudioWorklet`. |
| Wired headphones | Your browser cancels most of the echo from your speakers, but not all of it on every browser, and Step 5 is where the difference shows. |
| A terminal | macOS, Linux, Windows, and WSL all work — under WSL the audio is Windows' problem, not yours. |

Being comfortable with Python helps. An audio or machine-learning background doesn't.

::alert[If you aren't a developer, or you're on a locked-down laptop, Chromebook, or tablet, the [Easy Mode](/easy-mode) track runs this same workshop entirely in the browser. No install, no terminal, and its labs map one to one onto the steps.]{type="info" header="A different path through the same material"}

## Get the code

The workshop's Python lives in one repository. Clone it wherever you keep projects:

:::code{language=bash showCopyAction=true showLineNumbers=false}
git clone https://github.com/deepgram-devs/deepgram-workshop-py.git
cd deepgram-workshop-py
:::

## Set up your environment

Pick one of these three. They end in the same place: dependencies installed and a `.env` waiting for your key.

::::tabs

:::tab{id="local" label="On your machine"}
Install [uv](https://docs.astral.sh/uv/getting-started/installation/), then from the repository root:

::code[uv sync]{showCopyAction=true language=bash}

One virtual environment serves every step in this workshop. You run this once.

Then create your `.env` file:

::code[cp .env.example .env]{showCopyAction=true language=bash}

Open `.env` and paste in your Deepgram API key.
:::

:::tab{id="codespaces" label="GitHub Codespaces"}
Nothing local at all. From the [repository page](https://github.com/deepgram-devs/deepgram-workshop-py), choose **Code → Codespaces → Create codespace on main**.

The container builds, `uv sync` runs, and `post-create.sh` seeds `.env` from `.env.example`. Paste your key into `.env` and start at Step 1.

You can skip the paste entirely: set `DEEPGRAM_API_KEY` as a [Codespaces secret](https://github.com/settings/codespaces) and it arrives as an environment variable. Every step calls `load_dotenv()`, which does not overwrite a variable that is already set, so the secret wins and `.env` can stay empty.

The page opens at `https://….app.github.dev`, which is HTTPS and therefore a secure context, so the microphone works.
:::

:::tab{id="devcontainer" label="VS Code Dev Containers"}
Needs only Docker. Install the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers), open the cloned folder, and accept **Reopen in Container**.

The container builds, `uv sync` runs, and `post-create.sh` seeds `.env` from `.env.example`. Paste your key into `.env` and start at Step 1.

Port 8000 is forwarded, so the page opens at `http://127.0.0.1:8000` on your host — a secure context, so the microphone works.
:::

::::

::alert[**The microphone always lives outside the container.** A container has no audio hardware and does not need any — the browser is the microphone and the speaker, and the browser is on your machine. The one thing a container takes away is the `--local` flag described below. If the page opens twice in a container, the auto-forward and the step both got there; add `--no-open` and let the forward do it.]{type="info" header="Containers and audio"}

## Your Deepgram key

That key is the only Deepgram credential you need. The agent you build uses OpenAI's `gpt-4o-mini` as its brain until Step 6, but Deepgram brokers that call, so no OpenAI account, no second key, no second bill.

Deepgram applies **$200 in credit** to new accounts automatically. This workshop costs well under a dollar, so you'll have plenty left to keep building afterward.

## AWS access

Step 6 moves the agent's LLM onto Amazon Bedrock in an AWS account. How you get there depends on how you're taking this workshop.

::::tabs

:::tab{id="ws-account" label="Workshop Studio account"}
Your event provisions an AWS account for you, with Bedrock model access already granted in the event's Region. Nothing to request and nothing to wait for.

You'll open the Workshop Studio credentials panel in Step 6 and copy three values into `.env`: the access key ID, the secret access key, and the session token. They're temporary credentials, which is exactly what the step expects.
:::

:::tab{id="own-account" label="Your own AWS account"}
You need two things, and one of them takes calendar time:

1. **Bedrock model access for `zai.glm-4.7-flash`**, granted at [Bedrock → Model access](https://console.aws.amazon.com/bedrock/home#/modelaccess) in the Region you plan to use.
1. **An IAM user or STS session** with `bedrock:InvokeModelWithResponseStream` on that model's ARN, and an access key pair you can paste into `.env`.

`bedrock:InvokeModel` alone is not enough — the agent streams.
:::

::::

::alert[**Bedrock grants model access per model and per Region**, and some model families additionally require a one-time use case form that is not instant. Enabling a model in `us-east-1` does nothing for an agent pointed at `us-east-2`. If you're bringing your own account, request access days ahead — a week if you're running this for a room.]{type="warning" header="Request Bedrock access before the day"}

## Verify it works

Once you have uv and a key:

:::code{language=bash showCopyAction=true showLineNumbers=false}
uv run steps/01-setup/main.py
:::

That command checks your key in the terminal, then opens a browser page that checks the rest: microphone permission, the audio APIs, input level, and output. Better here than mid-conversation later. [Step 1](/build-the-agent/setup) walks through it in full.

::alert[Every step from Step 2 on serves a page at `http://127.0.0.1:8000` and opens it for you. **It has to be `127.0.0.1`** — browsers only grant microphone access on a secure context, and a LAN address is not one.]{type="warning"}

## Running in another Deepgram region

Deepgram serves the same APIs from more than one place. Global is the default; the EU and AU endpoints process audio inside those geographies. Pick one in `.env`:

:::code{language=bash showCopyAction=true showLineNumbers=false}
DEEPGRAM_REGION=eu        # global (default), eu, or au
:::

That is the whole change. Your key works in every region, no step's code names one, and every step reads the same `.env` — so one line moves Steps 1 through 8 together.

Step 1 then starts the agent against whichever endpoint you picked and reports whether that endpoint serves all three models. Model availability differs by region and moves over time, so that — not the key, and not whether the host answers — is the question worth asking before a room of people hits Step 2.

[`web/region.py`](https://github.com/deepgram-devs/deepgram-workshop-py/blob/main/web/region.py) has the details, and the same mechanism points the workshop at a Deepgram Dedicated or self-hosted deployment.

## Prefer the terminal?

Add `--local` to any step and it uses your system microphone and speaker through PortAudio instead, with no browser involved:

:::code{language=bash showCopyAction=true showLineNumbers=false}
uv run steps/99-final/main.py --local
:::

Same agent code either way — see [`web/README.md`](https://github.com/deepgram-devs/deepgram-workshop-py/blob/main/web/README.md) for what changes underneath. This is the one flag a container takes away.
