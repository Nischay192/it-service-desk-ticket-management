# Screenshot Plan

The repository should use screenshots as evidence of implementation, not as a substitute for documentation.

## Add these screenshots

| Screenshot | Where to place it | Purpose |
|---|---|---|
| SharePoint Tickets list | README and SharePoint Data Model | Shows the ticket schema and sample data |
| Flow 1 canvas | Power Automate Flows, Flow 1 | Shows trigger and notification action |
| Flow 2 canvas | Power Automate Flows, Flow 2 | Shows Critical condition and escalation |
| Flow 3 successful run | README or Power Automate Flows, Flow 3 | Proves the complete SLA path succeeded |
| Flow 4 condition | Power Automate Flows, Flow 4 | Shows Status expression and Completed comparison |
| Flow 4 successful run | README or Testing and Validation | Proves the resolution path succeeded |
| Status trigger output | Implementation Journey, Flow 4 troubleshooting | Shows how the broken Choice-field reference was diagnosed |
| Initial resolution email | Implementation Journey, blank-field roadblock | Shows the first output and why it was incomplete |
| Final resolution email | Testing and Validation, Flow 4 | Shows the completed user-facing result |

## Existing screenshots

The local project package prepared for this repository contains sanitized screenshots for the SharePoint list, SLA success, Flow 4 condition, Flow 4 success, Status output, and the initial resolution email.

## Screenshots still to capture

1. Flow 1 canvas.
2. Flow 2 canvas.
3. Final Flow 4 email with Resolution and Resolved Date populated.

Save the final email as:

`docs/screenshots/10-final-resolution-email.png`

## Privacy Checklist

Before publishing:

- Crop or redact account names.
- Remove personal email addresses.
- Avoid exposing private tenant URLs when unnecessary.
- Never show passwords, tokens, API keys, or connection secrets.
- Use synthetic test tickets.
- Check every screenshot before committing it to a public repository.
