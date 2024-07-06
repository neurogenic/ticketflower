# Asssistnat Guide Chat

================

Thank you for providing the detailed document on the TicketFlower Assistant Workflow Guide. I've reviewed the information, and I have several comments and suggestions for you to consider:

1. Overall Structure:
The system seems well-designed, with clear separation between workflow definitions, task definitions, ticket definitions, and transitions. This modular approach should provide flexibility and scalability.

2. Versioning:
The versioning approach for workflows is good, allowing for backwards compatibility. However, you might want to consider a more automated versioning system for minor changes to reduce manual intervention.

3. Task Types:
The three task types (FORM, SYNC_ACTION, ASYNC_ACTION) cover a wide range of use cases. You might consider adding a fourth type for long-running background tasks that don't require immediate callback.

4. Transitions:
The transition system is flexible, but the error handling for multiple transitions could be more robust. Consider implementing a conflict resolution mechanism instead of treating conflicting transitions as errors.

5. Parallel Task Execution:
You mention that multiple tasks are executed in parallel. This could lead to race conditions or inconsistent states. Consider adding a mechanism to control task execution order or dependencies.

6. Company Role Access:
The company role access for tickets is a good feature. You might want to extend this to workflow and task levels for more granular access control.

7. Field Schemas:
Using JSON schemas for defining fields is flexible, but it might be beneficial to provide some predefined schemas for common field types to ensure consistency across different workflows.

8. Error Handling:
The document mentions error handling for tasks, but it would be helpful to have more detailed information on how errors are logged, reported, and potentially recovered from.

9. Auditing:
Consider adding fields for tracking who last modified a workflow, task, or ticket definition, in addition to the creation information.

10. Internationalization:
If your system might be used in multiple languages, consider adding language fields for titles and descriptions.

11. API Documentation:
While this document is thorough for internal understanding, you might want to create separate API documentation for developers who will integrate with your system.

12. Testing Workflows:
Consider adding a feature to test or simulate workflows before setting them to 'ACTIVE' status.

These are initial thoughts based on the provided document. Would you like to discuss any of these points in more detail?

======================

Great points. Let me start by saying we are working on a proof of concept. The goal is to get the system running quickly and later features can be added as deemed important for the popular use cases.

Also, the point of this document is to instruct how to create a workflow. It is not an extensive document on how to use the system. Further, this is intended to be read by an AI assistant to help implement a workflow suggested by a workflow creator.

I am recording all the comments and will think about them as the documentation and system evolves in this early state. I can give some initial thougts here:

2. Versioning will be automated, in the sense that if an update is done, the version will automatically be incremented. So I guess it is up to the user to decide wether to update a workflow or deprecate it and create a new one. This will have to be spelled out clearly.

3. ASYNC_ACTION is intended for long running action. There is no restriction on when the callback should be made. It looks like that should be stated more explicitly.

4. At this point I don't want to get too fancy. The conditions I mentioned should not be allowed in the workflow. If they happen, I don't want to try to recover automatically. I will flag an error and that will kick off the error recover.  Later, with more experience, we can come up with better error handling.

5. This is definitely something to keep in mind. Right now a used should only define parallel tasks if they do not interfere. But I think it would be a good option to allow parallel or sequential execution. With sequential execution, the priority would be used. We will see how this goes with more experience. I'm leaning towards keeping it as is for now.

6. The system itself has other access control. Only some people can create workflows, for instance. I am not sure if this is what  you meant. If so, I am leaving the description out since it isn't really relevant to creating workflows.

7. We should look more into this. I am not sure how uniform the implementation of the special JSON formats is.

8. That is a good point. I will thank about if and how to include more information here.

9, 10, 11. These are definitely important. They are beign left out for now but will be added later.

12. This is also important. I am not sure how the testing will work for now. I will see how things go with some experience.

I think I will have to change some of the fields in the DB and also present the document based on the interface the Assistant will be used to create the workflows.

Right now, I have basic forms to update to the site. I will need to fill them all in.

I want to discuss the following JSON formats:

- JSON schema for workspace definition and ticket definition `fields` and task definition `input_schema` and `output_schema`.
- JSON transform for task definition `input_transform` and `output_transform`.
- JSON condition for workflow transitoin `condition`.

These values will be used in code so they must be of the proper format. How can this be done reliably?

=======================

Thank you for your detailed response and clarifications. Your approach of starting with a proof of concept and evolving the system based on experience is sensible. Let's focus on the JSON formats you mentioned, as these are critical for the proper functioning of the system.

To ensure that these JSON formats are reliable and properly structured, I suggest the following approaches:

1. JSON Schema for fields, input_schema, and output_schema:

For these, you can use JSON Schema itself. JSON Schema is a powerful tool for validating the structure of JSON data. Here's how you can approach this:



```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "The name of the ticket or workflow"
    },
    "priority": {
      "type": "integer",
      "minimum": 1,
      "maximum": 5,
      "description": "The priority of the ticket or workflow"
    },
    "due_date": {
      "type": "string",
      "format": "date",
      "description": "The due date of the ticket or workflow"
    },
    "tags": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Tags associated with the ticket or workflow"
    }
  },
  "required": ["name", "priority"]
}

```

This JSON Schema defines a structure with a name (required string), priority (required integer between 1 and 5), an optional due date (in date format), and optional tags (an array of strings).

2. JSON transform for input_transform and output_transform:

For these transformations, you could use a simplified version of JSONPath or a custom transformation language. Here's an example of a simple transform format:



```json
{
  "task_name": "$.workflow.name",
  "task_priority": "$.workflow.priority",
  "assigned_to": "$.user.email",
  "due_date": {
    "source": "$.workflow.due_date",
    "transform": "date_add(7)"
  }
}

```

In this example, `$.workflow.name` means "take the name from the workflow object". The `due_date` field shows how you could incorporate simple transformations.

3. JSON condition for workflow transition condition:

For conditions, you could use a simplified boolean logic format. For example:



```json
{
  "AND": [
    {"field": "status", "operator": "equals", "value": "IN_PROGRESS"},
    {"OR": [
      {"field": "priority", "operator": "greater_than", "value": 3},
      {"field": "due_date", "operator": "less_than", "value": "today()"}
    ]}
  ]
}

```

This condition would be true if the status is "IN_PROGRESS" AND either the priority is greater than 3 OR the due date is before today.

To implement these reliably:

1. Define clear specifications for each JSON format.
2. Implement validators for each format.
3. Use these validators when saving data to ensure it's correctly formatted.
4. Provide clear documentation and examples for users creating workflows.
5. Implement error handling to provide useful feedback when invalid JSON is entered.

You could also consider creating a user interface that helps users construct these JSON structures without having to write raw JSON. This could be particularly helpful for complex structures like conditions.

Would you like to discuss any of these formats in more detail or explore how to implement validators for them?

==================

Remember this document is not intended for a human use. This is part of a prompt that will be given to an AI assistant during a request. An Ai assistant has extensive knowledge of anything in its training set. Ideally I could reference a standard that would be in the AI assistants training set rather than trying to teach it how to create a given JSON (or alternate) format inside of the prompt.

Does this change the content you would recommend for the guide or the formats you would recommend?

====================

I apologize for the misunderstanding. You're absolutely right, and this context significantly changes the approach we should take. Given that this document is intended for an AI assistant with extensive knowledge, we can indeed leverage existing standards and formats without needing to explain them in detail. Here's a revised approach:

1. For JSON Schema (fields, input_schema, output_schema):
We can simply specify that these should follow the JSON Schema specification, latest draft (currently draft 2020-12). An AI assistant would be familiar with this standard and able to generate and interpret compliant schemas.

2. For JSON transforms (input_transform, output_transform):
We could specify the use of JSONPath, which is a well-established query language for JSON. An AI assistant would be familiar with JSONPath syntax and able to construct appropriate transformations.

3. For JSON conditions (condition):
We could specify the use of a subset of MongoDB's query operators. These provide a rich set of comparison and logical operators that an AI assistant would be familiar with and able to use effectively.

Given this context, here's how we might update the relevant parts of the guide:





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

When constructing these JSON structures, ensure they are valid according to their respective specifications. The system will validate these structures when they are saved or updated.


This approach leverages standards and formats that an AI assistant would be familiar with, eliminating the need for detailed explanations or examples within the guide itself. The AI can then generate appropriate JSON structures based on these specifications as needed.

Does this revised approach align better with your intentions for the document and its use by an AI assistant?

=========================

That is perfect. I will include that.

Let's turn our attention away from the guide. We will refine that after I have the chance to create a few workflows. Now, I want to try to muddle through making a workflow.

I want to start with a simple, contrived workflow.

- Workflow: "Document Edit"
    - Workflow Fields:
        - "title" - The title for the document
        - "submitted_doc" - The text to be edited
        - "doc_notes" - Any notes from the submitter
        - "questions" - Questions from the editor for the author
        - "author_responses" - Responses from the author for the questions.
        - "update_doc" - The updated document
        - "edit_notes" - Notes on the edited document

- Ticket: "Document Edit Ticket"
    - Ticket Fields:
        - "title" - The title for the document
        - "submitted_doc" - The text to be edited
        - "doc_notes" - Any notes from the submitter

Task: "Edit Document"
    - Task Inputs:
        - "title"
        - "submitted doc"
        - "doc_notes"
    - Task Outputs:
        - "questions": Optional: questions from the editor 
        - "updated_doc": OPtional: The modified document
        - "edit_notes": Optional: Notes from the editor
    - Task Description: An editor will take the inputs and do one of the following:
        - Ask questions of the author, populating the "questions" field of the response, leaving "updated_doc" and "edit_notes" empty.
        - Return an edited document, populating "updated_doc" and optionally "edit_notes", leaving questions empty.

Task: "Question Responses"
    - Task Inputs:
        - "questions": Optional: questions from the editor 
    - Task Outputs:
        "author_responses"
    - Task Description: The author responds to any questions from the editor.

Transition: "Document Completed"
    - From:  "Document Edit"
    - To Status: "Completed"
    - Condition: The "updated_doc" field is populated and the "questions" field is not

Transition: "Editor Questions"
    - From task:  "Document Edit"
    - To Task: "Question Responses"
    - Condition: The "questions" field is populated and the "updated_doc" field is not.

Transition: "Start Workflow":
    - From Task: NULL
    - To Task: "Document Edit"
    - Condition: always

Transition: "Questions Answered":
    - From Task: "Question Responses"
    - To Task: "Document Edit"
    - Condition: always

NOTES:

- A problem here is the questions and intermediate document will be overwritten each cycle. 
- I think we need to detect in SW if a workflow has no active tasks and is still active and flag this as an error.

Before we try to do any work on this, I would like and comments on this workflow and how the system accomadates it.

==============================

Your proposed "Document Edit" workflow is a good starting point for testing the system. It covers several key aspects of the workflow system, including multiple tasks, conditional transitions, and data flow between tasks. Here are my comments and observations on how this workflow fits into the system:

1. Workflow Structure:
The workflow is well-structured with clear tasks and transitions. It demonstrates a simple but realistic editing process, allowing for iteration between the editor and author.

2. Fields and Data Flow:
The workflow fields cover all necessary data points for the editing process. The system's use of JSON schemas for field definitions should accommodate this structure well.

3. Tasks:
The two tasks, "Edit Document" and "Question Responses," are clearly defined with appropriate inputs and outputs. The system's task definition structure should be able to represent these tasks accurately.

4. Transitions:
The transitions cover all possible paths in the workflow, including the initial transition from the ticket submission. The use of conditions in transitions is well demonstrated here.

5. Ticket:
The ticket definition aligns well with the initial workflow fields, providing a clear entry point for the workflow.

Regarding your notes and potential issues:

1. Overwriting of Questions and Intermediate Documents:
This is indeed a limitation of the current workflow design. To address this, you might consider:
   - Modifying the workflow fields to store arrays of questions and responses, allowing for multiple rounds of Q&A.
   - Adding version tracking to the document field to maintain a history of changes.

2. Detecting Stalled Workflows:
Your observation about detecting workflows with no active tasks is important. This isn't explicitly covered in the current system description, but it would be a valuable addition. You could consider:
   - Adding a periodic check for workflows that are still active but have no pending tasks.
   - Implementing a timeout mechanism for tasks to prevent indefinite waits.

Additional Considerations:

1. User Assignments: The current description doesn't specify how tasks are assigned to users. You might want to add a field for user assignments in the task instances.

2. Notifications: Consider how users will be notified of new tasks or updates in the workflow.

3. Permissions: Ensure that the system can enforce appropriate permissions (e.g., only the author can respond to questions, only designated editors can edit).

4. Validation: For each task, consider what validation might be necessary on the inputs and outputs.

Overall, this workflow serves as a good test case for the system. It uses many of the key features described in the guide, including conditional transitions and multi-step processes. Implementing this workflow will likely highlight areas where the system might need refinement or additional features.

Would you like to proceed with implementing this workflow using the JSON formats we discussed earlier? Or would you prefer to refine the workflow design further before implementation?

====================

--------------------

NOTES TO ME:

- In the above workflow, a task is reflected back to the author. But that is not really allowed in the workflow, since tasks are only done be task workers.
    - This could be addressed by making a automated task to alert the user
    - This could also be addressed by making company roles for doing tasks
        - BUT we also want to make sure only the USER does the question answering! Not any user! 

---------------------

Hmmm, there are a few issues:

1. Right now I have system roles that determine who can work on given objects, like a task. I had imagined there would be workers who would do the tasks, like in the lats company I worked for. But that is not sufficiently general. You mentioned this before. Here an arbitrary user who submits the document edit ticket needs to respond to the questions. So the roles need to be generalized to company roles can be set for all the actions.

2. Then, as you point out here, we need a way to assing to an individual. That is not factored in either. Assignments were to the worker group.
I assume there are multiple options, including giving creating a role for each user. Are there other common methods?

3. I would like to keep a history of all the changes. I have ideas for that but let's discuss that later. 

=====================

You're right to identify these issues. Let's address them one by one:

1. Generalizing Roles:
Your observation about needing more flexible role assignments is spot on. Instead of system-wide roles, a more general approach would be to use company-specific roles that can be assigned to various actions within a workflow. This allows for greater flexibility and customization for each company using the system.

2. Individual Assignments:
For individual assignments, there are indeed multiple approaches. Here are some common methods:

   a. User-specific Roles: As you mentioned, creating a role for each user. This can work but may become unwieldy in larger organizations.

   b. Dynamic Assignment: Allow the workflow to assign tasks based on certain criteria. For example:
      - The person who initiated the workflow (like the document submitter in your case)
      - A manager or supervisor role that can assign tasks to individuals
      - Round-robin assignment within a group
      - Assignment based on workload or availability

   c. Claim System: Tasks are placed in a pool, and eligible users can claim them.

   d. Matrix Assignment: Use a combination of role and other attributes (like department, expertise level, etc.) to determine eligibility for a task.

   e. Rule-based Assignment: Define rules in the workflow that determine who should be assigned to each task based on the workflow data and company structure.

A flexible system might implement a combination of these methods, allowing workflow designers to choose the most appropriate assignment method for each task.

3. Change History:
Keeping a history of changes is indeed important and we can discuss this in more detail later. However, it's worth noting that this requirement might influence how we design the data structures for workflows and tasks.

Given these considerations, here are some suggestions for adapting your system:

1. Modify the `TaskDefinition` to include an `assignment_method` field. This could be an enumeration of different assignment types (e.g., 'INITIATOR', 'ROLE', 'CLAIM', 'RULE').

2. Add an `AssignmentRule` model that can be linked to a `TaskDefinition`. This would allow for complex assignment logic when needed.

3. Modify the `WorkflowInstance` and `TaskInstance` models to include fields for tracking assignments and assignment history.

4. Consider adding a `CompanyRole` model that can be used to define custom roles for each company. These roles can then be referenced in workflow definitions and assignment rules.

5. Add a `UserAssignment` model to track which individual users are assigned to which tasks, including historical assignments.

Here's a sketch of how these new models might look:



```python
class CompanyRole(models.Model):
    company = models.ForeignKey(Company, on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)

class AssignmentRule(models.Model):
    task_definition = models.ForeignKey(TaskDefinition, on_delete=models.CASCADE)
    rule_type = models.CharField(max_length=20, choices=[
        ('INITIATOR', 'Workflow Initiator'),
        ('ROLE', 'Company Role'),
        ('CLAIM', 'Claim System'),
        ('ROUND_ROBIN', 'Round Robin'),
        ('CUSTOM', 'Custom Rule'),
    ])
    company_role = models.ForeignKey(CompanyRole, null=True, blank=True, on_delete=models.SET_NULL)
    custom_rule = models.TextField(blank=True)  # For storing custom assignment logic

class UserAssignment(models.Model):
    task_instance = models.ForeignKey(TaskInstance, on_delete=models.CASCADE)
    user = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    assigned_at = models.DateTimeField(auto_now_add=True)
    assigned_by = models.ForeignKey(CustomUser, related_name='assignments_made', on_delete=models.SET_NULL, null=True)
    status = models.CharField(max_length=20, choices=[
        ('ASSIGNED', 'Assigned'),
        ('ACCEPTED', 'Accepted'),
        ('DECLINED', 'Declined'),
        ('COMPLETED', 'Completed'),
    ])

# Update to existing models
class TaskDefinition(models.Model):
    # ... existing fields ...
    assignment_method = models.CharField(max_length=20, choices=[
        ('INITIATOR', 'Workflow Initiator'),
        ('ROLE', 'Company Role'),
        ('CLAIM', 'Claim System'),
        ('RULE', 'Assignment Rule'),
    ])

class TaskInstance(models.Model):
    # ... existing fields ...
    current_assignment = models.ForeignKey(UserAssignment, related_name='current_task', on_delete=models.SET_NULL, null=True)

class WorkflowInstance(models.Model):
    # ... existing fields ...
    initiator = models.ForeignKey(CustomUser, on_delete=models.SET_NULL, null=True)

```

These changes would allow for more flexible role and assignment management, addressing the issues you've identified. The system could then use these structures to determine who can perform each task and how tasks are assigned to individuals.

Would you like to discuss how these changes might be implemented in your current system, or do you want to explore any of these concepts further?

============================

I have some other concerns related to changing the workflow which is why I want to put off talking about history.

Let's continue discussing roles, permissions and assignment. This time, however, let's just discuss ideas and not write code yet. I want to work at a higher level in our discussion and not let it get muddled in details. 

First, in terms of permissions, there are many cases where you want to make sure only an individual has permission for a "task". For example, if it is an HR task given to an employee. How could permission be assigned to individuals? (Please respond more briefly than before.)

=========================

Certainly, I'll keep the response brief and focused on high-level ideas for assigning permissions to individuals:

1. Direct User Assignment: Assign tasks directly to specific users.

2. Role-Based + User Filter: Combine role-based permissions with a user filter.

3. Dynamic Attributes: Use user attributes (e.g., department, position) to determine permissions.

4. Task Metadata: Include metadata in tasks that specify eligible individuals.

5. Permission Matrix: Create a matrix mapping tasks to individual users.

6. Hierarchical Permissions: Use organizational hierarchy to determine individual access.

7. Token-Based Access: Provide secure tokens to individuals for specific tasks.

8. Rule Engine: Implement a flexible rule engine to determine individual permissions based on multiple factors.

These approaches can be used individually or in combination to achieve the desired level of granularity in permission assignment.

========================

I was thinking in the case of a normal role based permission system. How do you grant permissions to an individual?

==================

For granting permission to just one individual for a given task in your workflow system, considering we have the assignment portion worked out, here are some of the best approaches:

1. Task-specific Permission Override: Implement a system where you can add a permission override directly on the task instance, specifying the individual user who has permission.

2. Dynamic Task-User Linking: Create a linking table or model that associates specific task instances with individual users who have permission to perform them.

3. Token-based Access: Generate a unique, secure token for the task and provide it only to the intended individual.

4. Contextual Role: Create a temporary, context-specific role for the duration of the task that is assigned only to the specific individual.

5. Task Metadata: Include a field in the task instance that specifies the ID or username of the permitted individual.

The choice between these methods would depend on factors like the frequency of such specific assignments, the overall architecture of your system, and how it integrates with your existing permission structure. The key is to find a balance between flexibility, security, and ease of management.

====================

So let me see if I understand ow we could do #1. The task_instant could have a field that is populated with to provide an additional constraint on the permission for that task. We could choose it to be a single ID, a list of IDs, or a list of roles strings, with the option for a specially fomratted role string $user_[userd_id]. I guess we'd have to decide how general we would want this additional constraint.  

================

I will have to think more about the type of additional constraint. Would we want roles? Would we want one user? Would we want a list of users?

For now I'd like to talk about another issue that was bothering me. Defining these flows is very cumbersome.

Here's another approach which I think works well in some situations.

- There's a group of tasks each with its own Input dependencies and output.
- A task is executed when it's dependencies change. At this point it updates its output.

I guess you could call this a reactive system, like a spreadsheet is. This avoids the need of defining flows. In some cases it can become or some though.

What do you think of this approach? Are there any other approaches you think would work well?

=============

Your idea of a reactive, dependency-based system is indeed an interesting alternative to traditional workflow definitions. Let's analyze this approach and consider its implications:

Pros of the reactive approach:
1. Simplicity: Easier to define individual tasks without complex flow logic.
2. Flexibility: Tasks automatically execute when relevant data changes.
3. Parallelism: Tasks that don't depend on each other can run concurrently.
4. Adaptability: Easier to modify the system by adding or removing tasks.

Cons:
1. Complexity in large systems: Dependencies might become hard to track in complex scenarios.
2. Potential for circular dependencies: Need mechanisms to prevent infinite loops.
3. Lack of explicit sequencing: Might be challenging for processes that require strict ordering.
4. Debugging: Could be harder to trace the execution path when issues arise.

This approach works well for:
- Data processing pipelines
- Document approval systems with multiple independent reviewers
- Systems where the order of operations is flexible

Other approaches to consider:

1. Event-driven workflows: Define events that trigger specific tasks or transitions.

2. State machine-based: Define states and transitions between them, which can be simpler than full-fledged workflows.

3. Rule-based systems: Define rules that determine when and how tasks should be executed.

4. Hybrid approach: Combine aspects of traditional workflows with reactive elements for different parts of the system.

5. Visual programming: Use a drag-and-drop interface to create workflows, which can be more intuitive for non-technical users.

6. Template-based: Provide pre-defined workflow templates that users can customize, reducing the complexity of defining workflows from scratch.

The best approach often depends on the specific use cases, the complexity of the processes you're modeling, and the skills of the people who will be defining and maintaining these workflows. Your reactive approach could work very well for certain scenarios, particularly those with independent tasks and clear data dependencies.

===================

I wrote this up the same example using triggers. I would consider this kind of an reactive approach.

A specific task will be triggered by certain conditions regarding values and changes in values.

- Workflow: "Document Edit"
    - Workflow Fields:
        - "title" - The title for the document
        - "submitted_doc" - The text to be edited
        - "doc_notes" - Any notes from the submitter
        - "questions" - Questions from the editor for the author
        - "author_responses" - Responses from the author for the questions.
        - "update_doc" - The updated document
        - "edit_notes" - Notes on the edited document

- Ticket: "Document Edit Ticket"
    - Ticket Fields:
        - "title" - The title for the document
        - "submitted_doc" - The text to be edited
        - "doc_notes" - Any notes from the submitter

Task: "Edit Document"
    - Task Inputs:
        - "title"
        - "submitted doc"
        - "doc_notes"
    - Task Outputs:
        - "questions": Optional: questions from the editor
        - "updated_doc": OPtional: The modified document
        - "edit_notes": Optional: Notes from the editor
    - Task Description: An editor will take the inputs and do one of the following:
        - Ask questions of the author, populating the "questions" field of the response, leaving "updated_doc" and "edit_notes" empty.
        - Return an edited document, populating "updated_doc" and optionally "edit_notes", leaving questions empty.
    - Trigger:
        - "submitted_doc" is not empty
        - "submitted_doc" changes

Task: "Respond to Questions"
    - Task Inputs:
        - "questions": Optional: questions from the editor 
    - Task Outputs:
        - "author_responses"
    - Task Description: The author responds to any questions from the editor, placing them in "author_responses".
    - Trigger:
        - "questions" is not empty
        - "questions" changes

Task: "Mark Task Completed"
    - Task Outputs:
        -status
    - Task Description: set status completed (automated)
    - Trigger:
        - "updated_doc" not empty
        - "updated_doc" changes

A simple sequential workflow would have one task depending on the task before it completing.

Here we can also record the history of each field, so have the states over time.

Give me some pros and cons of this approach for workflows.

==========================

This reactive, trigger-based approach to workflows indeed offers an interesting alternative to traditional sequential workflows. Let's examine the pros and cons of this method:

Pros:

1. Flexibility: Tasks are executed based on data changes, allowing for more dynamic workflows.

2. Simplicity: Easier to define individual tasks without complex flow logic or explicit sequencing.

3. Parallelism: Tasks that don't depend on each other can potentially run concurrently.

4. Reactivity: The system automatically responds to changes, potentially reducing delays in the workflow.

5. Modularity: Easy to add or remove tasks without redesigning the entire workflow.

6. Data-centric: Focuses on the state of the data rather than the sequence of operations, which can be more intuitive for certain processes.

7. History tracking: As you mentioned, it's easier to record the history of each field, providing a clear audit trail.

8. Adaptability: Can handle variations in the process more easily than rigid, sequential workflows.

Cons:

1. Complexity in large systems: As the number of tasks and triggers increases, it might become difficult to understand and predict the overall behavior of the system.

2. Potential for race conditions: If multiple tasks are triggered simultaneously, you need to ensure proper handling of concurrent updates.

3. Debugging challenges: It might be harder to trace the execution path when issues arise, as the order of operations is not explicitly defined.

4. Lack of explicit sequencing: For processes that require a strict order of operations, this approach might be less intuitive to design and maintain.

5. Potential for circular dependencies: Need to implement safeguards to prevent infinite loops of triggering tasks.

6. Performance considerations: Constantly checking for trigger conditions could potentially impact system performance, depending on implementation.

7. Complexity in defining triggers: For complex workflows, defining the right triggers for each task can become challenging and error-prone.

8. Visualization difficulties: It might be harder to visually represent the workflow compared to traditional sequential diagrams.

9. Learning curve: Users accustomed to traditional workflow systems might find this approach less intuitive initially.

Overall, this trigger-based approach offers significant flexibility and can be very powerful for certain types of workflows, especially those that are more data-centric or require dynamic responsiveness to changes. However, it also introduces its own set of challenges, particularly in terms of complexity management and ensuring correct behavior in all scenarios. The suitability of this approach would depend on the specific requirements of your system and the types of workflows you need to support.

=======================

You mentioned an event based system. Could you illustrate how this example would be done in such a system?

=======================

Certainly. An event-based system for this document editing workflow would focus on actions and reactions to specific events. Here's how the example could be structured in an event-based system:

1. Events:
   - DocumentSubmitted
   - QuestionsAsked
   - QuestionsAnswered
   - DocumentEdited
   - WorkflowCompleted

2. Event Handlers:

   DocumentSubmitted Handler:
   - Triggered by: New document submission
   - Actions: 
     - Create new workflow instance
     - Assign to editor
     - Emit EditorAssigned event

   EditorAssigned Handler:
   - Triggered by: EditorAssigned event
   - Actions:
     - Notify editor of new document to review

   QuestionsAsked Handler:
   - Triggered by: Editor submitting questions
   - Actions:
     - Update workflow with questions
     - Assign to author
     - Notify author of questions

   QuestionsAnswered Handler:
   - Triggered by: Author submitting answers
   - Actions:
     - Update workflow with answers
     - Reassign to editor
     - Notify editor of answers

   DocumentEdited Handler:
   - Triggered by: Editor submitting edited document
   - Actions:
     - Update workflow with edited document
     - Check if questions were asked
     - If no questions were asked, emit WorkflowCompleted event
     - If questions were asked, assign to author for final review

   FinalReviewCompleted Handler:
   - Triggered by: Author approving final document
   - Actions:
     - Update workflow status
     - Emit WorkflowCompleted event

   WorkflowCompleted Handler:
   - Triggered by: WorkflowCompleted event
   - Actions:
     - Update workflow status to completed
     - Notify relevant parties of completion

3. Workflow State:
   The workflow state would still contain the fields you described (title, submitted_doc, questions, etc.), but instead of being directly modified by tasks, they would be updated by event handlers.

In this event-based system:

1. Each action in the workflow emits an event.
2. Event handlers listen for specific events and perform actions in response.
3. The workflow progresses through the emission and handling of events, rather than through predefined task sequences.

This approach allows for:
- Flexibility in adding new steps or branching logic by introducing new events and handlers.
- Easy extension of functionality by adding new handlers for existing events.
- Clear separation of concerns, with each handler responsible for a specific part of the workflow.
- Potential for easy scaling and distribution, as event handlers can be implemented as separate services.

The main difference from the trigger-based approach is that this system responds to specific actions (events) rather than changes in data state. This can make the flow of the process more explicit while still maintaining flexibility.

=============================

Here is another process. Could you give some biref comments about different approaches for this workflow. You don't need to generate all the workflows.

**Venue Map Creation Process**

- Submit Venue Data
- Create Venue Object in DB
    - DEP: "Venue Data"
- Geocode Venue Sources
    - DEP: "Venue Data", "Venue Object in DB"
- Digitize Venue Sources
    - DEP: "Venue Data", "Venue Object in DB"
- Add Venue Addresses to Map (eg room numbers)
    - DEP: "Venue Data", "Venue Object in DB", "Digitized Venue Sources"
- Create Entities (eg stores)
    - DEP: "Venue Data", "Venue Object in DB"
- Match Entities to Map where needed (when addresses not available)
    - DEP: "Venue Data", "Venue Object in DB", "Digitized Venue Sources", "Entities", "Venue Addresses"
- QA
    - DEP: "Venue Data", "Venue Object in DB", "Digitized Venue Sources", "Geocode", "Entities", "Venue Addresses"

============================

Certainly. This Venue Map Creation Process is an interesting workflow with multiple interdependent steps. Here are brief comments on how different approaches might handle this workflow:

1. Traditional Sequential Workflow:
   Pros: Clear, step-by-step process. Easy to visualize.
   Cons: Might be inefficient if some steps could be parallelized.

2. Parallel Workflow:
   Pros: Could speed up process by running independent steps simultaneously (e.g., Geocode and Digitize).
   Cons: Needs careful management of dependencies and potential resource conflicts.

3. Event-driven Approach:
   Pros: Flexible, can easily handle asynchronous processes (like geocoding).
   Cons: Might be complex to set up all the necessary event listeners and handlers.

4. Data-driven/Reactive Approach:
   Pros: Natural fit for this process, as many steps depend on data from previous steps.
   Cons: Need to carefully manage data dependencies to avoid circular references.

5. State Machine:
   Pros: Could clearly represent different states of the venue creation process.
   Cons: Might be overly complex for this linear process.

6. Microservices Architecture:
   Pros: Each step could be its own service, allowing for scalability and independent development.
   Cons: Adds complexity in terms of service communication and data consistency.

7. Rule-based System:
   Pros: Could handle complex logic for steps like entity matching.
   Cons: Might be overkill for the more straightforward steps in the process.

The best approach likely depends on factors like the scale of operation, frequency of map creation, available resources, and the need for flexibility. A hybrid approach, perhaps combining aspects of parallel workflow with data-driven triggers, could be effective for this particular process.

============================

This is the process we had at a Venue map company where I previously worked. The data approach works well here because it makes it run the steps in parallel and introduce maps back into the workflow if problems were found in QA.

I would like to think about some more workflows. Could generate a number of different kinds of workflows. Each one can be expressed in whatever terms is appropriate for that example, like I did above. 

========================

Certainly! I'll generate a variety of workflows from different domains, using appropriate terms and structures for each example. These will showcase different complexities and requirements that workflow systems might need to handle.

1. Customer Support Ticket Resolution

- Create Ticket (Customer)
- Assign to Support Agent
- Initial Response (Agent)
- Investigate Issue (Agent)
- Escalate to Specialist (if needed)
- Propose Solution
- Implement Solution
- Verify Resolution (Customer)
- Close Ticket

2. Software Development Sprint

- Sprint Planning
- Task Creation
- Development Work
- Code Review
- Testing
- Bug Fixing
- Documentation
- Sprint Review
- Sprint Retrospective

3. E-commerce Order Fulfillment

Events:
- OrderReceived
- PaymentProcessed
- InventoryChecked
- OrderPicked
- OrderPacked
- ShippingLabelCreated
- OrderShipped
- DeliveryConfirmed

4. Human Resources Onboarding

Tasks:
- Offer Accepted
- Create Employee Record
- Assign Equipment
- Set Up Accounts
- Schedule Orientation
- Complete Paperwork
- Conduct Training
- Assign Mentor
- First Week Check-in
- 30-day Review

Triggers:
- New hire start date
- Equipment received
- Training completed

5. Content Approval Process

States:
- Draft
- Submitted for Review
- In Review
- Revisions Requested
- Approved
- Published

Transitions:
- Submit: Draft -> Submitted for Review
- Start Review: Submitted for Review -> In Review
- Request Changes: In Review -> Revisions Requested
- Resubmit: Revisions Requested -> Submitted for Review
- Approve: In Review -> Approved
- Publish: Approved -> Published

6. Restaurant Kitchen Order Processing

Reactive Approach:
- Order Received -> Triggers: Ingredient Check, Prep Station Assignment
- Ingredients Available -> Triggers: Start Cooking
- Cooking Complete -> Triggers: Quality Check
- Quality Check Passed -> Triggers: Plating
- Plating Complete -> Triggers: Delivery to Server

7. Academic Paper Peer Review

- Submit Paper
- Editor Initial Review
- Assign Reviewers
- Conduct Reviews (parallel)
- Collect Reviews
- Editor Decision
- Revise and Resubmit (if needed)
- Final Decision
- Publication Preparation

8. Insurance Claim Processing

Data-Driven Approach:
- Claim Submission: {claimData, supportingDocuments}
- Initial Assessment: {claimData} -> {assessmentResult}
- Fraud Check: {claimData, assessmentResult} -> {fraudCheckResult}
- Adjudication: {claimData, assessmentResult, fraudCheckResult} -> {adjudicationResult}
- Payment Processing: {adjudicationResult} -> {paymentStatus}
- Claim Closure: {paymentStatus, allDocuments} -> {closureStatus}

These examples showcase various workflow structures and approaches, from traditional sequential processes to event-driven systems and data-dependent flows. Each could be further elaborated based on specific implementation needs.

=================================

From workflow software you are familiar with, how common are the different approaches for defining a workflow?

=======================

Based on my knowledge of various workflow software systems, I can provide an overview of the prevalence of different approaches for defining workflows. However, please note that my information might not reflect the very latest trends, as my training data has a cutoff date.

1. Traditional Sequential Workflows:
   Very common. Most workflow systems support this as it's intuitive and easy to understand.
   Examples: Microsoft Power Automate, Kissflow, Nintex

2. Parallel Workflows:
   Common in more advanced systems, often as an extension of sequential workflows.
   Examples: Camunda BPM, Pega

3. Event-driven Approaches:
   Increasingly common, especially in modern, cloud-native systems.
   Examples: AWS Step Functions, Zapier

4. Data-driven/Reactive Approaches:
   Less common in traditional workflow systems, but gaining popularity, especially in data processing pipelines.
   Examples: Apache NiFi, Luigi

5. State Machines:
   Moderately common, often used in complex systems with many possible states.
   Examples: AWS Step Functions, XState

6. Visual Programming / Drag-and-Drop Interfaces:
   Very common in business-oriented workflow systems.
   Examples: Microsoft Power Automate, Nintex, Kissflow

7. BPMN (Business Process Model and Notation):
   Common in enterprise-grade systems, provides a standardized approach.
   Examples: Camunda BPM, Bonita BPM

8. Rule-based Systems:
   Moderately common, often used in conjunction with other approaches.
   Examples: Drools, IBM Operational Decision Manager

9. Low-Code/No-Code Platforms:
   Increasingly common, often incorporating multiple approaches.
   Examples: OutSystems, Mendix

10. Microservices-oriented Workflows:
    Growing in popularity, especially in cloud-native environments.
    Examples: Netflix Conductor, Uber Cadence

Generally, traditional sequential and parallel workflows remain the most common, as they're intuitive and suit many business processes. Event-driven and data-driven approaches are gaining traction, especially in more technical environments. Visual programming interfaces are very popular in business-oriented tools, while BPMN is common in enterprise settings.

Many modern systems are adopting hybrid approaches, allowing users to define workflows using a combination of these methods to suit different needs.

===============================

I am interested in cases of operations workflows, with the intention of automating some of the steps over time. This is probably most meaningful in cases where there's a large number of workflow instances being tracked. Below I list the applicability of the examples we've talked about for this type of system:  
0. Map Example [YES]
1. Customer Support Ticket Resolution [YES] 
2. Software Development Sprint [NO]
3. E-commerce Order Fulfillment [YES]
4. Human Resources Onboarding [YES]
5. Content Approval Process [NO]
6. Restaurant Kitchen Order Processing [MAYBE] - Not really a good condidate for the automation I am thinking about, yet. (Maybe later)
7. Academic Paper Peer Review [NO]
8. Insurance Claim Processing [YES]
What type of workflow management systems are popular for these cases, both in terms of large estagblished players and smaller new players?

===========================

For operations workflows with high volume and potential for automation, especially in areas like map creation, customer support, e-commerce fulfillment, HR onboarding, and insurance claim processing, several workflow management systems stand out. Here's an overview of popular systems in this space:

Large Established Players:

1. ServiceNow: 
   - Widely used for IT service management, customer service, and HR processes.
   - Strong in automation and has AI capabilities.

2. Pega:
   - Known for its robust business process management and automation capabilities.
   - Used in customer service, insurance, and healthcare industries.

3. IBM Business Automation Workflow:
   - Part of IBM's Cloud Pak for Business Automation.
   - Handles complex, high-volume operational workflows.

4. SAP Workflow Management:
   - Integrated with SAP's broader ecosystem.
   - Good for companies already using SAP for ERP.

5. Appian:
   - Low-code platform with strong process automation features.
   - Used in financial services, insurance, and healthcare.

Smaller/Newer Players:

1. Camunda:
   - Open-source workflow engine with strong BPMN support.
   - Growing in popularity for microservices architectures.

2. Kissflow:
   - No-code platform focused on simplicity and ease of use.
   - Good for medium-sized businesses looking to automate operations.

3. Flokzu:
   - Cloud-based BPM solution.
   - Focuses on easy-to-implement process automation.

4. ProcessMaker:
   - Open-source workflow and BPM platform.
   - Strong in banking, higher education, and manufacturing.

5. Creatio:
   - Low-code platform for process management and CRM.
   - Used in financial services and professional services.

6. Nintex:
   - Known for its user-friendly interface and integration capabilities.
   - Popular for document-centric workflows.

7. Flowable:
   - Open-source business process engine.
   - Good for embedding workflow capabilities into other applications.

8. Bonitasoft:
   - Open-source BPM platform.
   - Strong in customization and extension.

These systems often provide:
- Visual workflow designers
- Integration capabilities with other systems
- Automation features (RPA, AI/ML integration)
- Analytics and reporting
- Mobile access
- Cloud and on-premises deployment options

The choice between these systems often depends on factors like:
- Scale of operations
- Existing tech stack and integration needs
- Level of customization required
- In-house technical expertise
- Budget
- Specific industry requirements (e.g., compliance in insurance)

For operations with a high volume of workflow instances and a focus on gradual automation, systems that offer strong integration capabilities, scalability, and advanced automation features (like ServiceNow, Pega, or Camunda) are often preferred. However, the specific choice would depend on the unique needs and constraints of the organization.

============================

Could you briefly describe the type of automation thee platforms do, first for the big established players, and then for any new players that offer something different.

=============================

Certainly. I'll describe the types of automation offered by the big established players and then highlight any unique automation approaches from newer players.

Big Established Players:

1. ServiceNow:
   - Offers robotic process automation (RPA)
   - AI-powered chatbots for customer service
   - Machine learning for predictive analytics and task routing
   - Natural language processing for ticket classification and sentiment analysis

2. Pega:
   - AI-driven process optimization
   - Robotic automation for repetitive tasks
   - Intelligent routing of work based on skills and workload
   - Predictive analytics for next-best-action recommendations

3. IBM Business Automation Workflow:
   - RPA for task automation
   - AI-powered document processing and data extraction
   - Cognitive capture for unstructured data
   - Decision automation using business rules and machine learning

4. SAP Workflow Management:
   - Intelligent routing of tasks
   - RPA integration
   - Machine learning for process mining and optimization
   - Conversational AI for user interactions

5. Appian:
   - Low-code RPA
   - AI-powered document processing
   - Intelligent task routing
   - Process mining for workflow optimization

Newer Players with Unique Approaches:

1. Camunda:
   - Focuses on orchestrating human tasks with automated microservices
   - Provides a lightweight, embeddable workflow engine for developers
   - Offers process automation as code, allowing version control of processes

2. UiPath:
   - Combines RPA with AI for end-to-end automation
   - Offers computer vision for interacting with legacy systems
   - Provides attended and unattended bots for different automation scenarios

3. Automation Anywhere:
   - Cloud-native, web-based RPA
   - AI-powered bots that can learn from human actions
   - Offers a marketplace for pre-built automation

4. Celonis:
   - Focuses on process mining and AI-powered process optimization
   - Automatically discovers inefficiencies in processes
   - Provides real-time, AI-driven recommendations for process improvements

5. Workato:
   - Recipe-based automation that combines API integrations with business logic
   - AI-powered suggestions for automation recipes
   - Offers both UI-based and code-based automation design

6. Nintex:
   - Combines workflow automation with document generation and e-signatures
   - Offers process mapping tools for visualization before automation
   - Provides RPA capabilities integrated with workflow management

The key difference in newer players often lies in their focus on specific aspects of automation (like Celonis with process mining), their integration capabilities (like Workato), or their approach to making automation more accessible to non-technical users. They also tend to be more agile and quicker to adopt emerging technologies compared to the established players.

======================

Tell me more about the automation that they provide at Camunda, UiPath and Workato.

=======================

Certainly. Let's dive deeper into the automation approaches of Camunda, UiPath, and Workato:

1. Camunda:

Camunda takes a developer-centric approach to process automation:

- Workflow Engine: Provides a lightweight, embeddable Java-based workflow engine that can be integrated directly into applications.

- BPMN Execution: Supports direct execution of BPMN 2.0 diagrams, allowing visual process designs to be run as-is.

- Microservices Orchestration: Specializes in coordinating automated tasks across distributed microservices architectures.

- Human Task Management: Integrates human tasks seamlessly within automated processes.

- Decision Automation: Uses DMN (Decision Model and Notation) for business rule automation.

- Automation as Code: Allows processes to be defined, versioned, and deployed using standard development practices.

- Cockpit and Optimize: Provides tools for monitoring and analyzing process performance in real-time.

2. UiPath:

UiPath focuses on Robotic Process Automation (RPA) with AI enhancements:

- Studio: A visual designer for creating automation workflows, including recording user actions.

- Robots: Software bots that can mimic human interactions with digital systems, including desktop applications and web interfaces.

- AI Center: Incorporates machine learning models into RPA processes for tasks like document understanding or decision making.

- Orchestrator: Manages and monitors robot deployment across the enterprise.

- Process Mining: Analyzes system logs to discover and visualize as-is processes before automation.

- Task Capture: Helps document human processes for analysis and automation.

- Test Suite: Automates software testing processes.

3. Workato:

Workato provides a platform for building automated workflows across different applications:

- Recipe-based Automation: Uses a concept of "recipes" (similar to IFTTT) to define integration and automation workflows.

- App Integrations: Offers a vast library of pre-built connectors to integrate with thousands of applications.

- AI-powered Suggestions: Uses machine learning to suggest automation recipes based on common patterns.

- Workbot: A chatbot that can be integrated with team communication tools to trigger and interact with automations.

- OpsAdvisor: AI-driven tool that analyzes automation usage and suggests optimizations.

- RPA Connector: Allows integration of RPA tools within broader automation workflows.

- Low-code/No-code Interface: Provides both a visual interface for non-technical users and a coding environment for more complex automations.

Key Differences:

- Camunda is more focused on developers and complex, distributed systems. It's particularly strong in scenarios where automation needs to be deeply embedded in custom applications.

- UiPath specializes in automating interactions with existing user interfaces, making it particularly useful for automating legacy systems or processes that span multiple applications.

- Workato excels in creating automated workflows that connect different cloud applications, with a focus on making this accessible to business users while still providing depth for technical users.

Each of these platforms offers unique strengths and would be suitable for different types of automation projects depending on the specific needs and technical environment of an organization.

=============================

Tell me a little more about the use cases for Camunda.

==================================

Certainly. Camunda is particularly well-suited for several specific use cases due to its architecture and approach to process automation. Here are some key use cases:

1. Microservices Orchestration:
   - Ideal for organizations with distributed architectures
   - Coordinates processes across multiple microservices
   - Manages long-running processes that span multiple services

2. Customer Onboarding in Financial Services:
   - Handles complex, multi-step onboarding processes
   - Integrates automated checks (e.g., KYC, credit checks) with human decision-making
   - Ensures compliance with regulations through auditable processes

3. Insurance Claims Processing:
   - Manages end-to-end claims workflows
   - Integrates automated assessments with manual reviews
   - Handles complex decision logic and routing of claims

4. Telecom Service Provisioning:
   - Coordinates the technical steps required to activate new services
   - Manages interactions between different systems and departments
   - Handles both automated tasks and human interventions

5. E-commerce Order Fulfillment:
   - Orchestrates the entire order process from placement to delivery
   - Integrates with inventory, payment, and shipping systems
   - Manages exceptions and special cases requiring human intervention

6. Healthcare Patient Management:
   - Coordinates patient care processes across different departments
   - Manages referrals, appointments, and treatment plans
   - Ensures adherence to healthcare protocols and regulations

7. IoT and Edge Computing Scenarios:
   - Manages processes that involve IoT devices and edge computing
   - Coordinates data collection, processing, and response actions
   - Handles complex event processing in real-time scenarios

8. DevOps and CI/CD Pipelines:
   - Automates software development and deployment processes
   - Coordinates build, test, and deployment steps across different tools and environments
   - Manages approvals and quality gates in the deployment process

9. Government and Public Sector Processes:
   - Handles complex, regulation-driven processes
   - Manages citizen service requests across multiple departments
   - Ensures transparency and auditability in government operations

10. Manufacturing and Supply Chain:
    - Coordinates production processes across different systems and machines
    - Manages quality control workflows
    - Handles supply chain processes involving multiple parties

Camunda is particularly effective in these scenarios because it:
- Allows for fine-grained control over process execution
- Supports both automated steps and human tasks within the same process
- Provides visibility and monitoring capabilities for complex, distributed processes
- Allows for process-as-code, fitting well into modern development practices
- Scales well for high-volume, mission-critical processes

Organizations often choose Camunda when they need deep integration of process automation into their custom applications, especially in scenarios involving microservices architectures or when they need granular control over process execution.

===================================
===================================
===================================

Takeways:

- In my system, I should be more flexible about who can do tasks. The company should define general roles for all actions: creating workflows, executing tasks, etc. My very first example task involved giving a task back to the person who submitted the ticket which I can't support with my current task workers' assumption.
- Again with permissions and that initial example it'd be nice to give permissions to an individual user for a task rather than just a role. So we should be able to say this task can only be done by this user.
    - A possible way to do this is to add a field onto a task_instance, adding a further restriction on who can do it. Maybe it should be a user id or a list of user ids? Maybe we allow roles? (I assume we can anticipate the roles for the task_definition.)
- I think workflows are very cumbersome as compared to dependencies or events. 
    - For a truly general system, maybe we could do multiple
    - More likely, I need a better idea of the application for this. And I can pick one method. I should probably just continue with what I have.
- There are huge existing products in this space. And there are lots of smaller companies coming out with AI automation. The bar is pretty high to contribute here.

If I want to contribute:

- I need to pick a sufficiently small area where I can build out a solution.
    - It should be an area where I am knowledgeable enough to contribute
    - It should be an area I am interested enough in.
    - I probably need contacts or other exposure.
    - I need to do my research.

The area that I am most interested in now:

- Using an AI assistant in an open-ended task where it relies on its training knowledge more than teaching it my specific rules.
- Interactive tasks involving AI and humans, along with purely automated tasks.
- Using AI for programming

I should continue this exercise at a minimum as a coding exercise.

Some possible outcomes related to coding:

- Go a better workflow for using AI for coding
    - automatically apply file edit and terminal commands
    - Get better with prompting and setting context
- Work towards automated context generation for the coding assistant
- Get the UI built!

