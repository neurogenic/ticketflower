# TicketFlower Project Overview

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


## Schema SQL

Here is the schema as designed in SQL. This can be modified as needed to fit Django.

```sql

CREATE TYPE system_roles AS ENUM ('admin', 'user', 'ticket_admin', 'task_worker', 'workflow_creator');

-- Companies Table
CREATE TABLE companies (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Users Table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES companies(id),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- User System Roles (Junction Table)
CREATE TABLE user_system_roles (
    user_id INTEGER REFERENCES users(id),
    role INTEGER REFERENCES system_roles(id),
    PRIMARY KEY (user_id, role_name)
);

-- User Compnay Roles (Junction Table)
CREATE TABLE user_company_roles (
    user_id INTEGER REFERENCES users(id),
    role  INTEGER REFERENCES company_roles(id),
    PRIMARY KEY (user_id, role_name)
);

-- Company Roles - Managed by company, used for ticket submit permissions
CREATE TABLE company_roles (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES company(id),
    role_name VARCHAR(100),
    description VARCHAR(255)
    -- add unique constraint on names
);

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES companies(id),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

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

## User Management

- Company Sign-up Page
    - Fields: Company Name, Admin User Email, Admin User Password, Admin User First Name, Admin User Last Name
    - Function: Creates a new company and the first admin user
- User Login Page
    - Fields: Email, Password
- User Landing Page (populated later)

LATER:

- Reset Password Page
- Simple Profile Page (allows editing fields)


For MVP, for starters, no functionality for creating additional company users. As a demo, users will be able to sign up as a company/person and do their own workflows and tickets. The full functionality will be built into the system apart from the UI part.

## User Ticket Pages (user role)

- Submit Tickets
- My Tickets (Active, Archive)
- Ticket Detail

Authorization note: "Company role" field is used for user access to ticket types.

## Ticket Team Pages

### Ticket Management: (ticket_admin role)

- Tickets
- Ticket Detail
- Tasks
- Task Detail
- Ticket Errors
- Ticket Error Detail

- Workflow Definitions
- Workflow Definition Detail
- Task Definitions
- Task Definition Detail
- Ticket Definitions
- Ticket Definition Detail

Initially we will automatically give them to the available worker with the shortest list. Later we can think about giving options here.

Initially we will not have any error handling capability in the UI.

Later we will support different options for assigning and tracking tickets along with more advanced ticket tracking and analytics.

### Ticket Execution: (task_workers role)

- My Tasks (To execute)
- Task Execution (For manual/Form type tasks)

### Workflow Creation (workflow_creator role)

- Workflow Definition Upload (Create and edit)
- Task Definition Upload (Create and edit)
- Ticket Definition Upload (Create and edit)

I want a minimal way to upload to these tables. Later we will work on detailed workflow definition with an AI assistant.