---
title: "Lab 5: Let it call a function"
weight: 55
---

**Goal:** Understand how an agent reaches outside its own head.

Tell your agent you're calling about your bank account and ask for your balance. It will give you a number with total confidence, because a language model has no database. Function calling closes that gap and every gap like it: your database, your API, the user's order history, today's date.

The flow has four hops:

1. The configuration advertises a function — a **name**, a **description**, and a schema for its arguments.
1. The model decides a turn warrants calling it, and Deepgram emits a **function call request** with the arguments it chose.
1. Something runs the function and returns a result.
1. The model works that result into its reply, and the agent speaks it.

## See it happen

Open the **Functions** section of the settings panel. Two example functions ship with the playground:

- **Arithmetic (Example)** — `do_arithmetic`, which takes an `operation` (add, subtract, multiply, divide) and a list of `numbers`.
- **End Conversation (Example)** — `end_conversation`, which the agent calls when it hears a phrase that means the conversation is over.

Toggle **Arithmetic (Example)** on, then expand it. You get the **Function name** and the **Function JSON** — the exact schema the model reads. Read the `description` fields carefully; you're about to see why they matter.

Start a conversation and ask something arithmetic: *"what's four hundred and seventeen times nineteen?"* Watch the message list. You'll see the function call request with the arguments the model picked, then the response, then the agent speaking the answer. Expand those events and compare the arguments to the schema you just read.

Then toggle **End Conversation (Example)** on, start again, and say goodbye. Different function, same four hops.

## The two things worth remembering

**The description is a prompt.** It is the only thing the model reads when deciding whether to call your function. Write it to say *when* to use the function, not just what it does. "Get the balance" is a weak description; "Get the current balance for a customer account by its last four digits. Use this whenever the customer asks how much money they have" is one the model can act on. Nine times out of ten, a function that never fires has a description problem, not a wiring problem.

**Where the function runs is one field.** In the JSON, a function definition can carry an `endpoint`:

:::code{language=json showCopyAction=true showLineNumbers=false}
{
  "name": "get_order_status",
  "description": "Look up the status of a customer order by order number. Use this whenever the caller asks where their order is.",
  "parameters": {
    "type": "object",
    "properties": {
      "order_number": { "type": "string", "description": "The order number the caller reads out." }
    },
    "required": ["order_number"]
  },
  "endpoint": { "url": "https://api.example.com/orders", "method": "post" }
}
:::

With `endpoint` set, Deepgram calls that URL itself — server-side, no application of yours involved. Leave `endpoint` out and Deepgram sends the call down the connection to your app and waits for you to answer, which is what you want when the function touches local state or credentials you'd rather not put on a public URL. That single omitted field is the whole distinction, and the code track's [Step 7](/make-it-yours/function-calling) turns on it.

The playground's Functions section ships those two examples rather than an editor, so writing your own is where Easy Mode hands off. You now know exactly what to write, and the [function calling documentation](https://developers.deepgram.com/docs/voice-agents-function-calling) has the rest.

::alert[Sketch a function for your own use case: what would your agent need to look up that a model cannot possibly know? Share one with the room. This is the most useful two minutes in the workshop.]{type="info" header="Pause: check in with the instructor"}
