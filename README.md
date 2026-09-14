# IT Service Desk Ticket Management & Automation

A practical IT service desk automation project built with **Microsoft SharePoint** and **Microsoft Power Automate**.

The solution models a small internal IT help desk where SharePoint stores tickets and Power Automate handles technician notifications, critical-ticket escalation, SLA monitoring, and requester resolution notifications.

> **Scope:** This portfolio version intentionally focuses on SharePoint + Power Automate. Power Apps, Teams, and Power BI are out of scope for the current implementation.

## What This Project Demonstrates

- SharePoint list and view design for IT service management
- Event-driven Power Automate flows
- Conditional routing and priority-based escalation
- Scheduled SLA monitoring
- OData filtering
- Power Automate expressions for date/time comparison
- Dynamic Outlook email notifications
- Reminder-state tracking to prevent repeated SLA emails
- Runtime troubleshooting using Power Automate run history
- End-to-end workflow testing and validation

## Architecture

```text
                         SharePoint
                     ┌─────────────────┐
                     │  Tickets List   │
                     │                 │
                     │ Issue           │
                     │ Description     │
                     │ Priority        │
                     │ Status          │
                     │ Assigned To     │
                     │ Requester       │
                     │ Resolution      │
                     │ Resolved Date  │
                     │ SLA Due Date    │
                     │ SLA Reminder    │
                     └────────┬────────┘
                              │
              ┌───────────────┼────────────────┐
              │               │                │
              ▼               ▼                ▼
       New Ticket       Critical Ticket   Completed Ticket
       Notification     Escalation        Resolution Notice
              │               │                │
              ▼               ▼                ▼
       Assigned tech     Assigned tech      Requester

                     Scheduled SLA Flow
                           │
                      Every 1 hour
                           ▼
                     Get eligible items
                           ▼
                      Apply to each
                           ▼
                        SLA check
                           ▼
                   Send reminder + flag
```

## Technology Stack

| Technology | Role |
|---|---|
| Microsoft SharePoint | Ticket data store and operational views |
| Microsoft Power Automate | Workflow orchestration and automation |
| Microsoft 365 Outlook | Automated email notifications |
| OData | SharePoint item filtering |
| Power Automate expressions | Date/time and Choice-field logic |

## SharePoint Data Model

The `Tickets` list contains:

- Issue
- Quick steps
- Issue description
- Priority
- Status
- Assigned to
- Associated files
- Related issue
- Category
- Requester
- Resolution
- Resolved Date
- SLA Due Date
- SLA Reminder Sent

Views include **Open Tickets**, **My Tickets**, and **Critical Tickets**.

![SharePoint Tickets list](docs/screenshots/01-sharepoint-tickets-list.png)

See [SharePoint Data Model](docs/sharepoint-schema.md).

## Power Automate Flows

### 1. New Ticket Notification

**Trigger:** SharePoint — When an item is created

Emails the assigned technician when a new ticket is created. The notification contains issue, description, priority, status, category, and requester information.

**Status:** Complete and tested.

### 2. Critical Ticket Escalation

**Trigger:** SharePoint — When an item is created

**Condition:** `Priority Value = Critical`

Critical tickets generate a high-importance email to the assigned technician.

**Status:** Complete and tested.

### 3. SLA Overdue Reminder

**Trigger:** Recurrence — every 1 hour

The flow retrieves eligible tickets, processes them with `Apply to each`, checks whether the SLA deadline has passed, sends an overdue reminder, and sets `SLA Reminder Sent = Yes`.

OData filter:

```text
SLADueDate ne null and Status ne 'Completed' and SLAReminderSent eq 0
```

Date comparison:

```text
ticks(item()?['SLADueDate']) <= ticks(utcNow())
```

**Status:** Complete and successfully tested.

![Successful SLA run](docs/screenshots/05-sla-overdue-success.png)

### 4. Ticket Resolution Notification

**Trigger:** SharePoint — When an item is created or modified

**Condition:** Status equals `Completed`

**Action:** Send an email to the Requester containing the issue, description, category, priority, resolution, and resolved date.

**Status:** Complete and successfully tested.

![Successful resolution flow](docs/screenshots/07-resolution-flow-success.png)

## Testing & Validation

Sample test tickets were used to validate the different paths:

- `Test - Wi-Fi Connection`
- `Test - Password Reset`
- `Test - Critical Server Issue`
- `Test - Critical Escalation 2`
- `Test - SLA Overdue`

| Scenario | Expected result |
|---|---|
| New ticket created | Assigned technician receives notification |
| Critical ticket created | High-importance escalation is sent |
| Non-critical ticket created | Critical branch does not execute |
| Overdue unresolved ticket | SLA reminder is sent and flag is updated |
| Completed ticket | Requester receives resolution notification |

The project was validated through Power Automate run history, email delivery, and SharePoint record updates.

## Implementation Journey

The project was built incrementally and required troubleshooting several real Power Automate issues:

- Renamed SharePoint `Title` field initially produced a blank Issue value in email output.
- Critical Choice-field condition initially did not route correctly.
- SLA comparison failed when blank SLA Due Dates produced a `Null` versus `String` type mismatch.
- The resolution flow initially displayed a broken `?Status.Value` reference.
- The first resolution email exposed incomplete test data because Resolution and Resolved Date were blank.

Each issue was diagnosed from runtime behavior and corrected. The complete troubleshooting journey is documented in [Implementation Journey](docs/implementation-journey.md).

## Project Status

| Component | Status |
|---|---|
| SharePoint IT Help Desk site | Complete |
| Tickets list/data model | Complete |
| Open Tickets view | Complete |
| My Tickets view | Complete |
| Critical Tickets view | Complete |
| New Ticket Notification | Complete |
| Critical Ticket Escalation | Complete |
| SLA Overdue Reminder | Complete |
| Ticket Resolution Notification | Complete |
| Documentation | Complete |
| Power Apps | Out of scope |
| Teams | Out of scope |
| Power BI | Out of scope |

## Repository Structure

```text
.
├── README.md
├── .gitignore
└── docs/
    ├── architecture.md
    ├── implementation-journey.md
    ├── power-automate-flows.md
    ├── screenshots.md
    ├── sharepoint-schema.md
    ├── testing-and-validation.md
    └── screenshots/
```

## Security & Privacy

This repository should contain documentation and sanitized screenshots only.

Do not publish passwords, API keys, authentication tokens, connection references, personal email addresses, or sensitive tenant information. Use synthetic test data and crop/redact private information from screenshots.
