# Productboard API Postman

Public Postman artifacts for working with the Productboard API v2.

## What is in this repository

- `Productboard v2.postman_collection.json`: a ready-to-import Postman collection for Productboard API v2.
- `Productboard.postman_environment.json`: a Postman environment with common variables and placeholders.

This repository intentionally publishes the v2 collection only. The legacy v1 collection is deprecated and is not included.

## What this collection is for

Use this package when you want to:
- explore Productboard API v2 routes in Postman
- start from working request shapes instead of building every request from scratch
- fill in your own IDs and token values while keeping secrets out of the repository

The request descriptions in the collection call out important route-shape details and known caveats where relevant.

## Prerequisites

You need:
- a Productboard account with API access
- a valid Productboard access token
- Postman

## Import into Postman

1. Import `Productboard v2.postman_collection.json`.
2. Import `Productboard.postman_environment.json`.
3. Set `productboard_token` in the imported environment.
4. Fill in any entity or relationship IDs required by the requests you want to run.

## Environment variables

The environment file includes placeholders for common values such as:
- `productboard_token`
- `entity_id`
- `entity_type`
- `product_id`
- `component_id`
- `feature_id`
- `team_id`
- `member_id`
- `note_id`

Leave unused variables empty until you need them.

## Notes on usage

- The collection targets `https://api.productboard.com/v2`.
- Some requests depend on IDs from your own Productboard workspace.
- Route behavior and available fields can vary by Productboard workspace configuration.
- Always review write requests before sending them to a live workspace.

## Security

This repository does not include secrets, credentials, or workspace-specific IDs.

Before committing changes, verify that you have not added:
- access tokens
- personal data
- internal workspace identifiers
- tenant-specific payload samples

## Updating the collection

If you update the collection:
- keep the repository public-safe
- avoid adding internal-only notes or references
- avoid embedding secrets or real identifiers
- preserve the general-purpose environment placeholders

## License

See [LICENSE](LICENSE).
