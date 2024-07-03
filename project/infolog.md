# Infolog: TicketFlower

## 7/2/24

progress:

- Added the workflow design application.


Notes:

- I have some authentication problems:
    - The admin login asks for username
    - The authenticate function takes the field "username" (your new code is identical to the existing code, I believe)
    - Our user table doesn't have a field called username. Only email.
    - The login took clear text.
- ALSO, I don't think JWT is a good match for my simple static react website.
    - recommendation: use JWT in a HTTP only cookie?
- ALSO, I think maybe I should be doing django forms rather than react. It keeps wanting to make an SPA from react, which makes sense.



Stimula Notes:

- I should exclude the migrations for the directory display in context
- Change line numbers to be 1 based (maybe make it a setting)
- Add a color to the label for Claude in the terminal UI

- Claude did a line number update. The start line was right but I think the end was one too high. He also added one line. TBD what happened and if we should make any updates.
- it has also been doing different combinations on the update, such as:
    - update with complete content, not action
    - update on complete content, with action "relcae"
    - updates using "append"
- DOH! I got an error! had to restart. I need to handle errors.


## 7/1/24

- Added some updates to stimula to improve claude coder.
    - Shared files are either with line num or without. I always forgot to add linnums if it was a separate function.
    - Shared files can be specified from the project directory.
    - Added a "Project Info" feature, which prints a file to the context, for project info.
- Setup the ticketflower_server project
    - see server_build/chat_proj_setup.md
- Implemented basic user management
    - see server_build/chat_user_mgmt_1.md


Next:

- Start on the ticket/workflow application
