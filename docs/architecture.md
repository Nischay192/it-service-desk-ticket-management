# Architecture

## High-Level Architecture

```text
SharePoint Online
      |
      | Tickets list
      v
Power Automate
      |
      +--> New Ticket Notification --> Assigned technician
      |
      +--> Critical Ticket Escalation --> Assigned technician
      |
      +--> SLA Overdue Reminder --> Assigned technician
      |
      +--> Resolution Notification --> Requester
      |
      v
Microsoft Outlook
```

## Event-Driven Flows

Flows 1, 2, and 4 respond to SharePoint item events:

`SharePoint event -> Power Automate trigger -> processing/condition -> Outlook email`

## Scheduled Flow

Flow 3 runs every hour:

`Recurrence -> Get eligible tickets -> Apply to each -> SLA check -> email -> update ticket`

## Data Flow

```text
Requester
   |
   v
SharePoint Ticket
   |
   +--> Assigned technician notification
   +--> Critical escalation
   +--> SLA monitoring
   +--> Requester resolution notification
```

## Key Design Decisions

- SharePoint remains the system of record for ticket state.
- Power Automate handles workflow logic and notifications.
- Outlook provides user-facing email notifications.
- SLA monitoring filters records before processing.
- `SLA Reminder Sent` prevents repeated overdue reminders.