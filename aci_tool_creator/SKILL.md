---
name: api-json-builder
description: >
  Build app.json and function.json integration files for any API or service. Use this skill
  whenever the user wants to create, generate, or produce app.json and function.json files
  for an API integration — even if they just say "make me the JSON files for Stripe" or
  "I need integration files for GitHub". Handles all auth types: No Auth, API Key, and OAuth2.
  Always use this skill when the task involves researching an API and outputting structured
  app.json + function.json pairs. Also triggers when editing or fixing existing app.json /
  function.json files.
---

# API JSON Builder

Creates `{appname}_app.json` and `{appname}_function.json` files for any API. The agent must
research the target API, then produce correctly structured output files following the schemas
and rules below.

---

## Step 1 — Gather Requirements

Collect from the user (or infer from context):

| Field | Notes |
|---|---|
| **API / Service name** | e.g. "Stripe", "GitHub", "OpenWeatherMap" |
| **Auth type** | No Auth / API Key / OAuth2 — if unknown, research it |
| **Which endpoints** | Specific endpoints, or "the most useful ones" |
| **Max functions** | Default: include all clearly useful public endpoints (cap at ~10) |

If any field is missing and can't be inferred, ask before researching.

---

## Step 2 — Research the API

Use web search or browse official docs to find:

**Always needed:**
- Official API base URL (e.g. `https://api.stripe.com/v1`)
- Auth method: No Auth / API Key (header name, prefix) / OAuth2 (authorize URL, token URL, scopes)
- Provider / company name
- Short description of what the API does
- Available categories (e.g. Payments, Communication, Search)

**Logo (always required):**
- Search for an official SVG or PNG logo URL (try the provider's GitHub, their press/brand kit page, or `https://logo.clearbit.com/{domain}` as a fallback)
- If nothing is found, use `""` (empty string) — never omit the `logo` field

**Per endpoint:**
- HTTP method (GET, POST, PUT, DELETE, PATCH)
- Path (e.g. `/charges`)
- Required and optional query/body/header parameters
- Parameter types, descriptions, enums, defaults, min/max

**Key research sources to check:**
1. Official docs site (search `{api name} API documentation`)
2. OpenAPI / Swagger spec if published (search `{api name} openapi.yaml` or `swagger.json`)
3. Postman public collections
4. RapidAPI listing

If auth type is ambiguous after research → default to **API Key**.

---

## Step 3 — Build the Files

Produce two flat files named:
- `{APPNAME}_app.json` — lowercase app name, e.g. `stripe_app.json`
- `{APPNAME}_function.json` — e.g. `stripe_function.json`

Read the auth-specific schema from the reference files before writing:

| Auth Type | Reference File |
|---|---|
| No Auth | `references/no-auth.md` |
| API Key | `references/api-key.md` |
| OAuth2 | `references/oauth.md` |

### Universal Rules (apply to ALL auth types)

1. **Function name prefix**: Every function name MUST start with `{APP_NAME}__` (double underscore). The APP_NAME is the `name` field from app.json, uppercased. Example: if `app.json` name is `STRIPE`, functions must be `STRIPE__CREATE_CHARGE`, `STRIPE__LIST_CUSTOMERS`, etc.

2. **Naming convention**: `name` fields use UPPER_SNAKE_CASE for both app and functions.

3. **`visible` arrays**: List only the parameter keys a human/agent should care about. Internal/auth params go in `required` but NOT `visible`.

4. **`additionalProperties`**: Always `false` on every object.

5. **Parameter placement**: Use `query` for GET params, `body` for POST/PUT/PATCH, `path` for URL variables, `header` for header params.

6. **`active` and `visibility`**: Always `"active": true` and `"visibility": "public"` unless told otherwise.

7. **Descriptions**: Write clear, concise descriptions — 1 sentence per parameter minimum.

8. **`tags`**: Add 2–4 relevant lowercase tags per function (e.g. `["payments", "customers"]`).

---

## Step 4 — Validate Before Saving

Run through this checklist mentally before writing the files:

- [ ] `logo` field is present in app.json (URL, clearbit fallback, or `""` — never missing)
- [ ] Every function name starts with `{APP_NAME}__`
- [ ] Auth credentials are in `app.json`'s `security_schemes`, NOT hardcoded in functions
- [ ] `visible` arrays exclude internal/auth params
- [ ] All objects have `"additionalProperties": false`
- [ ] `required` arrays only list truly required params (not optional ones)
- [ ] `protocol` is `"rest"` (or `"connector"` for special flows like Gmail send)
- [ ] `server_url` has no trailing slash
- [ ] Paths start with `/`
- [ ] No placeholder values like `YOUR_API_KEY` or `CLIENT_ID_HERE` — use `{{ VAR_NAME }}` format for secrets

---

## Step 5 — Save and Present

Save files to the output directory as flat files:
- `{appname}_app.json`
- `{appname}_function.json`

Then briefly summarize:
- App name, auth type used
- Number of functions generated
- Any endpoints skipped and why (e.g. "skipped admin-only endpoints")
- Any assumptions made about auth scopes or parameter types
