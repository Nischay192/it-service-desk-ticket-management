# IT Service Desk Ticket Management & Automation

A Microsoft **Power Automate + SharePoint** solution that automates a realistic IT support workflow from ticket creation through resolution.

I built this project to demonstrate practical workflow automation rather than simply creating a SharePoint list. The solution handles technician notifications, priority-based escalation, SLA monitoring, and requester communication, with testing and troubleshooting evidence throughout.

## Architecture

```text
                         SharePoint Online
                       ┌───────────────────┐
                       │   Tickets List    │
                       │  System of Record │
                       └─────────┬─────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
        Item created       Item created       Item modified
              │                  │                  │
              ▼                  ▼                  ▼
       New Ticket          Critical Check      Completed Check
       Notification              │                  │
              │             Critical?              │
              ▼                  │                  ▼
       Assigned Tech             ▼          Requester Notice
                            Escalation
                            Assigned Tech

                         Scheduled Flow
                           Every hour
                              │
                              ▼
                     Get eligible tickets
                              │
                              ▼
                         SLA evaluation
                              │
                    Overdue and unresolved?
                         /           \
                       Yes            No
                        │
                        ▼
                  Send reminder
                        │
                        ▼
              Mark reminder as sent
```

The **SharePoint Tickets list** acts as the system of record, while Power Automate provides the event-driven and scheduled workflow logic. Outlook delivers the user-facing notifications.

## What It Does

**1. New Ticket Notification**  
When a ticket is created, Power Automate notifies the assigned technician with the information needed to begin work.

**2. Critical Ticket Escalation**  
Critical-priority tickets are detected automatically and generate a high-importance escalation to the assigned technician.

**3. SLA Overdue Reminder**  
An hourly scheduled flow identifies unresolved tickets whose SLA deadline has passed, sends a reminder, and records that the reminder has been sent so the same ticket is not repeatedly notified.

**4. Ticket Resolution Notification**  
When a ticket reaches `Completed`, the requester receives an automated message containing the resolution details and resolved date.

## How to Test

The workflows were validated using synthetic SharePoint tickets representing normal, critical, overdue, and completed scenarios.

| Scenario | Expected result | Result |
|---|---|---|
| Standard ticket created | Assigned technician receives notification | Passed |
| Critical ticket created | High-importance escalation is sent | Passed |
| Non-critical ticket created | Critical branch is skipped | Passed |
| Overdue unresolved ticket | Reminder is sent and reminder state is updated | Passed |
| Completed ticket | Requester receives resolution notification | Passed |

Power Automate run history was used to verify trigger execution, condition results, email actions, and SharePoint updates rather than relying only on the final email output.

## Running the Workflows

The solution runs directly in Microsoft Power Automate using the SharePoint `Tickets` list as its data source.

- **Flows 1, 2, and 4** are event-driven and respond to SharePoint item creation or modification.
- **Flow 3** runs every hour to evaluate SLA status.
- Outlook is used for automated technician and requester notifications.

The same test cases can be recreated by creating or modifying records in the SharePoint list and reviewing the resulting Power Automate run history.

## Automation Flows

| Flow | Trigger | Core logic | Outcome |
|---|---|---|---|
| New Ticket Notification | Item created | Build notification from ticket data | Technician notified |
| Critical Ticket Escalation | Item created | Check `Priority = Critical` | High-importance escalation |
| SLA Overdue Reminder | Every hour | Filter eligible tickets → compare SLA → update reminder state | Overdue ticket escalated once |
| Ticket Resolution Notification | Item created or modified | Check `Status = Completed` | Requester receives resolution |

## Workflow Logic

### SLA filtering

The scheduled flow filters records before processing them:

```text
SLADueDate ne null and Status ne 'Completed' and SLAReminderSent eq 0
```

This keeps blank SLA dates, completed tickets, and already-reminded tickets out of the processing loop.

### SLA date comparison

```text
ticks(item()?['SLADueDate']) <= ticks(utcNow())
```

Using timestamp values provides a consistent comparison between the stored deadline and the current UTC time.

### SharePoint Choice-field handling

The resolution workflow ultimately used the runtime field structure returned by SharePoint:

```text
triggerOutputs()?['body/Status/Value']
```

The expression was identified by inspecting the trigger output in run history after the visual condition reference failed.

## Project Structure

```text
it-service-desk-ticket-management/
├── README.md
├── .gitignore
└── docs/
    ├── architecture.md
    ├── implementation-journey.md
    ├── power-automate-flows.md
    ├── sharepoint-schema.md
    └── testing-and-validation.md
```

## Anatomy of a Workflow

The SLA workflow is representative of the project's more advanced automation pattern:

```text
Recurrence
    ↓
Get eligible SharePoint items
    ↓
Apply to each ticket
    ↓
Is SLA deadline <= current time?
    ↓
   Yes
    ↓
Send overdue notification
    ↓
Set SLA Reminder Sent = Yes
```

The important part is not the number of actions. The workflow demonstrates **scheduled processing, data filtering, conditional evaluation, date/time expressions, state tracking, and controlled notifications**.

## Why I Built It This Way

A few design decisions demonstrate the reasoning behind the implementation:

**Event-driven triggers for immediate actions.** Ticket creation and completion are events, so SharePoint triggers allow the workflow to respond when the state changes instead of relying on manual checks.

**Scheduled processing for SLA monitoring.** An SLA is time-based, so the overdue check runs on a recurring schedule rather than waiting for another ticket event.

**Filter before processing.** The SLA flow excludes invalid and ineligible records before the loop. This reduces unnecessary processing and prevents null values from reaching the date comparison.

**State tracking for recurring automation.** `SLA Reminder Sent` records whether an overdue notification has already been issued, preventing repeated reminders on subsequent hourly runs.

**Runtime-driven troubleshooting.** When a SharePoint Choice field behaved differently from the visual condition reference, I inspected the actual trigger output and built the condition from the runtime structure.

**End-to-end validation.** I tested both positive and negative paths and checked the actual resulting emails and SharePoint updates, not just whether the flow showed a green status.

## Troubleshooting Highlights

The project required solving several real implementation issues:

- A renamed SharePoint `Title` field initially produced a blank Issue value in the notification email.
- The Critical escalation condition initially failed to reference the Choice-field value correctly.
- The SLA comparison failed when blank SLA dates produced a `Null` versus `String` type mismatch.
- The resolution flow initially contained a broken `?Status.Value` reference.
- The first resolution email exposed incomplete source data because Resolution and Resolved Date were blank on the test ticket.

Each issue was diagnosed from runtime behavior, corrected, and retested. The detailed troubleshooting path is documented in [Implementation Journey](docs/implementation-journey.md).

## Extending It

The design can be extended without changing the core ticket data model or replacing the existing workflows. Examples include adding additional priority rules, introducing new SLA policies, expanding notification paths, or connecting the ticket data to other Microsoft 365 services.

The current implementation deliberately focuses on the completed **SharePoint + Power Automate** workflow so the portfolio demonstrates depth in automation rather than breadth through unfinished components.

## Lessons Learned

- Built practical experience designing event-driven and scheduled Power Automate workflows.
- Strengthened understanding of SharePoint as a structured data source for workflow automation.
- Learned how Choice and Person fields behave when passed between SharePoint and Power Automate.
- Developed experience using OData filtering and Power Automate expressions for reliable scheduled processing.
- Learned to use run history and trigger output as primary debugging evidence when connector references behave unexpectedly.
- Strengthened understanding of state-based automation and preventing duplicate notifications.
- Improved testing discipline by validating both successful and negative workflow paths.

## Documentation

- [Power Automate Flows](docs/power-automate-flows.md) — detailed workflow implementation
- [Implementation Journey](docs/implementation-journey.md) — problems, fixes, and technical reasoning
- [SharePoint Data Model](docs/sharepoint-schema.md) — data model and design rationale
- [Testing & Validation](docs/testing-and-validation.md) — test scenarios and results

## Privacy

Use synthetic test data and never publish passwords, API keys, authentication tokens, connection references, or private tenant information.
