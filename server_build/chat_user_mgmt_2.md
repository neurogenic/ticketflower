# Chat User App

===================


TicketFlower is a ticketing and workflow system. 

## Workflow Functions

NOTE: This is a multi-tenant web site. Authorization for the workflow function requires a user to have the proper system role PLUS the user must be in the proper company.

System roles: 'admin', 'user', 'ticket_admin', 'task_worker', 'workflow_creator'

### General User Pages

Ticket submission functionality for general users in a given company.

- Submit Tickets
- My Tickets
- Ticket Detail

System role: 'user'
Company role: The customer has the option of assigning company roles to restrict access to specific tickets

### Ticket Admin

Access to workflow tickets and tasks. A ticket display will show the associated workflow.

- Tickets
- Ticket Detail
- Tasks
- Task Detail

We will support different options for assigning and tracking tickets, but for starters I think we will automatically give them to the available worker with the shortest list. Later we can think about giving options here. (Also, more advanced ticket tracking and analytics will be added later.)

System role: 'ticket_admin'

### Task Workers

- My Tasks (To execute)
- Task Execution (For manual/Form type tasks)

System role: 'task_worker'

### Workflow Creators

This will be for workflow designers. For now we will build out minimal functionality to upload workflow, task and ticket definitions. Later this will form the heart of the services value proposition.

- Workflow Definitions
- Workflow Definition Detail
- Task Definitions
- Task Definition Detail
- Ticket Definitions
- Ticket Definition Detail

- Workflow Definition Upload (Create and edit)
- Task Definition Upload (Create and edit)
- Ticket Definition Upload (Create and edit)

System role: 'workflow_creator'

===================

# Shared Files

/share requirements.txt ticketflower/settings.py ticketflower/urls.py users/models.py users/views.py users/apps.py users/admin.py users.tests.py

===============

We have done a bit of work on user management, but we might have to rethink some of it.

We are using JWT, however we are interested in implementing a lot of crud operations as simple static react pages, one for "my tickets", one to submit a ticket, etc. We will also have other servers for some more compute intensive and complex operations, which is why we wanted JWT.

I want to examine how we can use JWT with our simple web pages. One approach I want to look at is HTPP-only cookies.

**At this point I only want to have discussions. I want to fully understand the options before we start wrinting any code.**

For starters, briefly tell me about using JWT with HTTP-only cookies for simple web pages.

===========================

> Coder_Assistant: Certainly! I'll provide a brief overview of using JWT with HTTP-only cookies for simple web pages.

Using JWT with HTTP-only cookies is a good approach for securing simple web pages while maintaining the benefits of JWTs. Here's a brief explanation:

1. JWT Storage:
   Instead of storing the JWT in localStorage or sessionStorage (which are vulnerable to XSS attacks), you store it in an HTTP-only cookie.

2. HTTP-only Cookie:
   This is a special type of cookie that can't be accessed by JavaScript. It's only sent to the server with HTTP requests, making it more secure against XSS attacks.

3. Authentication Flow:
   - User logs in, and the server sends the JWT in an HTTP-only cookie.
   - For subsequent requests, the browser automatically includes this cookie.
   - The server validates the JWT on each request.

4. CSRF Protection:
   While HTTP-only cookies protect against XSS, they're still vulnerable to CSRF attacks. You'll need to implement CSRF protection (e.g., using CSRF tokens).

5. Refresh Tokens:
   You can use a similar approach for refresh tokens, storing them in a separate HTTP-only cookie.

6. Static React Pages:
   Your React pages would make API calls to your Django backend. The browser automatically includes the JWT cookie in these requests.

7. Cross-Origin Considerations:
   If your frontend and backend are on different domains, you'll need to configure CORS appropriately and use the 'SameSite' cookie attribute.

This approach combines the flexibility of JWTs with the security benefits of HTTP-only cookies, making it suitable for simple web pages while still allowing for more complex operations on other servers.

Would you like to discuss any specific aspect of this approach in more detail?

==============================

I've shared several files from the Django project. Could you take a look at those and briefly tell me what we would need to do to start using jwt in an http only cookie.

Again I don't want to write any code yet.

=====================

> Coder_Assistant: Certainly! I've reviewed the shared files from your Django project. To start using JWT in an HTTP-only cookie, you'll need to make some modifications to your existing setup. Here's a brief overview of the changes you'd need to consider:

1. JWT Configuration:
   In `settings.py`, you already have JWT configuration with `SIMPLE_JWT`. You'll need to add settings for using cookies.

2. Authentication Views:
   In `users/views.py`, you have a `UserLoginView`. This would need to be modified to set the JWT in an HTTP-only cookie instead of returning it in the response body.

3. Middleware:
   You'll need to add a custom middleware to extract the JWT from the cookie and add it to the request for authentication.

4. CORS Settings:
   If your frontend is on a different domain, you'll need to configure CORS in `settings.py` to allow credentials.

5. Cookie Settings:
   You'll need to set appropriate flags for the cookie (HTTP-only, Secure, SameSite) when creating it.

6. Refresh Token Handling:
   The `TokenRefreshView` in `urls.py` would need to be customized to work with cookies.

7. Logout Functionality:
   You'll need to implement a logout view that clears the JWT cookie.

8. CSRF Protection:
   Implement CSRF protection, possibly using Django's built-in CSRF middleware and tokens.

9. Frontend Adjustments:
   Your React frontend would need to be adjusted to work with cookies instead of manually handling JWTs.

These changes would primarily affect your authentication flow and how you handle JWTs in your application. The core of your user management (`users/models.py`) and most of your existing views wouldn't need significant changes.

Remember, this is just an overview. Each of these points would require careful implementation and testing to ensure security and proper functionality.

Would you like to discuss any of these points in more detail?

==============

Suppose we do this. will we be able to support JWT for a rest API and the HTTP-only tokens for the web pages? (Please answer briefly.)

===============

> Coder_Assistant: Yes, you can support both JWT for a REST API and HTTP-only cookies for web pages simultaneously. Here's a brief explanation:

1. For web pages: Use HTTP-only cookies to store JWTs.
2. For REST API: Continue to use token-based authentication, returning JWTs in the response body.

You'd implement different authentication classes or middleware to handle each case. The server would check the request type or headers to determine which authentication method to use.

This dual approach allows you to maintain a secure method for web pages while still supporting API clients that expect traditional JWT handling.

=================

This may sound like a silly question but how would they react web pages work. Suppose the user fills out a form to create a new ticket. When he submits the information he should be redirected to a new page I guess. Could we use arrest api for the submission? What's the standard practice?

====================

> Coder_Assistant: Not a silly question at all. It's an important consideration for user experience and application architecture. Here's a brief overview of how this typically works:

1. Form Submission:
   Yes, you can use a REST API for the submission. This is a common practice in modern web applications.

2. React Handling:
   - The React component handles the form submission event.
   - It sends the data to your Django backend using an API call (e.g., using fetch or axios).
   - The HTTP-only cookie with the JWT is automatically included in this request.

3. After Submission:
   - Once the API call is successful, you have two main options:
     a) Use React Router to programmatically navigate to a new page (e.g., a "Ticket Created" confirmation page or the ticket details page).
     b) Update the current page to show a success message and clear/reset the form.

4. Standard Practice:
   - Single Page Applications (SPAs) often update the current view rather than fully reloading.
   - However, redirecting to a new "page" (still within the SPA) is also common, especially for major state changes like creating a new resource.

The choice between updating the current view or navigating to a new one depends on your specific UX design and application flow. Both approaches are valid and widely used.

Would you like to discuss the pros and cons of these approaches?

===================

I've done a lot of SPAs but this case seems more like a simple crud website. It seems like a non-SPA would be a better choice. How would this work? (I haven't done this in a long time.)

===================

> Coder_Assistant: You're right, for a simple CRUD website, a non-SPA approach can be more straightforward and often more suitable. Let's briefly discuss how this would work:

1. Server-Side Rendering:
   - Django would handle both the backend logic and rendering of HTML pages.
   - Each page (like "create ticket", "view tickets", etc.) would be a separate Django view.

2. Form Submission:
   - Forms would use traditional HTML form submissions, not AJAX.
   - When a user submits a form, it sends a POST request to the server.

3. Page Navigation:
   - After form submission, Django would process the data and redirect to a new page (e.g., a success page or the newly created ticket's detail page).
   - This redirect is handled server-side using Django's redirect function.

4. Templates:
   - You'd use Django's template system instead of React components.
   - Each view would render a specific template.

5. Authentication:
   - You could still use JWT stored in HTTP-only cookies for authentication.
   - Django's session middleware could also be an option for simpler setups.

6. JavaScript Usage:
   - You'd use less JavaScript, mainly for enhancing user interactions rather than managing the entire page lifecycle.
   - Libraries like htmx or Alpine.js could be used for dynamic interactions without building a full SPA.

7. Workflow Example:
   - User navigates to "Create Ticket" page (GET request).
   - Django renders the form using a template.
   - User fills out the form and submits (POST request).
   - Django processes the form, creates the ticket, and redirects to a success page or ticket detail page.

This approach simplifies development, reduces client-side complexity, and often results in faster initial page loads. It's well-suited for straightforward CRUD operations and simpler user interfaces.  

Would you like me to elaborate on any specific part of this non-SPA approach?

====================

I was thinking of using static react pages rather than Django's template system. As part of the ticketing system and as part of the workflow we'll be using custom forms. I was thinking of doing these and react. How would the site work with static react pages?

=======================

> Coder_Assistant: Ah, I see. Using static React pages for a non-SPA approach is an interesting hybrid solution. This can work well, especially for custom forms. Let's break down how this would function:

1. Static React Pages:
   - Each page (e.g., "create ticket", "view tickets") would be a separate React application.
   - These React apps would be built and served as static files.

2. Server Setup:
   - Django would serve these static React pages.
   - You'd configure URLs in Django to serve the appropriate static page for each route.

3. Form Handling:
   - React would manage form state and validation on the client side.
   - On form submission, React would make an API call to your Django backend.

4. API Interaction:
   - Your Django backend would expose REST APIs for CRUD operations.
   - React pages would interact with these APIs using fetch or axios.

5. Authentication:
   - JWT in HTTP-only cookies would still work here.
   - The browser would automatically include the cookie in API requests.

6. Navigation:
   - Instead of client-side routing, you'd use traditional page loads.
   - After form submission, you could redirect to a new page using window.location or similar.

7. Workflow Example:
   - User navigates to "Create Ticket" page (loads a static React page).
   - React handles form display and client-side validation.
   - On submit, React sends data to Django API.
   - After successful submission, React code redirects to a new page (e.g., ticket detail).

8. Custom Forms:
   - React is excellent for building complex, dynamic forms.
   - You can easily implement conditional fields, real-time validation, etc.

9. Performance Considerations:
   - Initial page load might be slower than server-rendered pages.
   - Subsequent interactions would be fast and smooth.

10. Development Workflow:
    - You'd need to set up a build process to compile React pages into static files.
    - These built files would then be served by Django.

This approach gives you the flexibility of React for complex UI elements while maintaining a traditional multi-page application structure. It's a good middle ground between a full SPA and a server-rendered application.

Would you like to discuss any specific aspect of this approach in more detail?

=====================

It sounds like I might be non conventional to use react for a non-SPA. Is that true?

