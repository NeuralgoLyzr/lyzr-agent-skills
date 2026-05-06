# No Auth — Schema & Example

Use this when the API requires no authentication at all (public APIs).

---

## app.json — No Auth Schema

```json
{
  "name": "APPNAME",
  "display_name": "Human Readable Name",
  "logo": "https://path/to/logo.svg",
  "provider": "Provider Company Name",
  "version": "1.0.0",
  "description": "One paragraph describing what this API does.",
  "security_schemes": {
    "no_auth": {}
  },
  "default_security_credentials_by_scheme": {},
  "categories": ["Category1", "Category2"],
  "visibility": "public",
  "active": true
}
```

**Notes:**
- `security_schemes` contains only `"no_auth": {}`
- `default_security_credentials_by_scheme` is always `{}`
- `logo`: **always include this field** — use official SVG/PNG URL if available; try `https://logo.clearbit.com/{domain}` as a fallback; if nothing works use `""`. Never omit the field.
- `categories`: pick from common values — Research, Search & Scraping, Data, Finance, Communication, Productivity, Developer Tools, Weather, News, Social

---

## function.json — No Auth Schema

```json
[
  {
    "name": "APPNAME__FUNCTION_NAME",
    "description": "What this endpoint does in one sentence.",
    "tags": ["tag1", "tag2"],
    "visibility": "public",
    "active": true,
    "protocol": "rest",
    "protocol_data": {
      "method": "GET",
      "path": "/endpoint/path",
      "server_url": "https://api.example.com"
    },
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "object",
          "description": "Query parameters",
          "properties": {
            "param1": {
              "type": "string",
              "description": "What this param does."
            },
            "param2": {
              "type": "integer",
              "description": "How many results to return.",
              "default": 10,
              "minimum": 1,
              "maximum": 100
            }
          },
          "required": ["param1"],
          "visible": ["param1", "param2"],
          "additionalProperties": false
        }
      },
      "required": ["query"],
      "visible": ["query"],
      "additionalProperties": false
    }
  }
]
```

**Notes:**
- No auth headers needed anywhere
- For GET requests: params go in `query` object
- For POST requests: params go in `body` object (see api-key.md for body example)
- `header` object is optional; only include it if the endpoint needs specific Accept headers

---

## Real Example: arXiv Search

```json
// app.json
{
  "name": "ARXIV",
  "display_name": "arXiv",
  "logo": "https://raw.githubusercontent.com/aipotheosis-labs/aipolabs-icons/refs/heads/main/apps/arxiv.svg",
  "provider": "Cornell University",
  "version": "1.0.0",
  "description": "arXiv is an open-access repository of scientific preprints in mathematics, physics, computer science, and related fields.",
  "security_schemes": { "no_auth": {} },
  "default_security_credentials_by_scheme": {},
  "categories": ["Research"],
  "visibility": "public",
  "active": true
}

// function.json (excerpt)
[
  {
    "name": "ARXIV__SEARCH_PAPERS",
    "description": "Search for papers on arXiv by query, category, and other criteria.",
    "tags": ["research", "academic", "science"],
    "visibility": "public",
    "active": true,
    "protocol": "rest",
    "protocol_data": {
      "method": "GET",
      "path": "/api/query",
      "server_url": "https://export.arxiv.org"
    },
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "object",
          "description": "Query parameters for searching papers",
          "properties": {
            "search_query": {
              "type": "string",
              "description": "Search query string. Supports Boolean operators (AND, OR, NOT) and field-specific searches."
            },
            "max_results": {
              "type": "integer",
              "description": "Maximum number of results to return.",
              "default": 10,
              "minimum": 1,
              "maximum": 1000
            },
            "sortBy": {
              "type": "string",
              "description": "Sorting criterion for results.",
              "enum": ["relevance", "lastUpdatedDate", "submittedDate"],
              "default": "relevance"
            }
          },
          "required": ["search_query"],
          "visible": ["search_query", "max_results", "sortBy"],
          "additionalProperties": false
        }
      },
      "required": ["query"],
      "visible": ["query"],
      "additionalProperties": false
    }
  }
]
```
