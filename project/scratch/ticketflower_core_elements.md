# TicketFlower: Core Elements Structure

## 1. Task

- **Name**: String
- **Description**: Text
- **References**: Array of {URI: String, Type: String}
- **Data**:
  - Inputs: Array of {FieldName: String, FieldType: String}
  - Outputs: Array of {FieldName: String, FieldType: String}
- **Display Layout**: TBD (will be used to generate HTML)

## 2. Workflow

- **Name**: String
- **Description**: Text
- **References**: Array of {URI: String, Type: String}
- **Data**: Array of {FieldName: String, FieldType: String}
- **Tasks**: Array of Task IDs
- **Display Layout**: TBD (will be used to generate HTML for overall workflow status)

## 3. Ticket

- **Name**: String
- **Description**: Text
- **References**: Array of {URI: String, Type: String}
- **Workflow**: Workflow ID
- **Data**: Object of {FieldName: Value} (subset of Workflow Data fields)
- **Input Layout**: TBD (will be used to generate HTML form for ticket creation)
- **Display Layout**: TBD (will be used to generate HTML for ticket status display)

## 4. Implementation Considerations

### 4.1 Data Storage
- Use a flexible schema database (e.g., MongoDB) to accommodate the varying structure of Tasks, Workflows, and Tickets.
- Store layouts as JSON objects that can be easily translated into HTML.

### 4.2 AI-Assisted Creation
- Develop an API endpoint to accept document uploads and text input.
- Implement a conversational interface for the AI assistant.
- Create a processing pipeline that can:
  1. Extract relevant information from uploaded documents.
  2. Interpret user intent and requirements from conversation.
  3. Generate proposed Task, Workflow, and Ticket definitions.
  4. Allow for iterative refinement based on user feedback.

### 4.3 Layout Generation
- Develop a simple layout engine that can convert JSON layout definitions into HTML.
- Start with a set of basic components (e.g., text fields, dropdowns, file uploads) that can be easily composed.
- Allow for custom CSS classes to be applied for styling.

### 4.4 Reference Handling
- Implement a flexible URI handler that can manage various types of references (e.g., web links, file uploads, internal documents).
- Create a simple viewer interface that can display different types of references appropriately.

### 4.5 Workflow Execution
- Implement a basic state machine to manage the progression of a ticket through its associated workflow.
- Develop a simple task assignment system (e.g., a queue of available tasks).
- Create interfaces for users to claim and complete tasks, updating the ticket data as they progress.

## 5. MVP Focus Areas

1. AI-assisted creation of Tasks, Workflows, and Tickets
   - Implement document upload and processing
   - Develop conversational AI interface for refining definitions
2. Flexible data model to accommodate various ticket and workflow types
3. Simple layout generation for task, workflow, and ticket displays
4. Basic workflow execution and task assignment system

## 6. Future Considerations

- Enhanced layout capabilities (drag-and-drop interface, more complex components)
- Advanced workflow features (conditional branching, parallel tasks)
- Integration with external systems for task automation
- Analytics and reporting features
- Mobile-friendly interfaces

========================

## COMMENTS

1. Layout Generation - I am leaning towards defining the layout by having the assistant directly generaing HTML, or using a similar known framework from its training set.
2. I think we will want to define "software tasks" fairly early, possibly directly in the MVP. These will consist of actions taken with code or an assistant as steps in a workflow. One such example will be converting data as needed from ticket inputs or task outputs to be used for task inputs. On top of this we can provide additional processing functionality. Here I think we will be using code, probably python, generated by the agent.

In both the cases above, I want to leverage the ability of the AI assistant to write code to avoid restricting the user to specified display layouts or action flows. This is one of the common complaints of using low-code system that I think we can overcome.

