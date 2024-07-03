# TicketFlower Project Info

TicketFlower is a ticketing and workflow system. We have completed an initial version of user management and now we will work on the workflow design application of our Django server.

Workflow design will be the key differentiator of this application when released.  We will use AI assistance to define workflows and associated forms. Initially we will concentrate on manual tasks, for which a user wil be given a form to complete to input their results from the task. In the next version we will start to work on generating automated and assisted actions for the workflow tasks.

For our initial build, which we are working on now, we will create a _minimal interface_ to upload the workflow task and ticket information. We will be building out the surrounding workflow functionality before we concentrate on really developing the AI assistance workflows.

## TicketFlower Workflow Design Django Application

The 'workflow_design' application is responsible for creating and managing workflow definitions, task definitions, and ticket definitions. It's used by users with the 'workflow_creator' role.

### Key Functionalities

- Create and edit workflow definitions
- Create and edit task definitions
- Define workflow transitions
- Create and edit ticket definitions

### User Roles

In addition to being in the proper company, the user must have the following role to use the workflow design functionality.

- System role: 'workflow_creator'

### Planned Views

- Workflow Definitions List/Detail
- Task Definitions List/Detail
- Ticket Definitions List/Detail
- Workflow Definition Upload (Create and edit)
- Task Definition Upload (Create and edit)
- Ticket Definition Upload (Create and edit)

### Development Environment

- Django framework
- PostgreSQL database
- Dev machine: Windows PC + VSCode