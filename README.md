# AI Email Writer Monorepo

## Overview
An AI-powered email reply generator consisting of:

- **Chrome Extension**: Injects an "AI Reply" button into Gmail and inserts generated replies.
- **React Web App**: Simple UI to paste an email and generate a reply.
- **Spring Boot Backend**: REST API that calls Google Gemini to generate the reply text.

All clients call the backend at `http://localhost:8080`.

## Repository Structure
- `AI_Email_Chrome_Extension/`
  - Chrome Extension (Manifest V3) with `content.js` injected into Gmail.
- `AI_Email_Writer/email-writer-sb/`
  - Spring Boot backend API (Maven).
- `email-writer-react/`
  - React + Vite app using MUI.

## Tech Stack
- **Frontend**: React 18, Vite, MUI, Axios.
- **Extension**: Chrome Manifest V3 content script targeting Gmail.
- **Backend**: Spring Boot 3, WebFlux (WebClient), Lombok, Jackson.
- **AI Provider**: Google Gemini (HTTP API).

## Backend (Spring Boot)
Path: `AI_Email_Writer/email-writer-sb/`

### Requirements
- Java JDK 23 (as per `pom.xml` `java.version=23`)
- Maven 3.9+
- A Google Gemini API Key

### Configuration
Set the following in `src/main/resources/application.properties`:

```
# App
spring.application.name=email-writer-sb

# Gemini API (example values)
# Base URL should include the model and method, leaving only the `key` to be appended by the service.
# Common example (adjust model/version as needed):
# https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=

gemini.api.url=YOUR_GEMINI_GENERATE_CONTENT_URL_WITH_KEY_PARAM_PREFIX
# e.g. gemini.api.url=https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=

gemini.api.key=YOUR_GEMINI_API_KEY
```

The service constructs the final request URL as `gemini.api.url + gemini.api.key` and sends a JSON body with the prompt.

### Run
```
# from AI_Email_Writer/email-writer-sb/
mvn spring-boot:run
# or
mvn clean package && java -jar target/email-writer-sb-0.0.1-SNAPSHOT.jar
```
The server listens on `http://localhost:8080` by default.

### API
- `POST /api/email/generate`
  - Request JSON:
    ```json
    {
      "emailContent": "<original email text>",
      "tone": "professional" | "casual" | "friendly" | "" 
    }
    ```
  - Response: `text/plain` body with the generated reply.
  - CORS: `@CrossOrigin(origins = "*")` enabled.

Key classes:
- `EmailWriterSbApplication` (entry): `com.email.writer.EmailWriterSbApplication`
- `EmailGeneratorController`: `POST /api/email/generate`
- `EmailGeneratorService`: builds prompt and calls Gemini via `WebClient`
- `EmailRequest`: DTO with `emailContent` and `tone`

## React App (Web UI)
Path: `email-writer-react/`

### Scripts
```
npm install
npm run dev      # start Vite dev server
npm run build    # production build
npm run preview  # preview built app
```

The app posts to `http://localhost:8080/api/email/generate` (hardcoded in `src/App.jsx`). Ensure the backend is running.

## Chrome Extension
Path: `AI_Email_Chrome_Extension/`

### What it does
- Loads on Gmail pages (`*://mail.google.com/*`).
- Observes compose windows and injects an "AI Reply" button.
- On click, posts the current email content and optional tone to the backend.
- Inserts the generated text into Gmail's compose textbox.

### Permissions and Hosts
Defined in `manifest.json`:
- `permissions`: `activeTab`, `storage`
- `host_permissions`: `http://localhost:8080/*`, `*://mail.google.com/*`

### Load Unpacked (Chrome)
1. Go to `chrome://extensions`.
2. Enable Developer Mode.
3. Click "Load unpacked" and select the `AI_Email_Chrome_Extension/` folder.
4. Open Gmail, compose/reply, and use the injected "AI Reply" button.

## Development Notes
- The backend expects valid Gemini configuration. Without it, `POST /api/email/generate` will fail.
- If using a different port or host, update:
  - Chrome: `AI_Email_Chrome_Extension/manifest.json` and `content.js` fetch URL.
  - React: change the URL in `email-writer-react/src/App.jsx`.
- For production, consider moving API base URLs to environment configs and securing the API.

## Troubleshooting
- 404/Network errors from clients:
  - Ensure backend is running and reachable at `http://localhost:8080`.
  - Check CORS and host permissions.
- Backend startup issues:
  - Verify JDK 23 is installed and `JAVA_HOME` set.
  - Ensure `gemini.api.url` and `gemini.api.key` are configured.
- Gemini API errors:
  - Confirm model path and API key are valid.

