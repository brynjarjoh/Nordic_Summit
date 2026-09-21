# Step 4 — Connect

**Tool:** HttpClient / API pages
**Cheap move:** Daily count, source vs BC, alert on mismatch

---

## The real problem

An outbound integration call gets built, tested, and shipped. It works. Six weeks later, someone in finance asks why a handful of orders never showed up on the partner side. Nobody gets an error — because there wasn't one, in the sense anyone was watching for.

The actual failure mode: the HTTP call returned `200 OK` (or the AL code never checked the status at all), but the receiving system rejected the payload for a business reason downstream of the transport layer — a validation failure, a duplicate it silently discarded, a queue that dropped the message under load. Transport success and business success are two different things, and code that only checks the first one has no way to notice the second.

This is why the framework's rule for Connect is blunt: **silent failure is the expensive kind.** A loud failure gets fixed the same day. A silent one gets found in a month-end reconciliation, by someone who wasn't in the room when the integration was built.

## The API page — exposing what the partner needs, nothing more

```al
page 50102 "ZTH Export Ready Sales Orders"
{
    PageType = API;
    APIVersion = 'v1.0';
    APIPublisher = 'zth';
    APIGroup = 'integration';

    EntityCaption = 'Export Ready Sales Order';
    EntitySetCaption = 'Export Ready Sales Orders';
    EntityName = 'exportReadySalesOrder';
    EntitySetName = 'exportReadySalesOrders';

    ODataKeyFields = SystemId;
    SourceTable = "Sales Header";
    SourceTableView = where("Export Ready" = const(true));
    DelayedInsert = true;

    layout
    {
        area(content)
        {
            repeater(Group)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                    Editable = false;
                }
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                }
                field(exportReady; Rec."Export Ready")
                {
                    Caption = 'Export Ready';
                }
            }
        }
    }
}
```

`SourceTableView` scopes the entity set to only what the partner should see — narrower is safer than filtering client-side.

## The outbound call — checking both layers of success

```al
codeunit 50102 "ZTH Export HttpClient"
{
    procedure SendSalesOrder(var SalesHeader: Record "Sales Header"): Boolean
    var
        Client: HttpClient;
        RequestMsg: HttpRequestMessage;
        ResponseMsg: HttpResponseMessage;
        Content: HttpContent;
        ContentHeaders: HttpHeaders;
        Payload: JsonObject;
        PayloadText: Text;
    begin
        Payload.Add('orderNo', SalesHeader."No.");
        Payload.Add('exportReady', SalesHeader."Export Ready");
        Payload.WriteTo(PayloadText);

        Content.WriteFrom(PayloadText);
        Content.GetHeaders(ContentHeaders);
        if ContentHeaders.Contains('Content-Type') then
            ContentHeaders.Remove('Content-Type');
        ContentHeaders.Add('Content-Type', 'application/json');

        RequestMsg.SetRequestUri('https://partner.example.com/api/orders');
        RequestMsg.Method('POST');
        RequestMsg.Content(Content);

        // Layer 1: did the call complete at all?
        if not Client.Send(RequestMsg, ResponseMsg) then begin
            LogFailure(SalesHeader."No.", 'Transport failure — no response received');
            exit(false);
        end;

        // Layer 2: did the partner accept it?
        if not ResponseMsg.IsSuccessStatusCode() then begin
            LogFailure(SalesHeader."No.", StrSubstNo('HTTP %1', ResponseMsg.HttpStatusCode()));
            exit(false);
        end;

        // A 200 here still isn't proof the record now exists correctly on their side —
        // that's what the daily reconciliation below is for.
        exit(true);
    end;

    local procedure LogFailure(DocumentNo: Code[20]; Reason: Text)
    begin
        // Write DocumentNo, Reason, and CurrentDateTime to an integration log table.
        // This table is what the alert in the reconciliation job queries.
    end;
}
```

Two checks, not one — `Client.Send` returning `true` only means a response arrived, not that it was a good one. Both have to pass before this counts as sent.

## The reconciliation — the cheap move itself

The HttpClient code above catches failures it can see. It can't catch the case where the partner's system accepted the call, returned 200, and then silently dropped the record on their end. Nothing in Business Central will ever know that happened — unless something checks.

A scheduled job queue entry, run daily:

1. Count export-ready sales orders in BC for the period (the `SourceTableView` filter above gives you this for free).
2. Call a partner-side count or list endpoint for the same period.
3. Compare. Log the difference.
4. If the difference is non-zero, raise it — a Teams message, an email to the integration owner, an entry on the same dashboard the flow registry from Step 2 lives on.

This is deliberately simple: a count, not a full record-by-record diff. A count that's caught in 24 hours is cheap to fix. A record-level mismatch found in a month-end close is not.

## Cheap move, restated

If nothing is comparing a count on your side against a count on theirs, on a schedule, you don't have an integration — you have a hope.
