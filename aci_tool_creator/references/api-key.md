# API Key — Schema & Example

Use when the API authenticates via a key passed in a header or query string.

---

## Research Checklist for API Key APIs

Find these before writing the files:
- [ ] Header name (e.g. `X-Subscription-Token`, `X-API-Key`, `Authorization`)
- [ ] Prefix if any (e.g. `Bearer`, `Token`, or none)
- [ ] Whether it's a header or query param (header is most common)

**Logo:** Always include the `logo` field. Search for an official SVG/PNG, try `https://logo.clearbit.com/{domain}` as a fallback, or use `""`. Never omit it.

---

## app.json — API Key Schema

```json
{
  "name": "APPNAME",
  "display_name": "Human Readable Name",
  "logo": "https://path/to/logo.svg",
  "provider": "Provider Company Name",
  "version": "1.0.0",
  "description": "One paragraph describing what this API does.",
  "security_schemes": {
    "api_key": {
      "location": "header",
      "name": "X-API-Key",
      "prefix": null
    }
  },
  "default_security_credentials_by_scheme": {},
  "categories": ["Category1"],
  "visibility": "public",
  "active": true
}
```

**`security_schemes.api_key` fields:**
| Field | Description |
|---|---|
| `location` | `"header"` or `"query"` — almost always `"header"` |
| `name` | The exact header/param name (e.g. `"X-API-Key"`, `"Authorization"`) |
| `prefix` | String prefix before the key (e.g. `"Bearer"`, `"Token"`) or `null` if none |

---

## function.json — API Key Schema

The API key is handled by the framework via `security_schemes` — do NOT add it as a manual parameter. Functions look identical to No Auth functions.

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
      "path": "/endpoint",
      "server_url": "https://api.example.com/v1"
    },
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "object",
          "description": "Query parameters",
          "properties": {
            "q": {
              "type": "string",
              "description": "The search query."
            },
            "count": {
              "type": "integer",
              "description": "Number of results to return.",
              "default": 10,
              "maximum": 50,
              "minimum": 1
            }
          },
          "required": ["q"],
          "visible": ["q", "count"],
          "additionalProperties": false
        },
        "header": {
          "type": "object",
          "properties": {
            "Accept": {
              "type": "string",
              "default": "application/json"
            }
          },
          "required": [],
          "visible": [],
          "additionalProperties": false
        }
      },
      "required": ["query", "header"],
      "visible": ["query"],
      "additionalProperties": false
    }
  }
]
```

**POST request body example:**
```json
"parameters": {
  "type": "object",
  "properties": {
    "body": {
      "type": "object",
      "description": "Request body",
      "properties": {
        "name": { "type": "string", "description": "Name of the resource." },
        "amount": { "type": "number", "description": "Amount in cents." }
      },
      "required": ["name", "amount"],
      "visible": ["name", "amount"],
      "additionalProperties": false
    }
  },
  "required": ["body"],
  "visible": ["body"],
  "additionalProperties": false
}
```

**Key rules:**
- `header` object with `Accept: application/json` is standard boilerplate — include it
- `header` goes in `required` if present, but NOT in `visible` (it's internal)
- Never put the actual API key value anywhere in function.json

---

## Real Example: Brave Search

```json
// app.json
{
  "name": "BRAVE_SEARCH",
  "display_name": "Brave Search",
  "provider": "Brave Software, Inc.",
  "version": "1.0.0",
  "description": "Brave Search API supports web, image, video, and news search.",
  "security_schemes": {
    "api_key": {
      "location": "header",
      "name": "X-Subscription-Token",
      "prefix": null
    }
  },
  "default_security_credentials_by_scheme": {},
  "categories": ["Search & Scraping"],
  "visibility": "public",
  "active": true
}

// function.json (excerpt)
[
  {
    "name": "BRAVE_SEARCH__WEB_SEARCH",
    "description": "Search the web using Brave Search.",
    "tags": ["search"],
    "visibility": "public",
    "active": true,
    "protocol": "rest",
    "protocol_data": {
      "method": "GET",
      "path": "/web/search",
      "server_url": "https://api.search.brave.com/res/v1"
    },
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "object",
          "properties": {
            "q": { "type": "string", "description": "Search query (max 400 chars).", "maxLength": 400 },
            "count": { "type": "integer", "description": "Number of results (max 20).", "default": 20, "maximum": 20, "minimum": 1 }
          },
          "required": ["q"],
          "visible": ["q", "count"],
          "additionalProperties": false
        },
        "header": {
          "type": "object",
          "properties": { "Accept": { "type": "string", "default": "application/json" } },
          "required": [],
          "visible": [],
          "additionalProperties": false
        }
      },
      "required": ["query", "header"],
      "visible": ["query"],
      "additionalProperties": false
    }
  }
]
```
