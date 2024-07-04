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