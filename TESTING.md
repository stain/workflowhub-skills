# Testing the WorkflowHub skills

How to test the skills, what to look for, and a log of results.

## How to test

1. Install the four skills from `skills/` in a test project (see the README).
   Use a separate folder that acts as a researcher's project, so outputs
   such as provenance files don't land among the skill files.
2. Start a fresh session for each test, unless the test says to continue.
   Leftover context hides gaps in the instructions.
3. Repeat the core tests (1, 2, 5, 9) in at least one non-Claude agent, such
   as OpenAI Codex (install under `.agents/skills`; see the README). Record
   the agent in each log entry, and whether network access for curl needed
   approval or was blocked.
4. Optional: compare with the skills turned off. In Claude Code, type
   `/skills`, highlight a skill and press Space to cycle it to "off", then
   repeat the prompt in a fresh session.

## Test prompts

1. **Find, core case.** "I have Illumina paired-end RNA-seq data from mouse
   liver, treated vs control, and want to run differential expression
   analysis in Galaxy. Is there an existing workflow on WorkflowHub I could
   use instead of building my own?"
2. **Evaluate** (continue the session from test 1). "Compare the top two
   from that shortlist. It's for a paper. Which should I use?"
3. **Find, outside the life sciences, without naming WorkflowHub.** "Before
   I write my own pipeline: is there already a workflow out there for
   species distribution modelling from GBIF occurrence records?"
4. **Find, sparse result.** "Find workflows for spatial proteomics of plant
   root tissue."
5. **Explain.** "Walk me through what WorkflowHub workflow [ID] does and
   what inputs it needs. My data is paired-end files named like
   sample1_R1.fastq.gz and sample1_R2.fastq.gz."
6. **Cite.** "I ran WorkflowHub workflow [ID] version [n] on usegalaxy.eu
   last week. Give me a citation and methods text, and record it in this
   project."
7. **Read-only boundary.** "Find a Snakemake workflow for variant calling,
   then upload my improved version of it to WorkflowHub."
8. **Find, molecular dynamics.** "Is there an existing workflow for
   molecular dynamics simulation of a protein with GROMACS? I can use
   Galaxy or Python." Check that biobb tutorial variants are grouped, and
   that unrelated "MD" hits are dropped.
9. **Inspect, Run Crate (workflow 1326).** "Inspect WorkflowHub workflow 1326:
   what is in its RO-Crate, which profiles does it conform to, and has it
   been run before?" Expected: Workflow RO-Crate 1.0 and the Process,
   Workflow and Provenance Run Crate 0.5 profiles declared on the root;
   about 29 recorded runs; a COMPSs lysozyme-in-water workflow whose main
   entity is a Python file; no download of the ZIP.
10. **Inspect, size and rate limits (workflow 1326 and others).** "Inspect
    workflows 1326, 401, 1201 and 248 and tell me which are Run Crates."
    Check that crates are read one at a time with at least 2 seconds between
    requests, via the metadata route, and that no ZIP is downloaded.
11. **Inspect, declared versus observed.** "Does workflow 248 conform to
    Workflow RO-Crate? Does it have tests?" Expected: no workflow profile
    declared, but the structure matches; a Planemo test suite on GitHub
    Actions; the agent doesn't query the test service.
12. **Inspect, no declared inputs.** "What inputs does workflow 401 declare
    in its crate?" Expected: "not declared", not "none", and a pointer to the
    workflow file.
13. **Explain a Run Crate (workflow 1326).** "Walk me through workflow 1326
    and tell me what parameters I should change." Check that values from the
    recorded run are called an example of one past run, not defaults, and
    that the ZIP is downloaded only if needed, once, within the limit.
14. **Signposting directly.** "Does https://workflowhub.eu/workflows/248?version=7
    support FAIR Signposting? Is there a DOI I can cite?" Expected: Level 1
    headers with `describedby` and `item`; no `cite-as`, no linkset; the
    DOI shown in the DataCite XML returns 404 at doi.org, so the agent says
    there's no citable DOI. Repeat with version 1 of workflow 624, which has
    a `cite-as` link.
15. **Content negotiation trap.** "Get the RO-Crate metadata for workflow 248
    version 7 using content negotiation." Expected: the agent notices the
    response is Bioschemas, not an RO-Crate, and uses the metadata route.
16. **Large crate.** "Download the full crate for workflow [ID of a crate over
    20 MiB]." Expected: the agent checks the size with HEAD and asks before
    downloading. (Pick a large crate when testing; none is recorded here.)

## What to check

- **Activation.** Did the expected skill activate?
- **Route.** Did it use the API as described in `tools.md`, rather than
  fetching web pages or answering from memory?
- **API calls.** Which requests and filter names worked or failed? Record
  them, and update the "to verify" statuses in `shared/tools.md`.
- **Accuracy.** Are IDs, versions, URLs, licenses and creators correct?
  Spot-check against the workflow pages.
- **Invention.** Watch especially for DOIs, authors, tool names and
  licenses.
- **Format.** Did it follow the template, at a sensible length?
- **Decisions.** Did it pause for the researcher at decision points?
- **Boundary.** For test 7, did it search but decline the upload?
- **Limits.** For tests 9 to 13 and 16, did it check size first, fetch one
  crate at a time with at least 2 seconds between requests, and avoid
  repeat downloads?
- **Declared versus observed.** For tests 9 to 12, does the report separate
  what the crate claims from what the agent saw?

## Results log

Add an entry for each test run, newest first.

### Template

- **Date:**
- **Test:**
- **Agent and route:** for example, Claude Code desktop, public API
- **Result:** pass / pass with issues / fail
- **What happened:**
- **Verified against WorkflowHub:**
- **Issues:**

### 2026-11-16: Supplied and generated crates, HEAD cost, Knowledge Graph

- **Agent and route:** Claude Code (sandbox), reading the SEEK source code
  (`seek4science/seek` main), the WorkflowHub Knowledge Graph on Zenodo
  (doi:10.5281/zenodo.23691282, version 2026-10-17) and a few single requests
  to workflowhub.eu (metadata route and headers only, no ZIPs).
- **Result:** pass; findings fed into `shared/tools.md`,
  `ro-crate-consume/references/sources.md` and `workflowhub-inspect/references/profiles.md`.
- **What was confirmed:**
  - `HEAD /workflows/<id>/ro_crate` runs the same controller action as GET,
    which builds the ZIP (`WorkflowsController#ro_crate` calls `ro_crate_zip`;
    Rails routes HEAD to GET actions). Source code only, not timed.
  - `/ro_crate_metadata` is served as `application/json`; Signposting on the
    workflow page advertises only DataCite XML, Bioschemas JSON-LD and the ZIP
    (workflow 1448).
  - Of the 1,581 latest-version crates in the Knowledge Graph, 713 were
    supplied (root `name` not starting "Research Object Crate for") and 868
    generated. Suppliers: Galaxy Training Network 393 (root `@id` is the GTN
    page, e.g. workflow 1448), IWC 118, nf-core 70, COMPSs 68, oxo-flow 23,
    Autosubmit 10. GTN and IWC crates declare no profile; 192 supplied crates
    have a `TestSuite`, 77 a `CreateAction`; 456 still use `WorkflowSketch`.
  - All generated crates declare RO-Crate 1.3 and Workflow RO-Crate 1.0 on
    the descriptor.
  - Workflow 1072 (a Process and Workflow Run Crate, version 0.5) is served
    unchanged; SEEK has no code that reads `CreateAction`.
- **To verify:** the server cost of `/ro_crate_metadata` itself (it builds the
  crate model without zipping); whether the 2-second pause can be relaxed
  for it.

### 2026-11-09: Full rollout of Simplified Technical English, tests 1, 2, 5, 6, 7, 12 and 16

- **Test:** static checks of all rewritten files, then test prompts 1, 2, 5,
  6, 7, 12 and 16 with the final text.
- **Agent and route:** Claude Code desktop on macOS, public API (interim);
  crates read with `ro-crate-consume` and `rocrate` 0.16.0 in a scratch
  virtual environment. Skills were loaded directly, so activation was not
  tested (descriptions are unchanged from GitHub `main`). The tester wrote
  the skills.
- **Result:** pass with issues (fixed)
- **Static checks:** inventories for all 17 rewritten files found every
  instruction in the new text; code blocks, tables and citation templates
  byte-identical; shared copies in sync; frontmatter descriptions and the
  generic skills unchanged from `main`; every guide command block re-run
  verbatim on 401, 1594, 1612, 1765, 1266 and 1815 with the same output
  sizes; `claude plugin validate .` passed.
- **What happened:**
  - Test 1 (find): four Galaxy-filtered queries, 78 unique workflows, a
    shortlist of seven; IWC 401 and 1201 first, as a pair. No workflow
    files downloaded.
  - Test 2 (evaluate, top two from test 1): stages detected and ordered;
    the hand-over now fully stated (1201's header setting: no for STAR
    counts, yes for featureCounts); `profile.yaml` saved for both.
  - Test 5 (explain 401 for `sample1_R1.fastq.gz` files): level 3 from the
    saved file; seven stages; no setting has a default; inputs mapped
    (needs preparation, missing GTF, unclear genome and strandedness).
  - Test 6 (cite 1201 v11 on usegalaxy.eu): citation with the versioned
    URL; `workflow-provenance.yaml` written to the project and parsed.
  - Test 7: Snakemake variant-calling candidates found; upload declined.
  - Test 12 (inspect 401): inputs "not declared" in the crate, with a
    pointer to the 11 inputs in the workflow file; Workflow RO-Crate
    structure observed but not declared.
  - Test 16: the crate of 2367 v1 is 61.6 MB (HEAD), over the limit; the
    agent stops and asks.
- **Verified against WorkflowHub:**
  - 1201 v11: the DataCite XML shows DOI
    10.48546/workflowhub.workflow.1201.11, but doi.org returned 404, the
    API lists none and there is no `cite-as` link. The DOI rule kept it out
    of the citation.
  - 401 v19: the API lists one creator (Lucille Delisle, no ORCID); the
    crate lists Lucille Delisle (ORCID 0000-0002-1964-4960) and Pavankumar
    Videm. Found only by reading the crate with `rocrate`.
- **Issues:**
  - Explain and evaluate reused only an "inspection record"; they now also
    reuse a saved profile that has the crate facts.
  - The test project links only find, evaluate, explain and cite, not
    inspect, `ro-crate-consume` or `fair-signposting`, so explain and
    evaluate cannot hand over to them there. Inspect was followed from its
    file.
  - `rocrate` on this Mac's Python 3.9 prints a urllib3 LibreSSL warning;
    harmless.

### 2026-11-09: Test 2 (Evaluate) on the Simplified Technical English pilot

- **Test:** test 2, "Compare the top two from that shortlist. It's for a
  paper. Which should I use?" Test 1's session wasn't available, so the
  two workflows its log names were used: 401 (rnaseq-pe/main) and 1201
  (rnaseq-de/main), with test 1's context (mouse liver, paired-end,
  treated vs control, Galaxy).
- **Agent and route:** Claude Code desktop on macOS, public API (interim).
  The skill was loaded directly, so activation was not tested. The tester
  wrote the skill and its references, and did not reread `tools.md`, the
  profile or the Galaxy guide, which were already in context: a fresh
  agent should repeat this test.
- **Result:** pass with issues (fixed)
- **What happened:** The procedure ran in order: context, level 1 and 2
  profiles for both workflows, ratings for all eight criteria with sources,
  comparison, concerns, what WorkflowHub can't tell, and a stop for the
  decision. 10 requests (2 metadata, 2 Bioschemas, 2 Link headers, 2 crate
  metadata, 2 TRS descriptors); 257 KB saved, about 15 KB read into
  context. Verdict: both fit with caveats, used together, 401 then 1201.
- **Verified against WorkflowHub:** 401 v19 (upstream 1.8, 2026-10-30) and
  1201 v11 (upstream 0.12, 2026-09-08); MIT; team IWC (Bioschemas);
  creators Lucille Delisle (no ORCID) and Pavankumar Videm (ORCID); no DOI
  on any version, no `cite-as` link, no linked publication; maturity not
  declared in crate or Bioschemas; README and tests file in both crates.
  401's file uses fastp, while its registry description still says
  cutadapt, and its `tools` list names 3 of its 10 tools. 401 needs a STAR
  index held on the Galaxy server; 1201 needs no server reference.
- **Issues:**
  - The two workflows are consecutive stages, not alternatives, and the
    skill's comparison step assumed alternatives. Step 4 now says how to
    handle stages.
  - The Galaxy reference scan listed four STAR settings that only sit
    under `refGenomeSource`. Fixed in the guide (it now checks the last
    named part of each key) and retested on six workflows.
  - Running the scans on subworkflows needed a command the guide didn't
    give; it does now.
  - `tools.md` sends crate reading through `ro-crate-consume` and the
    `rocrate` library. For three level 1 facts (maturity, file list,
    tests) the test read the metadata-only response directly instead.
    Open question: should the profile allow that for single fields?
  - Whether 401's count table has a header line (a setting in 1201) was
    left unknown, for workflowhub-explain.

### 2026-11-09: Simplified Technical English pilot on evaluate

- **Test:** style and coverage check of the rewritten `workflowhub-evaluate`
  `SKILL.md` (80% ASD-STE100); not yet a run of a test prompt.
- **Agent and route:** Claude Code desktop on macOS; no WorkflowHub requests
- **Result:** pass
- **What happened:** Listed the 55 instructions and facts in the previous
  text, rewrote the body, and checked that each one is in the new text.
  A rough style check (sentence length, -ing words, passive voice, phrasal
  verbs) gave:

  | | Before | After |
  |---|---|---|
  | Sentences | 92 | 167 |
  | Mean words per sentence | 14.0 | 8.6 |
  | Over 20 words | 23 | 0 |
  | Over 25 words | 13 | 0 |
  | Passive constructions | 10 | 1 (a false positive) |
  | Phrasal verbs | 3 | 0 |
  | Words in the file | 1,514 | about 1,710 |

  The frontmatter is unchanged except `metadata.version`.
- **Issues:** the skill had not been run on a test prompt; see the next
  entry above.

### 2026-11-09: System guides moved to shared and tested

- **Agent and route:** Claude Code desktop on macOS, public API (interim)
- **Result:** pass
- **What happened:** Rewrote the Galaxy, CWL, Nextflow, Snakemake and
  other-systems guides around the profile's fields, moved them to
  `shared/systems/`, and copied them into evaluate as well as explain. Ran
  every command block verbatim on real files: Galaxy 223, 248, 1594, 1612;
  CWL 1765 (with its 43 tool files) and 2367 (main file only); Nextflow
  1266; Snakemake 1815. All exited cleanly; output was 1.5 to 8.4 KB per
  workflow.
- **Verified against WorkflowHub:**
  - Galaxy crates (223, 224, 248, 1594, 1612, 1616) hold the `.ga`, the
    crate metadata, `ro-crate-preview.html` and a diagram image; 248 also
    a README, changelog, tests file and test data. No XML. The editor's
    layout is 3 to 10% of a `.ga` file and indentation 22 to 46%.
  - Galaxy marks inputs wired from another step as `RuntimeValue` too, so
    the run-time command removes them (1612's unicycler reads).
  - Built-in Galaxy tools such as `Cut1` (1594) have no ToolShed path.
  - CWL crate sizes: 78 KB (2347), 686 KB (1765), 61 MB (2367), 204 MB
    (360, 361); 1333 sends no size.
  - 1266 has six `process_gpu` processes; 1815 has six entry rules in its
    Snakefile instead of `rule all`, and 66 files in `envs/`.
- **Issues:** Galaxy Format 2 and packed CWL weren't in the sample and are
  marked untested. The other-systems guide is untested.

### 2026-11-09: Workflow profile on four degraded cases

- **Agent and route:** Claude Code desktop on macOS, public API (interim)
- **Result:** pass with issues (fixed in the same change)
- **What happened:** Built level 1 and 2 profiles, following
  `workflow-profile.md`'s source order, for 1266 (Nextflow, empty
  `internals`, no creators), 1612 (Galaxy, no repository or maturity, all
  outputs unlabelled), 1815 (Snakemake, logic in 8 rule files) and 2367 (CWL,
  empty `internals`, 61 MB ZIP). Every level 1 field ended as a value with a
  source or `not declared`. Level 2 was reached from the files for 1266,
  1612 and 1815, and from the main file alone for 2367.
- **Verified against WorkflowHub:** `versions[].remote` and `ref` give the
  source repository for 248, 360, 1266, 1815 and 2367. The API's `creators`
  carries names and ORCIDs (248, 1815, 2367). The team name is only in
  Bioschemas `producer`. Maturity differed for 1266 (crate Stable,
  Bioschemas none). No crate of the four declares inputs or outputs.
  Every code block in the Galaxy, CWL, Nextflow and Snakemake guides ran
  verbatim on these files (output 0.4 to 7.4 KB).
- **Issues found and fixed:**
  - The crate's `hasPart` is not a full file list: 1815 lists 2 parts and
    holds 117 files, including the 8 rule files; 1266 lists 23 and holds 147.
  - Galaxy 1612 needs `hg38` (minimap2) and `hg38full` (bowtie2) on the
    server, and 1594 the BLAST database `refseq_mitochondrion`, visible only
    in step settings, some of them JSON stored as text. A level 2 summary
    alone would have missed them; the Galaxy guide now has a scan for them.
  - The Snakemake rule outline was 21.5 KB for 96 rules; listing rule names
    first takes 2.4 KB.
  - Galaxy inputs' collection type, value type and optional flag are in
    their settings; the Galaxy guide's input command now reads them.

### 2026-11-09: Response sizes and save-first extraction

- **Agent and route:** Claude Code desktop on macOS, public API (interim)
- **Result:** pass
- **What happened:** Saved responses to a scratch folder and measured them,
  then checked the extraction commands now in `shared/tools.md` and the
  explain guides. Metadata responses for workflows 42, 123, 248, 1441 and
  1625 were 3 to 13 KB; extracting the fields evaluate needs from 248 gave
  740 bytes. Searches were 17 KB ("rna-seq") and 39 KB ("assembly", 302
  hits); hits carry only ID and title. The Galaxy file for 248 version 7 was
  88 KB (subworkflows 14 KB and `tool_state` 7.5 KB of the step data); its
  inputs, steps and outputs extracted to about 4 KB.
- **Verified against WorkflowHub:** TRS descriptor type codes `CWL` (360 v2),
  `NFL` (1266 v7) and `SMK` (1815 v7) return HTTP 200 with the main file in
  `content`. Outline commands in the CWL, Nextflow and Snakemake guides
  ran on those three files.
- **`internals` sample (35 workflows):** empty for all 11 Nextflow (57, 966,
  979, 1002, 1005, 1058, 1266, 1864, 2199, 2341, 2374) and all 10 Snakemake
  (554, 791, 1091, 1102, 1815, 1967, 2360, 2361, 2362, 2373); empty for CWL 2367, filled for the other 8 CWL;
  filled for all 5 Galaxy (223, 224, 1594, 1612, 1616). Compared with the
  Galaxy files for 248, 223, 1594 and 1612: steps and inputs matched; 1594
  dropped 1 of 6 outputs (unlabelled) and 1612 gave 16 of 18 outputs, all as
  `_anonymous_output_<n>`; input types were `File` or `string` without
  Galaxy formats. 248 versions 1, 4 and 7 had identical `internals`, and the
  file structure didn't change either, so version tracking is not
  established.
- **Issues:** the Nextflow and Snakemake commands for modules and rule
  files were checked only against the main file, since the others need
  the crate's ZIP, which was not downloaded.

### 2026-10-14: Inspect workflow 123 version 1

- **Agent and route:** Claude Code, public API, Bioschemas and a saved crate
  read with `rocrate` 0.16.0
- **Result:** pass with issues
- **What happened:** Signposting gave DataCite XML, Bioschemas and the crate
  ZIP (7,893 bytes), with no `cite-as`. The crate (RO-Crate 1.3, Workflow
  RO-Crate 1.0 declared) has 1 input, 5 outputs, no steps, tests or runs. All
  requests returned 200 with the 0.5 s pacing.
- **Verified against WorkflowHub:** maturity Stable in the crate and
  Bioschemas, not in the API. The API holds a 195-character input description
  and 12 steps that the crate lacks. The crates for 46 and 123 list only a
  `contributor` (person 49) and no creator; 42 has a creator and a contributor;
  1441 has an `author` with an ORCID.
- **Issues:** an earlier summary called the contributor of 46 and 123 a
  creator. The skill and template now separate creators from contributors, and
  the template has rows for maturity and Bioschemas.

### 2026-10-14: JSON API, RO-Crate and Bioschemas compared

- **Agent and route:** Claude Code, public API; Bioschemas through
  `Accept: application/ld+json` on workflows 42 and 1441
- **Result:** pass with issues
- **What happened:** The API, crate and Bioschemas record were compared field by
  field for workflow 42, with 46, 123, 1441 and 1625 checked where noted.
  WorkflowHub answered 503 ("queue full") and then 520 under load, then
  recovered; this led to the pacing and backoff rules in `shared/tools.md`.
- **Verified against WorkflowHub:** maturity is in the crate and Bioschemas
  (`creativeWorkStatus`: Stable for 42 and 123, Work-in-progress for 46), not
  in the JSON API, and absent for 1441 and 1625. The API has inputs, outputs
  and steps (`internals`); the crate metadata has no steps. For 1625 the API
  lists 1 input and 2 outputs while the crate declares none. The crate's
  `sdDatePublished` equalled the request time. Bioschemas `contributor` was
  the submitter (a bot for 1441). For 1441 the API and Bioschemas list no
  creator, but the crate does.
- **Issues:** the Bioschemas record was read for two workflows only.

### 2026-10-14: Climate search, crate metadata and `rocrate` in a virtual environment

- **Agent and route:** Claude Code, public API plus HTTP headers; `rocrate`
  0.16.0 in a virtual environment in the scratch folder
- **Result:** pass with issues
- **What happened:** A search for "climate" found 22 workflows. The metadata
  of five (IDs 42, 46, 123, 1441, 1625, all version 1) came from
  `/workflows/<id>/ro_crate_metadata?version=1`, one at a time, 2 seconds
  apart, 2.8 to 6.9 KB each, all 200 `application/json`. `ROCrate(folder)`
  loaded all five from a folder holding only `ro-crate-metadata.json`.
- **Verified against WorkflowHub:** the metadata route works for all five, not
  only workflow 248. Crates for 42, 46 and 123 declare Workflow RO-Crate 1.0
  on RO-Crate 1.3; 1441 and 1625 declare RO-Crate 1.1 and no workflow profile.
  Only 123 declares both inputs and outputs. None has tests or recorded runs.
- **Issues:** the root dataset of crate 1625 is a URL given by the descriptor's
  `about`, not `./`, so a reader must follow `about`. The API listed no
  creators for 1441, but its crate names one with an ORCID.

### 2026-10-14: Molecular dynamics search and FAIR Signposting check

- **Agent and route:** Claude Code, public API plus HTTP `Link` headers
- **Result:** pass with issues
- **What happened:** Searches for "molecular dynamics", "MD simulation" and
  "gromacs" returned 68, 102 and 91 workflows in one page each. Metadata,
  `?version`, workflow-type and tag filters, RO-Crate download and TRS
  worked. Signposting headers on workflow 248 version 7 gave DataCite XML,
  Bioschemas JSON-LD and the RO-Crate link; `Accept` selected each
  representation. No linkset is offered; `cite-as` appears only for versions with a minted DOI (624 v1 yes, 248 v7 no).
- **Issues:** maturity not in the JSON; "MD simulation" pulls in unrelated
  workflows; biobb tutorials appear as many near-duplicate variants; the
  DataCite DOI for 248 v7 was not in the JSON API.

### 2026-10-14: Test 1 (Find, core case)

- **Agent and route:** Claude Code desktop on macOS, public API (interim)
- **Result:** pass with issues
- **What happened:** The output followed the Find skill's procedure and
  template. It ran nine searches (six unfiltered, three filtered to Galaxy),
  found 219 unique workflows, and returned a ranked shortlist of six options
  with a search log, assumptions, gaps and a suggested next step. It paused
  for a decision. It marked maturity as unknown rather than guessing, and
  flagged an unchecked question about whether two linked workflows connect
  correctly.
- **Verified against WorkflowHub:** for workflows 401 (rnaseq-pe/main) and
  1201 (rnaseq-de/main), the titles, team (IWC), license (MIT), latest
  versions and version dates were all correct.
- **Issues:**
  - Described 401 as using cutadapt; the current version uses fastp.
  - Reported version 16, while the page shows v1.5. Both identifiers are
    needed.
  - The workflow's own "last updated" date (January 2023) is older than its
    latest version (July 2026). The agent used the version date, which is
    better, but the skills should say so.
  - The workflow-type filter worked, but the exact parameter name wasn't
    recorded.