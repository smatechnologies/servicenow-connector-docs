# Operation

Once the ServiceNow connector has been installed and configured, there is very little operational requirements.

## Adding Jobs to ServiceNow Group in Notification Manager
Depending on the Notification Manager configurations, newly defined jobs may need to be included in the ServiceNow Group.

## Viewing Incident Status
It is possible to display the status of the Incident directly from OpCon by using Solution Manager.

Select the failed job and then ‘right click’ to get the Job Selection dialogue.

In the Job Selection Dialogue ‘click’ on the Incident Ticket value.

If required enter the user / password for ServiceNow and the ServiceNow ticket information will be displayed.

## Incident States updates
If an Incident ticket already exists for the failed job and update of ServiceNow Incident tickets is allowed the state of the existing Incident ticket will me modified as follows:

Current State | New State     | Description
------------- | ------------- | -----------------------
New           | no change     | Incident Ticket will remain in the New state.
In-Progress   | no change     | Incident Ticket will remain in the In-Progress state.
On-Hold       | no change     | Incident Ticket will remain in the On-Hold state.
Resolved      | In-Progress   | Incident Ticket state will be changed to In-Progress.
Cancelled     |               | A new Incident Ticket will be created.
Closed        |               | A new Incident Ticket will be created.
