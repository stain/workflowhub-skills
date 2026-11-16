# Repository-specific routes

Use a host's metadata-only route first when one is listed here; otherwise use
these when the generic steps (signposting, then content negotiation) fail or
are known not to work. Tested 2026-10-14; WorkflowHub notes updated
2026-11-16.

## WorkflowHub (workflowhub.eu)

| Route | Result |
|---|---|
| `HEAD /workflows/<id>/ro_crate?version=<n>` | `Content-Length` of the crate ZIP (about 66 KB for workflow 248, version 7). **Costs the server as much as a GET:** WorkflowHub (Rails) runs the same action for HEAD, which builds the ZIP, for a git-backed workflow by checking out the repository. Don't use HEAD to check the size; don't request the ZIP at all unless the files are needed. |
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
- The metadata route was confirmed on one workflow, and on the latest version
  of all 1,581 public workflows by the WorkflowHub Knowledge Graph (below). If
  it fails for a workflow, report that and stop. Fetch the ZIP instead only
  if the researcher agrees, never only to get the metadata.
- **Supplied or generated.** A crate uploaded or imported as a full RO-Crate
  is served as uploaded, without later edits on WorkflowHub. Otherwise
  WorkflowHub generates it, and its root `name` starts "Research Object Crate
  for". 45% of crates were supplied (2026-10-17), mostly by the Galaxy
  Training Network, IWC, nf-core and COMPSs. Supplied crates use RO-Crate 1.1
  to 1.3 and may declare no profile.
- A WorkflowHub crate is a Workflow RO-Crate. The `mainEntity` is the
  workflow file, whose `programmingLanguage` names the workflow system.
- **Bulk reading.** For questions across many workflows, use the WorkflowHub
  Knowledge Graph (<https://doi.org/10.5281/zenodo.23691282>), a dated RDF
  snapshot of the latest version of every public crate, instead of fetching
  crates one by one.
- The w3id permalink `https://w3id.org/workflowhub/workflow-ro-crate` returned
  `application/ld+json` for the RO-Crate profile; it points to the profile
  description, not a particular workflow.

## Other repositories

Not tested. Use the generic steps, and record what worked here when it has
been confirmed.
