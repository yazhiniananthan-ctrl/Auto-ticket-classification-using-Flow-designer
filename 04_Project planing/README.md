Auto Ticket Classification Using Flow Designer in ServiceNow
1. What is auto ticket classification?
Auto ticket classification means automatically analyzing an incoming ServiceNow ticket and determining values such as:

Category

Subcategory

Assignment group

Priority

Impact

Urgency

Service

Configuration item

Classification source/confidence

For example, a user submits:

Short description: VPN is not connecting
Description: Cisco AnyConnect fails when I try to connect from home.

Instead of a Service Desk agent manually reading and routing it, automation can produce:

Category          = Network
Subcategory       = VPN
Assignment Group  = Network Support
Impact            = 2 - Medium
Urgency           = 2 - Medium
Priority          = 3 - Moderate

ServiceNow's Flow Designer/Workflow Studio is designed around triggers, actions, flow logic, and reusable subflows, which makes it suitable for orchestrating this process. 
S
ServiceNow
+1

2. Business problem
Imagine an organization receives 5,000 incidents every month.

A typical manual process is:

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
Determines subcategory
  ↓
Determines assignment group
  ↓
Assigns ticket
  ↓
Support Team

This creates several problems:

Manual effort

Incorrect assignment

Delayed routing

Inconsistent categorization

Increased Service Desk workload

SLA risk

Repeated work

The automated process becomes:

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

3. Overall architecture
A good architecture is:

                    INCIDENT CREATED
                           |
                           v
                  +------------------+
                  |  FLOW DESIGNER   |
                  |     TRIGGER      |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | Validate Ticket  |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | Prepare Ticket   |
                  | Data              |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | Classification   |
                  | Engine            |
                  +--------+---------+
                           |
             +-------------+-------------+
             |                           |
             v                           v
       Decision Rules                 AI/ML
             |                           |
             +-------------+-------------+
                           |
                           v
                  +------------------+
                  | Confidence /     |
                  | Validation       |
                  +--------+---------+
                           |
                  +--------+--------+
                  |                 |
                  v                 v
             High confidence    Low confidence
                  |                 |
                  v                 v
            Auto classify      Human review
                  |
                  v
          Assignment Group
                  |
                  v
           Update Incident
                  |
                  v
              Logging

ServiceNow's current documentation describes flows as a trigger followed by actions and flow logic, with execution details and error handling available for monitoring. 
S
ServiceNow

4. Main components
You can build the solution using these components:

Component	Purpose
Incident table	Stores tickets
Flow Designer / Workflow Studio	Orchestrates the process
Record trigger	Starts classification
Conditions	Applies business rules
Decision Table	Stores classification rules
Subflow	Makes classification reusable
Predictive Intelligence	AI/ML classification
Update Record	Updates the Incident
Notifications	Notifies support teams
Custom fields	Stores classification information
Reports	Measures performance

5. Step 1 — Define the classification model
Before creating your Flow, define what your organization considers a classification.

For example:

Category	Subcategory	Assignment Group
Hardware	Laptop	EUC Support
Hardware	Printer	Hardware Support
Network	VPN	Network Support
Network	Wi-Fi	Network Support
Network	LAN	Network Support
Software	Outlook	Messaging Support
Software	Office	Application Support
Access	Password	Service Desk
Access	Account Locked	Service Desk
Application	SAP	SAP Support

This mapping becomes the basis of the automation.

6. Step 2 — Determine what information the Flow will analyze
For incident classification, the most useful inputs are generally:

Short description
Description
Caller
Business service
Configuration item
Location
Channel

For example:

Short description:
"Cannot access VPN"

Description:
"I am working from home and Cisco AnyConnect
fails when I enter my credentials."

The classification logic can analyze both fields rather than relying only on the short description.

7. Step 3 — Create the Flow
In current ServiceNow releases, Workflow Studio is the current workflow-authoring experience; the underlying concepts remain familiar from Flow Designer.

Go to:

All
   ↓
Process Automation
   ↓
Workflow Studio
   ↓
New
   ↓
Flow

Create something like:

Name:
Auto Classify Incident

ServiceNow documents creating a flow by adding a trigger and then configuring the actions/logic that follow it. 
S
ServiceNow

8. Step 4 — Configure the trigger
Use a Record trigger against the Incident table.

For example:

Trigger:
Record Created

Table:
Incident [incident]

Conceptually:

WHEN
    Incident is created

THEN
    Start automatic classification

ServiceNow supports record triggers for created, updated, and created-or-updated scenarios. Trigger conditions can also be used to limit which records start the flow. 
S
ServiceNow Developers
+1

Why "Created" is a good starting point
If your requirement is:

"Every new incident should be automatically classified."

then:

Record Created

is straightforward.

If you use:

Created or Updated

you have to be much more careful about repeated executions.

9. Step 5 — Prevent unnecessary execution
You don't necessarily want the Flow running for every possible Incident.

For example, you might only want to classify:

Active = true
AND
Category is empty
AND
Classification status != Completed

Conceptually:

Incident Created
       |
       v
Is Active?
       |
       +---- NO ---> Stop
       |
      YES
       |
       v
Already classified?
       |
       +---- YES ---> Stop
       |
      NO
       |
       v
Continue

Precise trigger conditions are important for performance and to avoid conflicting or repeated flows. ServiceNow specifically recommends defining precise trigger conditions and taking care with update-based triggers to avoid excessive triggering. 
S
ServiceNow

10. Step 6 — Get the ticket information
The trigger provides the Incident record and its data pills.

Conceptually:

Trigger
   |
   +-- Incident
        |
        +-- Number
        +-- Short description
        +-- Description
        +-- Caller
        +-- Category
        +-- Subcategory
        +-- Assignment group
        +-- Impact
        +-- Urgency
        +-- Service
        +-- Configuration item

You can use these data pills as inputs to later actions.

11. Classification method #1 — Keyword rules
The simplest implementation is keyword-based classification.

Example:

IF
Short description contains "VPN"

THEN

Category = Network
Subcategory = VPN
Assignment Group = Network Support

Another:

IF
Description contains "printer"

THEN

Category = Hardware
Subcategory = Printer
Assignment Group = Hardware Support

Another:

IF
Description contains "password"

THEN

Category = Access
Subcategory = Password
Assignment Group = Service Desk

12. Build keyword groups
Instead of having only one keyword per category, use multiple related terms.

VPN
VPN
AnyConnect
GlobalProtect
remote access
VPN connection
VPN client

Printer
printer
printing
print queue
printer offline
cannot print

Password
password
forgot password
reset password
locked account
credentials

Email
Outlook
email
mailbox
Exchange
mail not received

Then:

IF any VPN keyword matches
       ↓
Network / VPN

ELSE IF any printer keyword matches
       ↓
Hardware / Printer

ELSE IF any password keyword matches
       ↓
Access / Password

ELSE IF any email keyword matches
       ↓
Software / Email

ELSE
       ↓
Manual classification

13. Why keyword classification alone isn't enough
Consider this ticket:

"I'm working remotely and cannot access company resources."

There is no word:

VPN

A simple keyword engine might not classify it.

Another example:

"Outlook opens but I can't send messages."

The word "Outlook" helps, but you may need more context to distinguish an email issue from a broader application problem.

This is where Decision Tables and AI/ML become useful.

14. Classification method #2 — Decision Tables
For a larger implementation, I would strongly recommend using a Decision Table rather than creating hundreds of nested IF/ELSE blocks.

ServiceNow supports using Decision Tables from within flows. The table can take flow data as inputs and return results, and ServiceNow documents Decision Tables specifically for use cases such as incident assignment. 
S
ServiceNow

Conceptually:

Flow
 |
 v
Prepare Inputs
 |
 v
Decision Table
 |
 +---- Category
 |
 +---- Subcategory
 |
 +---- Assignment Group
 |
 +---- Other result
 |
 v
Update Incident

15. Example Decision Table
You could have:

Rule	Input/Condition	Category	Subcategory	Assignment
1	VPN	Network	VPN	Network Support
2	Wi-Fi	Network	Wi-Fi	Network Support
3	Printer	Hardware	Printer	Hardware Support
4	Laptop	Hardware	Laptop	EUC Support
5	Outlook	Software	Email	Messaging
6	Password	Access	Password	Service Desk
7	SAP	Application	SAP	SAP Support

The advantage is that the business rules are maintained in the Decision Table rather than buried inside the Flow.

16. Why Decision Tables are better for large implementations
Imagine you have:

10 categories
50 subcategories
20 assignment groups
100+ rules

This:

IF
ELSE IF
ELSE IF
ELSE IF
ELSE IF
...

becomes very difficult to maintain.

Instead:

Flow
 ↓
Decision Table
 ↓
Classification result

The Flow stays relatively simple.

ServiceNow's documentation explicitly supports creating a Decision Table while authoring a flow and then maintaining its actual rules separately. 
S
ServiceNow

17. Classification method #3 — Predictive Intelligence
For a more advanced implementation, use ServiceNow's Predictive Intelligence.

This is different from simple keyword matching.

Predictive Intelligence's classification framework can use machine learning to predict categorical fields—for example, predicting an Incident's category from the short description—and can support automatic categorization and routing. 
S
ServiceNow

Architecture:

Incident
   |
   v
Flow Designer
   |
   v
Predictive Intelligence
   |
   v
Predicted Category
   |
   v
Predicted Subcategory
   |
   v
Assignment

18. How Predictive Intelligence works
Conceptually, you train a model using historical incidents.

For example:

Historical incidents

"VPN doesn't connect"
Category = Network

"Printer is offline"
Category = Hardware

"Outlook won't open"
Category = Software

"Forgot my password"
Category = Access

The model learns relationships between the input text and the classification values.

Then a new ticket arrives:

"My GlobalProtect connection keeps failing."

The model may predict:

Category = Network

rather than requiring an exact "VPN" keyword.

ServiceNow describes Predictive Intelligence classification as a machine-learning framework for predicting categorical field values and routing work based on historical record-handling patterns. 
S
ServiceNow

19. Training data is important
The quality of the model depends heavily on your historical data.

If historical incidents look like:

VPN issue → Network
VPN issue → Software
VPN issue → Hardware
VPN issue → Network

the model may have difficulty learning a consistent relationship.

You should therefore clean and normalize historical data before relying heavily on ML classification.

ServiceNow's Predictive Intelligence documentation describes defining the training records, input fields, target/output field, and retraining frequency as part of the predictive model setup. 
S
ServiceNow

20. Classification confidence
An advanced implementation can use a confidence or prediction-quality check.

Conceptually:

AI Prediction
     |
     v
Category = Network
     |
     v
Confidence / quality check
     |
     +------ High ------> Auto classify
     |
     +------ Low -------> Human review

For example:

Prediction:
Network / VPN

Confidence:
High

→ Automatically update.

But:

Prediction:
Application / SAP

Confidence:
Low

→ Send for manual review.

The actual threshold should be established using your organization's historical test data rather than choosing an arbitrary number.

21. Human-in-the-loop design
This is one of the most important production concepts.

Don't design the system as:

AI says it → automatically accepted

Instead:

                  New Incident
                       |
                       v
                  Classifier
                       |
                       v
                  Prediction
                       |
             +---------+---------+
             |                   |
             v                   v
        Acceptable           Uncertain
             |                   |
             v                   v
       Auto classify        Agent review
             |                   |
             +---------+---------+
                       |
                       v
                Final classification

This gives the Service Desk control over exceptions.

22. Complete Flow example
Let's build a realistic example.

Incoming incident
Number:
INC0012345

Short description:
VPN is not connecting

Description:
Cisco AnyConnect fails when I try to connect
from home.

Flow
1. Incident Created
          ↓
2. Validate Incident
          ↓
3. Read Short Description
          ↓
4. Read Description
          ↓
5. Classify
          ↓
6. Category = Network
          ↓
7. Subcategory = VPN
          ↓
8. Assignment Group = Network Support
          ↓
9. Update Incident
          ↓
10. Add Work Note
          ↓
11. End

23. What the Incident looks like afterward
Before automation:

Number:
INC0012345

Short description:
VPN is not connecting

Category:
Empty

Subcategory:
Empty

Assignment group:
Empty

After automation:

Number:
INC0012345

Short description:
VPN is not connecting

Category:
Network

Subcategory:
VPN

Assignment group:
Network Support

24. Assignment logic
Classification and assignment should normally be connected.

For example:

Category        Subcategory       Assignment
------------------------------------------------
Network         VPN               Network Support
Network         Wi-Fi             Network Support
Hardware        Laptop            EUC Support
Hardware        Printer           Hardware Support
Software        Email             Messaging
Application     SAP               SAP Support
Access          Password          Service Desk

The classification result therefore drives routing.

25. Priority
You can also include impact and urgency.

For example:

Incident
   |
   +-- Impact
   |
   +-- Urgency
          |
          v
       Priority

However, don't unnecessarily duplicate ServiceNow's existing priority calculation logic.

If your instance already calculates priority from Impact/Urgency, your classification Flow can set the appropriate input fields and let the established platform/business logic calculate the resulting priority.

26. Error handling
Suppose the Decision Table or AI classification fails.

Your Flow should have an error path:

Classification failed
       |
       v
Set classification status
       |
       v
Route to Service Desk
       |
       v
Add work note
       |
       v
Log error

ServiceNow flows provide an error-handler section for catching and responding to errors. 
S
ServiceNow

27. Fallback classification
Always create a fallback.

For example:

IF classification found
       ↓
Update incident

ELSE
       ↓
Classification status = Manual Review
       ↓
Assignment Group = Service Desk

Consider this ticket:

"System isn't working."

There isn't enough information.

The system should not confidently guess.

28. Handling multiple matches
Consider:

"SAP cannot connect through VPN."

Possible matches:

SAP
VPN
Network
Application

You need rule precedence.

For example:

1. SAP + VPN specific rule
2. VPN rule
3. SAP rule
4. Generic Network rule
5. Generic Application rule

This avoids unpredictable results.

29. Subflow architecture
Instead of putting everything in one Flow, use reusable Subflows.

Main Flow
Incident Created
      ↓
Validate
      ↓
Classify Incident
      ↓
Determine Assignment
      ↓
Update Incident
      ↓
Notify

Classification Subflow
Inputs:

short_description
description
service
configuration_item

Outputs:

category
subcategory
assignment_group
classification_status
classification_method
confidence

Then:

MAIN FLOW
     |
     v
CLASSIFICATION SUBFLOW
     |
     +---- category
     +---- subcategory
     +---- group
     +---- confidence
     |
     v
UPDATE INCIDENT

Flow Designer supports reusable subflows as a core workflow component. 
S
ServiceNow

30. Preventing infinite/repeated processing
This is a very important implementation detail.

Suppose your Flow triggers when an Incident is updated:

Incident Updated
       ↓
Flow changes Category
       ↓
Incident Updated
       ↓
Flow runs again
       ↓
Flow changes Assignment
       ↓
Incident Updated
       ↓
Flow runs again

You can end up with repeated execution.

Use conditions such as:

Classification Status != Completed

or a custom field:

u_auto_classification_processed = false

After classification:

u_auto_classification_processed = true

You can also carefully configure the record trigger's run behavior. ServiceNow provides trigger options for created/updated records and different update execution behaviors. 
S
ServiceNow Developers
+1

31. Agent override
Suppose the system predicts:

Category = Network
Subcategory = VPN

but an agent realizes:

Category = Application
Subcategory = SAP

The agent should be able to override the automation.

A useful custom field is:

Classification Source

Possible values:

Rule
AI
Agent
Integration

Then your automation can respect:

Classification Source = Agent

and avoid overwriting the agent's decision.

32. Logging
For an enterprise implementation, store classification information.

For example:

Classification Status:
Automatically Classified

Classification Method:
Decision Table

Predicted Category:
Network

Predicted Subcategory:
VPN

Final Category:
Network

Final Subcategory:
VPN

For AI:

Classification Method:
Predictive Intelligence

Prediction:
Network / VPN

Prediction quality:
High

Final result:
Network / VPN

This makes auditing and reporting much easier.

33. Reporting
After implementation, don't just ask:

"Is the Flow working?"

Measure it.

Useful metrics:

Automation rate
Automatically classified incidents
-----------------------------------
Total incidents

Manual correction rate
Agent-corrected classifications
--------------------------------
Automatically classified incidents

Assignment accuracy
Correctly assigned incidents
----------------------------
Automatically assigned incidents

Unclassified rate
Unclassified incidents
----------------------
Total incidents

You can also monitor:

Average routing time

Classification accuracy

Assignment accuracy

Manual override percentage

Classification failure rate

AI prediction quality

Tickets routed to fallback

34. Testing
Before production deployment, create a test matrix.

Input ticket	Expected category	Expected group
VPN not connecting	Network / VPN	Network Support
Wi-Fi disconnected	Network / Wi-Fi	Network Support
Printer not printing	Hardware / Printer	Hardware Support
Laptop won't boot	Hardware / Laptop	EUC Support
Outlook cannot send email	Software / Email	Messaging
Forgot password	Access / Password	Service Desk
SAP transaction failing	Application / SAP	SAP Support

Also test ambiguous cases:

"SAP can't connect through VPN"

and poor descriptions:

"Not working"

and duplicate terms:

"VPN VPN VPN"

and tickets with missing descriptions.

35. Recommended production architecture
For a serious implementation, I'd recommend:

                         INCIDENT
                            |
                            v
                   +----------------+
                   | Flow Designer   |
                   | Record Trigger  |
                   +-------+--------+
                           |
                           v
                   +----------------+
                   | Validation     |
                   +-------+--------+
                           |
                           v
                   +----------------+
                   | Classification |
                   | Subflow        |
                   +-------+--------+
                           |
              +------------+------------+
              |                         |
              v                         v
       Decision Table             Predictive
                                  Intelligence
              |                         |
              +------------+------------+
                           |
                           v
                   +----------------+
                   | Validate       |
                   | Result         |
                   +-------+--------+
                           |
                  +--------+--------+
                  |                 |
                  v                 v
              Confident          Uncertain
                  |                 |
                  v                 v
             Auto update        Agent review
                  |                 |
                  +--------+--------+
                           |
                           v
                   Assignment Group
                           |
                           v
                    Update Incident
                           |
                           v
                      Audit/Log
                           |
                           v
                       Reporting

36. Flow Designer vs Business Rule
For this requirement, Flow Designer is a good orchestration layer.

Flow Designer
Good for:

Record-triggered automation

Conditions

Decisions

Notifications

Assignment

Calling subflows

Integrations

Updating records

ServiceNow describes Flow Designer as a low-code automation capability built around reusable flows, subflows, actions, triggers, and conditions. 
S
ServiceNow

Business Rules
Use them when you specifically need server-side database logic or behavior that is better handled synchronously at the record level.

You generally don't want to put your entire classification process into one enormous Business Rule.

37. Simple vs advanced implementation
There are three sensible levels.

Level 1 — Basic
Flow Designer
+
IF/ELSE
+
Keyword matching

Good for a small number of categories.

Level 2 — Enterprise rules
Flow Designer
+
Classification Subflow
+
Decision Table
+
Fallback
+
Logging

Better for a larger ServiceNow environment.

Level 3 — AI-assisted
Flow Designer
+
Classification Subflow
+
Predictive Intelligence
+
Confidence/quality check
+
Human review
+
Decision Table fallback
+
Reporting

This is the more sophisticated architecture.

38. Recommended implementation approach
If you are actually building this in a ServiceNow Developer Instance, I would implement it in this order:

Phase 1
Define Categories
       ↓
Phase 2
Define Subcategories
       ↓
Phase 3
Map Assignment Groups
       ↓
Phase 4
Create Classification Rules
       ↓
Phase 5
Create Decision Table
       ↓
Phase 6
Create Classification Subflow
       ↓
Phase 7
Create Incident Flow
       ↓
Phase 8
Add Fallback
       ↓
Phase 9
Add Logging
       ↓
Phase 10
Test
       ↓
Phase 11
Add AI/ML if required
       ↓
Phase 12
Reporting and optimization

39. The complete example
Let's put everything together.

A user creates:

INC0012345

Short Description:
"Cannot connect to VPN"

Description:
"I am working from home and Cisco AnyConnect
fails when I try to connect."

Flow execution
1. Trigger
   Incident Created

2. Validate
   Active = true
   Classification Status != Completed

3. Extract
   Short Description
   Description
   Service
   CI

4. Classification
   Detect VPN / AnyConnect

5. Decision
   Category = Network
   Subcategory = VPN

6. Assignment
   Assignment Group = Network Support

7. Update
   Update Incident fields

8. Logging
   Classification Method = Decision Table

9. Status
   Classification Status = Completed

10. End

Final Incident:

INC0012345

Category:
Network

Subcategory:
VPN

Assignment Group:
Network Support

Classification:
Automatic

Status:
In Progress

40. The key concept to remember
Think of the system as four separate layers:

                 ┌─────────────────────┐
                 │ 1. TRIGGER          │
                 │ New Incident        │
                 └──────────┬──────────┘
                            ↓
                 ┌─────────────────────┐
                 │ 2. CLASSIFICATION   │
                 │ Rules / AI          │
                 └──────────┬──────────┘
                            ↓
                 ┌─────────────────────┐
                 │ 3. ROUTING         │
                 │ Assignment Group    │
                 └──────────┬──────────┘
                            ↓
                 ┌─────────────────────┐
                 │ 4. UPDATE & AUDIT   │
                 │ Incident + Logging  │
                 └─────────────────────┘

So, Flow Designer is primarily the orchestration layer. The actual classification can be implemented with simple conditions, a Decision Table, or ServiceNow's Predictive Intelligence. ServiceNow's current documentation explicitly supports Decision Tables inside flows and ML-based classification for categorical fields such as Incident category. 
S
ServiceNow
+1

A strong final architecture
Incident Created
      ↓
Flow Designer
      ↓
Validation
      ↓
Classification Subflow
      ↓
Decision Table / Predictive Intelligence
      ↓
Classification Result
      ↓
Confidence / Validation
      ↓
 ┌───────────────┐
 │               │
High            Low
 │               │
 ↓               ↓
Auto            Agent
classify        review
 │               │
 └───────┬───────┘
         ↓
Assignment Group
         ↓
Update Incident
         ↓
Logging
         ↓
Reporting

This approach is scalable because you can start with Flow Designer + Decision Tables and later introduce Predictive Intelligence without completely redesigning the workflow. 
S
ServiceNow
+1



SS
SS
Sources

You’ll get smarter responses and can upload
