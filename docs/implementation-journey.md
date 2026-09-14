# Implementation Journey

## Project Goal

Build a practical IT service desk automation portfolio project using **Microsoft SharePoint + Power Automate**. The solution models ticket intake, technician notification, critical escalation, SLA monitoring, and requester resolution notifications.

The project was intentionally kept focused on Power Automate rather than expanding into Power Apps, Teams, or Power BI.

## 1. SharePoint Foundation

Created an **IT Help Desk** SharePoint site with a `Tickets` list.

Core fields include Issue, Issue description, Quick steps, Priority, Status, Assigned to, Requester, Category, Associated files, Related issue, Resolution, Resolved Date, SLA Due Date, and SLA Reminder Sent.

Created operational views for Open Tickets, My Tickets, and Critical Tickets.

## 2. Flow 1 — New Ticket Notification

**Trigger:** SharePoint — When an item is created

**Action:** Outlook — Send an email (V2) to the assigned technician.

### Roadblock: renamed Title field

The default SharePoint `Title` column was renamed to `Issue`. The first test produced an email with a blank Issue value because Power Automate still exposed the underlying field as `Title`.

### Resolution

The notification was changed to use the `Title` dynamic content for the Issue value.

### Result

A second test populated the issue correctly.

**Lesson:** SharePoint display names and underlying field references are not always identical.

## 3. Flow 2 — Critical Ticket Escalation

**Trigger:** SharePoint — When an item is created

**Condition:** Priority Value equals `Critical`

**Action:** High-importance Outlook email to the assigned technician.

### Roadblock

The original condition did not correctly reference the Choice field value, so the expected escalation path did not execute.

### Resolution

The condition was corrected to compare the Priority Choice value with `Critical`.

The `Test - Critical Escalation 2` ticket was used to validate the corrected branch.

### Result

The critical branch executed and delivered the expected email.

**Lesson:** Choice fields should be validated using their actual runtime value representation.

## 4. Flow 3 — SLA Overdue Reminder

**Trigger:** Recurrence — every 1 hour

The flow retrieves tickets, processes them with `Apply to each`, checks the SLA deadline, sends a reminder when overdue, and updates the reminder flag.

### Roadblock: null SLA dates

The first SLA implementation attempted to compare a blank SLA Due Date with the current time. Power Automate returned a type mismatch involving `Null` and `String`.

### Resolution

The SharePoint `Get items` action was given this OData filter:

```text
SLADueDate ne null and Status ne 'Completed' and SLAReminderSent eq 0
```

The condition then compared timestamps using:

```text
ticks(item()?['SLADueDate']) <= ticks(utcNow())
```

When an overdue ticket matched, the flow sent the email and updated `SLA Reminder Sent` to `Yes`.

### Result

The overdue test ticket successfully passed through the recurrence, retrieval, loop, condition, email, and update steps.

**Lesson:** Scheduled flows should defensively filter null data before performing comparisons.

## 5. Flow 4 — Ticket Resolution Notification

**Trigger:** SharePoint — When an item is created or modified

**Condition:** Status equals `Completed`

**Action:** Outlook — Send an email (V2) to the Requester.

### Roadblock: unresolved Status reference

The first version of the condition showed a broken `?Status.Value` reference. The flow triggered, but the condition did not resolve the SharePoint Choice field correctly.

### Troubleshooting approach

Rather than continuing to guess at the expression, the trigger's raw output was inspected from Power Automate run history.

The runtime output showed:

```text
Status.Value    Completed
```

The final working expression became:

```text
triggerOutputs()?['body/Status/Value']
```

with the comparison:

```text
is equal to
Completed
```

### Result

The next test showed:

- Trigger — Succeeded
- Condition — True
- Send an email (V2) — Succeeded

The requester received the resolution email.

### Roadblock: blank Resolution and Resolved Date

The first successful resolution email contained the Issue, Description, Category, and Priority, but Resolution and Resolved Date were blank.

The automation was working; the missing values were caused by the corresponding SharePoint fields being blank on the test record at the time of that run.

The test record was then populated with the resolution and resolved date, and the email body was formatted with cleaner spacing and labels.

**Lesson:** Validate both workflow execution and test data. A successful flow run can still produce incomplete output when source fields are empty.

## 6. Final Architecture

```text
                    SharePoint Tickets
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
 New ticket          Critical ticket      Completed ticket
     │                    │                    │
     ▼                    ▼                    ▼
Notify Assigned      Escalate Assigned    Notify Requester

                    Scheduled every hour
                           │
                           ▼
                   Get eligible tickets
                           │
                           ▼
                      Apply to each
                           │
                           ▼
                       SLA check
                           │
                    Overdue ticket?
                       /       \
                     Yes        No
                      │
                      ▼
                 Send reminder
                      │
                      ▼
            SLA Reminder Sent = Yes
```

## 7. Final Scope Decision

The project originally considered additional Power Platform components. For the portfolio version, the implementation stops at **SharePoint + Power Automate**.

Power Apps, Teams, and Power BI are intentionally out of scope so the repository can demonstrate Power Automate workflow design, runtime troubleshooting, scheduled processing, conditional branching, and end-to-end validation without unnecessary scope expansion.