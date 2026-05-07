---
sidebar_label: 'Operation'
title: ServiceNow Connector operation
description: "Operate the ServiceNow Connector day-to-day, including viewing incident status from OpCon and understanding incident state transitions."
tags:
  - Procedural
  - Operations Staff
  - Connectors
---

# Operation

## What is it?

This page covers the day-to-day operation of the ServiceNow Connector after it has been installed and configured: adding new jobs to the ServiceNow notification group, viewing the status of an incident from OpCon, and the state transitions that the connector applies to existing incident tickets.

- Use this page after a job has failed and you need to find or update its ServiceNow incident from Solution Manager.
- Use this page when adding new jobs that should generate ServiceNow incidents on failure.

Once the ServiceNow Connector has been installed and configured, there are very few operational requirements.

## Adding jobs to the ServiceNow group in Notification Manager

Depending on the Notification Manager configuration, newly defined jobs may need to be included in the ServiceNow group.

## Viewing incident status

You can display the status of the Incident directly from OpCon by using Solution Manager.

To view the incident status for a failed job, complete the following steps:

1. In Solution Manager, select the failed job and right-click to open the Job Selection dialog.
2. In the Job Selection dialog, select the **Incident Ticket** value.
3. If required, enter the user and password for ServiceNow. The ServiceNow ticket information is displayed.

## Incident state updates

If an Incident ticket already exists for the failed job and update of ServiceNow Incident tickets is allowed, the state of the existing Incident ticket is modified as follows:

| Current state | New state | Description |
| --- | --- | --- |
| New | no change | Incident Ticket remains in the New state. |
| In-Progress | no change | Incident Ticket remains in the In-Progress state. |
| On-Hold | no change | Incident Ticket remains in the On-Hold state. |
| Resolved | In-Progress | Incident Ticket state is changed to In-Progress. |
| Cancelled | (new ticket) | A new Incident Ticket is created. |
| Closed | (new ticket) | A new Incident Ticket is created. |
