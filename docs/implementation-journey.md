# Implementation Journey

This document captures the most important engineering decisions and troubleshooting steps from building the service desk automation.

## 1. Establishing the ticket workflow

I started by designing SharePoint as the system of record for the ticket lifecycle. The data model supports assignment, priority, status, requester communication, resolution, and SLA tracking.

The automation was then built around the events that matter to an IT support team: a ticket being created, a critical ticket being raised, an SLA becoming overdue, and a ticket being completed.

## 2. Solving SharePoint field-reference issues

### Renamed Title field

The SharePoint `Title` column was renamed to `Issue`. The first notification email showed a blank issue because Power Automate continued to expose the underlying field as `Title`.

**Fix:** Use the underlying `Title` dynamic content for the ticket issue.

**Result:** The next test populated the issue correctly.

**Key lesson:** Display names in SharePoint are not always the same as the field references exposed to Power Automate.

### Choice-field values

The critical-ticket flow initially did not route correctly because the Choice field was not being referenced as expected.

**Fix:** Use the Choice field's `Value` representation when evaluating the priority.

**Result:** The critical branch executed and sent the expected high-importance notification.

## 3. Building reliable SLA monitoring

The SLA workflow runs every hour and processes only tickets that are eligible for an overdue check.

The first implementation attempted to compare a blank SLA Due Date with the current time and failed with a `Null` versus `String` type mismatch.

**Fix:** Filter invalid/ineligible records before the comparison:

```text
SLADueDate ne null and Status ne 'Completed' and SLAReminderSent eq 0
```

Then compare timestamps using:

```text
ticks(item()?['SLADueDate']) <= ticks(utcNow())
```

**Result:** The overdue test ticket successfully passed through retrieval, looping, condition evaluation, email delivery, and SharePoint update. The reminder flag was set to `Yes`.

**Key lesson:** Scheduled workflows should validate data before performing date operations and should maintain state to avoid repeated actions.

## 4. Diagnosing the resolution workflow

The resolution notification flow needed to send an email when a ticket changed to `Completed`.

The first condition displayed a broken `?Status.Value` reference and evaluated incorrectly.

Instead of continuing to change the visual condition builder blindly, I inspected the trigger output in Power Automate run history.

The runtime data showed:

```text
Status.Value    Completed
```

The final expression was:

```text
triggerOutputs()?['body/Status/Value']
```

**Result:** The condition evaluated to True and the requester notification was sent successfully.

A later test also revealed blank Resolution and Resolved Date values. The flow itself had succeeded; the source ticket simply did not contain those values yet. After populating the source record, the notification output was complete.

**Key lesson:** Troubleshooting should distinguish between a workflow failure and incomplete source data.

## 5. Final outcome

The completed solution demonstrates four different automation patterns working against one SharePoint data source:

- **Event-driven notification** for new tickets
- **Conditional escalation** for critical tickets
- **Scheduled monitoring** for SLA compliance
- **State-change notification** when work is completed

The final tests confirmed the expected branches, email actions, and SharePoint updates. More importantly, the build required practical use of Power Automate expressions, SharePoint field behavior, OData filtering, scheduled processing, state management, and run-history debugging.
