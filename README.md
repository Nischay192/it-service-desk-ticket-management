# IT Service Desk Ticket Management & Automation

A portfolio project demonstrating how **Microsoft SharePoint** and **Power Automate** can automate common IT service desk workflows.

The goal was not simply to create a ticket list. I designed the solution around a realistic support lifecycle: **ticket creation → technician notification → priority escalation → SLA monitoring → resolution communication**.

## What I Built

| Workflow | What it does |
|---|---|
| **New Ticket Notification** | Notifies the assigned technician when a ticket is created. |
| **Critical Ticket Escalation** | Detects critical-priority tickets and sends an immediate high-importance alert. |
| **SLA Overdue Reminder** | Runs hourly, identifies eligible overdue tickets, sends a reminder, and records that the reminder was sent. |
| **Ticket Resolution Notification** | Detects completed tickets and sends the requester the resolution details. |

**Result:** All four workflows were tested end-to-end using sample tickets. Trigger execution, conditional branches, email delivery, and SharePoint updates were validated through Power Automate run history and the SharePoint data source.

## Skills Demonstrated

**Power Automate**  
Event-driven triggers · Conditions · Scheduled flows · Apply to each · Dynamic content · Expressions · Run-history troubleshooting

**SharePoint**  
List design · Choice and Person fields · Views · Ticket state management · Data used as a workflow system of record

**Workflow Logic**  
Priority-based routing · SLA monitoring · Null-data handling · State flags · Conditional notifications · Duplicate-reminder prevention

**Microsoft 365**  
Outlook email automation · SharePoint integration

## Solution Architecture

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
              ▼                  │              Completed?
       Assigned Tech             ▼                  │
                            Escalation              ▼
                            Assigned Tech      Requester Notice

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

## Technical Decisions

### Event-driven automation for immediate actions

Ticket creation and status changes use SharePoint triggers so notifications happen as part of the ticket lifecycle rather than through manual checks.

### Scheduled processing for SLA monitoring

SLA enforcement uses an hourly recurrence because the deadline is time-based rather than event-based. The flow filters eligible records before processing them.

```text
SLADueDate ne null and Status ne 'Completed' and SLAReminderSent eq 0
```

This prevents blank dates, completed tickets, and already-reminded tickets from entering the SLA comparison.

### Reliable date/time comparison

The SLA condition compares timestamps with `ticks()`:

```text
ticks(item()?['SLADueDate']) <= ticks(utcNow())
```

This provides a consistent numeric comparison between the stored SLA deadline and the current UTC time.

### Runtime-driven troubleshooting

When a SharePoint Choice field did not behave as expected through the visual condition builder, I inspected the trigger output in run history instead of continuing to guess at the field reference.

The working Status expression was:

```text
triggerOutputs()?['body/Status/Value']
```

The key troubleshooting principle was to use **runtime data to determine what the connector actually returned**.

## Testing & Results

The solution was validated against both expected and negative paths.

| Test | Result |
|---|---|
| Standard ticket creation | Passed |
| Critical ticket creation | Passed |
| Non-critical ticket skips escalation | Passed |
| Overdue unresolved ticket | Passed |
| Completed ticket sends requester notification | Passed |
| SLA reminder flag prevents repeat processing | Validated |

Testing exposed real implementation issues, including SharePoint field references, Choice-field values, null SLA dates, and incomplete source data. Each issue was diagnosed and corrected before final validation.

## Engineering Lessons

- SharePoint display names do not always match the underlying field reference exposed to Power Automate.
- Choice fields should be validated against their actual runtime structure.
- Scheduled automations should handle null and ineligible records before performing comparisons.
- State-tracking fields can prevent duplicate notifications in recurring workflows.
- A successful flow run does not guarantee complete business output; source data must also be validated.
- Power Automate run history is a practical debugging tool, not just a success/failure log.

## Project Structure

```text
.
├── README.md
├── .gitignore
└── docs/
    ├── architecture.md
    ├── implementation-journey.md
    ├── power-automate-flows.md
    └── testing-and-validation.md
```

## Documentation

- [Architecture](docs/architecture.md) — system design and data flow
- [Power Automate Flows](docs/power-automate-flows.md) — workflow logic and technical implementation
- [Implementation Journey](docs/implementation-journey.md) — key troubleshooting decisions and lessons
- [Testing & Validation](docs/testing-and-validation.md) — scenarios and results

## Privacy

Use synthetic test data and never publish passwords, API keys, authentication tokens, connection references, or private tenant information.
