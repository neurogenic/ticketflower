# Workflow Design Chat

==================

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

### Relevant Schema

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


======================

# Shared Files

/share requirements.txt ticketflower/settings.py ticketflower/urls.py users/models.py

===============

I've shared some files and some information about the 'workflow_design' django application I would like to build for the web application TicketFlower. The site currently has some user management built out, including companies and users.

I would appreciate brief responses so we don't overload your context length too quickly. I would also like to work one step at a time.

Without writing any code, please suggest where we should start on workflow design.

=====================

> Coder_Assistant: Based on the information provided, I suggest we start with the following steps for the workflow design:

1. Create the 'workflow_design' Django app
2. Define models for WorkflowDefinition, TaskDefinition, WorkflowTransition, and TicketDefinition
3. Set up basic views and URLs for listing and creating these objects
4. Implement simple forms for uploading workflow, task, and ticket definitions

This approach will give us a foundation to build upon. We can start with the app creation and model definitions. Shall we begin with creating the 'workflow_design' Django app?

========================

Yes, let's get started with step 1.


===========================

> Coder_Assistant: Certainly! Let's create the 'workflow_design' Django app. We'll use the Django management command to create the new app.

First, let's run the following command in the terminal:

<smart_terminal>
<command>python manage.py startapp workflow_design</command>
<working_directory>/app/ticketflower/ticketflower_server</working_directory>
</smart_terminal>

This command will create a new Django app named 'workflow_design' with the basic structure. After running this command, we need to add the new app to the INSTALLED_APPS list in the settings.py file.

Let's update the settings.py file:

<update_file>
<path>/app/ticketflower/ticketflower_server/ticketflower/settings.py</path>
<content>
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'rest_framework_simplejwt',
    'users',
    'workflow_design',  # Add this line
]
</content>
<mode>replace</mode>
<start_line>26</start_line>
<end_line>37</end_line>
</update_file>

This completes step 1. The 'workflow_design' app has been created and added to the project. Shall we move on to step 2 and start defining the models?

================================

The updates have been done. Let's move to step 2.

===================

> Coder_Assistant: Great, let's move on to step 2: defining the models for WorkflowDefinition, TaskDefinition, WorkflowTransition, and TicketDefinition.

We'll update the models.py file in the workflow_design app. We'll use the schema you provided earlier to create these models.

Let's update the models.py file:

<update_file>
<path>/app/ticketflower/ticketflower_server/workflow_design/models.py</path>
<content>
from django.db import models
from users.models import Company, User

class WorkflowDefinition(models.Model):
    company = models.ForeignKey(Company, on_delete=models.CASCADE)
    title = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    ext_ref = models.JSONField(default=list)
    fields = models.JSONField(default=list)
    version = models.IntegerField(default=1)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    display_layout = models.TextField(blank=True)

class TaskDefinition(models.Model):
    TASK_TYPES = [
        ('FORM', 'Form'),
        ('SYNC_ACTION', 'Synchronous Action'),
        ('ASYNC_ACTION', 'Asynchronous Action'),
    ]
    company = models.ForeignKey(Company, on_delete=models.CASCADE)
    workflow = models.ForeignKey(WorkflowDefinition, on_delete=models.CASCADE)
    title = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    ext_ref = models.JSONField(default=list)
    task_type = models.CharField(max_length=20, choices=TASK_TYPES)
    input_schema = models.JSONField(default=dict)
    output_schema = models.JSONField(default=dict)
    version = models.IntegerField(default=1)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    edit_layout = models.TextField(blank=True)
    display_layout = models.TextField(blank=True)
    action_url = models.URLField(blank=True)
    input_transform = models.JSONField(default=dict)
    output_transform = models.JSONField(default=dict)
    timeout_seconds = models.IntegerField(null=True, blank=True)

class WorkflowTransition(models.Model):
    workflow = models.ForeignKey(WorkflowDefinition, on_delete=models.CASCADE)
    from_task = models.ForeignKey(TaskDefinition, on_delete=models.CASCADE, related_name='transitions_from')
    to_task = models.ForeignKey(TaskDefinition, on_delete=models.CASCADE, related_name='transitions_to')
    condition = models.TextField(blank=True)
    priority = models.IntegerField(default=0)

class TicketDefinition(models.Model):
    company = models.ForeignKey(Company, on_delete=models.CASCADE)
    title = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    ext_ref = models.JSONField(default=list)
    workflow = models.ForeignKey(WorkflowDefinition, on_delete=models.CASCADE)
    fields = models.JSONField(default=dict)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    version = models.IntegerField(default=1)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    company_role_access = models.JSONField(default=list)
    edit_layout = models.TextField(blank=True)
    display_layout = models.TextField(blank=True)
</content>
<mode>replace</mode>
</update_file>

This defines our four main models based on the schema you provided. A few notes:

1. We're using JSONField for complex data structures like ext_ref, fields, etc.
2. For task_type in TaskDefinition, we're using choices to limit the possible values.
3. We're using ForeignKey relationships to link models where appropriate.

Next, we need to create and run migrations for these new models. Shall we proceed with that?

=========================

Yes, please give me the migration commands.

==========================

> Coder_Assistant: Certainly! To create and apply the migrations for our new models, we'll use the following Django management commands. Let's run them one at a time:

First, let's create the migrations:

<smart_terminal>
<command>python manage.py makemigrations workflow_design</command>
<working_directory>/app/ticketflower/ticketflower_server</working_directory>
</smart_terminal>

This command will create a new migration file based on the changes we made to the models.

After the migrations are created, let's apply them to the database:

<smart_terminal>
<command>python manage.py migrate workflow_design</command>
<working_directory>/app/ticketflower/ticketflower_server</working_directory>
</smart_terminal>

This command will apply the migrations to your database, creating the necessary tables for our new models.

After running these commands, the database schema should be updated with our new workflow_design models.

Is there anything else you'd like me to do regarding the models or migrations?

========================

OK, the migrations have been applied. 

Above you gave this list of tasks. Are we ready to move to the next step, or is there anything else we should do right now?

1. [x] Create the 'workflow_design' Django app
2. [x] Define models for WorkflowDefinition, TaskDefinition, WorkflowTransition, and TicketDefinition
3. [ ] Set up basic views and URLs for listing and creating these objects
4. [ ] Implement simple forms for uploading workflow, task, and ticket definitions

========================

> Coder_Assistant: You're right, we've completed steps 1 and 2. We're now ready to move on to step 3: setting up basic views and URLs for listing and creating these objects.

Let's start by creating some basic views for our models. We'll use Django Rest Framework (DRF) to create API views, which will make it easier to interact with our data.

First, let's create a new file called serializers.py in the workflow_design app:

<create_file>
<path>/app/ticketflower/ticketflower_server/workflow_design/serializers.py</path>
<content>
from rest_framework import serializers
from .models import WorkflowDefinition, TaskDefinition, WorkflowTransition, TicketDefinition

class WorkflowDefinitionSerializer(serializers.ModelSerializer):
    class Meta:
        model = WorkflowDefinition
        fields = '__all__'

class TaskDefinitionSerializer(serializers.ModelSerializer):
    class Meta:
        model = TaskDefinition
        fields = '__all__'

class WorkflowTransitionSerializer(serializers.ModelSerializer):
    class Meta:
        model = WorkflowTransition
        fields = '__all__'

class TicketDefinitionSerializer(serializers.ModelSerializer):
    class Meta:
        model = TicketDefinition
        fields = '__all__'
</content>
</create_file>

Now, let's update the views.py file to include our new views:

<update_file>
<path>/app/ticketflower/ticketflower_server/workflow_design/views.py</path>
<content>
from rest_framework import viewsets
from .models import WorkflowDefinition, TaskDefinition, WorkflowTransition, TicketDefinition
from .serializers import WorkflowDefinitionSerializer, TaskDefinitionSerializer, WorkflowTransitionSerializer, TicketDefinitionSerializer

class WorkflowDefinitionViewSet(viewsets.ModelViewSet):
    queryset = WorkflowDefinition.objects.all()
    serializer_class = WorkflowDefinitionSerializer

class TaskDefinitionViewSet(viewsets.ModelViewSet):
    queryset = TaskDefinition.objects.all()
    serializer_class = TaskDefinitionSerializer

class WorkflowTransitionViewSet(viewsets.ModelViewSet):
    queryset = WorkflowTransition.objects.all()
    serializer_class = WorkflowTransitionSerializer

class TicketDefinitionViewSet(viewsets.ModelViewSet):
    queryset = TicketDefinition.objects.all()
    serializer_class = TicketDefinitionSerializer
</content>
</update_file>

Next, let's set up the URLs. Create a new file called urls.py in the workflow_design app:

<create_file>
<path>/app/ticketflower/ticketflower_server/workflow_design/urls.py</path>
<content>
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import WorkflowDefinitionViewSet, TaskDefinitionViewSet, WorkflowTransitionViewSet, TicketDefinitionViewSet

router = DefaultRouter()
router.register(r'workflows', WorkflowDefinitionViewSet)
router.register(r'tasks', TaskDefinitionViewSet)
router.register(r'transitions', WorkflowTransitionViewSet)
router.register(r'tickets', TicketDefinitionViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
</content>
</create_file>

Finally, let's update the main urls.py file to include our new workflow_design URLs:

<update_file>
<path>/app/ticketflower/ticketflower_server/ticketflower/urls.py</path>
<content>
from django.contrib import admin
from django.urls import path, include
from rest_framework_simplejwt.views import TokenRefreshView
from users.views import CreateCompanyView, UserLoginView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/create-company/', CreateCompanyView.as_view(), name='create_company'),
    path('api/login/', UserLoginView.as_view(), name='user_login'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('api/workflow/', include('workflow_design.urls')),  # Add this line
]
</content>
</update_file>

These changes set up basic API endpoints for listing and creating our workflow objects. We've completed step 3 now.

Would you like to move on to step 4 (implementing simple forms for uploading definitions), or do you want to test these new API endpoints first?

=============================

/share workflow_design/models.py workflow_design/views.py workflow_design/urls.py users/tests.py

I have made the suggested updates and additions. I have also added some new shared files to the session.

I would like to write some tests for these new endpoints.



