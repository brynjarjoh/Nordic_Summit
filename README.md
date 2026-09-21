# From Zero to Hero — Demo Artefacts

*Trigger, Automate, Extend, Connect, Intelligise. Five demo artefacts for BC consultants, from Nordic Summit 2026.*

Companion repo for **"From Zero to Hero: 5 Steps to Automation in Business Central"** — Nordic Summit 2026, Billund.
Speaker: Brynjar Jóhannesson, Wise.

The session itself runs as a tool-selection map delivered through five stories, not five live demos. These files are where the "five demos" promise actually lands — one artefact per step, each grounded in a real, recurring consulting problem. **Generalized and anonymized: no file here is tied to a specific client or engagement.**

For the tool-selection map, the data gate, and fillable story templates, see [`index.html`](./index.html) — live at **[brynjarjoh.github.io/Nordic_Summit](https://brynjarjoh.github.io/Nordic_Summit/)**.

## The five steps

| # | Step | File | Tool |
|---|---|---|---|
| 1 | Trigger | [`01-trigger.md`](./01-trigger.md) | BC Workflow Engine — zero code |
| 2 | Automate | [`02-automate.md`](./02-automate.md) | Power Automate + BC connector |
| 3 | Extend | [`03-extend.md`](./03-extend.md) | AL custom workflow events |
| 4 | Connect | [`04-connect.md`](./04-connect.md) | HttpClient / API pages |
| 5 | Intelligise | [`05-intelligise.md`](./05-intelligise.md) | AI Builder + Copilot |

Each file follows the same shape: the real problem behind the demo, the pattern that fixes it, working AL/config where the step involves code, and the cheap move from the talk.

## Setup

What you actually need before each demo works, not before the file opens.

**1 · Trigger** — Any Business Central environment (sandbox or production), functional consultant access to the Workflow page. No AL, no Power Platform licence.

**2 · Automate** — A Power Automate plan that includes premium connectors. The Business Central connector is a Premium-tier connector; a Business Central licence carries its own Power Automate use rights for flows against BC data, but a standalone Power Automate seat needs a Premium (or Dynamics 365–inclusive) plan for anything beyond that. [Automate workflows prerequisites](https://learn.microsoft.com/dynamics365/business-central/dev-itpro/powerplatform/automate-workflows) — check current terms, this tier structure moves.

**3 · Extend & 4 · Connect** — VS Code + the AL Language extension, a Business Central sandbox you can publish extensions to (a dev licence, not production), and your own object ID range — see the note below.

**5 · Intelligise** — AI Builder capacity. Worth knowing before you scope a client project on this: **the licensing model is mid-transition.** Seeded AI Builder credits from qualifying Power Apps / Power Automate / Dynamics 365 licences stop being issued to new or renewed licences from **1 November 2026**; usage is moving to Copilot Credits. [Licensing and AI Builder credits](https://learn.microsoft.com/ai-builder/credit-management) — verify current terms rather than trusting this file's date.

## Repo structure

```
.
├── README.md
├── LICENSE
├── index.html          ← the tool-selection map / cheap moves / templates / horizon pack
├── 01-trigger.md
├── 02-automate.md
├── 03-extend.md
├── 04-connect.md
└── 05-intelligise.md
```

## Licence

MIT — see [LICENSE](./LICENSE). Use it, fork it, take it into a client scoping call.
