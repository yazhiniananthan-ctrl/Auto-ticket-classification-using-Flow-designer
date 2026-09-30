# AUTO TICKET CLASSIFICATION USING FLOW DESIGNER

## Project Description

**Auto Ticket Classification Using Flow Designer** is an automation-based project developed using **ServiceNow Flow Designer** to simplify and improve the process of managing IT support tickets. In a traditional ticket-management system, support teams may need to manually review incoming tickets, identify their category, determine priority, and assign them to the appropriate support group. This manual process can consume time and may result in inconsistent classification.

The proposed system automates these activities using a workflow created in **ServiceNow Flow Designer**. When a new ticket is created, the flow is automatically triggered and evaluates the available ticket information, such as the issue description, category, urgency, and other relevant fields. Based on predefined conditions and business rules, the system automatically classifies the ticket and determines the appropriate category, priority, and assignment group.

After classification, the workflow can automatically update the ticket fields, assign the ticket to the appropriate support team, and trigger notifications or other required actions. This reduces repetitive manual work and helps support teams process incoming tickets more efficiently.

## Objectives

The main objectives of the project are:

* To automate the classification of support tickets.
* To reduce manual effort involved in ticket processing.
* To assign appropriate categories and priorities to tickets.
* To route tickets to the relevant support groups automatically.
* To improve consistency in ticket classification.
* To reduce delays in ticket assignment and processing.
* To provide an easily configurable workflow using ServiceNow Flow Designer.

## Working Principle

The system follows a simple automated workflow:

**Ticket Creation → Flow Trigger → Ticket Information Analysis → Classification → Priority Assignment → Support Group Assignment → Notification/Action → Ticket Processing**

When a ticket is submitted, the Flow Designer detects the new record and starts the configured flow. The flow evaluates the ticket information against predefined conditions. According to the identified issue, the ticket is categorized and assigned an appropriate priority. It is then routed to the relevant support group for further processing.

## Technologies Used

* **ServiceNow**
* **Flow Designer**
* **ServiceNow Tables and Records**
* **Business Rules/Conditions**
* **Notifications**
* **Workflow Automation**

## Expected Outcome

The project provides an automated and structured approach to ticket classification and routing. By reducing manual classification activities, it helps improve operational efficiency, provides faster assignment of tickets to the appropriate teams, and creates a more consistent ticket-management process.

## Conclusion

**Auto Ticket Classification Using Flow Designer** demonstrates how ServiceNow automation can be applied to IT service management. The project uses Flow Designer to automate ticket classification, prioritization, assignment, and related actions. The solution can be extended with additional rules, integrations, and intelligent classification techniques to support more complex ticket-management requirements.
