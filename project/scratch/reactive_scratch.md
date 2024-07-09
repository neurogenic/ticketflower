# Reactive Scratcj

## Editor Example

1. Submit Document Ticket
    - output:
        - title
        - orig_doc
        - author_notes

2. Edit Document Task
    - input:
        - title
        - orig_doc
        - author_notes
        - answers
    - output:
        - updated_doc
        - editor_notes
        - questions
    - output 1:
        - questions
    - output 2:
        - updated_doc
        - editor_notes

3. Answer Questions Task
    - input:
        - questions
    - output:
        - answers

4. Completion:
    - input:
        - updated_doc
        - editor_notes

### Workflow Fields

1. title
2. orig_doc
3. author_notes
4. questions
5. answers
6. updated_doc
7. editor_notes

## Progression

### no questions

1. Submit ticket:
    - title
    - orig_doc
    - author_notes

2. Edit document task:
    - updated_doc
    - editor_notes

3. Completion


### questions

1. Submit ticket:
    - title
    - orig_doc
    - author_notes

2. Edit document task:
    - questions

3. Answer questions task:
    - answers

4. Edit document task:
    - questions (updated)

5. Answer questions task:
    - answers (updated)

6. Edit document task:
    - updated_doc
    - editor_notes

### error?

1. Submit ticket:
    - title
    - orig_doc
    - author_notes

2. Edit document task:
    - questions
    - updated_doc
    - editor_notes

3a. Answer questions task
3b. Completion

?: Error because completion triggered with questions asked.

## Alternate approach

1. Each task has inputs and a condition.

2. If any inputs change, the task condition is checked to see if the task should be run.

    - Order of operations:
        - All tasks done simultaneously, based on initial data state
            - It is up to the designer not to define a nonsensical circular reference update
        - Error: multiple parallel tasks write to the same value

3. Updated value is applied at when all data collected.

    - Workflow data is "versioned". Version number increments after each stage of task completions
    - tasks are given input_version defined by the workflow data version.

4. Based on workflow fields that are updated, next tasks are checked.

Issues:

- You can make a circular reference that doesn't make sense.
    - Resolution: Yes. Don't do that.
- You can define parallel tasks that have outputs updating the same workflow field
    - Resolution: Error!
- Supposed we want to assign multiple reviewers to a document and have the review the document in parallel. Works, but we can't have a dynamic number.
    - We could have a max number of reviewers, and then not fill some in some reviewers.

### Edit Document Workflow

note:

- We can probably implicitly include an "and" condition with the first half being any input was updated.
- This is contrived. We could add to workflow requests for change from the editor and have the author update it.

1. Submit Document Ticket
    - output:
        - title
        - orig_doc
        - author_notes

2. Edit Document Task
    - input:
        - title
        - orig_doc
        - author_notes
        - answers
    - condition:
        - (answers updated) or ((title updated) or (orig_doc updated) or (author_notes updated))
    - output:
        - updated_doc
        - editor_notes
        - questions
        - edit_completed

3. Answer Questions Task
    - input:
        - questions
    - condition:
        - (questions updated) and not (questions empty) and not (edit_completed)
    - output:
        - answers

4. Completion:
    - input:
        - updated_doc
        - editor_notes
        - edit_completed
    - condition:
        - (edit_completed)

