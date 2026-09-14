# Power Automate Flows

## 1. New Ticket Notification

**Trigger:** SharePoint — When an item is created

**Logic:** A new ticket is created in the Tickets list, and an email is sent to the assigned technician.

**Notification includes:**
- Issue
- Description
- Priority
- Status
- Category
- Requester

## 2. Critical Ticket Escalation

**Trigger:** SharePoint — When an item is created

**Condition:** Priority equals `Critical`

**Action:** Send a high-importance email to the assigned technician requesting immediate attention.

The workflow was tested with a critical ticket and successfully generated the expected escalation notification.

## 3. SLA Overdue Reminder

**Trigger:** Recurrence — every 1 hour

**Current structure:**

`Recurrence → Get items → Apply to each → Condition`

The condition evaluates whether the SLA due date has passed and whether the ticket remains unresolved. A reminder flag prevents repeated notifications for the same ticket.

**Planned actions when the condition is true:**

1. Send an overdue SLA email to the assigned technician.
2. Update `SLA Reminder Sent` to Yes.

## Design Principles

- Use SharePoint as the centralized ticket data source.
- Use automated notifications to reduce manual follow-up.
- Use priority-based escalation for critical incidents.
- Use scheduled SLA monitoring to identify overdue work.
- Use a reminder flag to avoid duplicate SLA notifications.
