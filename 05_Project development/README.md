Auto Ticket Classification Using Flow Designer in ServiceNow
Auto ticket classification in ServiceNow is the process of automatically analyzing an incoming incident and determining its category, subcategory, assignment group, priority-related attributes, and other routing information without requiring a Service Desk agent to classify the ticket manually.

A practical implementation can use Flow Designer/Workflow Studio + Decision Tables + business rules, and, for more advanced implementations, Predictive Intelligence/AI-based classification.

1. Business scenario
Assume a company receives incidents such as:

"VPN is not connecting from my laptop."

"My Outlook is not sending emails."

"The printer is showing offline."

"I forgot my password."

Without automation, an agent has to read each ticket and determine where it belongs.

With automation:

User creates Incident
        ↓
Flow Designer
        ↓
Read ticket information
        ↓
Classify ticket
        ↓
Category / Subcategory
        ↓
Determine Assignment Group
        ↓
Update Incident
        ↓
Notify support team
For example:

Ticket	Category	Subcategory	Assignment Group
VPN not connecting	Network	VPN	Network Support
Printer offline	Hardware	Printer	Hardware Support
Outlook not sending mail	Software	Email	Messaging Support
Forgot password	Access	Password	Service Desk
SAP transaction error	Application	SAP	SAP Support
2. What is Flow Designer?
Flow Designer is ServiceNow's low-code workflow automation capability.

A Flow generally consists of:

Trigger
   ↓
Conditions / Flow Logic
   ↓
Actions
   ↓
Subflows / Decisions
   ↓
Record Updates
For this project, Flow Designer acts as the orchestration layer.

It can:

Detect a new Incident

Read Incident fields

Evaluate conditions

Call a Decision Table

Call a reusable Subflow

Update the Incident

Assign the Incident

Send notifications

Handle errors

Record the classification result

3. Overall architecture
A production-oriented solution can look like this:

                       ┌──────────────────┐
                       │ User creates     │
                       │ Incident        │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ Flow Designer    │
                       │ Record Trigger   │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ Validate Ticket  │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ Classification   │
                       │ Subflow          │
                       └────────┬─────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    ▼                       ▼
             Decision Table          AI / ML Model
                    │                       │
                    └───────────┬───────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ Validate Result  │
                       └────────┬─────────┘
                                │
                         ┌──────┴──────┐
                         │             │
                         ▼             ▼
                      Confident     Uncertain
                         │             │
                         ▼             ▼
                   Auto classify   Agent review
                         │
                         ▼
                 Assignment Group
                         │
                         ▼
                  Update Incident
                         │
                         ▼
                     Reporting
4. Components required
A typical implementation contains:

Component	Purpose
Incident table	Stores tickets
Flow Designer	Controls the automation
Record Trigger	Starts the Flow
Conditions	Determines whether classification should run
Decision Table	Contains classification rules
Subflow	Encapsulates reusable classification logic
Predictive Intelligence	Optional ML classification
Update Record	Updates Incident fields
Assignment Group	Receives the classified ticket
Custom fields	Store classification status/source
Reports	Measure performance
5. Step 1 — Define the classification requirements
Before building anything, define what your organization considers a classification.

For example:

Category
   ↓
Subcategory
   ↓
Assignment Group
Example:

Network
 ├── VPN
 ├── Wi-Fi
 └── LAN

Hardware
 ├── Laptop
 ├── Desktop
 └── Printer

Software
 ├── Outlook
 ├── Microsoft Office
 └── Browser

Access
 ├── Password
 ├── Account Locked
 └── Access Request

Application
 ├── SAP
 ├── Salesforce
 └── HR Application
Then map those classifications to support groups.

6. Step 2 — Define your classification matrix
Create something like:

Category	Subcategory	Assignment Group
Network	VPN	Network Support
Network	Wi-Fi	Network Support
Network	LAN	Network Support
Hardware	Laptop	EUC Support
Hardware	Printer	Hardware Support
Software	Email	Messaging Support
Software	Office	Application Support
Access	Password	Service Desk
Access	Account Locked	Service Desk
Application	SAP	SAP Support
This becomes the business requirement for the Flow.

7. Step 3 — Identify input fields
The classification engine needs information from the Incident.

The most useful fields are:

Short description
Description
Caller
Business service
Configuration item
Location
Channel
For example:

Short description:
VPN is not connecting

Description:
Cisco AnyConnect fails when I try to connect
from home.
The Flow can analyze both:

Short description
+
Description
rather than depending only on one field.

8. Step 4 — Create the Flow
In ServiceNow, depending on your release and interface, you'll access this through Flow Designer/Workflow Studio.

Create a new Flow:

Name:
Auto Classify Incident
Conceptually:

Auto Classify Incident
        |
        +-- Trigger
        |
        +-- Validate
        |
        +-- Classify
        |
        +-- Determine Assignment
        |
        +-- Update Incident
        |
        +-- Log Result
9. Step 5 — Configure the trigger
Use a ServiceNow record trigger.

For example:

Table:
Incident [incident]

When:
Created
The logic is:

WHEN
    A new Incident is created

THEN
    Start automatic classification
You could also use an update trigger, but that needs more careful design because changing the Incident inside the Flow can itself cause another update.

For a first implementation, Record Created is easier to control.

10. Step 6 — Add trigger conditions
You don't necessarily want every Incident to be classified repeatedly.

For example:

Active = true
AND
Category is empty
AND
Classification Status != Completed
So:

Incident Created
       ↓
Is it eligible?
       ↓
     YES
