# Implementation Journey

This document records the most important implementation issues and how they were resolved.

## SharePoint Field References

### Renamed Title field

The SharePoint `Title` column was renamed to `Issue`, but Power Automate continued to expose the underlying field as `Title`. The first notification email therefore showed a blank Issue value.

**Fix:** Use the underlying `Title` dynamic content.

### Choice-field values

The critical-ticket flow initially did not reference the Priority Choice value correctly.

**Fix:** Reference the Choice field's `Value` and compare it with `Critical`.

**Result:** The critical branch executed and sent the expected high-importance notification.

## SLA Monitoring

The first SLA implementation attempted to compare a blank SLA Due Date with the current time and failed with a `Null` versus `String` type mismatch.

**Fix:** Filter eligible records before processing:

```text
SLADueDate ne null and Status ne 'Completed' and SLAReminderSent eq 0
```

Then compare timestamps with:

```text
ticks(item()?['SLADueDate']) <= ticks(utcNow())
```

**Result:** The overdue ticket was retrieved, processed, notified, and updated successfully. The reminder flag was set to `Yes`.

**Lesson:** Scheduled workflows should validate data before date operations and maintain state to prevent repeated actions.

## Resolution Workflow

The resolution notification condition initially displayed an unresolved `?Status.Value` reference and evaluated incorrectly.

The trigger output was inspected in Power Automate run history, which showed:

```text
Status.Value    Completed
```

The final expression was:

```text
triggerOutputs()?['body/Status/Value']
```

**Result:** The condition evaluated to True and the requester notification was sent successfully.

A later test showed blank Resolution and Resolved Date values. The workflow had succeeded; the source ticket simply had incomplete data. After populating the ticket, the notification output was complete.

**Lesson:** Run history helps distinguish workflow failures from incomplete source data.

## Outcome

The completed solution demonstrates four automation patterns against one SharePoint ticket source:

- Event-driven notification for new tickets
- Conditional escalation for critical tickets
- Scheduled monitoring for SLA compliance
- State-change notification when work is completed

The implementation required practical use of SharePoint field behavior, Power Automate expressions, OData filtering, scheduled processing, state management, and run-history debugging.
