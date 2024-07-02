I am working on a Ticket and workflow web service. The main focus of the site will be on using a AI assistant to help set up
workflows and tickets. Later, the AI assistant will also help with setting up and executing autonomous tasks as a part of workflows.

The initial target will be for small businesses to easily set up a ticket system.

This will be an MVP. I want to get it up and running quickly. The emphasis will not be on full features for the workflow management. The
main point will be to illustrate the capabilities of the AI assistant. We of course still want workable functionality.

The initial code we write will actual leave out the functionality of the AI assistant. We will be developing that. Here we will be 
building the ticket and workflow framework the AI assistant will fit into.

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
- System roles define access to different web site features, within a given company. They are global and we will use hard coded values for them.
- Company roles can be defined by the company to control access to specific ticket types for submission, which are of course defined by the company.

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

For MVP, for starters, no functionality for creating additional company users. As a demo, users will be able to sign up as a company/person and do their own workflows and tickets. The full functionality will be built into the system apart from the UI part.

## User Ticket Pages

- Submit Tickets
- My Tickets
- Ticket Detail

Authorization note: "Company role" field is used for user access to ticket types.

## Ticket Team Pages

### Ticket Team: Admin

Access to all tickets (by company)

- Tickets (Active, Archive)
- Ticket Detail

- Tasks - (Active, Archive)
- Task Detail

We will support different options for assigning and tracking tickets, but for starters I think we will automatically give them to the available worker with the shortest list. Later we can think about giving options here. (Also, more advanced ticket tracking and analytics will be added later.)

### Ticket Team: Task Workers

- My Tasks (To execute)
- Task Execution (For manual/Form type tasks)

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

We will have a detailed mechanism for creating workflows/tasks/tickets but for starters I want a minimal way to upload to those tables. (This would be  OK to be on the backend side. This is not an intended feature of the service.)

---

## Tech Stack

I am considering the following for the tech stack.

Back End:

- Django
- Django Rest Framework
- JWT
- Python with Type annotations
- Postgres
- Django's built-in TestCase

Front End:

- Typescript
- Vanilla React (including things like use reducer)
- Vite
- Tailwind and css modules
- Jest with React Testing Library

Docker/Server setup:

- Django and Postgres running in separate containers (set up with docker compose)
    - Later we will use a hosted solution for postgres.
    - For dev, I do not want the server docker file to start the django server. I want to be able to start and stop it with a separate command.
- Django server used for initial development.
    - Later with AI assistant we will probably switch to a server like Uvicorn, Django Channels that will support websockets. 
- Vite set up on PC for development.
    - Later static files will be served from the server.
- Environment variables: .env file (with python-dotenv) for dev
    - Likely docker secrets for prod

Dev Machine:

- Windows PC
- VSCode (with debugger usage)

---

I would like to you to break this down into a series of steps, giving a high level description of each. The server project is called ticketflower_server and the client is called ticketflower_client.

This step should involve no code. Next we will work on plans for each step, still involving no code. After that we will start writing code.

===========================================

Certainly. I'll break down the project into a series of high-level steps for both the server (ticketflower_server) and client (ticketflower_client) sides. Here's a proposed sequence of steps:

1. Project Setup
   - Set up the development environment
   - Initialize Git repositories for both server and client
   - Create basic project structures for Django and React

2. Database and Model Design
   - Set up PostgreSQL in a Docker container
   - Implement the database schema in Django models
   - Create initial migrations

3. Authentication and Authorization
   - Implement JWT authentication in Django
   - Set up user registration and login endpoints
   - Implement company and role-based authorization

4. Core API Development
   - Develop API endpoints for CRUD operations on main entities (tickets, workflows, tasks)
   - Implement business logic for workflow and task management

5. Client-Side Setup
   - Set up React with TypeScript using Vite
   - Configure Tailwind CSS and CSS modules
   - Set up routing and basic layout components

6. User Management Frontend
   - Implement company sign-up, user login, and profile pages
   - Create user dashboard

7. Ticket Management Frontend
   - Develop interfaces for submitting and viewing tickets
   - Create ticket detail view

8. Workflow and Task Management Frontend
   - Implement workflow and task definition interfaces
   - Create task execution views for workers

9. Admin Interfaces
   - Develop admin views for managing tickets, tasks, and workflows
   - Implement basic analytics and reporting features

10. Integration and Testing
    - Integrate frontend with backend APIs
    - Implement comprehensive test suites for both frontend and backend

11. Docker Configuration
    - Create Dockerfiles for both server and client
    - Set up docker-compose for local development

12. Documentation and Cleanup
    - Write API documentation
    - Create user guides
    - Code cleanup and optimization

13. Deployment Preparation
    - Set up environment variable management
    - Prepare for production deployment (though not actually deploying in this MVP phase)

14. Final Testing and Bug Fixes
    - Conduct thorough system testing
    - Address any bugs or issues found

This breakdown provides a high-level roadmap for developing the MVP. Each step can be further detailed into specific tasks and user stories as we move forward. The focus is on creating a functional system that demonstrates the core capabilities, with the AI assistant integration to be developed separately.

=============================

I have empty respositories for both ticketflower_server and ticketflower_client. I would like to to write out a plan for step one. This should be a self contained report/prompt on what to do for this step. Again, I don't want it to include any code. That work will be done by the worker who follows this report.

1. Project Setup
   - Set up the development environment
   - Initialize Git repositories for both server and client
   - Create basic project structures for Django and React

=================================

Project Setup Plan for TicketFlower MVP

This plan outlines the steps to set up the development environment and initialize the basic project structures for both the server (ticketflower_server) and client (ticketflower_client) components of the TicketFlower MVP.

1. Development Environment Setup

   a. Python Environment:
   - Install Python 3.11 or later if not already installed
   - Set up a virtual environment for the server project:
     - Navigate to the ticketflower_server directory
     - Create a virtual environment named 'venv'
     - Activate the virtual environment

   b. Node.js Environment:
   - Install Node.js 18.x LTS or later if not already installed
   - Install yarn package manager globally

   c. Docker:
   - Install Docker Desktop for Windows
   - Ensure Docker is running and accessible from the command line

   d. Visual Studio Code:
   - Install VS Code if not already present
   - Install recommended extensions for Python, JavaScript/TypeScript, and Docker

2. Git Repository Initialization

   a. Server (ticketflower_server):
   - Navigate to the ticketflower_server directory
   - Initialize git repository if not already done
   - Create a .gitignore file appropriate for Python/Django projects
   - Make an initial commit with the .gitignore file

   b. Client (ticketflower_client):
   - Navigate to the ticketflower_client directory
   - Initialize git repository if not already done
   - Create a .gitignore file appropriate for React/TypeScript projects
   - Make an initial commit with the .gitignore file

3. Server Project Structure (Django)

   a. Install Django and initial dependencies:
   - With the virtual environment activated, install Django, Django REST framework, and other initial dependencies
   - Create requirements.txt file

   b. Create Django project:
   - Use django-admin to start a new project named 'ticketflower'
   - Rename the outer ticketflower directory to 'server' for clarity

   c. Configure Django settings:
   - Update settings.py with appropriate configurations for databases, installed apps, etc.
   - Set up python-dotenv for environment variable management

   d. Create initial Django apps:
   - Create apps for 'users', 'tickets', 'workflows', and 'tasks'

   e. Set up Django REST framework:
   - Add REST framework to INSTALLED_APPS
   - Configure basic REST framework settings

   f. Create a docker-compose.yml file:
   - Define services for Django and PostgreSQL
   - Set up volume for PostgreSQL data persistence

   g. Commit changes to git repository

4. Client Project Structure (React with TypeScript)

   a. Create Vite project:
   - Use Vite to create a new React project with TypeScript template

   b. Install additional dependencies:
   - Install React Router for navigation
   - Install Tailwind CSS and configure it
   - Set up any other initial frontend dependencies

   c. Set up project structure:
   - Create directories for components, pages, services, and utils
   - Set up a basic routing structure

   d. Configure TypeScript:
   - Ensure tsconfig.json is properly set up for the project

   e. Set up testing environment:
   - Install and configure Jest and React Testing Library

   f. Commit changes to git repository

5. Documentation

   a. Create README.md files:
   - For both server and client projects, create detailed README files
   - Include setup instructions, development workflow, and any other relevant information

   b. Create a CONTRIBUTING.md file:
   - Outline the process for contributing to the project
   - Define coding standards and commit message formats

6. Final Steps

   a. Review the setup:
   - Ensure all components are working as expected
   - Test running the server and client applications

   b. Update .gitignore files if necessary

   c. Make final commits to both repositories

   d. Push changes to remote repositories if applicable

This plan provides a comprehensive guide for setting up the initial project structure for both the server and client components of the TicketFlower MVP. It ensures a solid foundation for further development while maintaining best practices in project setup and version control.

