# TicketFlower Assistant Workflow Guide

TicketFlower is a ticket and workflow system. Workflows are defined in the DB objects 'WorkflowDefinition', 'TaskDefinition', 'WorkflowTransitions' and 'TicketDefinition', with ticket submission being used to kick off a workflow. Workflow operations use the objects 'WorkflowInstance', 'TaskInstance' and 'TicketInstance' to describe a specific usage of a workflow.

This guide covers how to define a workflow and the associated tickets.

## Workflow Operations

1. Workflows:
    - A workflow can only be accessed by users of the company associated with the workflow.
    - A workflow instance has a set of fields, with a JSON schema given by `WorkflowDefinition.fields`.
    - These fields covers all the stored data for the workflow over the span of its use.
    - In a ticket submission, some subset of the fields of the workflow is populated and the workflow is kicked of.
    - Versioning:
        - If there is any material change to a workflow definition or its constituent objects rendering a potential incompatibility, it is recommended that the old workflow status be set to 'DEPRECATED' and a new instance of the workflow definition be created. This will keep the old workflow instances valid with new workflow instances being in the updated format.
        - If there is no issue of incompatibility with an update to a workflow definition or its constituents, no new workflow definition is needed.  

2. Tasks: Steps in a workflow
    - A task instance takes input arguments defined by the JSON schema given by `TaskDefinition.input_schema`, which pulls data from the workflow fields.
    - A task output is in the format of the JSON schema given by `TaskDefinition.output_schema`. These values are used to populate workflow fields of the same name.
    - In addition to the defined task outputs, the task returns a field `success` and, if false, an error message.
    - Task Types:
        - 'FORM': To execute this task, a task worker fills out a form.
            - The form is a Django Form whose content is given by `TaskDefinition.edit_layout`, in conjunction with values from the task input arguments.
            - The submitted form values are the task outputs.
        - 'SYNC_ACTION': To execute this task, the url `TaskDefinition.action_url` is called, with the return value determining the task outputs.
            - The task input arguments are mapped to the API using the json transform `TaskDefinition.input_transform`.
            - The task output arguments are mapped from the return value using the JSON transform `TaskDefinition.output_transform`.
            - A timeout for this call is given by `TaskDefinition.timeout_seconds`.
        - 'ASYNC_ACTION': To execute this task, the url `TaskDefinition.action_url` is called. The task outputs will be set in a separate callback to the service.
            - The task input arguments are mapped to the url using the json transform `TaskDefinition.input_transform`.
            - A timeout for this call is given by `TaskDefinition.timeout_seconds`.

3. Transitions: Transitions between workflow steps
    - Entering a transition: `WorkflowTransition.from_task` specifies a task that, on completion, triggers this transition check.
        - If the `from_task` is `NULL`, the transition is triggered by a ticket submission kicking off the workflow.
        - For a given initial state, there can be multiple transitions.
        - If a task exits with an error, workflow error handling is invoked and no transition is called.
    - Activating a transition: The JSON condition specified in `WorkflowTransition.condition` is used to determine is this transition is activated.
    - Transitioning to a new task: `WorkflowTransition.to_task` specifies the next task in the workflow.
    - Transitioning to a new status: `WorkflowTransition.to_status` specifies a change in status for the workflow. This signals a completion of the tasks in the workflow.
    - Between the two columns `to_task` and `to_status`, one should be populated and one should be `NULL`.
    - Multiple task transitions:
        - If there are multiple tasks to be transitioned to, they are executed in parallel, processed in arbitrary order.
        - The field `WorkflowTransition.priority` is ignored.
    - Error conditions with multiple transitions:
        - If different transitions indicate different `to_status` values, this is an error condition.
        - If some transitions indicate a status change and others indicate a new task, this is an error condition.

    - NOTES:
        - Tasks are executed in para priority if the tasks are executed sequentially?
 
    - Possible future additions:
        - Should we have an option to do serial versus parallel task execution?
        - Should we have some transitions that are always checked? (We would have to differentiate these from ones that are checked at the start.)
        - Should we add a status like active or inactive? I don't think we need one.

4. Tickets: Initiating a workflow
    - To submit a ticket instance, a form is filled out, given by the Django form content given in `TicketDefinition.edit_layout`.
    - When the user submits a ticket, this creates a new workflow of type `TicketDefinition.workflow`.
        - The fields from the ticket, defined by `TicketDefinition.field`, populate associated fields fom the workflow, defined by `WorkflowDefinition.fields`.
        - The ticket submission triggers a check of all transitions for this workflow with a `NULL` value for `from_task`.
    - There may be multiple ticket definitions associated with a single workflow.
    - Access to tickets within a company is optionally controlled by the list of "company roles" given in `TicketDefinition.company_role_access`
        - If this list of roles is empty or NULL, there is no restriction on access to this ticket for users of this company.
        - If the list of roles is not empty or NULL, a user must have at least one of these company roles to access the ticket.

## Workflow Object Fields

The following objects are used to define workflows and tickets.

### WorkflowDefinition

- **company**: ForeignKey to Company
- **title**: CharField, max length 255
- **description**: TextField, optional
- **ext_ref**: JSONField, optional, Array of {Label: String, Desc: String, URI: String}
- **fields**: JSONField, optional, JSON schema defining the fields for this workflow
- **version**: IntegerField, default 1
- **created_at**: DateTimeField, auto-set on creation
- **updated_at**: DateTimeField, auto-updated
- **created_by**: ForeignKey to CustomUser, optional
- **display_layout**: TextField, optional, Django form content to show the workflow
- **status**: CharField, max length 20
  - Options: 'DRAFT', 'ACTIVE', 'DEPRECATED', 'ARCHIVED'
  - Default: 'DRAFT'

### TaskDefinition

- **company**: ForeignKey to Company
- **workflow**: ForeignKey to WorkflowDefinition
- **title**: CharField, max length 255
- **description**: TextField, optional
- **ext_ref**: JSONField, optional, Array of {Label: String, Desc: String, URI: String}
- **task_type**: CharField, max length 20
  - Options: 'FORM', 'SYNC_ACTION', 'ASYNC_ACTION'
- **input_schema**: JSONField, optional
- **output_schema**: JSONField, optional
- **version**: IntegerField, default 1
- **created_at**: DateTimeField, auto-set on creation
- **updated_at**: DateTimeField, auto-updated
- **created_by**: ForeignKey to CustomUser, optional
- **edit_layout**: TextField, optional, Django form content to edit a task instance
- **display_layout**: TextField, optional, Django form content to display a task instance
- **action_url**: URLField, optional, url for async, sync action
- **input_transform**: JSONField, optional, input transform for async and sync action
- **output_transform**: JSONField, optional, output transform for sync action and async action
- **timeout_seconds**: IntegerField, optional, timeout for sync action

### TicketDefinition

- **company**: ForeignKey to Company
- **title**: CharField, max length 255
- **description**: TextField, optional
- **ext_ref**: JSONField, optional, Array of {Label: String, Desc: String, URI: String}
- **workflow**: ForeignKey to WorkflowDefinition
- **fields**: JSONField, optional, JSON schema defining the fields for this ticket type
- **created_at**: DateTimeField, auto-set on creation
- **updated_at**: DateTimeField, auto-updated
- **version**: IntegerField, default 1
- **created_by**: ForeignKey to CustomUser, optional
- **company_role_access**: ArrayField of CharField, optional, list of company role names that have access
- **edit_layout**: TextField, optional, Django form content to edit a ticket instance
- **display_layout**: TextField, optional, Django form content to display a ticket instance

### WorkflowTransitions

- **workflow**: ForeignKey to WorkflowDefinition
- **from_task**: ForeignKey to TaskDefinition, optional, related_name='transitions_from'
- **to_task**: ForeignKey to TaskDefinition, optional, related_name='transitions_to'
- **to_status**: CharField, max length 50, optional
  - Options: 'NOT_STARTED', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED', 'ON_HOLD'
- **condition**: JSONField, optional, JSON condition for this transition
- **priority**: IntegerField, default 0

## JSON Formats

The following JSON formats are used in various parts of the workflow definition:

1. JSON Schema (for WorkflowDefinition.fields, TaskDefinition.input_schema, TaskDefinition.output_schema, TicketDefinition.fields):
   - Use JSON Schema draft 2020-12 (https://json-schema.org/specification-links.html#2020-12)
   - Schemas should be valid according to this specification

2. JSON Transform (for TaskDefinition.input_transform, TaskDefinition.output_transform):
   - Use JSONPath expressions (https://goessner.net/articles/JsonPath/)
   - Transformations should be expressed as key-value pairs, where the key is the output field and the value is a JSONPath expression

3. JSON Condition (for WorkflowTransition.condition):
   - Use a subset of MongoDB query operators (https://docs.mongodb.com/manual/reference/operator/query/)
   - Supported operators include: $eq, $ne, $gt, $gte, $lt, $lte, $in, $nin, $and, $or, $not
   - Conditions should be expressed using these operators to form logical expressions
