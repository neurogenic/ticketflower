# Workflow Chat 7/2/24

======================

# TicketFlower Project Info

TicketFlower is a ticketing and workflow system. We have completed an initial version of user management and now we will work on the workflows application of our Django server.

Dev machine: Windows PC + VSCode

## Workflows Schema Design

CREATE TYPE task_status AS ENUM ('OPEN', 'IN_PROGRESS', 'PENDING', 'COMPLETED', 'CANCELLED', 'ERROR');
CREATE TYPE workflow_status AS ENUM ('NOT_STARTED', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED', 'ON_HOLD');
CREATE TYPE task_type AS ENUM ('FORM', 'SYNC_ACTION', 'ASYNC_ACTION');

-- Workflow Definition
CREATE TABLE workflow_definitions (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    ext_ref JSONB,  -- Array of {Label: String, Desc: String, URI: String}
    fields JSONB,  -- Array of {FieldName: String, FieldType: String}
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
    input_schema JSONB,
    output_schema JSONB,
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

-- Workflow Transitions
CREATE TABLE workflow_transitions (
    id SERIAL PRIMARY KEY,
    workflow_id INTEGER REFERENCES workflow_definitions(id),
    from_task_id INTEGER REFERENCES task_definitions(id),
    to_task_id INTEGER REFERENCES task_definitions(id),
    condition TEXT, -- JSON condition to evaluate
    priority INTEGER DEFAULT 0
);

-- Ticket Types
CREATE TABLE ticket_definitions (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    ext_ref JSONB,  -- Array of {Label: String, Desc: String, URI: String}
    workflow_id INTEGER REFERENCES workflow_definitions(id),
    fields JSONB,  -- Object of {FieldName: Value}
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    version INTEGER DEFAULT 1,
    created_by INTEGER REFERENCES users(id),
    company_role_access VARCHAR(100) [] -- Array of company role names that have access
    edit_layout TEXT,  -- react component source, to edit a ticket instance
    display_layout TEXT,  -- react component source, to display a ticket instance
);

-- Workflow Instance
CREATE TABLE workflow_instances (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    workflow_id INTEGER REFERENCES workflow_definitions(id),
    current_task_id INTEGER REFERENCES task_definitions(id),
    workflow_data JSONB,
    w_status workflow_status DEFAULT 'IN_PROGRESS',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Task Instance
CREATE TABLE task_instances (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    workflow_instance_id INTEGER REFERENCES workflow_instances(id),
    task_definition_id INTEGER REFERENCES task_definitions(id),
    t_status task_status DEFAULT 'PENDING',
    input_data JSONB,
    output_data JSONB,
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
    ticket_data JSONB,  -- Actual data submitted for this ticket
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by INTEGER REFERENCES users(id),
);

## Workflow Functions

NOTE: This is a multi-tenant web site. Authorization for the workflow function requires a user to have the proper system role PLUS the user must be in the proper company.

System roles: 'admin', 'user', 'ticket_admin', 'task_worker', 'workflow_creator'

### General User Pages

Ticket submission functionality for general users in a given company.

- Submit Tickets
- My Tickets
- Ticket Detail

System role: 'user'
Company role: The customer has the option of assigning company roles to restrict access to specific tickets

### Ticket Admin

Access to workflow tickets and tasks. A ticket display will show the associated workflow.

- Tickets
- Ticket Detail
- Tasks
- Task Detail

We will support different options for assigning and tracking tickets, but for starters I think we will automatically give them to the available worker with the shortest list. Later we can think about giving options here. (Also, more advanced ticket tracking and analytics will be added later.)

System role: 'ticket_admin'

### Task Workers

- My Tasks (To execute)
- Task Execution (For manual/Form type tasks)

System role: 'task_worker'

### Workflow Creators

This will be for workflow designers. For now we will build out minimal functionality to upload workflow, task and ticket definitions. Later this will form the heart of the services value proposition.

- Workflow Definitions
- Workflow Definition Detail
- Task Definitions
- Task Definition Detail
- Ticket Definitions
- Ticket Definition Detail

- Workflow Definition Upload (Create and edit)
- Task Definition Upload (Create and edit)
- Ticket Definition Upload (Create and edit)

System role: 'workflow_creator'

===================

# Shared Files

/share requirements.txt ticketflower/settings.py ticketflower/urls.py users/models.py 

===============

I've shared some files and some information about the ticketing and workflow project, TicketFlower. The site currently has some user management built out, including companies and users.

I'd like to get started on planning out the workflows for the server. I have listed the overall workflow schema and features in the project info from the context message.

I would appreciate brief responses so we don't overload your context length too quickly. I would also like to work one step at a time.

I am not especially experienced on Django and how applications are split out within a project.I was thinking we might want to split this into two parts, Design and operations. In design, the workflow creators would create the workflow, task and ticket definitions. In operations tickets would be submitted and executed.

Maybe we call the applications 'workflow_design' and 'workflow_operations'? I'd like to hear your thoughts and ideas on the design.

At this point I don't write any code.

=====================

> Coder_Assistant: Your idea to split the functionality into two applications, 'workflow_design' and 'workflow_operations', is a good approach. This separation of concerns aligns well with Django's philosophy and will make the project more manageable. Here's a brief overview of how we could structure this:

1. workflow_design:
   - Handle creation and management of workflow definitions, task definitions, and ticket definitions
   - Used by users with the 'workflow_creator' role

2. workflow_operations:
   - Handle ticket submission, task execution, and workflow progression
   - Used by general users, task workers, and ticket admins

This separation will allow for better organization of code, easier maintenance, and clearer permissions management.

To proceed, we should:
1. Create these two Django applications
2. Define models in each application
3. Set up views and URLs
4. Implement permissions and authentication

Shall we start by creating these two applications in the Django project?

=====================

I am thinking about starting a new conversation to just focus on the 'workflow_design' operation. This will remove some of the overhead from the other schema and functionality I've displayed.

Suppose I do this, preparing a new "project info" file. What information do you think would be most important to get started on the 'workflow_design' application. Specify which schema tables should be included, and feel free to craft text that could be included in project info.

======================

> Coder_Assistant: Focusing on the 'workflow_design' application is a good approach. Here's what I think should be included in the new project info for 'workflow_design':

1. Schema tables:
   - workflow_definitions
   - task_definitions
   - workflow_transitions
   - ticket_definitions

2. Key information:
   - Brief description of the 'workflow_design' application's purpose
   - System roles (specifically 'workflow_creator')
   - Main functionalities for workflow creators

Here's a draft of text you could include in the new project info:

```
# TicketFlower Workflow Design

The 'workflow_design' application is responsible for creating and managing workflow definitions, task definitions, and ticket definitions. It's used by users with the 'workflow_creator' role.

## Key Functionalities
- Create and edit workflow definitions
- Create and edit task definitions
- Define workflow transitions
- Create and edit ticket definitions

## Relevant Schema

[Include the CREATE TABLE statements for the four tables mentioned above]

## User Roles
- System role: 'workflow_creator'

## Planned Views
- Workflow Definitions List/Detail
- Task Definitions List/Detail
- Ticket Definitions List/Detail
- Workflow Definition Upload (Create and edit)
- Task Definition Upload (Create and edit)
- Ticket Definition Upload (Create and edit)

## Development Environment
- Django framework
- PostgreSQL database
```

This information should provide a solid foundation for starting work on the 'workflow_design' application.

================================
================================

From here I want to 'chat_workflow_design'.
