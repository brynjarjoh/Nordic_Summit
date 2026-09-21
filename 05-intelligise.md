# Step 5 — Intelligise

**Tool:** AI Builder + Copilot
**Cheap move:** Four numbers on one page — the lowest one is your next project

---

## The real problem

Once AI Builder and Copilot are on the table, they get reached for on things they were never the right tool for — a price band lookup, a currency conversion, a "if field A equals X, set field B to Y" rule that's been written down in a spec somewhere for years. All of that is deterministic. It has one correct answer, computable without inference, and an `IF` statement gets it right every time, with no per-call cost and no model drift to monitor.

The instinct to reach for AI isn't wrong — it's aimed at the wrong problems. The fix isn't "use AI less." It's a filter, applied before scoping:

> **Is this rule knowable and stable?** If yes, write it down as a rule. If the judgment genuinely depends on unstructured input, ambiguous context, or pattern-matching across cases too varied to enumerate — that's where AI Builder or Copilot earns its place.

## Where it actually fits: structured flagging

A good Intelligise candidate looks like this: not "calculate the number" (deterministic — write the rule) but "read unstructured context and flag what a rule can't cleanly express."

**Composite example — not a real deployed prompt, illustrative of the pattern:**

> **AI Builder prompt — Invoice Review Flag**
>
> *Input:* Vendor name, invoice line descriptions (free text), invoice total, the vendor's historical average invoice total, price variance percentage vs. the last accepted invoice for this vendor/item combination.
>
> *Instruction:*
> "Given the invoice line descriptions and the price variance shown, decide whether this invoice should be routed to Procurement for manual review before posting. Flag for review if the line descriptions suggest a change in scope, materials, or service level compared to what the vendor typically bills — not just because the price moved. Do not flag on price variance alone; that is handled separately. Return `Flag` or `NoFlag`, plus one sentence explaining which specific wording or pattern drove the decision."
>
> *Output schema:* `{ "decision": "Flag" | "NoFlag", "reason": string }`

Notice what's *not* in the prompt: the price variance threshold itself. That's a number, computed deterministically before the prompt ever runs, and passed in as context — not something the model is asked to calculate. The model's job is the part that's genuinely hard to write as a rule: does this line-item wording look like scope creep.

## The evaluation — before you trust it in production

Every AI-assisted suggestion needs a measured acceptance rate before it's allowed to influence a real process unsupervised. Four numbers, reviewed on a cadence, not assumed once and forgotten:

| Suggestion type | Acceptance rate | Sample / period | Action if below threshold |
|---|---|---|---|
| *(example)* Invoice review flag | 91% | 340 invoices / last 30 days | — |
| *(example)* Vendor match | 99% | 340 invoices / last 30 days | — |
| *(example)* Account suggestion | 96% | 340 invoices / last 30 days | — |
| *(example)* Contract match | 62% | 340 invoices / last 30 days | Route to human review until ≥85% over two consecutive periods |

Fill the "Action if below threshold" column *before* you go live, not after the lowest number embarrasses someone in a steering meeting. The lowest number on the page is never a footnote — it's the answer to "what should we build next," because it's the process still costing someone real review time.

## Cheap move, restated

An `IF` statement doesn't need an evaluation table. If you find yourself building one for something that turns out to always return the same answer for the same inputs, that's the signal you reached for AI Builder when the rule was already knowable — and it's cheaper to fix that now than to keep paying for inference on a deterministic lookup.
