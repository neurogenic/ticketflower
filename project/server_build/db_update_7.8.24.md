# TicketFlower Project Overview

I am updating the workflow process and the DB:

- Remove WorkflowTransitions, the "flow connectors".
- Add an implicit "flow connector" based on the input_fields of a task. There remains a "condition", moved from transmission to task, which indicates if the task should be run.
- Added a workspace completion, where previously this functionality was kept in the workspace transition.

On the ops side:

- Add a workspace data table with an iteration field. This stores the workspace data at each point during the workflow.
- Add a start_iteration and end_iteration field to the task instance. This references the iterations of the Workflow where the task starts (gets its input) and ends (sends its output). An iteration signifies the start of the workflow from a ticket submission or the completion of the single task in the workflow.
    - Note that multiple tasks can start on a single iteration, for the case of parallel task execution.
    - In the case of parallel task execution we will still create a new iteration for each task completion.
- Error case: It is an error if multiple tasks executing at the same time write an output to a common field.
    - This includes cases where two tasks are started at the same iteration or at different iteration.
- Circular references: Circular references are not an error. This workflow is reminiscent of a functional reaction approach but it is different. We are not determining an order of operations to update the state. Instead any triggered operation are executed in parallel. We also allow multiple tasks to write output to the same field, which is not allowed in the functional reactive approach. So this looks similar in some ways but is still a procedural approach. 
    - The approach we're using is similar to an event based approach where we implicitly make the task subscribe to change events for its input variables.
        - The "condition" field allows opting out of running the task in certain cases.
        - The input fields for a task can also be viewed as its dependencies. If you want to add different cases to trigger the task additional input fields/dependencies can be added.





## Schema SQL

Here is the schema as designed in SQL related to workflow_design and workflow_ops. This can be modified as needed to fit Django.

```sql

CREATE TYPE workflow_def_status AS ENUM ('DRAFT', 'ACTIVE', 'ARCHIVED');
CREATE TYPE task_status AS ENUM ('OPEN', 'IN_PROGRESS', 'PENDING', 'COMPLETED', 'CANCELLED', 'ERROR');
CREATE TYPE workflow_status AS ENUM ('NOT_STARTED', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED', 'ON_HOLD');
CREATE TYPE task_type AS ENUM ('FORM', 'SYNC_ACTION', 'ASYNC_ACTION');
CREATE TYPE error_type AS ENUM ('WORKFLOW_CREATION', 'TASK_CREATION', 'TASK_UPDATE', 'WORKFLOW_TRANSITION');
CREATE TYPE error_status AS ENUM ('NEW', 'REVIEWED', 'RESOLVED');

-- Workflow Definition
CREATE TABLE workflow_definitions (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    ext_ref JSONB,  -- Array of {Label: String, Desc: String, URI: String}
    fields_schema JSONB,  -- schema for workspace fields
    status workflow_def_status DEFAULT 'DRAFT',
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by INTEGER REFERENCES users(id)
    display_layout TEXT,  -- react component source, to show the workflow
);

-- Task Definition
CREATE TABLE task_definitions (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    workflow_id INTEGER REFERENCES workflow_definitions(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    ext_ref JSONB,
    task_type task_type NOT NULL,
    input_fields VARCHAR(100) [],
    output_fields VARCHAR(100) [],
    condition JSONB, -- JSON condition to evaluate
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by INTEGER REFERENCES users(id),
    edit_layout TEXT, -- react component source, to edit a task instance
    display_layout TEXT, -- react component source, to display a task instance
    action_url TEXT, -- url for async, sync action
    input_transform JSONB, -- input transform for async and sync action
    output_transform JSONB, -- output transform for sync action (handle response) and async action (process url call)
    timeout_seconds INTEGER -- timeout for sync action
);

-- Ticket Types
CREATE TABLE ticket_definitions (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    ext_ref JSONB,  -- Array of {Label: String, Desc: String, URI: String}
    workflow_id INTEGER REFERENCES workflow_definitions(id),
    fields VARCHAR(100) [],  -- workspace fields submitted
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    version INTEGER DEFAULT 1,
    created_by INTEGER REFERENCES users(id),
    company_role_access VARCHAR(100) [] -- Array of company role names that have access
    edit_layout TEXT,  -- react component source, to edit a ticket instance
    display_layout TEXT,  -- react component source, to display a ticket instance
);

-- Workflow Completions
CREATE TABLE workflow_completion (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    workflow_id INTEGER REFERENCES workflow_definitions(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    ext_ref JSONB,
    completion_status workflow_status NOT NULL,
    fields VARCHAR(100) [],
    condition JSONB, -- JSON condition to evaluate
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by INTEGER REFERENCES users(id),
    display_layout TEXT, -- react component source, to display a task instance
);

-- Workflow Instance
CREATE TABLE workflow_instances (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    workflow_id INTEGER REFERENCES workflow_definitions(id),
    status workflow_status DEFAULT 'IN_PROGRESS',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Workflow Instance
CREATE TABLE workflow_data (
    id SERIAL PRIMARY KEY,
    workflow_id INTEGER REFERENCES workflow_definitions(id),
    iteration: INTEGER,
    workflow_data JSONB
);

-- Task Instance
CREATE TABLE task_instances (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    workflow_instance_id INTEGER REFERENCES workflow_instances(id),
    task_definition_id INTEGER REFERENCES task_definitions(id),
    start_iteration INTEGER, -- iteration from which input data was taken
    end_iteration INTEGER, -- iteration to which output data was written
    status task_status DEFAULT 'PENDING',
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    assigned_to INTEGER REFERENCES users(id)
);

-- Ticket (for actual tickets created by users)
CREATE TABLE ticket_instances (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    ticket_type_id INTEGER REFERENCES ticket_definitions(id),
    workflow_id INTEGER REFERENCES workflow_instances(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by INTEGER REFERENCES users(id),
);

-- Ticket Errors (for errors in workflow execution)
CREATE TABLE ticket_errors (
    id SERIAL PRIMARY KEY,
    ticket_id INTEGER REFERENCES ticket_instances(id) NOT NULL,
    task_id INTEGER REFERENCES task_instances(id),
    error_type error_type NOT NULL,
    error_message TEXT NOT NULL,
    stack_trace TEXT,
    error_status error_status DEFAULT 'New',
    resolution_notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);



CREATE TABLE file_uploads (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    filename VARCHAR(255) NOT NULL,
    version INTEGER DEFAULT 1,
    ext_key TEXT NOT NULL,
    file_type VARCHAR(100),
    uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    uploaded_by INTEGER REFERENCES users(id)
);

```

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
- Task Definition Upload (Create and edit)
- Ticket Definition Upload (Create and edit)
- Workflow Transitions Upload (Create and edit)

The workflow upload pages are just a placeholder to allow population of the workflow pages. Later we will work on workflow development.


================

DB Changes:

Design:

- WorkspaceDefinitions
    - fields: JSONB -> fields_schema: JSONB
        - Simple name update
- TaskDefinitions
    - input_schema: JSONB -> input_fields: VARCHAR(100) []
        - Specify fields in the task, referencing fields and the associated schema from the workspace definition
    - output_schema: JSONB -> output_fields: VARCHAR(100) []
        - Specify fields in the task, referencing fields and the associated schema from the workspace definition
    - ADD condition: JSONB
        - If the task is triggered (by an update to dependency/input field), this condition tells if the task should be executed (formerly in WorkspaceTransition)
- TicketDefinitions:
    - fields: JSONB -> fields: VARCHAR(100) []
        - Specify fields in the ticket, referencing fields and the associated schema from the workspace definition
- WorkflowTransitions: DELETED
    - Transitions are now implicit based on a change in an input field for the task. Conditions can be used to further restrict when the task is run.
- WorkflowCompletion: ADD
    - Previously we the transition doubled as a completion condition. Now we explicitly include workflow completions. They are triggered wimilar to how a task is triggered.

Ops: