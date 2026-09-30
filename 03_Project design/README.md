1. What is Automatic Ticket Classification?
Automatic ticket classification is the process of having ServiceNow automatically analyze a newly created ticket and determine things such as:

Category

Subcategory

Assignment group

Priority

Impact

Urgency

Service

Configuration item

Classification confidence

Instead of an agent manually reading every incident and deciding where it belongs, Flow Designer automates the routing process.

Example
A user creates:

Short description: "VPN is not connecting"

Description: "I am working from home and Cisco AnyConnect keeps failing."

The automation can determine:

Category          → Network
Subcategory       → VPN
Assignment Group  → Network Support
Impact            → 2 - Medium
Urgency           → 2 - Medium
Priority          → 3 - Moderate
The ticket can then be automatically routed to the appropriate support team.

2. Why automate ticket classification?
In a traditional ServiceNow environment:

User
  ↓
Creates Incident
  ↓
Service Desk Agent
  ↓
Reads description
  ↓
Determines category
  ↓
Determines assignment group
  ↓
Assigns ticket
  ↓
Support team receives ticket
There can be delays because an agent has to manually inspect and route every ticket.

With automation:

User
  ↓
Creates Incident
  ↓
Flow Designer
  ↓
Classification
  ↓
Category/Subcategory
  ↓
Assignment Group
  ↓
Support Team
This can reduce manual routing work and provide more consistent classification.

3. High-Level Architecture
A good implementation looks like this:

                 ┌───────────────────┐
                 │   User creates    │
                 │     Incident      │
                 └─────────┬─────────┘
                           │
                           ↓
                 ┌───────────────────┐
                 │   Flow Designer   │
                 │     Trigger       │
                 └─────────┬─────────┘
                           │
                           ↓
                 ┌───────────────────┐
                 │ Read Incident     │
                 │ Information       │
                 └─────────┬─────────┘
                           │
                           ↓
                 ┌───────────────────┐
                 │ Classification    │
                 │ Logic             │
                 └─────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Category     Subcategory    Assignment
                                         Group
             │             │             │
             └─────────────┼─────────────┘
                           ↓
                 ┌───────────────────┐
                 │ Update Incident   │
                 └─────────┬─────────┘
                           │
                           ↓
                 ┌───────────────────┐
                 │ Notify / Route    │
                 │ Support Team      │
                 └───────────────────┘
4. Main components
The solution can contain the following ServiceNow components:

Component	Purpose
Incident table	Stores tickets
Flow Designer	Orchestrates automation
Trigger	Starts the flow
Conditions	Determines which rule applies
Decision Table	Stores classification logic
Subflow	Makes logic reusable
Assignment Rules	Determines support team
AI/ML	Handles natural-language classification
Custom fields	Store classification results
Notifications	Inform support teams
Reporting	Measure classification performance
5. Step 1 — Define your classification model
Before creating the Flow, decide what your ticket categories are.

For example:

Category	Subcategory	Assignment Group
Hardware	Laptop	EUC Support
Hardware	Printer	Hardware Support
Network	VPN	Network Support
Network	Wi-Fi	Network Support
Software	Outlook	Messaging Support
Software	Office	Application Support
Access	Password	Service Desk
Access	Account Locked	Service Desk
Application	SAP	SAP Support
This becomes your classification model.

6. Step 2 — Identify the input fields
The classification engine needs information from the incident.

The most useful fields are:

Short description
Description
Caller
Business service
Configuration item
Location
