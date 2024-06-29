# TicketFlower Schema and Site Design

## Schema SQL

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
    role_name system_roles,
    PRIMARY KEY (user_id, role_name)
);

-- User Compnay Roles (Junction Table)
CREATE TABLE user_company_roles (
    user_id INTEGER REFERENCES users(id),
    role_name  VARCHAR(100),
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
## Authorization and Security

- Strictly enforce a user from one company can not access any records labeled for a different company (eg ticket type or ticket instance)
    - Low priority on enforcing company id consistency between linked records (eg ticket type and ticket instance)
- System roles define access to assets within a given company. They are global and we will use hard coded values for them:
    - user
    - admin
    - ticket_admin
    - task_worker
    - workflow_creator
- Company specific roles can be defined to control access to ticket type submission. Default to companies having a role "user" and all users have this company role.

## User Management

- Company Sign-up Page
    - Fields: Company Name, Admin User Email, Admin User Password, Admin User First Name, Admin User Last Name
    - Function: Creates a new company and the first admin user
- User Login Page
    - Fields: Email, Password
- User Dashboard
    - Display user information
    - Links to other functionalities (Submit Tickets, My Tickets, etc.)
- Simple Profile Page (allows editing fields)
- Reset Password Page

NOTES: For MVP, no functionality for creating additional company users

## User Ticket Pages

- Submit Tickets
- My Tickets
- Ticket Detail

Authorization note: "Company role" field is used for user access to ticket types.
TBD: Should this use the same control as the Admin list?

## Ticket Team Pages

### Ticket Team: Admin

Access to all tickets (by company)

- Tickets (Active, Archive)
- Ticket Detail

- Tasks - (Active, Archive)
- Task Detail

### Ticket Team: Task Workers

- My Tasks (To execute)
- Task Execution (For manual/Form type tasks)

TBD: Should we use the page or control from "Ticket Team: Admin" to access my tasks? This means we would need to add the link to open the task execution page too.

### Ticket Team: Workflow Designers

- Workflow Definitions
- Workflow Definition Detail
- Task Definitions
- Task Definition Detail
- Ticket Definitions
- Ticket Definition Detail

- Workflow Definition Upload (Create and edit)
- Task Definition Upload (Create and edit)
- Ticket Definition Upload (Create and edit)

TBD: We will have a detailed mechanism for creating workflows/tasks/tickets but for starters I want a way to upload to those tables.

### Ticket Assignment

We will support different options for assigning and tracking tickets. For starters I think we will automatically give them to the available worker with the shortest list. Later we can think about giving options here. (Also, more advanced ticket tracking and analytics will be added later.)

===========================

Stack:

Back End:

- Django
- Django Rest Framework
- JWT
- Python with Type annotations
- Postgres
- pytest? (Or is there a django test framework that has an advantage?)

Front End:

- Typescript
- Vanilla React (including things like use reducer)
- Vite
- Tailwind and css modules
- TBD test framework(s)


================================

# Review of Ticket and Workflow Management Website

## Schema Review

1. **Companies Table**: As noted, there's no reference to companies in the provided schema. This should be added to support multi-tenant functionality.

2. **User Table**: Not provided in the schema. This table should be added to store user information and link to the companies table.

3. **Roles**: Consider creating a separate roles table instead of hardcoding roles. This would allow for more flexible role management.

4. **File Uploads**: Good inclusion. Consider adding a reference to the related entity (ticket, task, etc.) to easily track which uploads belong where.

5. **Indexes**: Add indexes on frequently queried columns to improve performance.

6. **Soft Deletes**: Consider adding a `deleted_at` column to relevant tables for soft deletes.

## Page Structure Review

### User Management

- Add pages for user management:
  - User List
  - User Detail/Edit
  - Company List
  - Company Detail/Edit
  - Role Management

### User Pages

- Current structure looks good.
- Consider adding:
  - Dashboard/Overview page
  - Notifications page or component

### Ticket Team Pages

#### Admin

- Current structure is good.
- Add:
  - Dashboard with key metrics
  - Search functionality for tickets and tasks
  - Reporting and analytics page

#### Task Workers

- Current structure is good.
- Add:
  - Dashboard with personal performance metrics
  - Calendar view of upcoming tasks

#### Workflow Designers

- Current structure is good.
- Add:
  - Workflow visualization tool
  - Version control for workflows, tasks, and ticket definitions
  - Import/Export functionality for definitions

### Ticket Assignment

- Add a page or section for configuring assignment rules
- Include a view for manually assigning tickets/tasks

