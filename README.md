# Productboard API Postman Collections

Public Postman collections for the Productboard API, based on live tenant validation and request-shape verification.

This repo is intentionally minimal. It contains only:
- `Productboard.postman_environment.json`
- `Productboard v1.postman_collection.json`
- `Productboard v2.postman_collection.json`
- `README.md`

## What This Repo Is For

Use this repo when you want ready-to-import Postman collections for Productboard without the larger private proof harness around them.

The collections were assembled from:
- official Productboard API documentation
- live Productboard API probing
- tenant-validated request shapes where the docs or earlier experiments were ambiguous

## Important Caveats

These collections are evidence-informed, but Productboard behavior is still context-sensitive.

The most important things to know:
- v2 entity work is configuration-first
  - fetch `GET /v2/entities/configurations` and `GET /v2/entities/configurations/{entityType}` before assuming field names or payload shapes
- OAuth and Productboard API token are not equivalent
  - some Productboard route families still work with API token but not OAuth bearer tokens
- the Productboard API has some real request-shape quirks
  - fresh `/v2/entities` list calls require `type[]`
  - fresh v2 field selectors require `fields[]`
  - `POST /v2/entities/search` expects `{"data":{"type":"..."}}`
  - v1 feature-link writes use `POST /objectives/{id}/links/features/{feature_id}` and `POST /initiatives/{id}/links/features/{feature_id}`

## Import Into Postman

1. Import both collection files.
2. Import `Productboard.postman_environment.json`.
3. Set `productboard_token`.
4. Fill in any ID variables you need for detail, relationship, update, or delete requests.
5. For any v2 entity export or write flow, fetch entity configuration first.

## What Is Included

### v1 collection

The v1 collection covers the main legacy Productboard surfaces, including:
- notes
- companies
- users
- products
- components
- features
- objectives
- key results
- initiatives
- releases
- release groups
- integrations and webhooks
- selected custom-field and feature-link write shapes

### v2 collection

The v2 collection covers:
- entity list/get/search/configuration routes
- entity relationships
- notes and note relationships
- Jira integrations
- members and teams routes

## Safety Notes

- The environment file ships with an empty `productboard_token`.
- Do not test mutating requests against production data unless you understand the impact.
- Prefer disposable sandbox records for write validation.
- Productboard route availability can differ by auth mode, tenant configuration, and feature enablement.

## Not Included

This public repo does not include the private proof harness, local evidence JSON, or helper scripts that were used to validate the collections.

## Source Docs

- [Productboard API v1 Introduction](https://developer.productboard.com/reference/introduction)
- [Productboard API v2 Introduction](https://developer.productboard.com/v2.0.0/reference/introduction)
- [Productboard Authentication](https://developer.productboard.com/reference/authentication)
- [Productboard Versioning and compatibility](https://developer.productboard.com/reference/versioning-and-compatibility)
- [Productboard v2 List Entities](https://developer.productboard.com/v2.0.0/reference/listentities)
- [Productboard v2 Search Entities](https://developer.productboard.com/v2.0.0/reference/searchentities)
