# SharePoint Data Model

SharePoint acts as the **system of record** for the service desk. The data model was designed around the information Power Automate needs to route, monitor, and close a ticket.

## Core Ticket Data

- **Issue & description** — identify and explain the support request.
- **Priority & status** — drive routing, escalation, and lifecycle decisions.
- **Assigned to & requester** — identify who owns the ticket and who needs the outcome.
- **Category** — provides consistent classification for support requests.
- **Resolution & resolved date** — capture the final outcome communicated to the requester.

## SLA & Automation State

- **SLA Due Date** — provides the deadline used by the scheduled SLA workflow.
- **SLA Reminder Sent** — stores workflow state so an overdue ticket is not repeatedly notified on every hourly run.

## Supporting Ticket Context

- **Quick steps** — initial troubleshooting guidance.
- **Associated files** — supporting evidence or attachments.
- **Related issue** — reference to another ticket when requests are connected.

## Operational Views

| View | Purpose |
|---|---|
| **Open Tickets** | Focuses technicians on active work. |
| **My Tickets** | Filters work assigned to the current technician. |
| **Critical Tickets** | Provides a focused view of high-priority incidents. |

## Design Rationale

The important design choice was to make the SharePoint list more than a storage table: its fields represent **workflow state and business rules** used by Power Automate. Choice fields provide consistent values for conditions, Person fields support targeted notifications, and the SLA reminder flag maintains state between scheduled runs.
