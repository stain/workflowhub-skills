# Repository-specific routes

Use these when the generic steps (signposting, then content negotiation) fail
or are known not to work. Tested 2026-10-14.

## WorkflowHub (workflowhub.eu)

| Route | Result |
|---|---|
| `HEAD /workflows/<id>/ro_crate?version=<n>` | `Content-Length` of the crate ZIP (about 66 KB for workflow 248, version 7). The size is available. |
| `GET /workflows/<id>/ro_crate?version=<n>` | The crate as a ZIP, signposted as `rel="item"` with the RO-Crate profile. `Range` requests are ignored, so the whole file is always sent. |
| `GET /workflows/<id>/ro_crate_metadata?version=<n>` | Only `ro-crate-metadata.json`, as `application/json`. Byte-identical to the file inside the ZIP for workflow 248, version 7. Not signposted. **Cheapest route; prefer it.** |
| `GET /workflows/<id>?version=<n>` with `Accept: application/ld+json;profile="https://w3id.org/ro/crate"` | **Not an RO-Crate.** Returns a Bioschemas record, which is the same as the signposted `describedby` link of that type. Don't treat it as the crate. |

Notes:

- Throttle as in `SKILL.md`: at least 2 seconds between crate requests, one
  at a time. This applies to the metadata route too.
- The metadata-only response is in effect a detached crate, but not a valid
  one: the data entities in the five crates tested (workflows 42, 46, 123, 1441
  and 1625) have relative `@id`s (file names such as `main-workflow.ga`), which
  only resolve inside the ZIP. ROHub crates (three tested) use absolute URIs.
  Not checked with a validator.
- The metadata route was confirmed on one workflow. If it returns 404 or HTML
  for another, fall back to the ZIP route, within the size limit.
- A WorkflowHub crate is a Workflow RO-Crate. The `mainEntity` is the
  workflow file, whose `programmingLanguage` names the workflow system.
- The w3id permalink `https://w3id.org/workflowhub/workflow-ro-crate` returned
  `application/ld+json` for the RO-Crate profile; it points to the profile
  description, not a particular workflow.

## Other repositories

Not tested. Use the generic steps, and record what worked here when it has
been confirmed.
