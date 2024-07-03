# GPT Arch Scratch

I want to think a little more about what sort of architecture I need for the service. I don't have too clear a picture yet.

I have the simple workflow operations page:

- user pages:
    - submit ticket
    - my tickets
    - ticket detail
- ticket admin pages
    - tickets
    - ticket detail
    - tasks (as in workflow steps)
    - task detail
- task workers
    - my tasks
    - task detail

There will also be workflow designer pages. This will be an involved process using an AI agent to design forms and workflows and even to create tasks, including scripts and AI assistant tasks.

Do you have suggestions on how to architect this? Perhaps I can keep the simple django server for "operations" and have the remote servers use REST calls to set (and retrieve) workflow definition data?

===========

...

===========

The workflow definition process will be complex but one of the outputs the workflow definitions should be pretty simple. I was thinking of hosting these in the operations database. When I say operations, I mean the simple Django Server. The workflow definition is referred to the workflow which includes transitions between tasks the task themselves, and the various tickets. Externally I want to have something called actions which are the way to execute a task. So the operation server will identify a task (along with its associated action, for the case of an automated action) and the operations server will call the action server to do the action.

So I guess there might be four types of servers - the operations server, the workflow designer server (which designs workflows, tasks and tickets), the action execution designer server (which defines automated actions, the hardest task) and the action execution server (which executes the tasks, the most compute intensive task)

