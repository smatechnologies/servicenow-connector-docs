---
sidebar_label: 'Installation'
title: ServiceNow Connector installation
description: "Install, configure, and integrate the ServiceNow Connector with OpCon and a ServiceNow instance."
tags:
  - Procedural
  - System Administrator
  - Installation
  - Connectors
---

# Installation

## What is it?

This page covers the end-to-end installation of the ServiceNow Connector — installing the connector and its supporting Windows Agent, configuring the connection to OpCon and ServiceNow, and importing the SMA OpCon ServiceNow Application into your ServiceNow instance.

- Use this page when installing the ServiceNow Connector for the first time.
- Use this page when configuring a self-signed certificate for the OpCon-to-ServiceNow connection.

For a conceptual overview of how the connector works at runtime, see the [ServiceNow Connector overview](./overview.md).

## Before you begin

The following software levels are required to implement this version (21.x.x) of the ServiceNow Connector:

| Component | Requirement |
| --- | --- |
| OpCon | Release 21.0 or higher |
| OpCon Rest-API | Configured to use TLS |
| OpCon Windows Agent | Installed on the same Windows server as the ServiceNow Connector |
| OpCon Notification Manager | Available |
| Java | Embedded OpenJDK 11 (part of the connector installation) |
| ServiceNow | An instance that supports the Rest API |
| ServiceNow credentials | A user with privileges to submit incidents and attachments — encrypted using `EncryptValue.exe` |

## Installation checklist

The installation is broken into six phases. Complete them in order — each phase depends on the previous.

| # | Phase | Where it runs |
| --- | --- | --- |
| 1 | [Install the OpCon Windows Agent](#opcon-windows-agent-installation) | Target Windows server |
| 2 | [Install the ServiceNow Connector files](#servicenow-connector-installation) | Target Windows server |
| 3 | [Configure the connector](#servicenow-connector-configuration) (Connector.config + at least one template) | Target Windows server |
| 4 | [Define the Notification Manager rule](#opcon-notification-manager-definition) | OpCon Solution Manager |
| 5 | [Import the SMA OpCon ServiceNow Application](#servicenow-sma-opcon-application-installation) | ServiceNow instance |
| 6 | [Configure self-signed certificates](#using-self-signed-certificates) (only if not using a CA certificate) | OpCon and ServiceNow |

## OpCon Windows Agent installation

The ServiceNow Connector requires the installation of a Windows Agent on the same system as the ServiceNow Connector. Either use an existing Windows Agent or complete the installation of the Windows Agent.

## ServiceNow Connector installation

The ServiceNow Connector can be installed on any Windows Server, as long as there is an OpCon MSLSAM Agent installed on that Windows Server.

To install the connector files, complete the following steps:

1. Copy the downloaded install file `SMAServiceNowConnector-win.zip` and extract it into a temp directory (`c:\temp`).
2. Extract the contents — including sub-directories — into the required installation directory.

After the extraction, the root installation directory contains:

| Item | Purpose |
| --- | --- |
| `SMASnowConnector.exe` | Connector executable. |
| `EncryptValue.exe` | Encryption utility for credentials. |
| `Connector.config` | Configuration file (described below). |
| `java/` | Java software required to run the connector (OpenJDK 11). |
| `joblogs/` | Temporary storage for job logs extracted from the OpCon system. |
| `log/` | Connector log files. |
| `templates/` | ServiceNow template files. Includes an initial `basic.json` template that you can modify. |

### Create the logfiles directory

Create a `/logfiles` directory off the installation directory. This directory is used as temporary storage when attaching job log files to incident tickets.

### Create the $SCHEDULE DATE-SNOW global property

Create the special **$SCHEDULE DATE-SNOW** global property that contains the schedule date in the `yyyy-MM-dd` format from the standard `$SCHEDULE DATE` property. Set the value to **yyyy-MM-dd**.

## ServiceNow Connector configuration

Configuring the ServiceNow Connector involves two files:

- The `Connector.config` file — defines the OpCon connection.
- One or more template files in `templates/` — define each ServiceNow instance and the rules used to create incidents.

:::caution
All user and password values placed in the configuration and template files must be encrypted using the `EncryptValue.exe` utility provided with the connector.
:::

### EncryptValue utility

The `EncryptValue` utility uses standard 64-bit encryption. It supports a `-v` argument and displays the encrypted value.

On Windows, the following example shows how to encrypt the value `abcdefg`:

```
EncryptValue.exe -v abcdefg

```

### Connector.config configuration

The `Connector.config` file lives in the installation directory and is split into three INI-style sections.

#### `[GENERAL]` section

| Property | Description |
| --- | --- |
| **LOG_FILES_DIRECTORY** | The name of the directory where retrieved log files are stored. After successful attachment to the ServiceNow incident, the log file is deleted. The name is a sub-directory of the installation directory (default `logfiles`). |
| **TEMPLATES_DIRECTORY** | The name of the directory where the template definitions are stored. The name is a sub-directory of the installation directory (default `templates`). |
| **DEBUG** | Connector debug mode. Set to `ON` only when capturing an error condition; otherwise set to `OFF`. Default `OFF`. |

#### `[PROXY CONNECTION]` section

Used to define the full URL of a proxy server, if required.

| Property | Description |
| --- | --- |
| **USE_PROXY** | Indicates whether this connector should use a proxy server connection. Values `True` or `False`. Default `False`. |
| **PROXY_ADDRESS** | The address of the proxy server. |
| **PROXY_PORT** | The port of the proxy server. |

#### `[OPCON API CONNECTION]` section

Used to define the connection to the OpCon API.

| Property | Description |
| --- | --- |
| **SERVER** | The server address of the OpCon API. |
| **PORT** | The port number used by the OpCon API server. |
| **USES_TLS** | Must be set to `True`. |
| **TOKEN** | An application token used for authentication when communicating with the OpCon-API. |
| **RET_SERVER** | The address of the OpCon Server. If using a DNS name, ensure that the DNS name is visible in the public domain. Used by the ServiceNow OpCon Application to submit report-back commands to OpCon. |
| **RET_PORT** | The port of the OpCon Rest-API Server. Used by the ServiceNow OpCon Application to submit report-back commands to OpCon. |

The `RET_SERVER` and `RET_PORT` values are combined to form the OpCon Rest-API address that is sent to ServiceNow.

#### Example `Connector.config`

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

### Templates

A template provides the information used when submitting an incident request to ServiceNow. Because some sites submit requests to an intermediate table instead of directly to the incident table, templates make these differences configurable.

A template includes the address of the ServiceNow instance, credentials, urls, tag routing definitions, working-hour definitions, attributes for working or non-working hours, and parsing information.

:::tip
A single connector can submit to multiple ServiceNow instances by using multiple templates — one template per instance.
:::

The sections below describe each top-level attribute. Each section is marked **Required**, **Optional**, or **Required when ...** so you can identify the minimum configuration first. See the [Example template](#example-template) at the end for a complete starter file.

A minimum working template includes: `address`, `rules`, `credentials`, `urls`, `attributes`, and `parseNotations`. Add the optional or conditional sections only when you enable the corresponding rule.

#### `address`

**Required.** Defines the ServiceNow instance address.

| Attribute | Description |
| --- | --- |
| **name** | The name associated with the ServiceNow instance (for example, `production`, `test`). |
| **value** | The address of the ServiceNow instance. |

#### `rules`

**Required.** Defines which functions are supported by the connector. All rules have defaults — only set the rules you want to change.

| Rule | Description | Default |
| --- | --- | --- |
| **includeJobLogAttachment** | Indicates whether the job log should be attached to the incident ticket. | `true` |
| **includeTagRouting** | Indicates whether user-defined tags should be used for incident routing. | `false` |
| **includeCorrelationId** | Indicates whether correlation information should be included in the information submitted to EasyVista. | `false` |
| **applicationIdRetrievalFromCmdb** | Indicates whether the application ID used during incident creation must be retrieved from the ServiceNow CMDB. | `false` |
| **allowTicketReopen** | Indicates whether, if a ticket already exists for a failed task and the ServiceNow ticket is in an appropriate state (not closed or cancelled), the ticket should be reopened. | `true` |
| **extractAppIdFromScheduleName** | Used with **applicationIdRetrievalFromCmdb**. Indicates that the value used to retrieve the application ID from the CMDB must be retrieved from the schedule name of the job. Mutually exclusive with **extractAppIdFromTagName**. | `false` |
| **extractAppIdFromTagName** | Used with **applicationIdRetrievalFromCmdb**. Indicates that the value used to retrieve the application ID from the CMDB must be retrieved from the first tag name in the tag list associated with the job. Mutually exclusive with **extractAppIdFromScheduleName**. | `false` |
| **extractXmlContentFromJobDescription** | Allows the extraction of XML content from the associated job documentation. The extracted content is appended to the description attribute submitted to ServiceNow. The **jobDescriptionXmlTag** attribute contains the name of the XML tag in the associated job documentation field. | `false` |
| **routingByDocumentationContent** | Implements a specific routing requirement based on whether the job documentation contains instructions on what action to take should the job fail. The instructions are contained within a set of XML tags. The XML tag name is defined in the `jobDescriptionXmlTag` attribute. The **documentationRouting** attribute contains the routing information to be included in the ServiceNow object. | — |
| **alwaysCreateNewTicket** | Indicates whether a new ticket must be created if the failed job already has an existing ticket assigned. | `false` |

#### `credentials`

**Required.** Defines the ServiceNow user and password used for the connection.

| Attribute | Description |
| --- | --- |
| **user** | The user that has the required privileges to connect to the ServiceNow System to submit requests. Must be encrypted using `EncryptValue.exe`. |
| **password** | The password of the user that has the required privileges to connect to the ServiceNow System to submit requests. Must be encrypted using `EncryptValue.exe`. |

#### `appIdLocation`

**Required when `extractAppIdFromScheduleName` is `true`.** Defines where in the schedule (or tag) name to find the string used to retrieve the application ID from the CMDB.

| Attribute | Description |
| --- | --- |
| **startLocation** | Indicates where in the schedule name of the failed job the string starts. |
| **numberOfChars** | Indicates the number of characters to extract. |
| **tagIdPrefix** | A character string that can be matched against the start of a tag name to determine which tag contains the indicator to use for the CMDB match. |

#### `jobDescriptionXmlTag`

**Required when `extractXmlContentFromJobDescription` is `true`.** Used with the **extractXmlContentFromJobDescription** rule.

| Attribute | Description |
| --- | --- |
| **xmlTag** | The name of the XML tag to search for in the associated job documentation. |

#### `urls`

**Required.** Defines the urls used when submitting requests to ServiceNow. Each entry is a `name`/`value` pair.

| Name | Required? | Description | Default |
| --- | --- | --- | --- |
| **incident** | Required | URL used when submitting a POST or PUT to ServiceNow (creating or updating an incident). | `api/now/table/incident` |
| **getincident** | Required | URL used when submitting a GET to ServiceNow (retrieving an existing incident). | `api/now/table/incident` |
| **attachment** | Optional (required if **includeJobLogAttachment** is `true`) | URL used when attaching the OpCon job log to the ticket. | `api/now/table/incident` |
| **cmdb** | Optional (required if **applicationIdRetrievalFromCmdb** and **extractAppIdFromScheduleName** are `true`) | URL used when retrieving the application id from the CMDB. | `api/now/cmdb` |

If your site does not use an intermediate table for creating incidents, the **incident** and **getincident** values are the same.

#### `tags`

**Required when `includeTagRouting` is `true`.** Defines OpCon user-defined tag routing.

| Attribute | Description |
| --- | --- |
| **indicator** | What part of the tag should be used to identify the request. Supports `TAG_END`, `TAG_START`, or `DEFAULT`. The `DEFAULT` value is used if there is no `TAG_END` or `TAG_START` match and tag routing is enabled. |
| **indicatorValue** | The value matched against the OpCon user-defined tag (either the end or the start). |
| **attribute** | The attribute name added to the JSON payload along with the value. |
| **value** | The value associated with the attribute. |

#### `workingHours`

**Required when using `workingHoursAttributes` or `nonWorkingHoursAttributes`.** Defines working hours per day, used to apply different attribute sets for working vs. non-working hours. Each day (`monday` through `sunday`) has a `start` and `stop` pair.

| Attribute | Description |
| --- | --- |
| **start** | The start time. Four digits, `HHMM`. |
| **stop** | The stop time. Four digits, `HHMM`. |

The `monday`, `tuesday`, `wednesday`, `thursday`, `friday`, `saturday`, and `sunday` keys each carry a `start`/`stop` pair in this format.

#### `attributes`

**Required.** Attributes added to all incident tickets. Each entry is a `name`/`value` pair.

| Name | Required? | Description |
| --- | --- | --- |
| **short_description** | Required | Default value: `OpCon Task Failure (schedule {0} job {1} server {2} error code {3})`. The placeholders are filled in by the connector with the schedule, job, server, and error code. The text can be changed, but consider the placeholder positions. |
| **description** | Required | Default value: `Technical data of the Opcon batch (schedule ({0}) job ({1}) server ({2}) error code ({3}) date ({4}) JOBID={5})`. The placeholders are filled in by the connector with the schedule, job, server, date, error code, and unique job id. The text can be changed, but consider the placeholder positions. |
| **correlation_id** | Optional (required if **includeCorrelationId** is `true`) | Used to pass information to ServiceNow so that the SMA ServiceNow OpCon Application can update job information. The value can be left as an empty string — the connector adds it. |
| **correlation_display** | Optional (required if **includeCorrelationId** is `true`) | Used to pass information to ServiceNow so that the SMA ServiceNow OpCon Application can update job information. Set to `SMA_OPEN` to indicate that the incident originated from OpCon. |
| **state** | Required | Used when a failed task already has an existing ticket — the ticket state is reset if allowed. The value can be left as an empty string — the connector adds it. |

#### `workingHoursAttributes` and `nonWorkingHoursAttributes`

**Optional.** Additional attributes submitted only during working hours or non-working hours, respectively. Each entry is a `name`/`value` pair. Requires `workingHours` to be defined.

| Attribute | Description |
| --- | --- |
| **name** | The name of the attribute. |
| **value** | The value of the attribute. |

#### `parseNotations`

**Required.** Tells the connector how to extract values from the JSON payloads ServiceNow returns. Uses JSONPath. Each entry has `name`, `attribute`, and `notation` values.

| Name | Attribute | Notation | Used after |
| --- | --- | --- | --- |
| **getincidentnumber** | `number` | `result.[0].number` | GET request |
| **getincidentsysid** | `sys_id` | `result.[0].sys_id` | GET request |
| **getincidentstate** | `state` | `result.[0].state` | GET request |
| **incidentnumber** | `number` | `result.number` | POST request |
| **incidentsysid** | `sys_id` | `result.sys_id` | POST request |
| **cmdbsysid** | `sys_id` | `result.sys_id` | GET request to the CMDB |

#### `documentationRouting`

**Required when `routingByDocumentationContent` and `extractXmlContentFromJobDescription` are both `true`.** Defines attributes submitted depending on whether the job documentation contains valid instructions on the action to take on job failure.

| Attribute | Description |
| --- | --- |
| **routingAttribute** | The name of the attribute to be included in the ServiceNow object. |
| **workingHoursWithDoc** | Destination if the job fails during working hours and has valid instructions in the job documentation field. |
| **workingHoursWithoutDoc** | Destination if the job fails during working hours and has no instructions in the job documentation field. |
| **nonWorkingHoursWithDoc** | Destination if the job fails during non-working hours and has valid instructions in the job documentation field. |
| **nonWorkingHoursWithoutDoc** | Destination if the job fails during non-working hours and has no instructions in the job documentation field. |

#### `variables`

**Optional.** Variables required by specific connector behaviors.

| Attribute | Description |
| --- | --- |
| **incidentReopenState** | The state the incident is set to when it is reopened after a restarted failed task fails again. Values are defined in ServiceNow — `1` is New, `2` is In-progress. |

#### Example template

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

## OpCon Notification Manager definition

Notification Manager is used to run the ServiceNow Connector when a task completes with a failure condition. Using this approach, jobs are added to a single rule rather than defining a failure event on every task.

To configure the Notification Manager rule, complete the following steps:

1. Open Notification Manager and select the **Jobs** tab.
2. Create a new **Group** called **ServiceNow**.
3. Select the **ServiceNow** group, right-click, and select **Add Job Trigger**.
4. In the **Add Job Trigger** selection, select **Job Failed**.
5. Select the **Run Command** tab and enter the following values:

   | Field | Value |
   | --- | --- |
   | **Command** | `C:\Connectors\ServiceNow\SMASnowConnector.exe -a [[$MACHINE NAME]] -s [[$SCHEDULE NAME]] -jn [[$JOB NAME]] -e [[$JOB TERMINATION]] -sd [[$SCHEDULE DATE-SNOW]] -si [[$SCHEDULE ID]] -sn [[$SCHEDULE INST]] -t basic.json` |
   | **Working Directory** | `C:\Connectors\ServiceNow` |
   | **Batch User** | Use Service Account (the batch user under which the task will be run) |

The command arguments resolve as follows:

| Argument | Resolves to |
| --- | --- |
| `C:\Connectors\ServiceNow\SMASnowConnector.exe` | The location of the connector. <!-- TODO: SME review — original referenced C:\Connectors\EasyVista\EasyVista.exe; updated to match the ServiceNow connector path used elsewhere on this page. Confirm correct path. --> |
| `-a [[$MACHINE NAME]]` | The agent name. |
| `-s [[$SCHEDULE NAME]]` | The schedule name. |
| `-jn [[$JOB NAME]]` | The job name. |
| `-e [[$JOB TERMINATION]]` | The job termination code. |
| `-sd [[$SCHEDULE DATE-SNOW]]` | The date (format `YYYY-MM-DD`). |
| `-si [[$SCHEDULE ID]]` | The schedule ID. |
| `-sn [[$SCHEDULE INST]]` | The schedule instance. |
| `-t basic.json` | The template in the `templates` folder to use. <!-- TODO: SME review — original example listed -t bas_easyvista.json in the table while the command shows -t basic.json. Aligned to basic.json; confirm. --> |

## ServiceNow SMA OpCon Application installation

The ServiceNow SMA Application provides business rules and outbound Rest messages that can be imported into ServiceNow using the **System Update Sets** function.

### Download the application

The application can be downloaded from the `opcon-servicenow-custom-application` repository in the SMA Technologies Innovation Lab (https://github.com/SMATechnologies).

To download the application:

1. Open the repository.
2. Go to the **Releases** section on the right-hand side and select the latest tag.
3. Select the required artifact (zip or tar.gz) to download the software.
4. Open the zip file and go to the application directory.
5. Extract the `sys_remote_update_set_a59fc0e9dbce00101e349b81ca96198f.xml` file and save it. Use this file during the import process.

### Import process

To import the ServiceNow SMA OpCon Application, complete the following steps:

1. Log in to the ServiceNow Application.
2. Go to the **System Update Sets** menu item and select **Retrieved Update Sets**.
3. Select the **Import Update Set from XML** button (below **Related Links** — bottom left of the **Retrieved Update Sets** screen).
4. Use the **Browse** option to find the extracted `sys_remote_update_set_a59fc0e9dbce00101e349b81ca96198f.xml` file.
5. Select the **Upload** button. The information appears on the **Retrieved Update Sets** list.
6. Select the application name **SMA OpConAPI**. The application information is displayed.
7. Select **Preview Update Set** in the top right-hand corner. This process detects problems that may occur if you commit the updates on the local instance. After you preview and before you commit an Update Set, follow this procedure to resolve all the problems that the preview process discovered. Once it indicates 100% succeeded, you can commit the new application into the ServiceNow instance.
8. Select the **Commit Update Set** button (top right-hand corner).
9. Go to the **System Definition** menu item and select **Business Rules**.
10. Enter `opcon` in the search field and submit. The opcon business rules are displayed.
11. Select a rule and mark it **active** by selecting the **Active** option.

:::note
The application import provides two rules that can be used when an incident is **Resolved**:

- **OpConTicketResolved** — sets the OpCon task to **Fixed** (operations staff must release the task).
- **OpConTicketRestart** — automatically restarts the OpCon task.

Activate one of these two rules according to your preferred behavior.
:::

### Configure the authentication header

Before using the business rules, the authentication header in the `updateJobStatusByJobId` Rest message must be updated to include a valid OpCon application token.

To update the authentication header, complete the following steps:

1. Create the OpCon Application token (see the OpCon-API documentation on how to create an application token).
2. Select the **updateJobStatusByJobId** Outbound REST Message.
3. Select the **postStatus** HTTP Method.
4. Update the **Authorization** header attribute, replacing the token value. Keep the leading `Token ` value.

## Using self-signed certificates

It is possible to use self-signed certificates when working with OpCon. However, for production systems, a certificate from an authorized CA supplier is recommended.

When working with self-signed certificates, you can either:

- Use the provided self-signed certificate from when OpCon was installed (which only uses the computer name for the CN field), or
- Create a new self-signed certificate using the full DNS name for the certificate CN field.

When using the existing self-signed certificate, the IP Address of the OpCon server must be used.

:::note
This is a six-step process that touches both Windows certificate stores and ServiceNow system properties. Plan to complete all six steps in one session.
:::

### Step 1. Create a new certificate

Using PowerShell, run the following commands to create a self-signed certificate in the **Personal** section of `certlm` (Certificates — local Computer) using the full DNS name. Set a password that can be used when moving the certificate.

```
New-SelfSignedCertificate -certstorelocation cert:\\localmachine\\my -dnsname "\<***full dns name***\>"
$pwd = ConvertTo-SecureString -String "\<***password***\>" -Force -AsPlainText

```

Once these commands have been processed, the certificate can be found by managing computer certificates. The certificate must be moved from the **Personal** section to the **Trusted Root Certification** section.

### Step 2. Export the encrypted certificate

To create an encrypted file containing the certificate, complete the following steps:

1. Right-click the created certificate and select **All Tasks**, then **Export**, then **Next**.
2. Select **Yes, export the private key**.
3. Select the **Personal Information Exchange – PKCS #12 (.PFX)** format. Ensure that **Include all certificates in the certification path if possible** and **Enable certificate privacy** are selected.
4. Enter a password that will be used when importing the certificate.
5. Use the default encryption method.
6. Select a file to write the encrypted certificate into.
7. Complete the final questions to create the encrypted file.

### Step 3. Import to Trusted Root Certification Authorities

To import the certificate into the **Trusted Root Certification Authorities** store, complete the following steps:

1. Go to **Certificates** in the **Trusted Root Certification** section.
2. Right-click the **Certificates** section.
3. Select **All Tasks**, then **Import**, then **Next**.
4. Browse to select the file you created in the previous step.
5. Enter the password you used when creating the file.
6. Select **Place all certificates in the following store**.
7. Ensure that **Trusted Root Certification Authorities** is displayed.
8. Complete the final questions to import the certificate into the store.

### Step 4. Register the certificate with the OpCon-API

Once the certificate has been loaded into the **Trusted Root Certification Authorities** store, register it with the OpCon-API Rest Server. This involves editing `SMAOpConRestApi.ini` to replace the `CertificateSerialNumber` with the new certificate's serial number.

To register the new certificate, complete the following steps:

1. Stop the **SMA OpCon RestAPI** service.
2. Display the **Serial Number** of the new certificate by selecting the certificate, then **Details**, then **Serial Number**.
3. Edit the `SMAOpConRestApi.ini` file in the `Program Data\OpConxps\SAM` folder.
4. Update the `CertificateSerialNumber` field with the new value.

   :::caution
   Do not use copy and paste — there are some hidden characters in the value.
   :::

5. Open an administrator `cmd` window.
6. Run the `C:\Program Files\OpConxps\SAM>SMAOpConRestApi.Controllers.exe setcertificate` command. You may see some errors, but the key items to check are:
   - `SSL Certificate successfully deleted`
   - `SSL Certificate successfully added`
7. Restart the **SMA OpCon RestAPI** service.

### Step 5. Export the public key and load it into ServiceNow

To export the public key and load it into ServiceNow, complete the following steps:

1. In `certlm`, right-click the certificate and select **All Tasks** then **Export**. You should only be allowed to export the public key.
2. Select the **DER encoded binary X.509 (.CER)** format and save the exported file (the file will have a `.cer` extension).
3. In your ServiceNow application, enter `certificates` in the **Filter navigator**.
4. Select the **New** button.
5. Enter a name for the certificate and select the **DER** format. You will get a message asking you to attach a DER encoded certificate file.
6. To attach the file, select the paperclip in the upper right-hand corner.
7. In the pop-up window, select **Choose file** to browse for the file you created. After the file has been loaded, the window can be closed.
8. Select the **Validate Stores/Certificates** button in the lower left-hand corner to validate the extracted certificate. If the certificate is valid, certificate information is displayed.
9. Select **Update** to save the key.

### Step 6. ServiceNow system properties

Some additional steps are required within ServiceNow to complete the process.

If you are using a **new** self-signed certificate, set the system property **com.glide.communications.httpclient.verify_revoked_certificate** to `false`. This property can be found by entering `sys_properties.list` in the Navigator filter. If this property cannot be found, create it by selecting **New** and entering:

- **Name:** `com.glide.communications.httpclient.verify_revoked_certificate`
- **Type:** `true|false`
- **Value:** `false`

When sending a Rest Message, the **full DNS name** of the OpCon server should be used in the URL — this name is verified against the CN values of the certificates in the certificate store.

If you are using the **existing** self-signed certificate, hostname verification must also be disabled and IP Address values should be used. To disable hostname verification, set the system property **com.glide.communications.httpclient.verify_hostname** to `false`. This property can be found by entering `sys_properties.list` in the Navigator filter.
