# Productboard API Postman Package

This folder is a practical, evidence-backed Productboard API package.

It is meant to answer four questions quickly:
- Which Productboard routes actually work on the tested tenant?
- Which routes only work with a Productboard API token and not OAuth?
- Which request shapes are correct when the docs or earlier experiments were ambiguous?
- How do you rerun the proof and inspect the evidence yourself?

The package is based on:
- official Productboard API docs
- live API probes performed on 2026-03-05, 2026-03-06, and targeted contract repros on 2026-03-07
- disposable sandbox write tests using a dedicated `pbproof` OAuth profile
- configuration-first stress runs generated from live v2 entity configuration snapshots

## At A Glance

Current evidence snapshot:
- latest broad proof artifact: `evidence/latest_proof.json` generated at `2026-03-06T22:47:40Z`
- latest broad proof mode: `smoke`
- v2 entity types discovered in config snapshot: `11`
- result entries recorded: `406`
- cleanup actions recorded: `16`
- latest targeted OAuth v1 contract repros: `2026-03-07`

What is currently true on the tested tenant:
- core v2 entity and note reads work with OAuth
- core v2 entity and note writes work with OAuth when the token has the right write/delete scopes
- some legacy v1 routes still work with OAuth
- several important route families still require a Productboard API token rather than OAuth
- the package prefers live proof over doc assumptions; when docs and live behavior differ, the live behavior wins

## What Is In This Folder

Main files:
- `Productboard.postman_environment.json`
- `Productboard v1.postman_collection.json`
- `Productboard v2.postman_collection.json`
- `evidence/endpoint_inventory.json`
- `evidence/latest_proof.json`
- `scripts/prove_productboard_api.py`
- `scripts/build_endpoint_inventory.py`

Use the collections when you want to call routes manually in Postman.
Use `latest_proof.json` when you want machine-readable evidence of what was actually exercised.
Use `endpoint_inventory.json` when you want a route-by-route inventory with notes.

## Import

1. Import both collection files into Postman.
2. Import `Productboard.postman_environment.json`.
3. Set `productboard_token`.
4. Fill in the id variables you need for detail, relationship, update, or delete requests.
5. Before doing any v2 entity export or write work, fetch entity configuration first.

## The Most Important Rules

### 1. v2 Is Configuration-First

For v2 entity work, do not guess field names or payload shapes.

Always start with:
- `GET /v2/entities/configurations`
- `GET /v2/entities/configurations/{entityType}`

Why this matters:
- field sets vary by workspace configuration
- a field that exists in config is not always safely selectable on every real entity
- nested config paths are not automatically valid `fields[]` selectors

Stable response-level fields are:
- `id`
- entity `type`
- `createdAt`
- `updatedAt`

### 2. OAuth And API Token Are Not Equivalent

Do not assume that an OAuth token can use every route just because the token appears to have relevant scopes.

Current practical split:
- OAuth is good for core v2 entities and notes, plus some v1 routes
- Productboard API token is still needed for `members`, `teams`, `analytics/member-activities`, and several legacy v1 surfaces

If a route is auth-limited, this package documents that explicitly in notes rather than pretending the route is universally available.

### 3. Live Proof Beats Guesswork

This package is not a copy of the official docs.

A route is included because it was exercised successfully against the live API, with auth caveats and payload-shape caveats recorded when needed.

## Fastest Way To Reproduce Evidence

Run from this folder or by absolute path.

Fast broad proof:

```bash
python3 "scripts/prove_productboard_api.py" \
  --profile pbproof \
  --suite all \
  --mode smoke \
  --include-writes \
  --include-api-token-surface \
  --api-token-op-ref "op://Private/Productboard API Access Token/credential"
```

Deep stress proof:

```bash
python3 "scripts/prove_productboard_api.py" --profile pbproof --mode deep
```

Target only the API-token-only surfaces:

```bash
python3 "scripts/prove_productboard_api.py" \
  --profile pbproof \
  --suite api-token-support \
  --mode smoke \
  --api-token-op-ref "op://Private/Productboard API Access Token/credential"
```

Rebuild the route inventory:

```bash
python3 "scripts/build_endpoint_inventory.py"
```

What gets written:
- proof harness: `evidence/latest_proof.json`
- endpoint inventory: `evidence/endpoint_inventory.json`

Useful proof flags:
- `--mode smoke|deep`
- `--seed <int>`
- `--sample-per-type <n>`
- `--list-pages <n>`
- `--types <csv>`
- `--include-writes`
- `--include-api-token-surface`

## How To Read The Evidence

The stress harness is read-heavy by default.

For each discovered v2 entity type, it probes:
- list capability
- search capability
- detail retrieval
- relationship retrieval
- config-derived field selection in small batches

Evidence includes:
- `config_snapshot`
- `type_capabilities`
- `sampling_plan`
- `field_matrix_results`
- `per_type_summary`
- `flaky_or_context_sensitive_findings`

The purpose is not just “does the route exist?”
It is also “does this field-selection pattern keep working across real sampled entities?”

## What Definitely Works

### v1

These were live-proved on the tested tenant.

General facts:
- v1 requests require `X-Version: 1`
- `GET /subfeatures` returned `404`, so subfeatures are represented through `/features` in v1
- `GET /hierarchy-entities/custom-fields?type=feature` returned `400`; that `type` parameter expects a custom-field kind, not an entity type

Core reads that returned `200`:
- `/notes`, `/notes/{id}`
- `/companies`, `/users`
- `/products`, `/components`, `/features`
- `/objectives`, `/initiatives`
- `/releases`, `/release-groups`
- `/jira-integrations`, `/plugin-integrations`, `/webhooks`

Core writes that were live-proved:
- `POST /components`, `PATCH /components/{id}`
- `POST /features`, `PATCH /features/{id}`, `DELETE /features/{id}`
- `POST /objectives`, `PATCH /objectives/{id}`, `DELETE /objectives/{id}`
- `POST /initiatives`, `PATCH /initiatives/{id}`, `DELETE /initiatives/{id}`
- `POST /objectives/{id}/links/features/{feature_id}`
- `POST /initiatives/{id}/links/features/{feature_id}`
- `PUT /hierarchy-entities/custom-fields-values/value`

Important shape notes:
- feature and subfeature writes needed explicit `type`, empty-string `description`, correct parent shape, and `status.id`
- key-result create requires `data.parent.id` pointing to an objective
- custom-field value write requires query params `customField.id` and `hierarchyEntity.id`, plus a typed `data` body such as `{type:"dropdown", value:{id}}`
- the earlier body-based feature-link shape was wrong; the valid contract puts `feature_id` in the path

### v2 Read Contracts

These were live-proved on the tested tenant.

Most important quirks:
- fresh `/v2/entities` list requests require `type[]`, not scalar `type`
- fresh field selectors require `fields[]`, not scalar `fields`
- Productboard may emit malformed `/v2/entities` `links.next` URLs with scalar `type`; clients should normalize those before following them
- `POST /v2/entities/search` requires `{"data":{"type":"..."}}`, not root-level `{"type":"..."}`

Confirmed working patterns:
- `GET /v2/entities/configurations`
- `GET /v2/entities/configurations/{entityType}`
- `GET /v2/entities?type[]=feature&fields[]=id`
- `POST /v2/entities/search` with `{"data":{"type":"feature"}}`
- `POST /v2/entities/search` with `{"data":{"type":"subfeature"}}`
- `GET /v2/entities/{id}/relationships`
- `GET /v2/notes`
- `POST /v2/notes/search`
- `GET /v2/notes/{id}`
- `GET /v2/notes/{id}/relationships`
- `GET /v2/notes/configurations`
- `GET /v2/notes/configurations/simple`
- `GET /v2/jira-integrations`
- `GET /v2/jira-integrations/{id}`
- `GET /v2/jira-integrations/{id}/connections`
- `GET /v2/jira-integrations/{id}/connections/{connectionId}`

Entity-type notes:
- `company` and `user` are valid v2 entity types in list/get/relationships mode
- `company` and `user` are not accepted in `POST /v2/entities/search` on the tested tenant
- `company` and `user` should be accessed through `/v2/entities`; guessed standalone `/v2/companies` and `/v2/users` routes remain excluded

### v2 Write Contracts

Live-proved with disposable sandbox entities using prefix `zz-codex-api-proof-<timestamp>`:
- `POST /v2/entities`
- `PATCH /v2/entities/{id}`
- `DELETE /v2/entities/{id}`
- `POST /v2/entities/{id}/relationships`
- `PUT /v2/entities/{id}/relationships/parent`
- `DELETE /v2/entities/{id}/relationships/{type}/{targetId}`
- `POST /v2/notes`
- `PATCH /v2/notes/{id}`
- `DELETE /v2/notes/{id}`
- `POST /v2/notes/{id}/relationships`
- `DELETE /v2/notes/{id}/relationships/link/{targetId}`

Important shape notes:
- on the tested tenant, `component`, `feature`, and `subfeature` creation required an inline parent relationship in the create payload
- `DELETE /v2/entities/{id}` required `entities:delete`; `entities:write` alone was not enough
- `DELETE /v2/notes/{id}` required `notes:delete`; `notes:write` alone was not enough
- `POST /v2/notes/{id}/relationships` worked with:

```json
{
  "data": {
    "type": "link",
    "target": {
      "id": "<entity-id>",
      "type": "link"
    }
  }
}
```

Earlier candidate payloads such as `targetType=entity` and `target.type=feature` were rejected and are intentionally not documented as valid.

## What Still Needs API Token Instead Of OAuth

These are valid live-proved surfaces, but OAuth still failed while Productboard API token worked.

v2 families:
- `GET /v2/members`, `GET /v2/members/{id}`, `GET /v2/members/{id}/relationships`
- `GET /v2/teams`, `GET /v2/teams/{id}`, `GET /v2/teams/{id}/relationships`
- `POST /v2/teams`, `PATCH /v2/teams/{id}`, `DELETE /v2/teams/{id}`
- `GET /v2/analytics/member-activities`

Legacy v1 families and routes:
- `POST /companies`, `PATCH /companies/{id}`, `DELETE /companies/{id}`
- `GET /companies/custom-fields`
- `GET /companies/{companyId}/custom-fields/{companyCustomFieldId}/value`
- `PUT /companies/{companyId}/custom-fields/{companyCustomFieldId}/value`
- `POST /users`, `PATCH /users/{id}`, `DELETE /users/{id}`
- `GET /key-results`, `GET /key-results/{id}`
- `POST /key-results`, `PATCH /key-results/{id}`, `DELETE /key-results/{id}`
- `GET /feedback-form-configurations`
- `GET /feedback-form-configurations/{id}`

Other API-token-proved routes:
- `PUT /v2/notes/{id}/relationships/customer`
- `DELETE /v2/notes/{id}/relationships/customer/{customerId}`

Operational rule:
- if you see a `403` on one of these families under OAuth, do not immediately assume the route is unsupported; check whether it is one of the known auth-limited families first

## Known Instability

The bounded broad proof recorded one flaky/context-sensitive finding:
- one `subfeature` detail field-selection request using `fields[]=name,archived,description` returned a `599` timeout in one sample while the same field batch succeeded elsewhere

This is tracked as instability, not as a proven contract failure.

## Coverage Classification

- `live-proved`: the request/response shape succeeded against the live API in at least one auth mode
- `auth-limited` in prose: the route is live-proved, but not with every auth mode
- this package intentionally excludes guessed families and negative placeholder routes from the importable collections

## Safety Notes

- the package does not request `members_pii:read`
- mutating requests are for disposable sandbox records only
- prefer a deterministic prefix such as `{{probe_prefix}}` when testing writes
- treat a write proof as incomplete if cleanup does not return `200`, `204`, or a safe `404` for already-removed records

## Sources

- [Productboard API v1 Introduction](https://developer.productboard.com/reference/introduction)
- [Productboard API v2 Introduction](https://developer.productboard.com/v2.0.0/reference/introduction)
- [Productboard Authentication](https://developer.productboard.com/reference/authentication)
- [Productboard Versioning and compatibility](https://developer.productboard.com/reference/versioning-and-compatibility)
- [Productboard v2 List Entities](https://developer.productboard.com/v2.0.0/reference/listentities)
- [Productboard v2 Search Entities](https://developer.productboard.com/v2.0.0/reference/searchentities)
- [Productboard v2 Get Entity Configuration](https://developer.productboard.com/v2.0.0/reference/getentityconfiguration.md)
- [Productboard v2 Features Export Recipe](https://developer.productboard.com/v2.0.0/recipes/features-export)
- [Productboard v2 List Members](https://developer.productboard.com/v2.0.0/reference/listmembers.md)
- [Productboard v2 Get Member](https://developer.productboard.com/v2.0.0/reference/getmember.md)
- [Productboard v2 Get Member Relationships](https://developer.productboard.com/v2.0.0/reference/getmemberrelationships.md)
- [Productboard v2 Get Members Activity](https://developer.productboard.com/v2.0.0/reference/getmembersactivity)
- [Productboard v2 List Teams](https://developer.productboard.com/v2.0.0/reference/listteams)
- [Productboard v2 Set Note Relationship](https://developer.productboard.com/v2.0.0/reference/setnoterelationship.md)

## Requested But Still Blocked

These routes were examined during the March 7, 2026 expansion pass but were not added because they are not yet safely live-proved for this package:

- `POST /companies/custom-fields`
  - the documented create shapes did not succeed on the tested tenant; `text` returned `400 type.inclusion` and `number`/`string` variants returned `500`
- `POST /plugin-integrations`, `PATCH /plugin-integrations/{id}`, `GET /plugin-integrations/{id}/connections`, `GET /plugin-integrations/{id}/connections/{featureId}`, `PUT /plugin-integrations/{id}`, `PUT /plugin-integrations/{id}/connections/{featureId}`
  - create is blocked by Productboard callback validation requirements; a real public callback endpoint that echoes the validation token as plain text is needed before the rest of the family can be proved safely
- `POST /feedback-forms`
  - submission returned `201`, but cleanup could not be proved through the obvious note read/delete contracts for the returned id, so it currently fails the package's sandbox-write cleanup bar
