---
title: "Lab 3: Tune turn detection"
weight: 53
---

**Goal:** See the trade-off between a snappy agent and one that lets you finish a sentence.

The agent's sense of "you're done talking" comes from Flux, Deepgram's conversational speech-to-text model. Turn detection lives inside the model rather than in a timer on the client, and it's tunable. The Voice Agent panel doesn't expose those dials, so take a short side trip to see them directly:

1. Open the **Streaming Speech to Text** playground and choose the turn-based streaming mode with the `flux-general-en` model.
1. Click **Start listening** and talk. Watch the events: `StartOfTurn`, `EagerEndOfTurn`, `TurnResumed`, `EndOfTurn`, each with a confidence score.
1. Find these three settings and move them:

| Setting | What it does | Default |
|---|---|---|
| **End of Turn Threshold** | How confident the model must be that you've finished before it fires `EndOfTurn`. Lower (0.5) answers faster and risks cutting you off; higher (0.9) waits for certainty. | 0.7 |
| **End of Turn Timeout** | Ends the turn after this much silence regardless of confidence. | 5000 ms |
| **Eager End of Turn Threshold** | A lower bar that fires an early "probably done" signal, so an agent can start drafting a reply speculatively. Must be at or below the End of Turn Threshold. | — |

Try 0.5, then 0.9, saying the same halting sentence both times: *"I'd like to order a… uh… large coffee."* At 0.5 the model calls the turn during your "uh." At 0.9 it waits you out, and the reply arrives noticeably later.

No value is correct in the abstract. A drive-through agent wants speed; an agent taking a credit card number over the phone wants patience. You're choosing which mistake to make.

::alert[Good moment to compare notes with the code track, who are measuring the same trade-off in milliseconds over in [Step 8](/optimize/turn-detection-and-latency).]{type="info" header="Pause: check in with the instructor"}
