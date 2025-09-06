# Controller: MakeController
Base Route: /make

Handles API requests related to webinars, authorization, and attendees.

## 1. Authorize API Key

Endpoint:
GET /make/authorize

Headers:

x-api-key (string, required) – The API key associated with a workspace.

Description:
Validates an API key and returns the associated admin user’s email.

Response Example:

{
  "status": 200,
  "message": "Clé API valide",
  "email": "admin@example.com"
}


Errors:

401 Unauthorized – Invalid or missing API key.

## 2. Get Webinar List

Endpoint:
GET /make/webinar/list?apiKey=YOUR_API_KEY

Query Parameters:

apiKey (string, required) – The API key linked to the workspace.

Description:
Fetches all webinars and their active sessions (LIVE or PENDING) for the given workspace. Dates are formatted in French locale.

Response Example:

{
  "result": [
    {
      "sessionId": "sess_123",
      "title": "Webinar Title",
      "scheduledFor": "15 juillet 20h55"
    },
    {
      "sessionId": "sess_456",
      "title": "Webinar Title",
      "scheduledFor": "16 juillet 18h00 evergreen"
    }
  ]
}


Errors:

404 Not Found – Workspace not found.

## 3. Register User in Webinar

Endpoint:
GET /make/register/webinar?session_id=...&email=...&name=...

Query Parameters:

session_id (string, required) – The webinar or evergreen session ID.

email (string, required) – The attendee’s email.

name (string, required) – The attendee’s full name.

Description:
Registers a user in a webinar or evergreen session. If the user is already registered, it returns the existing session. Generates links for live and replay.

Response Example:

{
  "email": "user@example.com",
  "name": "John Doe",
  "webinar_name": "Sales Masterclass",
  "link_live": "https://app.welya.io/live/attendee123",
  "link_replay": "https://app.welya.io/replay/attendee123",
  "day_live": "15 juillet 2025",
  "time_live": "20:55"
}


Errors:

404 Not Found – Session not found.

500 Internal Server Error – Registration error.

## 4. Mark Attendee as Buyer in Live Session

Endpoint:
GET /make/live/buyer?name=...&email=...

Query Parameters:

name (string, required) – Attendee’s name (used for updating the record).

email (string, required) – Attendee’s email.

Description:
Marks an attendee as a buyer during a live session (standard or evergreen). Sends a real-time event message via the messaging service.

Response Example:

{
  "name": "John Doe",
  "email": "user@example.com"
}


Errors:

404 Not Found – Session or attendee not found.

# Service: MakeService

Provides the business logic for handling authorization, webinars, and attendees.

## 1. authorizeUserApiKey(apiKey: string)

Validates the given API key.

Fetches the workspace and its admin user.

Throws UnauthorizedException if the key is invalid.

## 2. getWebinarListFromApiKey(apiKey: string)

Finds the workspace linked to the API key.

Retrieves webinars and active sessions (LIVE, PENDING).

Formats session dates using Luxon in French locale.

## 3. registerUserInWebinar(sessionId, email, name)

Looks for a standard or evergreen session.

If the attendee already exists, returns their session.

Otherwise, creates a new attendee and generates live/replay links.

Formats the date and time of the session in French.

## 4. getLiveBuyer(name, email)

Finds an active (LIVE) session (standard or evergreen) where the attendee is registered.

Updates the attendee as a buyer.

Emits a user-buyer event via the messaging service with details (attendee info, conversion message, timestamp, evergreen flag).
