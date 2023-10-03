# Release Notes 

## General

Requires **OpCon 20.7 STS** or greater due to OpCon-API requirements.

## Release 21.4

### New Features

**CONNUTIL-614**
                    Adjusted actions when a ticket exists and re-open of a ticket is allowed:
					- incident state New, In-Progress or On-Hold, no state change and log files are appended to the existing incident ticket.
					- incident state Resolved, state changed to In-Progress and log files appended to the existing incident ticket.
					- incident state Cancelled or Closed, new incident created and log files appended to the new incident ticket.

### Fixes

**CONNUTIL-6i5**
                    Fixed a problem where updating or inserting ticket information into the OpCon job record always returned a false value.
					
## Release 21.3

### New Features

**CONNUTIL-605**
                    Modify ServiceNow to accept dynamic value for query field name when retrieving the application id from the cmdb.
					
					Changed cmd url definition in factory to include replacement value for application query attribute name.   

					Added variable cmdbQueryFieldAttributeName to variables section of template.

					"variables": {
						"incidentReopenState": 1,
						"cmdbQueryFieldAttributeName":"u_trigramme"
					}

					value should be set to the required query field name (default value is **u_trigramme**).

### Migration Considerations
After installation add "cmdbQueryFieldAttributeName":"u_trigramme" to variables section of the json template.
Modify u_trigramme to required value or retain the default value.

## Release 21.2.1

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

