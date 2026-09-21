# Step 1 — Trigger

**Tool:** BC Workflow Engine · zero code
**Cheap move:** Read the workflow event list before scoping

---

## The real problem

A recurring pattern across BC engagements: a client asks for "an approval step" or "a notification when X happens," and it gets scoped as an AL extension — a new table field, a codeunit, a page customization. Days of estimated work.

Often, none of that is necessary. Business Central ships a configurable workflow engine with a long list of built-in events and responses, covering most of the "notify someone" and "require approval" scenarios a functional consultant hears in discovery. The AL quote gets invoiced; the client later discovers (from someone else) that it was a 20-minute configuration task. That's an expensive way to lose trust.

The fix isn't a tool. It's a five-minute habit: check the workflow event list before you scope anything that sounds like "when this happens, do that."

## What "zero code" actually looks like

1. Search **Workflows** in Business Central, open the **Workflow** page, choose **New**.
2. Fill in **Code**, **Description**, and a **Category** (e.g. `SALES`, `PURCH`).
3. On the first line, set **When Event** — this is a fixed list of events Microsoft has already published. Examples: *A vendor record is changed*, *A sales document is released*, *A purchase invoice is ready for approval*.
4. Set **On Condition** to scope which records the event applies to (e.g. only when a specific field changes).
5. Set **Then Response** — again a fixed list: create an approval request, revert a field change, send a notification, run a report.
6. Chain additional steps for what happens on approval, rejection, or delegation.
7. Turn on **Enabled**.

No extension. No deployment. No AppSource review cycle.

## Where the event list actually lives

Two places to check before quoting anything:

- **Workflow page → When Event dropdown** — the full list of events available in the current app + any installed extensions.
- **Workflow Templates page** — pre-built, importable workflows. Anything starting `MS-` shipped from Microsoft; copy and adapt rather than building from scratch.

If the scenario genuinely isn't in that list, *then* you're into Power Automate (Step 2) or a custom AL event (Step 3) — but check first.

## Moving workflows between environments

Workflows export and import as XML:

- **Export:** open the workflow, choose **Export to File**.
- **Import:** on the Workflow list, choose **Import from File**, select the XML.

This is how a workflow built in a sandbox moves to production, or how you'd hand a client a starting point without giving them AL.

> **Caution:** importing a workflow whose Code already exists in the target database overwrites it. Check for a naming collision first.

## Further reading (verified against Microsoft Learn, Sept 2026)

- [Workflows in Business Central](https://learn.microsoft.com/dynamics365/business-central/across-workflow)
- [Create workflows to connect tasks in business processes](https://learn.microsoft.com/dynamics365/business-central/across-how-to-create-workflows)
- [Export and import approval workflows](https://learn.microsoft.com/dynamics365/business-central/across-how-to-export-and-import-workflows)
- [Walkthrough: Set up and use a purchase approval workflow](https://learn.microsoft.com/dynamics365/business-central/walkthrough-setting-up-and-using-a-purchase-approval-workflow)
