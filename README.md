
Zendesk AI Summarizer Microservice
=================================
Testing has been done in a Windows 10 system.

What this project does (FLOW)
----------------------
This is a Node.js microservice that handles Zendesk webhook-style POST requests to generate a summarization of tickets using an AI API service. The service workflow is:

1.(Optional) Request verification — It can verify incoming requests using HMAC-SHA256 and a shared secret (ZENDESK_SECRET); currently, this is disabled for local testing.
2.Payload validation — Ensures the JSON body contains the required fields: id, title, and description.
3.Summary generation — Calls summarizeContent(title, description) in functions.js to generate a summary. Currently, this uses a mock API (jsonplaceholder.typicode.com) to return random latin text from this API.
4.Add internal note — Calls addInternalNoteToTicket(ticketId, summary) to add the summary as a private comment on the Zendesk ticket using Zendesk API.
5.Response handling — Returns a 200 response with the summary if successful, or an error code if validation or API calls fail.


---------------------------
Running the service (local)
---------------------------
Requirements:
- Node.js v22.21.0 or newer (the code uses the global `fetch` API which is available in Node 18+). Download link for Windows x64 https://nodejs.org/dist/v22.21.0/node-v22.21.0-x64.msi
(** IMPORTANT **) If you encounter the error "cannot be loaded because running scripts is disabled on this system" do the following:

Option 1:
Run CMD or Powershell with admin rights

Option 2:
-Run in PowerShell:
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted
-To revert the policy back to restricted, run...
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Restricted

Steps:
Commands can be done in CMD or Powershell (run as administration recommended)

1. Install dependencies:
cd 'path/to/projectroot/'
npm install

2. Start the service:
npm start

3. The server will log the URL, e.g. http://localhost:3000

---------------------------
Testing the endpoint locally
---------------------------
Since request verification is commented out by default, you can directly POST a JSON payload with the required fields to test the flow.

curl example (you can simply copy and paste):
curl -X POST http://localhost:3000/ai-summary -H "Content-Type: application/json" -d "{\"id\":123,\"title\":\"Test ticket\",\"description\":\"This is a test description\"}"




Files of interest
-----------------
- `app.js` — Express server, request validation, POST endpoint `/ai-summary`, signature verification (commented out for local testing).
- `functions.js` — Two helpers:
	- `summarizeContent(title, description)` — currently calls a mock API;variables are not actually used; returned message is a random Latin text from the API
	- `addInternalNoteToTicket(ticketId, summary)` — sends a Zendesk API request to add an internal note to the ticket.
- `.env` — environment variables used by the app (see below).

Environment variables
---------------------
The service reads configuration from environment variables.
Used variables:
- `PORT` — port the Express server listens on (default in repo: `3000`).
- `ZENDESK_DOMAIN` — Zendesk domain (e.g. `yourcompany.zendesk.com`).
- `ZENDESK_EMAIL` — Zendesk account email (used with `/token:` when creating the Basic auth header).
- `ZENDESK_API_KEY` — Zendesk API token (also used in Basic auth header).
- `ZENDESK_SECRET` — shared secret used to verify incoming webhook signatures.


Notes about the current implementation
-------------------------------------
- summarizeContent currently uses a mock external ai API (jsonplaceholder).
- addInternalNoteToTicket constructs a Basic auth header using the pattern `Basic base64(ZENDESK_EMAIL + '/token:' + ZENDESK_API_KEY)`.
- The response-checking block inside `addInternalNoteToTicket` is commented out in this repo for local testing;

Troubleshooting
---------------
- If you see "fetch is not defined" errors: upgrade Node to v18+ or install `node-fetch` and import/assign it to `globalThis.fetch`.
- If the server doesn't start: ensure `PORT` is set in the environment or `.env` and that another process isn't already using that port.
