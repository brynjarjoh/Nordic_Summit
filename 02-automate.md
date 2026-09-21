# Step 2 — Automate

**Tool:** Power Automate + BC connector
**Cheap move:** One page — every flow, its owner, where failures land

---

## The real problem

A flow triggers off a BC event — "when a record is created," "when a document is posted" — and does something downstream: creates a record in another system, posts a matching transaction, sends a notification with side effects. It works in testing. Weeks later, someone notices two of something that should exist once: two postings, two notifications, two records in the downstream system.

The cause is rarely a bug in the flow's logic. It's almost always one of:

- A connector trigger firing more than once for the same underlying change (retries, redelivery, or a record being touched twice in one business process).
- The flow being run manually a second time by someone who didn't know it had already fired.
- A downstream system timing out, the flow retrying, and the downstream system having actually succeeded the first time.

None of this shows up until someone reconciles two systems and finds a mismatch — which is why it's expensive. Nobody's watching for a duplicate at 2am.

## The pattern: an idempotency guard

Before a flow creates anything, it checks whether it already has. That check needs a key the flow can reliably re-derive — a document number, a combination of company + document type + document number, or a GUID field already on the record.

Flow structure:

1. **Trigger** — BC connector, "When a record is created or modified" (or the relevant BC event).
2. **Compose** — build the idempotency key from fields already on the trigger record (e.g. `Company + "-" + Document No.`).
3. **Lookup** — query the destination system, or a lightweight BC/Dataverse tracking table, for a record with that key.
4. **Condition** —
   - **Key found** → terminate the run with an outcome of *Skipped — already processed*, logged, not treated as a failure.
   - **Key not found** → proceed to create the downstream record, then **write the key back** to the tracking table in the same transaction as the create, so a crash between the two doesn't reopen the gap.
5. **On any step failure** — write to a shared error log (see the registry below), don't just let the flow's built-in retry hide it.

The BC side of this is usually one small addition: a text or GUID field the flow can query against, or a "Processing Status" flag on the source table the flow sets after a confirmed downstream success — not before.

## The flow registry

The actual cheap move isn't the guard — it's knowing the guard exists on every flow that touches financial or operational data, without having to open each flow to check. One page, kept current:

| Flow name | BC trigger | Idempotency key | Owner | Failures land in |
|---|---|---|---|---|
| *(example)* Sales order → 3PL | Sales order released | `Company + No.` | *(name)* | `#integration-alerts` Teams channel + error log table |
| | | | | |
| | | | | |

Fill a row in per flow at build time — not retroactively during an incident review. The empty rows are the template; copy this table into your own tracker.

## Cheap move, restated

If you can't point to this page during a client call, you don't actually know which of your flows can safely run twice and which can't. Find out before the client does.
