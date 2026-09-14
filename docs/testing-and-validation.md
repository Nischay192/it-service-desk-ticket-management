# Testing and Validation

## Test Tickets

The project used sample tickets including:

- `Test - Wi-Fi Connection`
- `Test - Password Reset`
- `Test - Critical Server Issue`
- `Test - Critical Escalation 2`
- `Test - SLA Overdue`

## Test Matrix

| Scenario | Expected result | Result |
|---|---|---|
| New ticket created | Assigned technician receives notification | Passed |
| Critical ticket created | High-importance escalation is sent | Passed |
| Non-critical ticket created | Critical branch does not execute | Passed |
| Overdue unresolved ticket | SLA reminder is sent and flag is updated | Passed |
| Completed ticket | Requester receives resolution notification | Passed |

## Flow 1 — New Ticket Notification

Create a new SharePoint ticket with an Assigned To user.

Expected:
- Trigger succeeds.
- Email action succeeds.
- Assigned technician receives ticket details.

## Flow 2 — Critical Ticket Escalation

Create a ticket with `Priority = Critical`.

Expected:
- Trigger succeeds.
- Priority condition evaluates to True.
- High-importance email is sent.

Also test a non-critical ticket to confirm the escalation branch is skipped.

## Flow 3 — SLA Overdue Reminder

Use a ticket with:

- SLA Due Date in the past
- Status not Completed
- SLA Reminder Sent = No

Expected:
- Recurrence succeeds.
- Get items returns the eligible ticket.
- Apply to each processes it.
- Overdue condition evaluates to True.
- Email succeeds.
- SLA Reminder Sent becomes Yes.

The successful run validated the recurrence, SharePoint retrieval, loop, condition, email, and update actions.

## Flow 4 — Ticket Resolution Notification

Modify a ticket so:

```text
Status = Completed
Resolution = Password reset completed successfully.
Resolved Date = populated
```

Expected:
- SharePoint trigger succeeds.
- Status condition evaluates to True.
- Requester email action succeeds.
- Email contains the resolution information.

The final test validated the trigger, condition, and email delivery.

## Failure-Driven Testing

### SLA null-date failure

The original date comparison attempted to compare a blank SLA date with the current time.

**Observed:** Type mismatch involving `Null` and `String`.

**Fix:** Exclude blank SLA dates with OData and compare timestamps with `ticks()`.

### Resolution Status failure

The original condition displayed an unresolved `?Status.Value` reference.

**Observed:** The condition did not correctly resolve the SharePoint Choice field.

**Fix:** Inspect trigger output and use:

```text
triggerOutputs()?['body/Status/Value']
```

### Incomplete email data

The first successful resolution email had blank Resolution and Resolved Date values.

**Cause:** The test record had not yet been populated with those fields.

**Fix:** Populate the source ticket and rerun the test.

## Why Run History Matters

Power Automate run history was used to distinguish trigger failures, condition results, skipped actions, successful email actions, and successful SharePoint updates. This made troubleshooting evidence-based rather than assumption-based.