---
sidebar_label: 'Release notes'
title: ServiceNow Connector release notes
description: "Version history and change details for the ServiceNow Connector, including new features, fixes, and migration considerations."
tags:
  - Reference
  - System Administrator
  - Connectors
---

# ServiceNow Connector release notes

## General

Requires **OpCon 20.7 STS** or greater due to OpCon-API requirements.

:::note
This history covers releases from 21.2.1 onwards. Earlier releases are not documented here.
:::

## 26

### 26.0

**Released:** 2026 May

#### What's new

- **CON-1331**: Removed vulnerability CVE-2022-41404.


## 25

### 25.0

**Released:** 2025 April

#### What's new

- **INTPLT-387**: Removed the old opcon-rest-api-client library, replacing it with direct calls to the OpCon Rest-API. This release is completely rebased: all OpCon communications (retrieving DailyJob information, updating the DailyJob Incident value, and retrieving log files) now use direct OpCon Rest-API calls.

#### Fixes

- **INTPLT-413**: When ticket reopen is enabled, the incident state is taken from the `incidentReopenState` variable in the variables section of the template. If the variable is not defined, the default value of 2 (IN PROGRESS) is used.

Incident states:

| Value | State |
| ----- | ----------- |
| 1 | NEW |
| 2 | IN PROGRESS |
| 3 | ON HOLD |
| 6 | RESOLVED |
| 7 | CLOSED |
| 8 | CANCELLED |

```
"variables": {
    "incidentReopenState": 2
}
```

## 21

### 21.5

**Released:** 2023 November

#### What's new

- **CONNUTIL-621**: Added a new attribute `tagIdPrefix` to the `appIdLocation` structure. The attribute can be used to identify the tag that contains the application id. The value of the attribute is used to match the start characters of the tag. The first tag in the list that matches will be used.

```
"appIdLocation": {
    "startLocation": 6,
    "numberOfChars": 3,
    "tagIdPrefix": "TAG002"
}
```

### 21.4

**Released:** 2023 September

#### What's new

- **CONNUTIL-614**: Adjusted actions when a ticket exists and reopening of a ticket is allowed:

- Incident state New, In-Progress or On-Hold: no state change and log files are appended to the existing incident ticket.
- Incident state Resolved: state changed to In-Progress and log files appended to the existing incident ticket.
- Incident state Cancelled or Closed: a new incident is created and log files appended to the new incident ticket.

#### Fixes

- **CONNUTIL-615**: Fixed a problem where updating or inserting ticket information into the OpCon job record always returned a false value.

### 21.3

**Released:** 2023 May

#### What's new

- **CONNUTIL-605**: Modified ServiceNow to accept a dynamic value for the query field name when retrieving the application id from the cmdb. Changed the cmd url definition in the factory to include a replacement value for the application query attribute name. Added the variable `cmdbQueryFieldAttributeName` to the variables section of the template.

```
"variables": {
    "incidentReopenState": 1,
    "cmdbQueryFieldAttributeName": "u_trigramme"
}
```

The value should be set to the required query field name. The default value is **u_trigramme**.

#### Migration considerations

After installation, add `"cmdbQueryFieldAttributeName": "u_trigramme"` to the variables section of the json template. Modify `u_trigramme` to the required value or retain the default value.

### 21.2.1

**Released:** 2023 February

#### What's new

- **CONNUTIL-594**: Implemented configurable incident state when reopening a ServiceNow incident after a restarted failed task fails. Added a variables section to the template.

```
"variables": {
    "incidentReopenState": 1
}
```

The value should be set to the state required by your organization when reopening the incident (1 is New, 2 is In-Progress).

#### Fixes

- **CONNUTIL-585**: Fixed an issue where files were not attached when a restarted failed job failed again.

- **CONNUTIL-586**: Fixed an issue when updating information for a restarted failed task that fails again — only the description is now updated.

- **CONNUTIL-587**: Fixed an issue where a failed task with an existing ticket in a 'Closed' or 'Cancelled' state tried to insert a new ticket number into the job information instead of updating the job information.

#### Migration considerations

The template used by the connector must be updated to include the new variables section.
