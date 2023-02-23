# Release Notes ServiceNow 21.2.1

## General

Requires **OpCon 20.7 STS** or greater due to OpCon-API requirements.

### New Features

**CONNUTIL-594**
                    Implement configurable incident state when re-opening ServiceNow incident when restarted failed task fails.
					Added variables section to template.

					"variables": {
						"incidentReopenState": 1
					}					  	    

					value should be set to the state required by your organization when reopening the incident (1 is New, 2 is In-Progress)
### Fixes

**CONNUTIL-585**
                    Files not attached when restarted failed job fails again.
**CONNUTIL-586**
                    When updating information for a restarted failed task that fails again, only update the description.	
**CONNUTIL-587**
                    Failed task with exiting ticket in a 'Closed' or 'Cancelled' tries to insert new ticket number into the job information instead of updating the job information.	

### Migration Considerations
The template used by the connector must be updated to include the new variable section.

