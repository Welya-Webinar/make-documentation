# MakeController

The controller defines the API routes under the /make prefix.

## GET /make/authorize

Description:
Validates an API key and returns the associated admin user’s email.

Headers:

x-api-key (string, required) – Workspace API key.

Responses:

200 OK: Returns a success message and admin email.

401 Unauthorized: If the API key is missing or invalid.

## GET /make/webinar/list

Description:
Fetches a list of upcoming webinars (standard and evergreen) for a workspace using its API key.

Query Parameters:

apiKey (string, required) – Workspace API key.

Responses:

200 OK: Returns an array of active sessions with:

sessionId

title

scheduledFor (formatted in French locale, e.g., 15 juillet 20h55)

404 Not Found: If no workspace is found.

401 Unauthorized: If API key is missing.

## GET /make/register/webinar

Description:
Registers a user for a specific webinar session (standard or evergreen).

Query Parameters:

session_id (string, required) – The webinar session ID.

email (string, required) – User’s email address.

name (string, required) – User’s full name.

Responses:

200 OK: Returns registration details including:

email, name, webinar_name

link_live (user-specific live session URL)

link_replay (replay link)

day_live and time_live (formatted in French locale).

400 Bad Request: If parameters are missing or email is invalid.

404 Not Found: If no session is found.

## GET /make/live/buyer

Description:
Marks a registered attendee as a buyer in an ongoing live webinar session and notifies the messaging service.

Query Parameters:

name (string, required) – Attendee’s name.

email (string, required) – Attendee’s email.

Responses:

200 OK: Returns the updated attendee’s name and email.

400 Bad Request: If parameters are missing or email is invalid.

404 Not Found: If no active session or attendee is found.

# MakeService

The service layer implements the business logic and interacts with the database using Prisma.

## authorizeUserApiKey(apiKey: string)

Validates the API key against the workspace table.

Ensures at least one admin user exists in the workspace.

Returns the admin’s email if valid.

Throws:

UnauthorizedException if key is missing or invalid.

## getWebinarListFromApiKey(apiKey: string)

Finds webinars linked to a workspace via its API key.

Fetches LIVE or PENDING sessions for both standard and evergreen webinars.

Formats scheduledFor dates in French locale.

Throws:

UnauthorizedException if key is missing.

NotFoundException if workspace does not exist.

## registerUserInWebinar(sessionId: string, email: string, name: string)

Validates input parameters and email format.

Checks if session exists (standard or evergreen).

Prevents duplicate registrations by checking existing attendees.

Creates a new webinarAttendee entry if not already registered.

Returns session links and formatted date/time.

Throws:

Error if parameters are missing or email invalid.

NotFoundException if no matching session is found.

## getLiveBuyer(name: string, email: string)

Validates input parameters and email format.

Checks if the attendee is part of an ongoing LIVE session (standard or evergreen).

Updates the attendee record with isBuyer: true.

Sends a user-buyer event through the MessagingService.

Returns attendee information.

Throws:

Error if parameters are missing or email invalid.

NotFoundException if no active session or attendee is found.
