# Project Setup Chat

==============

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

I want a minimal way to upload to these tables. Later we will work on detailed workflow definition with an AI assistant.sistant will also help with setting up and executing autonomous tasks as a part of workflows.

The initial target will be for small businesses to easily set up a ticket system.

This will be an MVP. I want to get it up and running quickly. The emphasis will not be on full features for the workflow management. The
main point will be to illustrate the capabilities of the AI assistant. We of course still want workable functionality.

The initial code we write will actual leave out the functionality of the AI assistant. We will be developing that later. Here we will be
building the ticket and workflow framework the AI assistant will fit into.

---

# TicketFlower Project Design Report

## Project Overview

TicketFlower is a web-based ticket and workflow management system designed for small businesses. The primary goal is to create a Minimum Viable Product (MVP) that demonstrates the core functionality of ticket submission, workflow management, and task execution. The system is designed with future AI assistant integration in mind, although this feature is not part of the initial implementation.

## Technical Stack

- Backend: Django (Python web framework)
- Database: PostgreSQL
- Frontend: Django templates (with future potential for a more advanced frontend)
- Development Environment: Windows with VSCode, PostgreSQL in Docker

## Key Design Decisions

### 1. Authorization and Multi-tenancy

The system is designed to support multiple companies (tenants) while ensuring data isolation and proper access control.

#### Authorization Model:

a. Authentication:
   - Utilizes Django's built-in User model and authentication system for simplicity and security.

b. Company Identification:
   - A Company model is created and linked to User with a ForeignKey, establishing the multi-tenant structure.

c. System Roles:
   - Leverages Django's built-in Group model for system-wide roles.
   - Permissions are assigned to these groups using Django's permission system, allowing for flexible role-based access control.

d. Company Roles:
   - A custom CompanyRole model is implemented to handle company-specific roles.
   - This model is linked to both Company and User with a many-to-many relationship, allowing users to have different roles in different companies.

e. Authorization Implementation:
   - Django's @permission_required decorator is used for view-level permissions, providing a straightforward way to restrict access to specific views.
   - Custom permissions are implemented for more granular control where needed.

f. Multi-tenancy Implementation:
   - The get_queryset method in views is overridden to filter by the user's company, ensuring data isolation.
   - A custom model manager is used to always filter by company, providing an additional layer of security.

g. Middleware (optional):
   - A light middleware is considered to set the current company in the request, simplifying company-based filtering throughout the application.

#### Handling Multi-tenancy:

a. Middleware:
   - The existing middleware sets the current company for each request.
   - For superusers, it checks for an 'impersonated_company_id' in the session, allowing for company impersonation.

b. Superuser Functionality:
   - A dedicated page for superusers is created to allow company impersonation.
   - The selected company ID is stored in the session for the duration of the impersonation.

c. User Interface:
   - The current company name is displayed in the standard page header or navbar, providing clear context for the user.

d. Session Management:
   - An option to end impersonation (clearing the session variable) is provided for superusers.

e. Logging:
   - Standard action logging includes both user and affected data, ensuring a comprehensive audit trail.

### 2. Development Approach

The development is structured into logical phases, focusing on building core functionality incrementally.

#### Task List:

1. Project Setup:
   - Initialize the Django project, set up the PostgreSQL database, and configure environment variables.

2. User Management (users app):
   - Implement company and user models.
   - Create views and templates for company signup, user login, and user dashboard.

3. Workflow, Task, and Ticket Definition (workflow_design app):
   - Create models for workflow, task, and ticket definitions.
   - Implement listing pages for definitions with appropriate access controls.
   - Create interfaces for uploading and editing definitions.

4. Workflow Management (workflow_ops app):
   - Implement models for workflow and task instances.
   - Create a WorkflowStateManager to handle workflow logic.
   - Develop user interfaces for ticket submission, listing, and details.
   - Create admin interfaces for ticket and task management.

5. Task Execution (workflow_ops app):
   - Implement interfaces for task workers to view and complete assigned tasks.
   - Integrate task completion with workflow state updates.

### 3. Workflow Transition Model

The workflow transition process is designed to be robust while maintaining simplicity.

1. Database Update:
   - User-initiated changes (e.g., ticket submission, task completion) are immediately reflected in the database.

2. Workflow State Management:
   - After the initial database update, the WorkflowStateManager is invoked.
   - This manager calculates the next transition, creates new tasks, or completes the workflow as needed.
   - These operations are performed outside the initial database transaction to prevent blocking.

3. Error Handling:
   - Any errors during the workflow management process are captured and stored for admin review.

### 4. Error Handling

A comprehensive error tracking system is implemented to capture and manage issues in the workflow process. 

1. Workflow Creation Error: Occurs when a ticket exists but workflow instance creation fails.
2. Task Creation Error: Happens when a workflow exists but the system is unable to create the next task.
3. Task Update Error: Occurs when the system is unable to update a task's status.
4. Workflow Transition Error: A catch-all category for issues related to transitioning the workflow, including data validation, condition evaluation, assignment, and completion problems.

## Conclusion

This design provides a solid foundation for the TicketFlower MVP. It emphasizes simplicity and functionality while allowing for future expansion and integration of more advanced features like AI assistance. The multi-tenant architecture ensures data isolation and security, while the workflow management system provides flexibility for various business processes. The error handling system allows for robust operation and easy troubleshooting. As development progresses, these design decisions can be revisited and refined based on real-world usage and emerging requirements.

============================

/share .env_MODIFIED docker-compose.yml

=================

I would like to set up a django server for the Ticketflower project as described above. I already have a git repository with a .env file, a docker-compose file for postgres and the gitignore file.

I plan on running the django project directly on my PC for now.

A description as well as an initial project task list is given above. Please review that and let me know if you have any questions.

=============

> Coder_Assistant: Thank you for providing the context and the existing setup for the Ticketflower project. I've reviewed the project description and the initial task list. It seems you have a solid foundation to start with. Let's begin by setting up the Django server for your project.

First, I'll outline the steps we'll take to set up the Django project:

1. Create a virtual environment
2. Install Django and other necessary packages
3. Start a new Django project
4. Configure the project settings to use the existing PostgreSQL database
5. Create the initial apps as outlined in the task list

Before we proceed, I have a couple of questions to ensure we're on the same page:

1. Are you using Python 3.x? If so, which specific version?
2. Do you want to use any specific Django version, or should we use the latest stable release?

Once you provide this information, we can start setting up the Django server for the Ticketflower project.

===================

I will be using python 3.12. I will be using the latest stable release for all packages. Feel free to specify versions, such as if you are giving me a requirements.txt file, but I will update these to the latest.

=====================

> Coder_Assistant: Great, thank you for the clarification. Let's proceed with setting up the Django server for the Ticketflower project using Python 3.12 and the latest stable releases of Django and other packages.

First, let's create a virtual environment and install Django:

<terminal>
<command>python -m venv venv</command>
<command>venv\Scripts\activate</command>
<command>pip install django psycopg2-binary python-dotenv</command>
</terminal>

Now, let's create a new Django project:

<terminal>
<command>django-admin startproject ticketflower .</command>
</terminal>

Next, we'll create a requirements.txt file:

<create_file>
<path>/app/ticketflower/ticketflower_ops_server/requirements.txt</path>
<content>
Django==5.0.1
psycopg2-binary==2.9.9
python-dotenv==1.0.0
</content>
</create_file>

Now, let's update the settings.py file to use the PostgreSQL database and load environment variables:

<update_file>
<path>/app/ticketflower/ticketflower_ops_server/ticketflower/settings.py</path>
<content>
import os
from pathlib import Path
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv('DJANGO_SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.getenv('DEBUG', 'False') == 'True'

ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', '').split(',')

# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'ticketflower.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'ticketflower.wsgi.application'

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME'),
        'USER': os.getenv('DB_USER'),
        'PASSWORD': os.getenv('DB_PASSWORD'),
        'HOST': os.getenv('DB_HOST'),
        'PORT': os.getenv('DB_PORT'),
    }
}

# Password validation
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files (CSS, JavaScript, Images)
STATIC_URL = 'static/'

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
</content>
</update_file>

Now, let's create the initial apps as outlined in the task list:

<terminal>
<command>python manage.py startapp users</command>
<command>python manage.py startapp workflow_design</command>
<command>python manage.py startapp workflow_ops</command>
</terminal>

Update the INSTALLED_APPS in settings.py to include these new apps:

<update_file>
<path>/app/ticketflower/ticketflower_ops_server/ticketflower/settings.py</path>
<content>
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'users',
    'workflow_design',
    'workflow_ops',
]
</content>
</update_file>

Finally, let's run the initial migrations:

<terminal>
<command>python manage.py makemigrations</command>
<command>python manage.py migrate</command>
</terminal>

This sets up the basic Django project structure for Ticketflower. The next steps would involve:

1. Implementing the models for each app (users, workflow_design, workflow_ops)
2. Creating views and templates for user management, workflow definition, and ticket management
3. Setting up URL routing
4. Implementing the business logic for workflow management and task execution

Would you like to proceed with implementing the models for the 'users' app, which will handle company and user management?


