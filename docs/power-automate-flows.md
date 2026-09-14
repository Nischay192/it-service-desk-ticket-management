# Power Automate Flows

This project contains four completed Power Automate workflows built around the SharePoint `Tickets` list.

## 1. New Ticket Notification

**Trigger:** SharePoint — When an item is created

**Action:** Outlook — Send an email (V2)

A new ticket sends an email to the assigned technician. The notification includes Issue, Description, Priority, Status, Category, and Requester.

### Roadblock and resolution

The SharePoint default `Title` column was renamed to `Issue`, but Power Automate continued to expose the underlying field as `Title`. The first test showed a blank Issue value. Using the `Title` dynamic content corrected the output.

**Status:** Complete and tested.

## 2. Critical Ticket Escalation

**Trigger:** SharePoint — When an item is created

**Condition:** `Priority Value = Critical`

**Action:** Outlook — Send an email (V2) with High importance

Critical tickets generate an immediate high-importance notification to the assigned technician.

### Roadblock and resolution

The initial condition did not correctly reference the Choice-field value. The condition was corrected to use **Priority Value** and compare it with `Critical`.

**Status:** Complete and tested.

## 3. SLA Overdue Reminder

**Trigger:** Recurrence — every 1 hour

### Structure

`Recurrence -> Get items -> Apply to each -> Condition -> Send email -> Update item`

### OData Filter Query

```text
SLADueDate ne null and Status ne 'Completed' and SLAReminderSent eq 0
```

The query prevents blank SLA dates, completed tickets, and already-reminded tickets from entering the processing loop.

### Date comparison

```text
ticks(item()?['SLADueDate']) <= ticks(utcNow())
```

### True branch

1. Send an overdue SLA email to Assigned To.
2. Update `SLA Reminder Sent` to `Yes`.

### Roadblock and resolution

The original condition failed when a ticket had a blank SLA Due Date. Power Automate reported a type mismatch involving `Null` and `String`.

Blank SLA dates were excluded in the SharePoint OData filter, and the comparison was then performed with `ticks()` against `utcNow()`.

**Status:** Complete and successfully tested.

## 4. Ticket Resolution Notification

**Trigger:** SharePoint — When an item is created or modified

**Condition:** Status equals `Completed`

**Action:** Outlook — Send an email (V2) to Requester

### Condition expression

```text
triggerOutputs()?['body/Status/Value']
```

Comparison: `is equal to Completed`

### Email content

- Issue
- Description
- Category
- Priority
- Resolution
- Resolved Date

### Roadblock and resolution

The first condition showed a broken `?Status.Value` reference and evaluated incorrectly. The trigger output was inspected in run history, where the runtime data showed `Status.Value = Completed`. The condition was then changed to:

```text
triggerOutputs()?['body/Status/Value']
```

The corrected condition evaluated to True and the email action succeeded.

The first successful email also had blank Resolution and Resolved Date values because those fields were blank on the test ticket. The source ticket was updated with the missing values and the email body was formatted with clear labels and spacing.

**Status:** Complete and successfully tested.

## Design Principles

- SharePoint is the centralized ticket data source.
- Event-driven automation handles immediate notifications.
- Conditional branching handles critical-ticket escalation.
- Scheduled processing handles SLA monitoring.
- OData filtering excludes invalid or ineligible records before processing.
- A reminder flag prevents repeated SLA notifications.
- Runtime trigger data is inspected when visual dynamic-content references do not behave as expected.
- Testing validates both flow execution and actual email content.