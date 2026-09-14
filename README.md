# IT Service Desk Ticket Management & Automation

A portfolio project demonstrating an IT help desk solution built with Microsoft SharePoint, Power Automate, and Power Apps. The solution is designed to support ticket submission, assignment, notifications, priority-based escalation, SLA monitoring, and technician workflows.

## Current Architecture

- **SharePoint** — ticket data store and IT Help Desk site
- **Power Automate** — notification, escalation, and SLA automation
- **Power Apps** — planned employee ticket submission and technician interface
- **Microsoft Teams** — planned support-team integration
- **Power BI** — optional reporting and service metrics dashboard

## Implemented So Far

### SharePoint IT Help Desk

Created an **IT Help Desk** SharePoint site with a **Tickets** list containing fields for:

- Issue
- Issue description
- Quick steps
- Priority
- Status
- Assigned to
- Requester
- Category
- Associated files
- Related issue
- Resolution
- Resolved Date
- SLA Due Date
- SLA Reminder Sent

Configured views include Open Tickets, My Tickets, and Critical Tickets.

### Power Automate Flows

#### 1. New Ticket Notification

**Trigger:** SharePoint — When an item is created

Automatically emails the assigned technician when a new ticket is created. The notification includes the issue, description, priority, status, category, and requester.

#### 2. Critical Ticket Escalation

**Trigger:** SharePoint — When an item is created

Checks whether the ticket priority is **Critical**. Critical tickets generate a high-importance escalation email requiring immediate attention.

#### 3. SLA Overdue Reminder

**Trigger:** Scheduled recurrence — every hour

Retrieves tickets and evaluates SLA due dates so overdue unresolved tickets can receive a reminder. The `SLA Reminder Sent` field is used to prevent repeated reminders for the same ticket.

This flow is currently under construction.

## Planned Components

- Employee-facing Power Apps ticket submission form
- Technician dashboard in Power Apps
- Additional status and resolution notifications
- Microsoft Teams support notifications and collaboration
- Optional Power BI help desk analytics dashboard
- Project documentation and screenshots

## Technologies

**Microsoft Power Platform:** Power Apps, Power Automate, SharePoint

**Microsoft 365:** Microsoft Teams, Outlook

**Planned Analytics:** Power BI

## Project Status

The SharePoint data layer and core Power Automate notification/escalation workflows are implemented. SLA monitoring is in progress, followed by the Power Apps user interface and additional integrations.

## Security & Portfolio Notes

This repository contains documentation and configuration descriptions only. No tenant URLs, credentials, connection details, personal information, or secrets are included.
