# Chat - Workflow Design App

=======================

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

============================

/share ticketflower/settings.py ticketflower/urls.py users/models.py users/middleware.py workflow_design/models.py workflow_design/views.py workflow_design/urls.py workflow_design/tests.py workflow_design/apps.py workflow_design/admin.py workflow_design/templates/workflow_design/base.html 

workflow_design/templates/workflow_design/workflow_definitions_list.html 
workflow_design/templates/workflow_design/workflow_definition_detail.html workflow_design/templates/workflow_design/task_definition_detail.html workflow_design/templates/workflow_design/ticket_definition_detail.html workflow_design/templates/workflow_design/transition_detail.html  

workflow_design/templates/workflow_design/workflow_components_upload.html
 
==========================

We have just completed working on some upload and display pages for workflow design for the TicketFlower server. Now I want to do some manual tests.

In order to do this, I want to create a few workflows that I can update.

First, I would like to you to review the included material, especially the document 'Workflow Definition Guide'. When you have completed that, please ask me any questions you have about the workflows in TicketFlower and on creating workflows.

======================

> Coder_Assistant: Certainly! I've reviewed the provided material, including the Workflow Definition Guide. The TicketFlower system seems to be a comprehensive workflow and ticket management platform with a focus on AI assistance for setup and execution.

Based on the information provided, I understand that workflows in TicketFlower consist of several components: Workflow Definitions, Task Definitions, Ticket Definitions, and Workflow Transitions. The system appears to be designed with multi-tenancy in mind, allowing different companies to have their own isolated workflows and tickets.

Before we proceed with creating workflows for manual testing, I have a few questions to ensure I understand the system correctly:

1. Are there any predefined task types or statuses that we should consider when creating workflows, or can we define these freely?

2. For the Workflow Transitions, how detailed should the 'condition' field be? Is it expected to be a simple text description, or should it contain some form of logic or code?

3. Are there any limitations on the number of tasks, transitions, or tickets that can be associated with a single workflow?

4. How should we handle the 'company_role_access' field in the Ticket Definition? Is there a predefined set of roles, or can we create custom roles for testing purposes?

5. For the various JSON fields (like 'ext_ref', 'fields', 'input_schema', etc.), are there any specific formats or structures we should follow?

Once I have clarity on these points, I'll be better equipped to help you create some test workflows. Do you have any specific types of workflows in mind that you'd like to create for testing?

==========================

1. workflow definition status and task type enumerations:

    - workflow definition status:
        - 'DRAFT' - A workflow definition in process of creation
        - 'ACTIVE' - An active workflow definition
        - 'ARCHIVED' - An archived workflow definition
    - task type:
        - 'FORM' - This is a task that is executed by a task worker filling out a form.
        - 'SYNC_ACTION' - This is a task the is executed by calling an API returning the task output values. **NOT SUPPORTED CURRENTLY**
        - 'ASYNC_ACTION' - This is a task the is executed by calling an API, which will asynchronously call back to set the task output values. **NOT SUPPORTED CURRENTLY**

2. I'm having trouble finding the documentation on the transition condition. This should be a JSON object defining conditions based on the workflow fields JSON. It appears it was set as a text field. Let discuss this.

3. There is no current limit on the count of tasks, transitions or tickets in a workflow.

4. For the time being, let's just require the company role "user", and not worry about permissions on tickets.

5. JSON field formats:
    - ext_ref: Array of {Label: String, Desc: String, URI: String}
    - WorkflowDefinition fields: JSON schema, list of types (!)
    - TaskDefintion input_schema: JSON schema
    - TaskDefintion output_schema: JSON schema
    - TaskDefintion input_transform: JSON schema
    - TaskDefintion output_transform: JSON schema
    - TicketDefinition fields: JSON schema, list of types (!)
    - TicketDefinition company_role_access: Array of company roles names (?)
    - WorkflowTransitions condition: JSON condition (!)

As you can see, there appear to be a few issues in the JSON field formats. Here are some questions for you.

- Would JSON schema make better sense for the 'fields', similar toe 'input_schema' and 'output_schema'?
- Conditions should have a been a JSOn condition. Are you familiar with that type? We should update this.
- Does using a JSON field for a list of role names make sense?

Before we propose any changes, let's discuss these questions.

========================

> Coder_Assistant: Thank you for providing that detailed information. Let's discuss the questions you've raised:

1. JSON schema for 'fields':
Yes, using JSON schema for the 'fields' in both WorkflowDefinition and TicketDefinition would make more sense and provide better consistency with the input_schema and output_schema fields in TaskDefinition. JSON schema allows for more detailed type definitions, constraints, and validations, which would be beneficial for defining the structure of workflow and ticket fields.

2. JSON condition for transitions:
I'm familiar with the concept of JSON conditions, though the exact implementation can vary. Typically, a JSON condition would be a structured way to represent logical conditions using JSON syntax. For example:

```json
{
  "operator": "and",
  "conditions": [
    {"field": "status", "operator": "equals", "value": "approved"},
    {"field": "amount", "operator": "greaterThan", "value": 1000}
  ]
}
```

This approach would indeed be more structured and easier to process programmatically than a text field. It would allow for more complex conditions and easier integration with automated systems.

3. JSON field for role names:
Using a JSON field for a list of role names is reasonable, especially if you want to maintain flexibility. However, if the role names are always going to be simple strings, a more straightforward approach might be to use a PostgreSQL array field of strings. This would be more efficient for querying and indexing. If you anticipate needing more complex role definitions in the future, keeping it as a JSON field might be more future-proof.

Given these considerations, here are my recommendations:

1. Update the 'fields' in WorkflowDefinition and TicketDefinition to use JSON schema.
2. Change the 'condition' field in WorkflowTransitions to a JSONField and implement a JSON condition structure.
3. For company_role_access, consider using a PostgreSQL ArrayField of strings if the roles will always be simple names, or keep it as a JSONField if you anticipate more complex role structures in the future.

These changes would improve the consistency and flexibility of your data model. Would you like me to propose specific changes to the models to implement these recommendations?

==============

Yes, let's do those updates. (I assume no update is needed for 'fields' except a documentation update?)

For the company role access, would it be possible to specify no roles for company_role_access and interpret this to mean no specific role is needed for the ticket?

=============

> Coder_Assistant: Certainly! I'll propose the updates for the models based on our discussion. You're correct that for 'fields', we mainly need a documentation update since it's already a JSONField.

Here are the proposed changes:

1. Update `workflow_design/models.py`:

```python
from django.contrib.postgres.fields import ArrayField
from django.db import models
from users.models import Company, CustomUser

class WorkflowDefinition(models.Model):
    # ... (other fields remain the same)
    fields = models.JSONField(blank=True, null=True, help_text="JSON schema defining the fields for this workflow")
    # ... (rest of the model remains the same)

class TaskDefinition(models.Model):
    # ... (no changes needed)

class TicketDefinition(models.Model):
    # ... (other fields remain the same)
    fields = models.JSONField(blank=True, null=True, help_text="JSON schema defining the fields for this ticket type")
    company_role_access = ArrayField(
        models.CharField(max_length=100),
        blank=True,
        null=True,
        help_text="List of company role names that have access. If empty, no specific role is needed."
    )
    # ... (rest of the model remains the same)

class WorkflowTransitions(models.Model):
    # ... (other fields remain the same)
    condition = models.JSONField(
        null=True,
        blank=True,
        help_text="JSON condition for this transition"
    )
    # ... (rest of the model remains the same)
```

2. Update the docstring or comments in the models to reflect these changes:

```python
class WorkflowDefinition(models.Model):
    """
    Represents a workflow definition.

    fields: JSON schema defining the structure of custom fields in the workflow.
    Example:
    {
        "type": "object",
        "properties": {
            "priority": {
                "type": "string",
                "enum": ["low", "medium", "high"]
            },
            "due_date": {
                "type": "string",
                "format": "date"
            }
        }
    }
    """
    # ... (rest of the model)

class TicketDefinition(models.Model):
    """
    Describes the structure of tickets that can be created for this workflow.

    fields: JSON schema defining the structure of fields for this ticket type.
    company_role_access: List of company role names that have access to this ticket type.
                         If empty, no specific role is needed for access.
    """
    # ... (rest of the model)

class WorkflowTransitions(models.Model):
    """
    Defines the flow between tasks in a workflow.

    condition: JSON condition for this transition.
    Example:
    {
        "operator": "and",
        "conditions": [
            {"field": "status", "operator": "equals", "value": "approved"},
            {"field": "amount", "operator": "greaterThan", "value": 1000}
        ]
    }
    """
    # ... (rest of the model)
```

These changes accomplish the following:

1. Update the documentation for 'fields' in WorkflowDefinition and TicketDefinition to clarify that it's a JSON schema.
2. Change the 'condition' field in WorkflowTransitions to a JSONField with appropriate documentation.
3. Update company_role_access to an ArrayField of strings, which can be null or empty to indicate no specific role is needed.

After making these changes, you'll need to create and run a migration to update the database schema. You may also need to update any forms or serializers that interact with these models to handle the new field types correctly.

Would you like me to prepare the migration file for these changes?

================

I made the updates for the database columns. (I held off on the inline docs. I will put that elsewhere.)

Please review the views.py file and see if there are any modifications needed caused by the changes we made:

- WorkflowDefinition fields - alternate JSON format
- TicketDefinition fields - alternate JSON format
- TicketDefinition company_role_access company_role_access: to array of strings
- WorkflowTransitions - to JSON 

======================

> Coder_Assistant: Certainly! I'll review the views.py file and suggest any necessary modifications based on the changes we made to the models. Let's go through each view and see if any adjustments are needed:

1. `workflow_definitions_list`: No changes needed.

2. `workflow_definition_detail`: No changes needed.

3. `task_definition_detail`: No changes needed.

4. `ticket_definition_detail`: We need to update how we display the company_role_access field since it's now an array of strings. Update the template to handle this new format.

5. `transition_detail`: We need to update how we display the condition field since it's now a JSON object. Update the template to handle this new format.

6. `workflow_definition_upload`: No changes needed, but ensure that the form can handle JSON input for the 'fields' attribute.

7. `workflow_components_upload`: We need to update how we handle the form data for TicketDefinition and WorkflowTransitions.

Here are the suggested modifications for the `views.py` file:

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required, permission_required
from django.forms import modelform_factory
from .models import WorkflowDefinition, TaskDefinition, TicketDefinition, WorkflowTransitions
import json

# ... (other imports and unchanged views)

@login_required
@permission_required(['workflow_design.view_ticketdefinition'], raise_exception=True)
def ticket_definition_detail(request, ticket_id):
    ticket = get_object_or_404(TicketDefinition, id=ticket_id, company=request.company)

    context = {
        'ticket': ticket,
        'fields': json.dumps(ticket.fields, indent=2),
        'company_role_access': ticket.company_role_access or []
    }
    return render(request, 'workflow_design/ticket_definition_detail.html', context)

@login_required
@permission_required(['workflow_design.view_workflowtransitions'], raise_exception=True)
def transition_detail(request, transition_id):
    transition = get_object_or_404(WorkflowTransitions, id=transition_id, workflow__company=request.company)

    context = {
        'transition': transition,
        'condition': json.dumps(transition.condition, indent=2) if transition.condition else 'No condition set'
    }
    return render(request, 'workflow_design/transition_detail.html', context)

@login_required
@permission_required(['workflow_design.add_workflowdefinition', 'workflow_design.change_workflowdefinition'], raise_exception=True)
def workflow_definition_upload(request, workflow_id=None):
    # ... (existing code)

    if request.method == 'POST':
        form = WorkflowForm(request.POST, instance=workflow)
        if form.is_valid():
            workflow = form.save(commit=False)
            try:
                # Validate JSON for 'fields'
                if workflow.fields:
                    json.loads(workflow.fields)
            except json.JSONDecodeError:
                form.add_error('fields', 'Invalid JSON format')
            else:
                workflow.company = request.company
                workflow.created_by = request.user
                workflow.save()
                return redirect('workflow_definition_detail', workflow_id=workflow.id)
    else:
        form = WorkflowForm(instance=workflow)

    return render(request, 'workflow_design/workflow_definition_upload.html', {'form': form, 'workflow': workflow})

@login_required
@permission_required(['workflow_design.add_taskdefinition', 'workflow_design.add_workflowtransitions', 'workflow_design.add_ticketdefinition'], raise_exception=True)
def workflow_components_upload(request, workflow_id):
    # ... (existing code)

    if request.method == 'POST':
        # ... (existing code for task_form)

        elif 'submit_transition' in request.POST and transition_form.is_valid():
            transition = transition_form.save(commit=False)
            try:
                # Validate JSON for 'condition'
                if transition.condition:
                    json.loads(transition.condition)
            except json.JSONDecodeError:
                transition_form.add_error('condition', 'Invalid JSON format')
            else:
                transition.workflow = workflow
                transition.save()
                return redirect('workflow_components_upload', workflow_id=workflow.id)

        elif 'submit_ticket' in request.POST and ticket_form.is_valid():
            ticket = ticket_form.save(commit=False)
            try:
                # Validate JSON for 'fields'
                if ticket.fields:
                    json.loads(ticket.fields)
            except json.JSONDecodeError:
                ticket_form.add_error('fields', 'Invalid JSON format')
            else:
                ticket.company = request.company
                ticket.workflow = workflow
                ticket.created_by = request.user
                ticket.save()
                return redirect('workflow_components_upload', workflow_id=workflow.id)

    # ... (rest of the view remains the same)
```

These modifications:

1. Update how we pass the `fields` and `company_role_access` data to the ticket_definition_detail template.
2. Update how we pass the `condition` data to the transition_detail template.
3. Add JSON validation for the `fields` attribute in workflow_definition_upload.
4. Add JSON validation for the `condition` attribute in the transition form processing.
5. Add JSON validation for the `fields` attribute in the ticket form processing.

You'll also need to update the corresponding templates to properly display the JSON data and the array of company roles. Let me know if you want me to provide the template updates as well.

===============

When I review the old code, I did not see any special handling for the existing JSON fields. For example, here is the code for the ticket definition detail. 

```
@login_required
@permission_required(['workflow_design.view_ticketdefinition'], raise_exception=True)
def ticket_definition_detail(request, ticket_id):
    ticket = get_object_or_404(TicketDefinition, id=ticket_id, company=request.company)

    context = {
        'ticket': ticket,
    }
    return render(request, 'workflow_design/ticket_definition_detail.html', context)
```

What is the difference?

===============

> Coder_Assistant: You're absolutely right, and I apologize for the confusion in my previous response. I made an incorrect assumption about the existing code. Let me clarify:

1. Django's template language can handle JSON fields directly. When you pass a model instance to the template, Django's template engine can access and display JSON fields without any special handling in the view.

2. The ArrayField (which we're using for company_role_access) is also handled automatically by Django's ORM and template engine.

Given this, you're correct that we don't need to make any changes to the views for displaying these fields. The existing code you showed is correct and sufficient.

The only potential changes we might need are:

1. In the templates, to ensure we're displaying the JSON and array fields correctly.
2. In the form handling, to ensure we're properly validating and saving JSON input.

For the ticket_definition_detail view, the code you provided is indeed correct and doesn't need to be changed. The same applies to the other views we discussed earlier.

I apologize for the confusion in my previous response. Thank you for bringing this to my attention. Is there anything specific you'd like me to focus on or clarify regarding the views or templates?

================

Do you have acces to the templates? I believe they are in shared files. Could you review those and let me know if any changes are required.

================



