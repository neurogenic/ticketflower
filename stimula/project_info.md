TicketFlower Project Info
=========================

# TicketFlower Project Overview

This is an overview of the Ticketflower project, a web site for ticket and workflow management.

The main focus of the site will be on using a AI assistant to help set up
workflows and tickets. Later, the AI as# TicketFlower Project Overview

This is an overview of the Ticketflower project, a web site for ticket and workflow management.

The main focus of the site will be on using a AI assistant to help set up
workflows and tickets. Later, the AI assistant will also help with setting up and executing autonomous tasks as a part of workflows.

The initial target will be for small businesses to easily set up a ticket system.

This will be an MVP. I want to get it up and running quickly. The emphasis will not be on full features for the workflow management. The
main point will be to illustrate the capabilities of the AI assistant. We of course still want workable functionality.

The initial code we write will actual leave out the functionality of the AI assistant. We will be developing that later. Here we will be
building the ticket and workflow framework the AI assistant will fit into.

## Tech Stack

- Django
- Python with Type annotations
- Postgres
- Django's built-in TestCase

Dev setup:

- Environment variables: .env file (with python-dotenv) for dev
- Windows machine + VSCode
- Postgres is running in a docker container


## Overview

The initial target will be for small businesses to easily set up a ticket/order and workflow system.

This will be an MVP. I want to get it up and running quickly. The emphasis will not be on full features for the workflow management. The
main point will be to illustrate the capabilities of the AI assistant. We of course still want workable functionality.

The initial code we write will actual leave out the functionality of the AI assistant. We will be developing that later. Here we will be
building the ticket and workflow framework the AI assistant will fit into.

## Multi-tenancy

Users are restricted to only view assets for their company. User management should contain some support for this.

## Workflow Design App Pages

### Definitions Pages (ticket_admin, workflow_creator roles)

- Workflow Definitions
- Workflow Definition Detail
    - workflow definition fields
    - tasks
    - transitions
    - tickets
- Task Definition Detail
- Ticket Definition Detail
- Transition Detail


### Workflow Upload Pages (workflow_creator role)

- Workflow Definition Upload (Create and edit)
- Task/Ticket.Trainsition Definition Upload - For a given workflow (Create and edit)

The workflow upload pages are just a placeholder to allow population of the workflow pages. Later we will work on workflow development.

---

# Workflow Definition Guide

This guide outlines the components of a workflow definition in the TicketFlower system, including workflows, tasks, transitions, and tickets.

## Workflow Definition

A workflow definition is the top-level object that describes a complete process.

Fields:
- Title: A short, descriptive name for the workflow
- Description: A detailed explanation of the workflow's purpose and function
- External References: JSON array of {Label: String, Desc: String, URI: String} for any external resources
- Fields: JSON array of {FieldName: String, FieldType: String} for custom fields in the workflow
- Version: Integer representing the version of this workflow definition
- Status: One of 'DRAFT', 'ACTIVE', or 'ARCHIVED'
- Display Layout: Django form content to display the workflow

## Task Definition

Tasks are individual steps within a workflow.

Fields:
- Title: A short, descriptive name for the task
- Description: A detailed explanation of what the task involves
- External References: JSON array of {Label: String, Desc: String, URI: String} for any external resources
- Task Type: One of 'FORM', 'SYNC_ACTION', or 'ASYNC_ACTION'
- Input Schema: JSON schema describing the expected input for this task
- Output Schema: JSON schema describing the expected output from this task
- Version: Integer representing the version of this task definition
- Edit Layout: Django form content for editing a task instance
- Display Layout: Django form content for displaying a task instance
- Action URL: URL for async or sync actions (if applicable)
- Input Transform: JSON describing how to transform input data (if applicable)
- Output Transform: JSON describing how to transform output data (if applicable)
- Timeout Seconds: Integer for timeout duration (for sync actions)

## Workflow Transitions

Transitions define the flow between tasks in a workflow.

Fields:
- From Task: The task this transition starts from (can be null for starting transitions)
- To Task: The task this transition leads to (can be null for ending transitions)
- To Status: The workflow status to transition to (used when To Task is null)
- Condition: A text field describing the condition for this transition
- Priority: Integer for ordering multiple possible transitions

Rules for Transitions:
1. A transition must have either a To Task or a To Status, but not both.
2. If From Task is null, it represents a starting point for the workflow.
3. If To Task is null and To Status is set, it represents an ending point for the workflow.
4. Multiple transitions can come from the same From Task, but they should have different conditions or priorities.
5. The Condition field can be used to specify when this transition should be taken (e.g., based on task output or workflow data).
6. Priority determines the order in which conditions are evaluated when multiple transitions are possible.

## Ticket Definition

Ticket definitions describe the structure of tickets that can be created for this workflow.

Fields:
- Title: A short, descriptive name for the ticket type
- Description: A detailed explanation of what this ticket type represents
- External References: JSON array of {Label: String, Desc: String, URI: String} for any external resources
- Fields: JSON object of {FieldName: Value} describing the fields for this ticket type
- Version: Integer representing the version of this ticket definition
- Company Role Access: JSON array of company role names that have access to this ticket type
- Edit Layout: Django form content for editing a ticket instance
- Display Layout: Django form content for displaying a ticket instance

## Best Practices

1. Start with a clear understanding of the entire process before defining the workflow.
2. Break down the process into distinct, manageable tasks.
3. Clearly define the input and output for each task.
4. Use meaningful names and descriptions for all components.
5. Consider all possible paths through the workflow, including error conditions and edge cases.
6. Use the condition field in transitions to create dynamic, data-driven workflows.
7. Leverage the priority field in transitions to control the flow when multiple transitions are possible.
8. Design ticket definitions to capture all necessary information for the workflow.
9. Use the company role access field in ticket definitions to control who can interact with different types of tickets.
10. Utilize the layout fields (display_layout, edit_layout) to create user-friendly interfaces for interacting with workflows, tasks, and tickets.

Remember, a well-designed workflow should be clear, efficient, and flexible enough to handle various scenarios within your business process.