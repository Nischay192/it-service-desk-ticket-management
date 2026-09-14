# SharePoint Schema

## Site

**Site:** IT Help Desk

## Tickets List

The `Tickets` list is the primary data store for service desk requests.

| Column | Purpose |
|---|---|
| Issue | Ticket title / issue name |
| Issue description | Detailed description of the problem |
| Quick steps | Initial troubleshooting steps |
| Priority | Critical, High, Normal, Low |
| Status | New, In progress, Completed, Blocked, Duplicate |
| Assigned to | Technician responsible for the ticket |
| Requester | Employee who submitted the request |
| Category | Service category such as Network or Account Access |
| Associated files | Supporting attachments/files |
| Related issue | Link/reference to a related ticket |
| Resolution | Details of the solution provided |
| Resolved Date | Date the ticket was resolved |
| SLA Due Date | Deadline used for SLA monitoring |
| SLA Reminder Sent | Tracks whether the overdue reminder was sent |

## Views

- **Open Tickets** — active service desk tickets
- **My Tickets** — tickets assigned to the current technician
- **Critical Tickets** — tickets where Priority is Critical

## Design Notes

The SharePoint list provides the backend data layer for the Power Platform solution. Person/Group fields are used for technician assignment and requester identification, while choice fields provide consistent ticket priority, status, and category values.
