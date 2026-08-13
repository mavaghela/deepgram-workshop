---
title: "Clean up"
weight: 70
---

**Goal:** Leave nothing behind that costs money or outlives the day.

This workshop provisions no AWS resources. Nothing was deployed, so there is no stack to delete and nothing sitting idle accruing cost. There are three small things worth doing anyway.

## Stop the local processes

Any step you left running is still holding port 8000 and still holding a WebSocket open. Ctrl+C in each terminal.

The agent bills per second of audio, and an idle connection with nobody talking to it costs nothing meaningful — but a page left open in a browser tab will keep a microphone light on, which people notice.

## Revoke the AWS credential

::::tabs

:::tab{id="ws-account" label="Workshop Studio account"}
Nothing to do. Terminate the event in Workshop Studio and the account, its credentials, and its Bedrock model access all go away with it. The temporary credentials you pasted into `.env` expire on their own.

Delete your local `.env` anyway if you're handing the machine to someone else — it still has your Deepgram key in it.
:::

:::tab{id="own-account" label="Your own AWS account"}
Delete the IAM access key you created for Step 6. It is the only credential this workshop asked you to create in your own cloud, and it has no reason to outlive the day.

**IAM → Users → your user → Security credentials → Access keys → Delete.**

If you created a dedicated IAM user for the workshop rather than a key on an existing one, delete the user as well.

You can leave Bedrock model access enabled. It costs nothing until a request is made against it, and you'll want it if you keep building.
:::

::::

## Your Deepgram key

Keep it. New accounts come with $200 in credit and this workshop spent well under a dollar of it, so there's plenty left to keep building with.

If you'd rather not: **console.deepgram.com → API Keys → Delete**.

---

That's everything. The agent still runs on your machine any time you want it — nothing about it depended on the event.
