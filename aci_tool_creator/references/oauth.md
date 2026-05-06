# OAuth2 — Schema & Example

Use when the API uses OAuth 2.0 authorization code flow (user grants access via a consent screen).

---

## Research Checklist for OAuth APIs

Find these before writing the files:
- [ ] `authorize_url` — where the user goes to grant permission (e.g. `https://accounts.google.com/o/oauth2/v2/auth`)
- [ ] `access_token_url` — where the app exchanges the code for a token (e.g. `https://oauth2.googleapis.com/token`)
- [ ] `refresh_token_url` — usually the same as access_token_url
- [ ] `scope` — space-separated list of permission scopes needed (research which scopes the endpoints require)
- [ ] `client_id` / `client_secret` — these are NEVER hardcoded; use `{{ APPNAME_CLIENT_ID }}` placeholder format
- [ ] Token location — almost always `header` with `Authorization: Bearer {token}`

**Logo:** Always include the `logo` field. Search for an official SVG/PNG, try `https://logo.clearbit.com/{domain}` as a fallback, or use `""`. Never omit it.

---

## app.json — OAuth2 Schema

```json
{
  "name": "APPNAME",
  "display_name": "Human Readable Name",
  "logo": "https://path/to/logo.svg",
  "provider": "Provider Company Name",
  "version": "1.0.0",
  "description": "One paragraph describing what this API does.",
  "security_schemes": {
    "oauth2": {
      "location": "header",
      "name": "Authorization",
      "prefix": "Bearer",
      "client_id": "{{ AIPOLABS_APPNAME_CLIENT_ID }}",
      "client_secret": "{{ AIPOLABS_APPNAME_CLIENT_SECRET }}",
      "scope": "scope1 scope2 scope3",
      "authorize_url": "https://provider.com/oauth/authorize",
      "access_token_url": "https://provider.com/oauth/token",
      "refresh_token_url": "https://provider.com/oauth/token"
    }
  },
  "default_security_credentials_by_scheme": {},
  "categories": ["Category1"],
  "visibility": "public",
  "active": true
}
```

**`security_schemes.oauth2` fields:**
| Field | Description |
|---|---|
| `location` | Always `"header"` for OAuth2 |
| `name` | Always `"Authorization"` |
  | `prefix` | Always `"Bearer"` |
| `client_id` | Placeholder: `"{{ AIPOLABS_APPNAME_CLIENT_ID }}"` |
| `client_secret` | Placeholder: `"{{ AIPOLABS_APPNAME_CLIENT_SECRET }}"` |
| `scope` | Space-separated scopes string. Research minimum required scopes for your endpoints. |
| `authorize_url` | OAuth2 authorization endpoint URL |
| `access_token_url` | Token exchange endpoint URL |
| `refresh_token_url` | Token refresh URL (usually same as access_token_url) |

---

## function.json — OAuth2 Schema

OAuth token injection is handled by the framework. Functions do NOT include auth headers manually.

For standard REST endpoints:
```json
[
  {
    "name": "APPNAME__FUNCTION_NAME",
    "description": "What this endpoint does.",
    "tags": ["tag1", "tag2"],
    "visibility": "public",
    "active": true,
    "protocol": "rest",
    "protocol_data": {
      "method": "GET",
      "path": "/resource/path",
      "server_url": "https://api.provider.com/v1"
    },
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "object",
          "description": "Query parameters",
          "properties": {
            "maxResults": {
              "type": "integer",
              "description": "Maximum number of results to return.",
              "default": 100,
              "maximum": 500
            },
            "q": {
              "type": "string",
              "description": "Filter query string."
            }
          },
          "required": [],
          "visible": ["maxResults", "q"],
          "additionalProperties": false
        }
      },
      "required": [],
      "visible": ["query"],
      "additionalProperties": false
    }
  }
]
```

**For "connector" protocol functions** (special send/action flows, like sending an email):
```json
{
  "name": "APPNAME__SEND_MESSAGE",
  "description": "Sends a message on behalf of the user.",
  "tags": ["messaging"],
  "visibility": "public",
  "active": true,
  "protocol": "connector",
  "protocol_data": {},
  "parameters": {
    "type": "object",
    "properties": {
      "recipient": { "type": "string", "description": "Recipient address.", "format": "email" },
      "subject": { "type": "string", "description": "Message subject." },
      "body": { "type": "string", "description": "Message body content." }
    },
    "required": ["recipient", "body"],
    "visible": ["recipient", "subject", "body"],
    "additionalProperties": false
  }
}
```

Use `"protocol": "connector"` when:
- The action involves sending/creating on behalf of the user
- The endpoint requires special SDK-level handling beyond a plain HTTP call
- Examples: Gmail send, Slack post message, calendar event creation

Use `"protocol": "rest"` for all standard read/list/search/update endpoints.

---

## Real Example: Gmail

```json
// app.json
{
  "name": "GMAIL",
  "display_name": "Gmail",
  "provider": "Google",
  "version": "1.0.0",
  "description": "The Gmail API enables sending, reading, and managing emails.",
  "security_schemes": {
    "oauth2": {
      "location": "header",
      "name": "Authorization",
      "prefix": "Bearer",
      "client_id": "{{ AIPOLABS_GMAIL_CLIENT_ID }}",
      "client_secret": "{{ AIPOLABS_GMAIL_CLIENT_SECRET }}",
      "scope": "https://www.googleapis.com/auth/gmail.send https://www.googleapis.com/auth/gmail.readonly https://www.googleapis.com/auth/gmail.modify",
      "authorize_url": "https://accounts.google.com/o/oauth2/v2/auth",
      "access_token_url": "https://oauth2.googleapis.com/token",
      "refresh_token_url": "https://oauth2.googleapis.com/token"
    }
  },
  "default_security_credentials_by_scheme": {},
  "categories": ["Communication"],
  "visibility": "public",
  "active": true
}

// function.json (excerpt)
[
  {
    "name": "GMAIL__SEND_EMAIL",
    "description": "Sends an email on behalf of the authenticated user.",
    "tags": ["email"],
    "visibility": "public",
    "active": true,
    "protocol": "connector",
    "protocol_data": {},
    "parameters": {
      "type": "object",
      "properties": {
        "sender": { "type": "string", "description": "Sender email address. Use 'me' for the authenticated user.", "default": "me" },
        "recipient": { "type": "string", "description": "Recipient email address.", "format": "email" },
        "subject": { "type": "string", "description": "Email subject line." },
        "body": { "type": "string", "description": "Plain text email body." }
      },
      "required": ["sender", "recipient", "body"],
      "visible": ["sender", "recipient", "subject", "body"],
      "additionalProperties": false
    }
  },
  {
    "name": "GMAIL__MESSAGES_LIST",
    "description": "Lists messages in the user's mailbox.",
    "tags": ["email", "messages"],
    "visibility": "public",
    "active": true,
    "protocol": "rest",
    "protocol_data": {
      "method": "GET",
      "path": "/users/me/messages",
      "server_url": "https://gmail.googleapis.com/gmail/v1"
    },
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "object",
          "properties": {
            "maxResults": { "type": "integer", "description": "Max messages to return.", "default": 100, "maximum": 500 },
            "q": { "type": "string", "description": "Gmail search query (e.g. 'from:user@example.com')." }
          },
          "required": [],
          "visible": ["maxResults", "q"],
          "additionalProperties": false
        }
      },
      "required": [],
      "visible": ["query"],
      "additionalProperties": false
    }
  }
]
```
