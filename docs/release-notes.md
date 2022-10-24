# Release Notes ServiceNow 21.2.0

## General

Requires **OpCon 20.7 STS** or greater due to OpCon-API requirements.

### New Features

**CONNUTIL-581**
                    Implement new rule alwaysCreateNewTicket to ensure a new ticket is created when a task that has a ticket fails again (this is a customer requirement).
					Includes addition in the template to provide the functionality
					1 new rule added
					"rules": {
					   "extractAppIdFromScheduleName": false,
					   "alwaysCreateNewTicket": false
					},

					If this rule is enabled, a new ticket will always be created when a job already has a ticket assigned and it fails again.				    

### Fixes

### Migration Considerations
The template used by the connector must be updated to include the new rule.

