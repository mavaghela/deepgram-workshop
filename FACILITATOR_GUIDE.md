# Facilitator Guide: Deepgram Voice Agent Workshop

Everything you need to run this workshop in a room. Attendees don't need this file.

---

## Workshop Overview

**What Participants Will Learn:** The Deepgram Voice Agent Workshop teaches participants how to build a real-time voice agent in Python, one runnable step at a time. By the end, they'll have an agent that listens on Flux, thinks with a model running in their own AWS account on Amazon Bedrock, answers in a natural voice, stops when interrupted, and calls their Python when it needs something it can't know.

**Learning Objectives:**

- Configure speech-to-text, an LLM, and text-to-speech through a single Deepgram WebSocket
- Explain why Flux performs turn detection server-side, and what that removes from client code
- Stream microphone audio without blocking the thread that plays the reply
- Clear queued playback the instant a user interrupts — the difference between a demo and something people will use
- Move the agent's LLM onto Amazon Bedrock in their own AWS account, with a correctly scoped credential
- Wire client-side function calls into a live conversation
- Trade turn-detection latency against accuracy, and measure it honestly

**Target Audience:** Application developers, solutions architects, and technical leads evaluating real-time voice for a product. Secondarily, sales engineers and developer advocates who need a demo they can explain line by line.

**Assumed skill:** comfortable reading and editing Python. That's the bar. No audio engineering, DSP, speech recognition, or machine learning background is assumed. No AWS knowledge is needed until Step 6, which assumes participants can find the Bedrock console and read a set of credentials.

### At a glance

| | |
|---|---|
| **Format** | Hands-on, attendees code on their own machines |
| **Full run** | ~3 hours (Steps 0 to 8) |
| **Short run** | ~90 minutes (Steps 0 to 5), and it still ends with a working voice agent |
| **Staffing** | One instructor, plus one floating helper per ~15 attendees |
| **Cost per attendee** | Well under $1 in Deepgram usage, plus cents of Bedrock tokens. New Deepgram accounts get $200 in credit |
| **Infrastructure** | None. The workshop provisions no AWS resources |

The single biggest predictor of whether this workshop goes well is how many people arrive with a working environment. Send the pre-event email.

---

## Prerequisites

### Facilitator Preparation

- Walk through the full workshop yourself (budget 3 hours) to understand the flow and catch issues early. Run Step 6 Part 2 against a real Bedrock account so you've seen the credential paths yourself.
- Test by launching a test event in Workshop Studio and confirm the provisioned account has Bedrock model access for `zai.glm-4.7-flash` in the event Region.
- Grab the projection deck — see **Resources** below.
- Read the Troubleshooting section so you're ready for common questions.

### Service Requirements & Quotas

- **AWS services used:** Amazon Bedrock only, and only in Step 6. Nothing else is called and nothing is provisioned.
- **Model access:** `zai.glm-4.7-flash` must be enabled in the event Region. Bedrock grants model access **per model and per Region**, and some model families additionally require a one-time use case form that is not instant.
- **Large events:** For 200+ participants, confirm Bedrock on-demand throughput for the chosen model in the event Region ahead of time. Each attendee holds one streaming conversation at a time, so concurrency tracks attendee count directly.
- **Account setup:** Attendees need either a Workshop Studio-provisioned account or their own. Either way they need credentials they can paste into a `.env` file — an access key pair, or temporary STS credentials with a session token.
- **Non-AWS dependency:** every attendee needs a free Deepgram API key. That is the credential Steps 1 to 5 run on.

### Participant Prerequisites

- Comfortable reading and editing Python. No audio or ML background needed.
- [uv](https://docs.astral.sh/uv/getting-started/installation/) installed — the only local dependency. It fetches Python 3.13 itself.
- A free [Deepgram API key](https://console.deepgram.com/signup?jump=keys).
- A current browser: Chrome, Firefox, or Safari. The browser is the microphone and the speaker, so it needs `getUserMedia` and `AudioWorklet`.
- **Wired headphones.** Browsers cancel most of the echo from laptop speakers, but not all of it on every browser, and Step 5 is where the difference shows.
- A terminal. macOS, Linux, Windows, and WSL all work — under WSL the audio is Windows' problem, not yours.

### Send this a week ahead

> **Before the workshop: 10 minutes of setup**
>
> Please do this before you arrive so we can start with the interesting part.
>
> 1. Install [uv](https://docs.astral.sh/uv/getting-started/installation/)
> 2. Sign up at [console.deepgram.com](https://console.deepgram.com/signup?jump=keys) and create an API key (new accounts get $200 free credit)
> 3. Clone the repo and run the setup check:
>    ```bash
>    git clone https://github.com/deepgram-devs/deepgram-workshop-py.git
>    cd deepgram-workshop-py
>    uv sync
>    cp .env.example .env    # paste your key in
>    uv run steps/01-setup/main.py
>    ```
> 4. **Bring wired headphones.** Your browser cancels most of the echo, but not all of it on every browser. Without headphones the agent can hear itself and interrupt itself, which makes Step 5 confusing.
>
> The setup check finishes in a browser page. Chrome, Firefox, or Safari, reasonably current.
>
> If the check prints anything other than all-OK, reply to this email and we'll sort it out before the day.

**If attendees are bringing their own AWS accounts**, add a fifth item — and send that version *two weeks* ahead, not one:

> 5. Enable Bedrock model access for `zai.glm-4.7-flash` in one Region at [Bedrock → Model access](https://console.aws.amazon.com/bedrock/home#/modelaccess), and have an IAM access key ready with `bedrock:InvokeModelWithResponseStream` on that model. Access is granted per model *and* per Region, and approval isn't always instant.

If Workshop Studio is provisioning accounts, leave that item out — model access is already granted and attendees copy temporary credentials from the event panel in Step 6.

### Running it in another Deepgram region

Deepgram's global endpoint is the default and is what the pre-event email assumes. If the room needs audio processed inside the EU or Australia, set one line in `.env` and hand out the repository as usual:

```bash
DEEPGRAM_REGION=eu        # global (default), eu, or au
```

Nothing in `steps/` names a region. Every step reads the same `.env`, so this moves all of them at once.

You don't have to test the region yourself. Step 1 opens the same WebSocket Step 2 opens, with the same three models, and reports whether the server accepted them:

```
[  OK  ] Region: eu (wss://api.eu.deepgram.com/v1/agent/converse)
[  OK  ] Agent started (flux-general-en + gpt-4o-mini + flux-alexis-en)
```

That is the check that matters regionally. Model availability is per-region and moves over time. A key that works and an endpoint that answers still don't tell you a model is served there. If one isn't, the agent names it in an `Error` frame and Step 1 prints it, along with the two ways out: `DEEPGRAM_REGION=global`, or swap the model the error names in every step.

:::alert{type="warning"}
**As of August 2026, `au` needs one change.** Flux TTS is not served there yet, so `flux-alexis-en` is refused. Flux STT and the LLM are fine. Set `speak` to `aura-2-thalia-en` in every step's `SETTINGS` and the workshop runs unchanged; Step 6 teaches changing the voice anyway, so it costs the room nothing. `eu` and `global` both run the workshop as shipped. Step 1 tells each attendee this directly, so you'll find out even if this note goes stale.
:::

The Deepgram region and the AWS Region are independent choices. Pair an EU speech endpoint with an EU Bedrock Region when that's what the audit requires.

---

## Recommended Agenda

### Standard 3-Hour Format

Times assume a 3-hour slot with one break. The **⏸** rows are the sync points marked on each page. Hold the room at these.

| Time | Duration | Activity | Key Focus & Tips |
|------|----------|----------|------------------|
| 0:00–0:10 | 10 min | **Step 0: Voice agent concepts** | You present. Three models, what the API orchestrates, what Flux changes. **⏸** Keep it to 10 min; they came to code |
| 0:10–0:25 | 15 min | **Step 1: Setup** | Everyone runs the checker. **⏸** Nobody proceeds without a green key line and a green `Agent started` |
| 0:25–0:45 | 20 min | **Step 2: Connect** | Handshake. **⏸** Everyone sees `>> Settings applied` |
| 0:45–1:10 | 25 min | **Step 3: Hear the agent** | Speaker output. **⏸** Everyone *hears* the greeting |
| 1:10–1:40 | 30 min | **Step 4: Talk to the agent** | Microphone, full loop. **⏸** Everyone holds a conversation. High point, so let it breathe |
| 1:40–1:50 | 10 min | **Break** | Use it to unstick stragglers |
| 1:50–2:15 | 25 min | **Step 5: Barge-in** | Interruption. **⏸** Everyone experiences the bug *before* fixing it |
| 2:15–2:50 | 35 min | **Step 6: Persona, voice, and your own brain** | Part 1 persona and voice (~20 min), Part 2 Bedrock (~15 min). **⏸** Go around the room and demo a few personas before starting Part 2 |
| 2:50–3:20 | 30 min | **Step 7: Function calling** | Phone banking agent. **⏸** Sketch a function for their own use case first |
| 3:20–3:35 | 15 min | **Step 8: Turn detection and latency** | Pace-recovery step. Stragglers catch up here, and it sheds time cleanly |
| 3:35–3:45 | 10 min | **Wrap-up & Next Steps** | Summary page, Q&A. Point at Discord and the docs |

**Pacing tips:**

- Steps 1 to 5 are the core. Finish those and the room has a working voice agent whatever else happens.
- **Step 6 is now two parts and it will run long the first time you deliver it.** Part 1 is fun and people linger on personas; Part 2 is where credentials go wrong. Budget the full 35 minutes and be willing to call time on personas.
- **If you're running long.** Cut Step 8 and Step 7's exercises. Both work fine as take-home, and Step 8 is deliberately last so it can go. Never cut Step 5; it's the step people remember.
- **If you're running short.** Step 8's "Going further" (`eager_eot_threshold`) and Step 7's `transfer_funds` exercise absorb time well. For a room that is genuinely ahead, the optional Step 7b (healthcare) is the richer fifteen minutes — it needs no extra credential and nothing after it depends on it, so it also makes the best take-home.
- With 20+ participants, keep per-step Q&A short and use your floating helpers.

### 90-Minute Format

Run Step 0 through Step 5 and stop. It ends with a working voice agent you can interrupt mid-sentence, which is a satisfying place to finish. Mention Steps 6 to 8 as take-home and point at the workshop URL.

---

## Delivery Tips

### Setup & Environment

- **The pre-event email is the highest-leverage thing you do.** Everything else on this list matters less than how many people arrive with a green Step 1.
- If Workshop Studio is provisioning accounts, open the credentials panel yourself before the session and confirm the values are where you'll tell people to look.
- Have one fully-working environment on your own machine to demo from when someone's setup is unrecoverable.
- Test screen sharing and audio before participants join. You will be playing agent audio through the room.

**Regional notes:**

- Good AWS Regions: `us-east-2` and `us-west-2` for broad Bedrock model availability.
- Deepgram region: `global` unless there's a data residency requirement. `eu` runs the workshop as shipped; `au` needs the TTS model swap noted above.
- Cost per participant: well under $1 of Deepgram usage plus cents of Bedrock tokens. No infrastructure to clean up.

### Facilitation Strategies

**What works well:**

- Explain the "why" before the "how". Step 0 exists for exactly this and it's worth the ten minutes.
- Say out loud at the first sync point that skipping ahead is allowed. Every step folder is complete, so someone stuck on Step 3 at the 1:40 mark should jump to `steps/05-barge-in/` and keep going. They lose the typing, not the workshop.
- At Step 5, make everyone experience the bug before writing the fix. The bug is far more memorable than the patch.
- At Step 6 Part 1, go around the room and demo four or five personas. Hearing them makes the prompt lesson land harder than any explanation.
- At Step 7, have people sketch a function for their own use case before writing code. It's the most useful two minutes in the workshop.

**Common mistakes:**

- Don't skip the architecture overview. Participants need that context before diving in.
- Don't assume everyone is on the same step. Check progress at every sync point.
- Don't debug live when the room is moving. The answer key for any step is the next folder's `main.py` — point people there.

**Mixed skill levels:**

- Beginners: pair them with experienced participants, and remind them the next folder is the answer key.
- Advanced: point them at the "Going further" section on each page. They're substantial.
- Non-developers, locked-down laptops, Chromebooks, tablets: send them to the **Easy Mode** chapter. It runs the same material in the Deepgram Playground with nothing installed, and its labs map one to one onto the steps so a mixed room stays in sync.

### Managing pace spread

The step folders exist for exactly this problem. Someone stuck on Step 3 at the 1:40 mark should **skip to `steps/05-barge-in/`** and keep going. That folder already contains a complete working Steps 1 to 4.

Say this out loud at the first sync point so people know it's allowed. Otherwise they'll quietly fall behind and disengage.

---

## Troubleshooting

### Top Issues

#### 1. Microphone permission

:::alert{type="warning"}
The prompt appears during Step 1 and people click past it. This is the most common issue in the workshop.
:::

**Cause:** The browser permission prompt was dismissed, or the OS is blocking the browser.

**Fix:**

- Click the icon at the left of the address bar, allow, press the button again.
- On macOS also check **System Settings → Privacy & Security → Microphone** and confirm the *browser* is enabled. Note it is the browser now, not the terminal.

#### 2. The page is open on the wrong address

**Cause:** `getUserMedia` and `AudioWorklet` need a secure context, which browsers grant to `localhost` and nothing else, so a LAN address makes the audio API silently vanish.

**Fix:** Use the `http://127.0.0.1:8000` the terminal printed. Symptom is the Connect button staying disabled with a red box above it.

This replaces WSL-has-no-audio as the environment failure to watch for; WSL now works, because the audio is Windows' problem.

#### 3. "AccessDenied" or a refused brain in Step 6

:::alert{type="warning"}
By far the longest issue to fix if it comes up, because you often can't fix it in the room. Rule out the cheap causes first.
:::

**Cause, in the order worth checking:**

- **Region mismatch** between the credentials and the endpoint URL. On a Workshop Studio account, `AWS_REGION` must match the Region the event was provisioned in.
- **Expired temporary credentials.** Workshop Studio credentials time out; re-copy them from the event panel.
- **`bedrock:InvokeModel` but not `bedrock:InvokeModelWithResponseStream`.** The agent streams, so the non-streaming permission alone isn't enough.
- **Model access not granted** for that exact model ID in that exact Region. This is the one you can't fix live for an attendee on their own account.

**Fix:** Work the list top down. If it's model access on a personal account, have them keep going — `think_settings()` guards on the presence of credentials and falls back to the brokered provider, so their file still runs and Step 7 continues either way.

Anyone arriving from the Pipecat edition will also reach for `AWS_BEARER_TOKEN_BEDROCK`, which does nothing here.

#### 4. Bluetooth headsets

**Cause:** Activating the microphone flips many headsets into a low-quality mono profile, and some then fail to open at all.

**Fix:** Reconnect, or switch to wired. This is why the pre-event email asks for wired.

#### 5. The agent talks to itself

**Cause:** Laptop speakers feeding the laptop microphone, past the echo canceller. Much rarer than it used to be and no longer universal, which makes it *more* confusing when it happens to one person in the room.

**Fix:** Headphones, or turn the volume down. Step 1's check reports whether the browser actually granted echo cancellation, which is worth reading when someone hits this.

#### 6. A truncated API key

**Cause:** Copy-paste drops characters or adds a space.

**Fix:** Copy it again from the console. Step 1 makes a real authenticated call rather than checking the string is non-empty, so this surfaces immediately with `Deepgram rejected the key`.

#### 7. "The agent refused these settings"

**Cause:** Not an environment failure at all. A Deepgram model isn't available where that person is connecting.

**Fix:** Step 1 prints which model. Same fix for the whole room, since they share a `.env` layout: `DEEPGRAM_REGION=global`, or change the named model everywhere.

### Step-Specific Issues

**Step 2: Connect**

- **Issue:** Nothing after `>> Connection opened`.
- **Solution:** `on_message` was defined but never passed to `bridge.run`.

**Step 4: Talk to the agent**

- **Issue:** "connected, not sending audio" stays on the page.
- **Solution:** TODO 4.2 isn't done — `on_media=on_media` never reached `bridge.run`.

**Step 5: Barge-in**

- **Issue:** The agent still talks over you, but only briefly.
- **Solution:** They cleared one queue and not the other. The Python-side `Outbox` is the one people forget.

**Step 6: Persona, voice, and your own brain**

- **Issue:** `Thinking with: OpenAI (AWS credentials are in .env, but think_settings() is not using them yet)`.
- **Solution:** The keys are fine; TODO 6b.2 and 6b.3 aren't both done. That line reports what the function returned, not what's in the environment.

**Step 7: Function calling**

- **Issue:** The function never gets called.
- **Solution:** Nine times out of ten it's the description. It has to say *when* to use the function, not just what it does.
- **Issue:** The agent apologizes vaguely, but the console clearly shows the call going out.
- **Solution:** The name in `FUNCTIONS` doesn't match the key in `FUNCTION_HANDLERS`. Those two halves only meet through that string, so a rename in one place and not the other produces exactly this.

### Service Limits & Quotas

:::expand{header="Bedrock issues during delivery"}
- **"Model access is not enabled"** — Cannot be fixed in the room for a personal account. Have the attendee continue on the brokered fallback; the guard in `think_settings()` exists for this.
- **Throttling on a large event** — Every attendee holds one streaming Bedrock conversation at a time, so concurrency tracks headcount. Confirm on-demand throughput for the chosen model in the event Region before a 200+ person session.
- **"Everything is slow"** — `total_latency` in Step 8 includes the LLM. A larger model, or a Region far from the room, both show up in that number.
:::

### When Nothing Works

If you hit something that isn't listed here:

1. Check the terminal. Every failure in this workshop prints something, and Step 1 diagnoses most of them by name.
2. Check Workshop Studio event logs for account or credential issues.
3. Ask the Workshop Studio Atlas Agent from the UI, or #workshop-studio-interest (Slack).
4. **Add it to this guide after you fix it.** The next facilitator will appreciate it.

---

## Resources

### For Facilitators

- **Slides:** `slides/deepgram-voice-agent-workshop.pptx` in the source repository. A PowerPoint-compatible projection deck that mirrors the agenda above. **The slides are a visual aid for the room, not a talk.** Most exist to be left on screen while people work, so anyone who looks up can see which step the room is on, the command to run, and what has to be true before we move. Only the Step 0 block is presented, and it's capped at ten minutes. Every slide carries speaker notes with the things worth saying, the failures to watch for, and the check-yourself answer for that step. Note the deck ships with Step 6b as a separate optional block; this edition merges it into Step 6, so adjust that section or skip it.
- **Source code:** [deepgram-devs/deepgram-workshop-py](https://github.com/deepgram-devs/deepgram-workshop-py)
- **Repository maintenance:** `uv run scripts/verify_steps.py` checks that every step compiles, no TODO markers leak into the next folder's answer key, docs are present, and ruff passes. Run it after editing any step. Nine near-identical copies of `main.py` drift quietly.
- **Support channel:** Workshop Studio Atlas Agent from the UI, or #workshop-studio-interest on Slack.

### Links for the room

Worth turning into QR codes on a slide, since attendees on a conference floor won't type URLs.

| What | URL |
|---|---|
| Sign up / API key | https://console.deepgram.com/signup?jump=keys |
| Workshop source code | https://github.com/deepgram-devs/deepgram-workshop-py |
| Voice Agent docs | https://developers.deepgram.com/docs/voice-agent |
| Flux docs | https://developers.deepgram.com/docs/flux |
| Voice catalogue | https://developers.deepgram.com/docs/tts-models |
| Bedrock model access | https://console.aws.amazon.com/bedrock/home#/modelaccess |
| Deepgram Playground (Easy Mode) | https://playground.deepgram.com/voice-agent |
| Discord | https://discord.gg/xWRaCDBtW4 |
| GitHub Discussions | https://github.com/orgs/deepgram/discussions |

### For Participants (share after the event)

- **Deepgram documentation:** [Voice Agent API](https://developers.deepgram.com/docs/voice-agent), [Flux](https://developers.deepgram.com/docs/flux), [TTS models](https://developers.deepgram.com/docs/tts-models)
- **AWS documentation:** [Amazon Bedrock](https://docs.aws.amazon.com/bedrock/), [Bedrock pricing](https://aws.amazon.com/bedrock/pricing/)
- **What to do next:** The Summary page lists seven directions — telephony, multilingual, eager end-of-turn, keyterms, mid-conversation updates, server-side functions, and conversation context.
- **Community:** [Discord](https://discord.gg/xWRaCDBtW4) and [GitHub Discussions](https://github.com/orgs/deepgram/discussions)

---

## Check-yourself answers

Each page poses these with the answer in an expandable block, so attendees can self-serve. Here they are in one place for when you ask the room.

**Step 0.** Speech-to-text, an LLM, and text-to-speech; the LLM holds the conversation history. · Turn-taking, provider handoffs, buffering, interruption signalling. · Turn detection happens inside the Flux model, server-side; the client is still responsible for clearing queued playback on barge-in. · The next folder.

**Step 1.** One Deepgram key. Deepgram brokers the OpenAI call, so your Deepgram key covers the LLM — until Step 6, where the model moves into your own AWS account and a second credential appears.

**Step 2.** `listen` configures speech-to-text; the LLM is named under `think`. · The agent discards any media that arrives before the handshake completes, so audio sent early is silently lost.

**Step 3.** The greeting starts arriving within milliseconds of `SettingsApplied`, and audio with nowhere to go is audio thrown away. The bridge waits for the browser's `start` message before it opens the Deepgram socket for exactly this reason.

**Step 4.** Flux does turn detection inside the model, so there's nothing for client-side VAD to do.

**Step 5.** Because the pump would immediately refill the browser's queue from the Python-side one. Clearing the far queue first and the near queue second means the agent talks over the user a moment later rather than immediately, which is worse, because it looks like it nearly works. (The PortAudio equivalent: `stop()` drains the buffer, playing everything already queued before stopping. `abort()` throws it away.)

**Step 6, Part 1.** Tell it that it's speaking (no markdown, bullets, or emoji), and tell it to be brief.

**Step 6, Part 2.** Deepgram brokers OpenAI, so a model name is the whole configuration. It doesn't broker Bedrock: it makes the call *as you*, which needs credentials to make it with (`provider.credentials`) and an address to make it to (`think.endpoint`).

**Step 7.** Setting `endpoint` moves execution to Deepgram; omitting it keeps the function client-side.

**Step 7b (optional).** The prompt is a request and the payload is a guarantee. Prose asking the model not to repeat a phone number can be talked around; a payload that never carried one cannot. Filter at the boundary and there is nothing to leak.

**Step 8.** Raise `eot_threshold`. It demands more confidence before Flux calls the turn over.

**Easy Mode, Lab 1.** Speech-to-text. Flux does turn detection inside the model, so the same model that transcribes also decides when the turn ended.

**Easy Mode, Lab 2.** Because of the audio Deepgram already sent, which is sitting in a client-side playback queue. Deepgram can stop generating; it can't reach into a buffer it already handed over.

**Easy Mode, Lab 4.** Same as Step 6 Part 1.

---

## Post-Event Actions

Terminate the event in Workshop Studio. Cleanup is automatic — the workshop provisions no AWS resources, so there is nothing to tear down beyond the account itself.

If attendees used their own AWS accounts, remind them to delete the IAM access key they created for Step 6. It's the only credential the workshop asked them to create, and the Clean up page walks through it.

Then, optionally:

- Share the repo link again and point people at the Summary page for extension ideas.
- Invite people into Discord while they still have the agent running.
- **Update this guide** if you found new issues or better approaches.

Worth capturing while it's fresh: which step consumed the most time, how many people finished Step 7, how many got a Bedrock reply on the first try, and which environment failures actually occurred. That's what you'll change before the next run.

---

## Additional Notes

**This edition merges Step 6b into Step 6.** The source repository at [deepgram-devs/deepgram-workshop-py](https://github.com/deepgram-devs/deepgram-workshop-py) treats Amazon Bedrock as an optional detour, skipped in its run of show, because attendees rarely arrive with model access. This edition makes it required, on the assumption that accounts are provisioned ahead of the event. If you're delivering to a room that has neither provisioned accounts nor prepared their own, run Part 1 only and mention Part 2 as take-home — `think_settings()` falls back to the brokered provider without credentials, so nothing downstream breaks.

**Where the AWS credential travels.** Worth saying out loud in Step 6: the attendee's access key and secret go into the Deepgram `Settings` message, over the WebSocket. That is what "Deepgram makes the call as you" means, and it's why the workshop asks for a credential scoped to `bedrock:InvokeModelWithResponseStream` on one model ARN rather than admin keys. Some rooms will have someone who needs to hear that stated plainly before they'll paste anything.

**No infrastructure.** Everything is either a local process the attendee started or a metered API call. Nothing accrues cost between sessions.

---

## Need Help?

- **Improve this guide:** Open an issue or PR against the workshop content repository.
- **Workshop Studio support:** Ask the Workshop Studio Atlas Agent from the UI, or #workshop-studio-interest (Slack).
- **Deepgram questions:** [Discord](https://discord.gg/xWRaCDBtW4) or [GitHub Discussions](https://github.com/orgs/deepgram/discussions).
- **Authoring docs:** [Workshop Studio Facilitator Guide Documentation](https://catalog.workshops.aws/docs/en-US/create-a-workshop/authoring-a-workshop/facilitator-guide)
