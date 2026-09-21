# Step 3 — Extend

**Tool:** AL custom workflow events
**Cheap move:** Search the base app for the event first

---

## The real problem

A consultant needs to hook custom logic onto something happening in Business Central. The workflow engine (Step 1) doesn't cover it — it's specific to a client's extension, not a documented workflow event. So a custom integration event gets published.

The recurring mistake: publishing that event without first checking whether the base application — or an already-installed extension — publishes an equivalent one on the same table, a few lines away. The result is two events doing overlapping jobs, a subscriber landscape nobody can reason about, and a maintenance surface that grows every time someone repeats the mistake on the next table.

The check costs five minutes. Skipping it costs a refactor eighteen months later, once three more subscribers depend on the event you shouldn't have published.

## How to check, concretely

Before writing `[IntegrationEvent]`, search the base app and any installed extensions for existing `OnBefore`/`OnAfter` events on the table or procedure you're extending:

- In VS Code, **Go to Definition** / **Find All References** on the table or procedure — installed extensions' symbols are available for this.
- Business Central's client has an **Events** page (search "Events") that records which events actually fire during a session — run the scenario once with recording on and see what's already published before assuming nothing is.
- Search the AL source for the base application on GitHub (`microsoft/ALAppExtensions` covers extensions; core base app symbols ship with the AL Language extension) for the table name + `IntegrationEvent`.

If nothing fits — genuinely custom logic, no equivalent exists — publish your own. Here's the pattern.

## Worked example (generic, not client-specific)

Scenario: a sales order needs to notify an external logistics partner the moment it's marked ready for export — a status that only exists because this client's process needs it. Nothing in the base app tracks "export ready," so a custom field and a custom event are the right call here.

**The event publisher:**

```al
codeunit 50100 "ZTH Export Readiness Events"
{
    [IntegrationEvent(false, false)]
    internal procedure OnAfterSalesOrderMarkedExportReady(var SalesHeader: Record "Sales Header")
    begin
    end;
}
```

**The call site — a table extension that raises it:**

```al
tableextension 50100 "ZTH Sales Header Ext" extends "Sales Header"
{
    fields
    {
        field(50100; "Export Ready"; Boolean)
        {
            Caption = 'Export Ready';
            DataClassification = CustomerContent;

            trigger OnValidate()
            var
                ExportReadinessEvents: Codeunit "ZTH Export Readiness Events";
            begin
                if "Export Ready" then
                    ExportReadinessEvents.OnAfterSalesOrderMarkedExportReady(Rec);
            end;
        }
    }
}
```

**A subscriber consuming it** — this is where Step 4's Connect logic would actually get called:

```al
codeunit 50101 "ZTH Export Notifier"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"ZTH Export Readiness Events",
        'OnAfterSalesOrderMarkedExportReady', '', false, false)]
    local procedure NotifyLogisticsPartner(var SalesHeader: Record "Sales Header")
    begin
        // See 04-connect.md — this is where the outbound HttpClient call belongs.
    end;
}
```

## Why this shape, specifically

- The publisher codeunit is separate from the table extension. Keeping event *definitions* in one place, independent of what raises them, means other extensions can subscribe without depending on the table extension that happens to raise it today.
- `internal` rather than `local` on the publisher — visible within the extension, not exposed as a public API surface by default. Widen it to a full public event only once something outside this extension genuinely needs to subscribe.
- The subscriber does nothing but hand off — no business logic inline in an event subscriber. If `NotifyLogisticsPartner` fails, that failure shouldn't be silently swallowed by the publish call; see the Connect step for the actual error handling.

## Cheap move, restated

Every custom event you publish is a contract you now maintain. Publish it because you checked and it genuinely doesn't exist — not because checking felt slower than writing.
