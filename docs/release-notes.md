# Release Notes ServiceNow 21.1.0

## General

Requires **OpCon 20.7 STS** or greater due to OpCon-API requirements.

### New Features

**CONNUTIL-562**
                    Implement support for extracting the incident application ID value from CMDB.
					Includes additions in the template to provide the functionality
					2 new rules added
					"rules": {
					   "applicationIdRetrievalFromCmdb": false,
					   "extractAppIdFromScheduleName": false
					},
					1 new URL added
					  "urls": 
						{
						  "name": "cmdb",
						  "value": "api/now/cmdb"
						}
					  
					new attribute to define the location of the string to extract from the schedule name which is used to find the application ID in the CMDB
					"appIdLocation": {
					   "startLocation": 2,
					   "numberOfChars": 3
				    },
					new parseNotation field indicating how to retrieve the application ID from the returned CMDB data
					"parseNotations": 
					   {
					     "name": "cmdbsysid",
					     "attribute": "sys_id",
					     "notation": "result.sys_id"
					   }
				    

**CONNUTIL-568**
                    Added new rule to extract the application id from the first tag in the list of tags associated with the job.
					Includes additions in the template to provide the functionality
					1 new rule added
					"rules": {
					   "extractAppIdFromTagName": false
					}
					
**CONNUTIL-569**
                    Added new rule to extract an xml formatted string from the documentation associated with the job and appending this information to the description attribute submitted to ServiceNow.
					Includes additions in the template to provide the functionality
					new rule added
					"rules": {
					   "extractXmlContentFromJobDescription": false
					},
					new attribute to define the name of the xml tag in the job documentation
				    "jobDescriptionXmlTag": {
					  "xmlTag": "xmltagname"
				    }
**CONNUTIL-571**
                    Implemented a new rule to support specific routing requirements based on if failed job instructions are present in the job documentation field.
					Includes additions in the template to provide the functionality
					new rule added
					"rules": {
					   "routingByDocumentationContent": false
					},
					new structure to define the routing information
				    "documentationRouting": {
					  "routingAttribute": "",
                      "workingHoursWithDoc": "",
                      "workingHoursWithoutDoc": "",
                      "nonWorkingHoursWithDoc": "",
                      "nonWorkingHoursWithoutDoc": ""
                    }
					
**Migration Notes**
					When upgrading to this version of the connector, the new template format must be used even if these new features are not being used (set the rules to false).
### Fixes

**CONNUTIL-557**
                    When creating the OpCon Rest-API web services client, changed deserilization feature of ObjectMapper to not fail when an unknown property is encountered.


