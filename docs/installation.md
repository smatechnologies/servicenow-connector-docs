# Installation

The ServiceNow Connector installation consists of multiple steps that are required to complete the installation successfully. 

The connector requires a SMA OpCon Windows Agent to provide the connection between Notification Manager in the OpCon System and the ServiceNow Connector software. 
It requires the OpCon Rest-API to extract the unique id of the task in the daily tables, the updating of the incident number into the task in the daily tables, the retrieval of the job log and the updating of the task status by the outbound Rest messages of the SMA OpCon ServiceNow Application.

## Supported Software Levels
The following software levels are required to implement this version (21.x.x) of the ServiceNow Connector.

- OpCon Release 21.0 or higher.
- Embedded Java OpenJDK 11 (part of installation).
- OpCon Rest API Configured to use TLS.
- OpCon Windows Agent to provide link to ServiceNow Connector.
- OpCon Notification Manager.
- An ServiceNow implementation that supports the Rest API.

## Installation
The installation process consists of the following steps:

- OpCon Windows Agent Installation.
- ServiceNow Connector Installation.
- ServiceNow Connector Configuration.
- Template creation. 
- OpCon Notification Manager Definition.
- ServiceNow SMA OpCon ServiceNow Application Installation.
 
### OpCon Windows Agent Installation
The ServiceNow Connector requires the installation of a Windows Agent on the same system as the ServiceNow Connector.
Either use an existing Windows Agent or complete the installation of the Windows Agent.

### ServiceNow Connector Installation
The ServiceNow connector can be installed on any Windows Server as long as there is an OpCon MSLSAM Agent installed on that Windows Server.

Copy the downloaded install file SMAServiceNowConnector-win.zip and extract it into a temp directory (c:\temp). Extract the information including sub-directories into the required directory.

After the extraction, the root installation directory contains the connector executable (SMASnowConnector.exe), the encryption software executable (EncryptValue.exe), the Connector.config file and four directories, java, joblogs, templates and log. The java directory contains the java software required to execute the connector (OpenJDK 11), the joblogs directory is used to temporarily contain job logs extracted from the OpCon system, the log directory contains the connector log files and the templates directory contains the ServiceNow template files. The templates directory that includes an initial basic.json template file that can be modified to provide the required template.

#### Create logfiles
Create a directory /logfiles off the installation directory that will be used as temporary storage when attaching job log files to the incident tickets.

#### Create $SCHEDULE DATE-SNOW Global Property
Create the special **$SCHEDULE DATE-SNOW** global property that contains the schedule date in the yyyy-MM-dd format from the standard $SCHEDULE DATE property. Set the value to **yyyy-MM-dd**.

### ServiceNow Connector Configuration
The configuration of the ServiceNow Connector requires setting the OpCon connection information for the OpCon system associated with the connector.

All user and password values placed in the configuration and template files must be encrypted using the EncryptValue.exe utility provided with the connector. 

#### EncryptValue Utility
The EncryptValue utility uses standard 64 bit encryption.

Supports a -v argument and displays the encrypted value

On Windows, example on how to encrypt the value "abcdefg":

```
EncryptValue.exe -v abcdefg

```

#### Connector.config configuration
Configure the Connector.config file in the installation directory setting the required information.
The Connector.config contains the following values

Property Name | Value
--------- | -----------
**[GENERAL]**                       | header
**LOG_FILES_DIRECTORY**             | The name of the directory where the retrieved log files are stored. After successful attachment to the ServiceNow Incident, the log file is deleted. The name is a sub directory of the installation directory (default logfiles).
**TEMPLATES_DIRECTORY**             | The name of the directory where the template definitions are stored. The name is a sub directory of the installation directory (default templates).
**DEBUG**                           | The Connector supports a debug mode which can be enabled by setting the value to ON. The connector should be run with DEBUG disabled (OFF) and enabled (ON) when requested to capture an error condition. Value either ON or OFF (default OFF).
**[PROXY CONNECTION]**              | header - Used to define the full URL of a proxy server if required
**USE_PROXY**                       | Indicates if this connector should use a Proxy Server connection. Values are True or False (default False).
**PROXY_ADDRESS**                   | The address of the Proxy Server.
**PROXY_PORT**                      | The port of the Proxy Server.
**[OPCON API CONNECTION]**          | header - Used to define the connection to the OpCon API
**SERVER**                          | The server address of the OpCon API.
**PORT**                            | The port number used by the OpCon API server.
**USES_TLS**                        | Must be set to True.
**TOKEN**                           | An application token used for Authentication when communicating with the OpCon-API. 
**RET_SERVER**                      | The address of the OpCon Server. If using a DNS name ensure that the DNS name is visible in the public domian. This information is used by ServiceNow OpCon Application to submit report back commands to OpCon.
**RET_PORT**                        | The port of the OpCon Rest-API Server. This information is used by ServiceNow OpCon Application to submit report back commands to OpCon.

Example configuration file. 

```
[GENERAL]
LOG_FILES_DIRECTORY=logfiles
TEMPLATES_DIRECTORY=templates
DEBUG=OFF

[PROXY CONNECTION]
USE_PROXY=True
PROXY_ADDRESS=10.200.180.18
PROXY_PORT=8080

[OPCON API CONNECTION]
SERVER=50.17.60.164
PORT= 9010
USES_TLS=True
TOKEN=35f3b848-1991-40ad-be7f-9631b7da5eda
RET_SERVER=50.17.60.164
RET_PORT= 9010


```
The PROXY CONNECTION section provides the information about a proxy server should this be required. The proxy server is defined using the USE_PROXY, PROXY_ADDRESS and PROXY_PORT statements.

The OPCON API CONNECTION section provides the information about connecting to the OpCon System using the OpCon Rest-API so the job information can be retrieved and the job log can be retrieved and appended to the incident ticket using the **attachment** URL defined in the url section of the template. The SERVER statement defines the web server address and the PORT statement defines the web server port used. The USES_TLS statement indicates if the connection uses TLS. The TOKEN statement contains an application token (see OpCon Rest-API documentation on how to generate an application token). The RET_SERVER and RET_PORT values are used when implementing the capability for ServiceNow to report back to OpCon. These two values are used to generate the OpCon Rest-API address that is sent to ServiceNow.

#### Templates
Templates provide information about the definitions that are submitted to ServiceNow as part of the request. As it is possible to submit a request to an intermediate table instead of directly into the incident table, templates provide the capability to configure these differences.

The template includes the address of the ServiceNow instance, the credentials, the urls, any tag routing definitions, working hour definitions, the attributes to submit for working hours or non-work hours and parsing information.

A template includes the following definitions:

Attribute Name Name | Value
--------- | -----------
**address**                             | header - Defines the ServiceNow Instance address
**name**                                | The name associated with the ServiceNow instance (i.e. production, test)
**value**                               | The address of the ServiceNow Instance.
**rules**                               | header - Defines what functions are supported by the connector.
**includeJobLogAttachment**             | Indicates if the job log should be attached to the incident ticket. Value either true or false (default true).
**includeTagRouting**                   | Indicates if User defined tags should be used for incident routing purposes. Value either true or false (default false). 
**includeCorrelationId**                | Indicates if the correlation information should be included in the information submitted to EasyVista. Value either true or false (default false).
**applicationIdRetrievalFromCmdb**      | Indicates if the application ID used during incident creation must be retrieved from the ServiceNow CMDB. Value either true or false (default false). 
**allowTicketReopen**                   | Indicates if a ticket already exists for a failed task, should the ServiceNow ticket be in an appropriate state (not closed or cancelled) the ticket will be reopened. Value either true or false (default true).
**extractAppIdFromScheduleName**        | Used in conjunction with the rule **applicationIdRetrievalFromCmdb** and indicates the value required to retrieve the application ID from the CMDB must be retrieved from the schedule name of the job. This attribute is mutually exclusive with the **extractAppliIdFromTagName** attribute. Value either true or false (default false).
**extractAppIdFromTagName**             | Used in conjunction with the rule **applicationIdRetrievalFromCmdb** and  indicates the value required to retrieve the application ID from the CMDB must be retrieved from the first tag name in the tag list associated with the job. This attribute is mutually exclusive with the **extractAppliIdFromScheduleName** attribute. Value either true or false (default false).
**extractXmlContentFromJobDescription** | This rule allows the extraction of xml content from the associated job documentation. The extracted content is appended to the description attribute submitted to ServiceNow. The **jobDescriptionXmlTag** attribute contains the name of the xml tag in the associated job documentation field. Value either true or false (default false).
**routingByDocumentationContent**       | This rule implements a specific routing requirement depending if the job documentation contains instructions on what action to take should the job fail. The instructions are contained within a set of xml tags. The xml tag name is defined in the  jobDescriptionXmlTag attribute. The **documentationRouting** attribute contains the routing information to be included in the ServiceNow object.
**alwaysCreateNewTicket**               | Indicates if a new ticket must be created if the failed job already has an existing ticket assigned. Value either true or false (default false). 
**credentials**                         | header	
**user**                                | The user which has the required privileges to connect to the ServiceNow System to submit requests. The name must be encrypted using the EncryptValue.exe utility.
**password**                            | The password of the user which has the required privileges to connect to the ServiceNow System to submit requests. The password must be encrypted using the EncryptValue.exe utility .
**appIdLocation**                       | header - Used in conjunction with the rule **extractAppIdFromScheduleName**
**startLocation**                       | Indicates where in the schedule name of the failed job the string starts.
**numberOfChars**                       | Indicates the number of characters to extract.
**tagIdPrefix**                         | Contains a character string that can be used to match against the start of a tag name to determine which tag contains the indicator to use for the CMDB match.  
**jobDescriptionXmlTag**                | header - Used in conjunction with the rule **extractXmlContentFromJobDescription**
**xmlTag**                              | Is the name of the xml tag to search for in the associated job documentation.
**urls**                                | header - Defines the urls to be when submitting requests to ServiceNow. NOTE The definition consists of name, value pairs.
**name**                                | A required name, value pair. The name consists of **incident** and the value indicates the url to be used when submitted a POST or PUT function to ServiceNow.
**value**                               | Defines the url value to use when creating or updating an incident (POST / PUT). Default value (api/now/table/incident)
**name**                                | A required name, value pair. The name consists of **getincident** and the value indicates the url to be used when submitting a GET function to ServiceNow.
**value**                               | Defines the url value to use when retrieving an existing incident (GET). Default value (api/now/table/incident)
**name**                                | An optional name, value pair. The name consists of **attachment** and the value indicates the url to be used when attaching the OpCon job log to the ticket. 
**value**                               | Defines the url to use when attaching the OpCon job log to the ticket. This attribute is required if the rule **includeJobLogAttachment** is set to True. Default value (api/now/table/incident).
**name**                                | An optional name, value pair. The name consists of **cmdb** and the value indicates the url to be used when retrieving the application id from the CMDB. 
**value**                               | Defines the url to use when retrieving the application id from the CMDB. This attribute is required if the rules  **applicationIdRetrievalFromCmdb** and **extractAppIdFromScheduleName** are set to True. Default value (api/now/cmdb).
**tags**                                | header - Defines information if OpCon User defined Tags are to be used to include attributes in the ServiceNow submission. This is enabled if the rule **includeTagRouting** is set to True.
**indicator**                           | Defines what part of the tag should be used to identify the request. Supports TAG_END,  TAG_START or DEFAULT. The DEFAULT value is used if there is no TAG_END or TAG_START match and TAG Routing is enabled.
**indicatorValue**                      | The value that is matched to the OpCon User defined tag (either the end or the start).
**attribute**                           | Defines the attribute name that will be added to the JSON payload along with the value.
**value**                               | The value associated with the attribute.
**workingHours**                        | header - Defines what is working hours. Using these definitions allows the definitions of a different set of attributes for working and non-workings hours. Note Consists of start,stop pairs.
**monday**	                            | header - Defines the start and stop hours for Monday.
**start**                               | The start time. Value consists of four digits (HHMM).
**stop**                                | The start time. Value consists of four digits (HHMM)
**tuesday**	                            | header - Defines the start and stop hours for Tuesday.
**start**                               | The start time. Value consists of four digits (HHMM).
**stop**                                | The start time. Value consists of four digits (HHMM)
**wednesday**                           | header - Defines the start and stop hours for Wednesday.
**start**                               | The start time. Value consists of four digits (HHMM).
**stop**                                | The start time. Value consists of four digits (HHMM)
**thursday**                            | header - Defines the start and stop hours for Thursday.
**start**                               | The start time. Value consists of four digits (HHMM).
**stop**                                | The start time. Value consists of four digits (HHMM)
**friday**	                            | header - Defines the start and stop hours for Friday.
**start**                               | The start time. Value consists of four digits (HHMM).
**stop**                                | The start time. Value consists of four digits (HHMM)
**saturday**                            | header - Defines the start and stop hours for Saturday.
**start**                               | The start time. Value consists of four digits (HHMM).
**stop**                                | The start time. Value consists of four digits (HHMM)
**sunday**	                            | header - Defines the start and stop hours for Sunday.
**start**                               | The start time. Value consists of four digits (HHMM).
**stop**                                | The start time. Value consists of four digits (HHMM)
**attributes**                          | header - Defines the attributes that will be submitted during working and non-working hours. NOTE Consists of name value pairs
**name**                                | A required name, value pair. The name consists of **short_description** and the value contains the text to be added.
**value**                               | Default value (OpCon Task Failure (schedule {0} job {1} server {2} error code {3})). The value includes place holders consisting of the schedule, job and server names and the error code that are filled in by the connector. The text can be changed, but consideration should be given to the location of the place holders.
**name**                                | A required name, value pair. The name consists of **description** and the value contains the text to be added.
**value**                               | Default value (Technical data of the Opcon batch (schedule ({0}) job ({1}) server ({2}) error code ({3}) date ({4}) JOBID={5})). The value includes place holders consisting of the schedule, job and server names, the date, the error code and the unique jobid that are filled in by the connector. The text can be changed, but consideration should be given to the location of the place holders.
**name**                                | An optional name, value pair, which is required if the rule **includeCorrelationId** is set to True. The name consists of correlation_id. This attribute is used to pass information to ServiceNow so ServiceNow can use the SMA ServiceNow OpCon Application to update job information. 
**value**                               | Can be left as an empty string as the value is added by the connector.
**name**                                | An optional name, value pair, which is required if the rule **includeCorrelationId** is set to True. The name consists of correlation_display. This attribute is used to pass information to ServiceNow so ServiceNow can use the SMA ServiceNow OpCon Application to update job information. 
**value**                               | Consists of the value SMA_OPEN which indicates to ServiceNow that the incident originated from OpCon.
**state**                               | A required name, value pair. The name consists of state. This is used if a failed task already has an existing ticket to reset the ticket state if this is allowed. 
**value**                               | Can be left as an empty string as the value is added by the connector.
**workingHoursAttributes**              | header - Defines the attributes that will be submitted during working hours. NOTE Consists of name value pairs
**name**                                | The name of an attribute.
**value**                               | The value of the attribute.
**nonWorkingHoursAttributes**           | header - Defines the attributes that will be submitted during nonworking hours. NOTE Consists of name value pairs
**name**                                | The name of an attribute.
**value**                               | The value of the attribute.
**parseNotations**                      | header - These are required definitions that indicate to the connector how to extract information from the returned JSON payload. The connector uses JSONPath formats to find the required values in the returned data. NOTE consists of name, attribute and notation values
**name**                                | A required parseNotation that indicates where to extract the incident **number** in the returned JSON payload following a GET request. Value getincidentnumber.
**attribute**                           | Indicates which attribute to extract the information from. Value **number**.
**notation**                            | Consists of the JSONPath notation indicating the location of the attribute in the returned JSON payload. Value **result.[0].number**.
**name**                                | A required parseNotation that indicates where to extract the incident **sys_id** in the returned payload data following a GET request. Value getincidentsysid.
**attribute**                           | Indicates which attribute to extract the information from. Value **sys_id**.
**notation**                            | Consists of the JSONPath notation indicating the location of the attribute in the returned JSON payload. Value **result.[0].sys_id**.
**name**                                | A required parseNotation that indicates where to extract the incident **state** in the returned JSON payload following a GET request. Value getincidentstate.
**attribute**                           | Indicates which attribute to extract the information from. Value **state**.
**notation**                            | Consists of the JSONPath notation indicating the location of the attribute in the returned JSON payload. Value **result.[0].state**.
**name**                                | A required parseNotation that indicates where to extract the incident **number** in the returned JSON payload following a POST request. Value incidentnumber.
**attribute**                           | Indicates which attribute to extract the information from. Value **number**.
**notation**                            | Consists of the JSONPath notation indicating the location of the attribute in the returned JSON payload. Value **result.number**.
**name**                                | A required parseNotation that indicates where to extract the incident sys_id in the returned payload following a POST request. Value incidentsysid.
**attribute**                           | Indicates which attribute to extract the information from. Value **sys_id**.
**notation**                            | Consists of the JSONPath notation indicating the location of the attribute in the returned JSON payload. Value **result.sys_id**.
**name**                                |A required parseNotation that indicates where to extract the application id in the returned JSON payload following a GET request to the CMDB. Value **cmdbsysid**.
**attribute**                           | Indicates which attribute to extract the information from. Value **sys_id**.
**notation**                            | Consists of the JSONPath notation indicating the location of the attribute in the returned JSON payload. Value **result.sys_id**.
**documentationRouting**                | header - Defines the attributes that will be submitted depending on if the job documentation contains valid instructions on what action should be taken on job failure. Requires **routingByDocumentationContent** and **extractXmlContentFromJobDescription** rule be set to True.
**routingAttribute**                    | Indicates the name of the attribute to be included in the ServiceNow object.
**workingHoursWithDoc**                 | The destination if the job fails during working hours and has valid instructions in the job documentation field.
**workingHoursWithoutDoc**              | The destination if the job fails during working hours and has no instructions in the job documentation field.
**nonWorkingHoursWithDoc**              | The destination if the job fails during nonworking hours and has valid instructions in the job documentation field.
**nonWorkingHoursWithoutDoc**           | The destination if the job fails during non working hours and has no instructions in the job documentation field.
**variables**                           | header - This defines variables required by the connector
**incidentReopenState**                 | the state the incident will be set to when the incident is re-opened when a restarted failed task fails again (values are defined in ServiceNow - 1 is New, 2 is In-progress).

The address section of the template defines a name associated with the ServiceNow Instance and the address associated with the ServiceNow Instance. This allows a single connector the ability to submit requests to multiple ServiceNow instances by creating multiple templates. The address section is a required attribute.

The credentials section of the template defines the ServiceNow user and password to be used for the connection. These values must be encrypted using the EncryptValues.exe program. The credentials section is a required attribute.

The urls section defines the urls that should be used for the connection. It is possible that when creating an incident, an intermediate table is used. This means that there are different urls for creating and retrieving incidents. The incident and getincident attributes define these urls. If there is no intermediate table for creating incidents, then these values will be the same. The attachment url defines the url that is used when attaching the job log to the incident. This value is required in the rule includeJobLogAttachment is set to ‘true’.

It is possible to extract the Incident application ID from the ServiceNow CMDB using a string extracted from the schedule name of the failed job. To use this feature, the rules applicationIdRetrievalFromCmdb and extractAppIdFromScheduleName must be set to ‘true’. When these rules are set, the cmdb url definition must contain the correct definition to submit a request to the CMDB and the appIdLocation must contain the location of the string to extract from the schedule name of the failed job.

The appIdLocation section defines the location of the string within the schedule or tag name to use to retrieve the Application ID to be used when submitted an incident. This is required when the rule extractAppIdFromScheduleName is set to ‘true’.

The tags section defines information if OpCon Job User defined tags are used to route tickets. Routing can be done by defining strings that either start (TAG_START) or end (TAG_END) a tag. If no values are found, then the DEFAULT value will be used. Tag routing is enabled when the rule includeTagRouting is set to ‘true’.

The workingHours section defines what is working hours. This allows a different set of attributes to defined when creating an incident for working and non-working hours.

The attributes section defines attributes that will be added to all tickets, while the workingHoursAttributes and nonWorkingHoursAttributes sections define additional attributes that will be included in those time slots.

The parseNotations sections defines attributes that need to be extracted and includes the JSONPath notation associated with extracting that attribute value from the returned JSON data.

It is possible to route the request to a specific destination within ServiceNow depending on if the failed job contains instructions on what action to perform on job failure. Implementing this function requires setting the extractXmlContentFromJobDescription and routingByDocumentationContent to true.

The jobDescriptionXmlTag definition contains the name of the XML structure in the job documentation that contains the instructions.

The documentationRouting definition contains the routing instructions which includes the ServiceNow attribute name and the routing destinations for working and nonworking hours as well as if the job documentation contains instructions or not.

Template Example

```
{
  "address" : {
    "name" : "production",
    "value" : "dev70313.service-now.com"
  }, 
  "rules" : {
    "includeJobLogAttachment" : true,
    "includeTagRouting" : false,
    "includeCorrelationId" : true,
    "applicationIdRetrievalFromCmdb" : false,
    "allowTicketReopen" : true,
    "extractAppIdFromScheduleName": false,
    "extractAppIdFromTagName": false,
    "extractXmlContentFromJobDescription": false,
    "routingByDocumentationContent": false
  },
  "credentials" : {
    "user" : "595752746157343d",
    "password" : "55335268636c6468636e4e534f574e7263773d3d"
  },
    "appIdLocation": {
    "startLocation": 2,
    "numberOfChars": 3,
    "tagIdPrefix":"TAG002"
  },
  "jobDescriptionXmlTag": {
    "xmlTag": ""
  },
  "urls" : [ {
    "name" : "incident",
    "value" : "api/now/table/incident"
  }, {
    "name" : "getincident",
    "value" : "api/now/table/incident"
  }, {
    "name" : "attachment",
    "value" : "api/now/attachment/file"
  } ],
   "tags" : [ {
    "indicator" : "TAG_END",
    "indicatorValue" : "ROUTE1",
    "attribute" : "category",
    "value" : "Software"
    },{
    "indicator" : "TAG_START",
    "indicatorValue" : "ROUTE2",
    "attribute" : "subcategory",
    "value" : "Operating System"
    },{
    "indicator" : "DEFAULT",
    "indicatorValue" : "DEFAULT",
    "attribute" : "category",
    "value" : "Network"
  } ],
  "workingHours" : {
    "monday" : {
      "start" : "0800",
      "stop" : "1900"
    },
    "tuesday" : {
      "start" : "0800",
      "stop" : "1900"
    },
    "wednesday" : {
      "start" : "0800",
      "stop" : "1900"
    },
    "thursday" : {
      "start" : "0800",
      "stop" : "1900"
    },
    "friday" : {
      "start" : "0800",
      "stop" : "1900"
    },
    "saturday" : {
      "start" : "0800",
      "stop" : "1100"
    },
    "sunday" : {
      "start" : "0000",
      "stop" : "0000"
    }
  },
  "attributes" : [ {
    "name" : "short_description",
    "value" : "OpCon Task Failure (schedule {0} job {1} server {2} error code {3})"
  }, {
    "name" : "description",
    "value" : "Technical data of the Opcon batch (schedule ({0}) job ({1}) server ({2}) error code ({3}) date ({4}) JOBID={5})"
  }, {
    "name" : "correlation_id",
    "value" : ""
  }, {
    "name" : "correlation_display",
    "value" : "SMA_OPCON"
  }, {
    "name" : "state",
    "value" : ""
  } ],
  "workingHoursAttributes" : [ {
    "name" : "urgency",
    "value" : "1"
  }, {
    "name" : "impact",
    "value" : "2"
  } ],
  "nonWorkingHoursAttributes" : [ {
    "name" : "urgency",
    "value" : "2"
  }, {
    "name" : "impact",
    "value" : "3"
  } ],
  "parseNotations" : [ {
    "name" : "getincidentnumber",
    "attribute" : "number",
    "notation" : "result.[0].number"
  }, {
    "name" : "getincidentsysid",
    "attribute" : "sys_id",
    "notation" : "result.[0].sys_id"
  }, {
    "name" : "getincidentstate",
    "attribute" : "state",
    "notation" : "result.[0].state"
  } , {
    "name" : "incidentnumber",
    "attribute" : "number",
    "notation" : "result.number"
  }, {
    "name" : "incidentsysid",
    "attribute" : "sys_id",
    "notation" : "result.sys_id"
  }, {
    "name" : "cmdbsysid",
    "attribute" : "sys_id",
    "notation" : "result.sys_id"
  } ],
  documentationRouting": {
    "routingAttribute": "",
    "workingHoursWithDoc": "",
    "workingHoursWithoutDoc": "",
    "nonWorkingHoursWithDoc": "",
    "nonWorkingHoursWithoutDoc": ""
  },
  "variables": {
    "incidentReopenState": 1,
    "cmdbQueryFieldAttributeName":"u_trigramme"
  }
}

```
### OpCon Notification Manager Definition
Notification Manager is used to execute the ServiceNow Connector when a task completes with a failure condition. Using this approach allows the tasks to be added to the rule instead of defining a failure event on every task. 

Using Notification Manager, select the Jobs tab and create a new Group called ServiceNow.
Once the Group has been created, select the ServiceNow Group, perform a ‘right-click’ and select Add Job Trigger. In the Add Job Trigger selection, select Job Failed.

In the Run Command tab, enter the following:
**Command**			       C:\Connectors\ServiceNow\SMASnowConnector.exe -a [[$MACHINE NAME]] -s [[$SCHEDULE NAME]] -jn [[$JOB NAME]] -e [[$JOB TERMINATION]] -sd [[$SCHEDULE DATE-SNOW]] -si [[$SCHEDULE ID]] -sn [[$SCHEDULE INST]] -t basic.json

                           Where 
                           C:\Connectors\EasyVista\EasyVista.exe 	is the location of the connector.
                           -a [[$MACHINE NAME]]		                resolves to the agent name
                           -s [[$SCHEDULE NAME]] 		            resolves to the schedule name
                           -jn [[$JOB NAME]]			            resolves to the job name
                           -e [[JOB TERMINATION]]		            resolves to the job termination code
	                       -sd [[$SCHEDULE DATE-SNOW]]	            resolves to the date (format YYYY-MM-DD)
	                       -si [[$SCHEDULE ID]]		                resolves to the schedule ID
	                       -sn [[$SCHEDULE INST]]		            resolves to the schedule instance
	                       -t bas_easyvista.json		            which template in the templates folder to use

**Working Directory**		C:\Connectors\ServiceNow
**Batch User**			    Use Service Account 	                The batch User under which the task will be run.

### ServiceNow SMA OpCon Application Installation
The ServiceNow SMA Application provides business rules and outbound Rest messages that can be imported into ServiceNow using the System Update Sets function.
The application can be downloaded from the opcon-servicenow-custom-application repository in the SMA Technologies Innovation Lab (https://github.com/SMATechnologies). 

Once the repository has been opened, oo to the Releases section on the right-hand side and select the latest tag and then 'click' on the required artifact (zip or tar.gz) to download the software.
 
After downloading the file, open the zip file and navigate to the application directory. Extract the sys_remote_update_set_a59fc0e9dbce00101e349b81ca96198f.xml file, save it and use this during the import process.

#### Import Process
To import the ServiceNow SMA OpCon Application:

- login into the ServiceNow Application.
- Navigate to the **System Update Sets** menu item and select **Retrieved Update Sets**.
- Select the **Import Update Set from XML** button (below Realted Links - bottom left of Retrieved Update Sets screen).
- Use the Browse Option to find the extracted sys_remote_update_set_a59fc0e9dbce00101e349b81ca96198f.xml file.
- 'Click' on the **Upload** button and the information should now appear on the Retrieved Update Sets list.
- Select the application by 'clicking' in the name **SMA OpConAPI** and the application information will appear.
- Select **Preview Update Set** in the top righthand corner. This process detects problems that may occur if you commit the updates on the local instance. After you preview and before you commit an Update Set, follow this procedure to resolve all the problems that the preview process discovered. Once it indicates 100% succeeded, it is possible to commit the new application into the ServiceNow instance and make it available.
- Select the **Commit Update Set** button (top righthand corner).
- Navigate to the **System Definition** menu item and select **Business Rules**.
- Enter the name **opcon** in the search field and transmit and the opcon business rules will be displayed.
- Select a rule and then mark it **active** by selecting the Active checkbox.

The application import provided two rules that can be used when an Incident is Resolved. The task in OpCon can be set to Fixed (rule OpConTicketResolved) which means operations staff must release the task or it can be automatically restarted (rule OpConTicketRestart).
Before using the business rules, the authentication header in the updateJobStatusByJobId rest message must be updated to include a valid OpCon application token.
- First create the OpCon Application token (see OpCon-API documentation on how to create an application token). 
- Select the **updateJobStatusByJobId** Outbound REST Message.
- Select the postStatus HTTP Method.
- Update the Authorization header attribute replacing the token value. Remember to keep the leading 'Token ' values.

#### Using Self-Signed Certificates
It is possible to use self-signed certificates when working with OpCon. However, it is recommended that for production systems a certificate from an authorized CA supplier should be used.

When working with self-signed certificates, it is possible to use the provided self-signed certificate when OpCon was installed (only uses the computer name for the CN field) or creating a new self-signed certificate using the full DNS name for the certificate CN field. When using the existing self-signed certificate, the IP Address of the OpCon server must be used.

##### Create New Certificate
Using PowerShell, enter the following commands to create a self-signed certificate in the Personal section of the certIm (Certificates - local Computer) using the full DNS name and setting a password that can be used when moving the certificate.

```
New-SelfSignedCertificate -certstorelocation cert:\\localmachine\\my -dnsname "\<***full dns name***\>"
$pwd = ConvertTo-SecureString -String "\<***password***\>" -Force -AsPlainText

```

Once these commands have been processed, the certificate can be found by managing computer certificates.

The created certificate must be moved from the Personal section to the Trusted Root Certification section. 

First create an encrypted file containing the certificate.
- Right-click on the created certificate and select All tasks, then Export, then Next. 
- Select Yes, export the private key.
- Select the Personal Information Exchange – PKCS #12 (.PFX) format.  (ensure that Include all certificates in the certification path if possible and Enable certificate privacy are selected).
- Enter a password that will be used when importing the certificate. 
- Use the default encryption method.
- Select a file to write the encrypted certificate into.
- Complete final questions to create encrypted file.

Go to Certificates in the Trusted Root Certification section.
- Right-click on the Certificates section.
- Select All Tasks, then Import, then Next.
- Use browse to select the file you created in the previous step.
- Enter the password you used in the first step when creating the file.
- Select the Place all certificates in the following store.
- Ensure that Trusted Root Certification Authorities is displayed.
- Complete final questions to import certificate into the store.

Once the certificate has been loaded into the Trusted Root Certification Authorities store, it must be registered with the OpCon-API Rest Server. This process consists of editing the SMAOpConRestApi.ini file replacing the CertificateSerialNumber with the value of the new certificate and registering the new certificate.
- Stop the SMA Opcon RestAPI service.
- Display the Serial Number of the new certificate by selecting the certificate, Details and the Serial Number.
- Edit the SMAOpConRestApi.ini file in the Program Data\OpConxps\SAM folder.
- Update the CertificateSerialNumber field with the new value. Do not use copy & paste as there are some hidden characters in the value.
- Open an administrator cmd window.
- Execute the “C:\Program Files\OpConxps\SAM>SMAOpConRestApi.Controllers.exe setcertificate” command
  - (you may see some errors, but the key items to check are)
  -	SSL Certificate successfully deleted
  -	SSL Certificate successfully added
- Restart the SMA Opcon RestAPI service

elect the certificate in certlm, perform a right-click, select All Tasks and Export. You should only be allowed to export the public key. Select the DER encoded binary X.509 (.CER) format and save the exported file (file will have a .cer extension).
Go to your ServiceNow application and enter certificates in the Filter navigator. 
 
- Select the New button.
- Enter a name for the certificate and select the DER format. You will get a message asking you to attach a DER encoded certificate file. 
- To attach the file, click on the paperclip in the upper right-hand corner. 
- A pop-up window will ask you to select the file to upload. When selecting Choose file, you can browse for the file you created. After the file has been loaded, you will see the following window which can be closed.
 
After the file has been loaded, click on the Validate Stores/Certificates button in the lower left-hand corner to validate the extracted certificate. If the certificate is valid, certificate information will be displayed.
Select Update to save the key.

Using the Certificate

There are some additional steps required within ServiceNow to complete the process. If you are using a new self-signed certificate, then you only need to set the following system property **com.glide.communications.httpclient.verify_revoked_certificate** must be set to false. This property can be found by entering the sys_properties.list value in the Navigator filter navigator. If this property cannot be found in the list, you can create it by selecting New button and entering the following:
-	Name	com.glide.communications.httpclient.verify_revoked_certificate
-	Type	true|false
-	Value 	false

When sending the Rest Message, the full DNS name of the OpCon server should be used in the URL as this name is verified against the CN values of the certificates in the certificate store.
When using the existing self-signed certificate, hostname verification must also be disabled, and IP Address values should be then used. To disable hostname verification set the system property **com.glide.communications.httpclient.verify_hostname** to false This property can be found by entering the sys_properties.list value in the Navigator Filter.

 
