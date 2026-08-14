---
name: research-conference-presenter-roster
description: Build current, evidence-backed rosters of conference presenters, speakers, panelists, moderators, profiled participants, and associated organizations over a configurable time range. Use when researching a conference, annual event, summit, festival, or event series and the output must identify people, current titles, organizations, participation type, contact routes, source URLs, and uncertainty without inferring attendance or fabricating contact data.
---

# Research Conference Presenter Roster

## Goal

Create a defensible, reusable roster of people and organizations connected to a specified conference or event series. The skill is designed for requests such as “find education journalists who presented at EWA", "find individuals who repsented at ASU+GSV or SXSW EDU during the last five years,” but must generalize to any sector, geography, event, and date range.

The primary unit is the person-event relationship, not merely the organization. Capture the person’s name, verified current or event-time title, organization, participation type, event and date, and evidence. Maintain a linked organization table so the result can also describe the organizations that make up the field.

Do not equate public mention, registration, or conference proximity with attendance. Report exactly what the evidence supports: `Presenter`, `Speaker`, `Panelist`, `Moderator`, `Interviewer`, `Profiled participant`, `Award recipient`, `Workshop leader`, `Exhibitor`, `Sponsor`, `Named attendee`, `Publicly mentioned`, or `Attendance unclear`.

## Research contract

At kickoff, write `research_contract.md` and state:

- Event or event series, official host, canonical event URL, and aliases.
- Sector and inclusion definition, such as education journalism, education media, assessment, philanthropy, or education technology.
- Geography and treatment of virtual, international, chapter, partner, and satellite events.
- Time range with inclusive start and end dates, plus the research date.
- Person inclusion rule: what counts as a qualifying participant and which evidence classes are accepted.
- Organization inclusion rule: organizations represented by qualifying people, event hosts/partners, or independently prominent field organizations.
- Desired completeness standard and stopping rule; never claim absolute completeness from web search alone.
- Required deliverables, output location, batch size, public-source budget, and freshness window.

If a choice is missing, make a conservative assumption, record it, and proceed. Ask only when the ambiguity would materially change the roster.

## Default operating model

Use a coordinator plus independent research batches. The coordinator owns scope, source mapping, candidate IDs, deduplication, schema, validation, exception handling, and final audit. Subagents only research assigned candidates or source slices; they must not redefine scope or merge entities.

Use a low-cost coordinator such as a Luna-class medium model for orchestration and bounded research subagents for evidence collection. Dispatch independent batches in parallel when the environment supports it. Prefer stable batches of 20–40 candidate people or one bounded source family per batch.

### Phase 1: Discovery

Build `candidate_ledger.csv` before deep verification. Capture:

`Candidate ID`, name as found, organization as found, event/year, participation clue, discovery URL, source class, geography, and provisional inclusion reason.

Search the official event program, speaker pages, session pages, archived agendas, videos, transcripts, awards pages, event news, host announcements, and official social posts. Use reputable secondary coverage and organizational biographies to discover candidates, then seek primary confirmation.

For recurring events, enumerate each year or edition in the requested range. Record missing or inaccessible editions rather than silently treating them as having no presenters.

### Phase 2: Normalize and prioritize

Normalize names, titles, organizations, domains, event editions, and dates. Deduplicate conservatively using exact names, official biographies, domains, employment history, and evidence of identity. Keep two people separate when identity is uncertain.

Prioritize candidates using event-source strength, prominence, sector fit, repeated participation, evidence availability, and expected effort. Place ambiguous or likely-important omissions in `exception_queue.csv`.

### Phase 3: Verification

Each subagent returns one structured record per assigned candidate. Verify:

1. Person identity and name variants.
2. The specific event edition and date.
3. The participation type supported by the source.
4. Organization and title at the time of the event, if available.
5. Current organization/title only when separately verified and clearly dated.
6. Public contact route, when available, without guessing personal emails.
7. Evidence URLs for every material claim.

Prefer official event pages, host archives, session pages, speaker bios, videos/transcripts, official organization pages, regulatory or professional directories, and reputable reporting. Search snippets are discovery aids only, never final evidence.

### Phase 4: Validate and merge

At every batch checkpoint, validate required fields, URL health, date range, person identity, organization identity, duplicate relationships, event edition, participation type, evidence coverage, and currentness. Merge clean records and route only failed, conflicting, low-confidence, or scope-ambiguous records to the exception queue.

Retain raw subagent returns separately from canonical tables. Preserve stable IDs and prior validated rows on resumed runs.

### Phase 5: Audit and report

Finish with reconciled counts for candidates, included, excluded, deferred, unresolved, people, organizations, event editions, and evidence classes. Report missing editions, source limitations, likely coverage gaps, duplicate decisions, and the percentage of included person-event rows with primary event evidence.

## Canonical outputs

Create these files in a project-specific folder:

- `research_contract.md`
- `candidate_ledger.csv`
- `person_event_roster.csv`
- `organization_roster.csv`
- `source_ledger.csv`
- `batch_manifest.csv`
- `progress.json` or `progress.csv`
- `exception_queue.csv`
- `unresolved_items.csv`
- `coverage_memo.md`
- optional audited `.xlsx` workbook and narrative report

Use one row per person-event relationship in `person_event_roster.csv`. Use one row per organization in `organization_roster.csv`. Use the source ledger for many-to-one evidence relationships.

### Person-event schema

Use these fields unless the user supplies a compatible schema:

`Person Record ID`, `Person Name`, `Name Variants`, `Organization Name`, `Organization Record ID`, `Title at Event`, `Current Title`, `Current Title Checked Date`, `Sector Role`, `Event Name`, `Event Edition`, `Event Date`, `Event URL`, `Participation Type`, `Attendance Evidence Status`, `Session / Panel / Program Role`, `Topic`, `Geography`, `Public Contact Route`, `Public Email`, `Public Profile URL`, `Primary Evidence URL`, `Additional Evidence URLs`, `Evidence Checked Date`, `Confidence`, `Research Status`, `Research Notes`.

Allowed `Attendance Evidence Status` values include `Direct event evidence`, `Official participant listing`, `Official recording/transcript`, `Official profile or announcement`, `Reputable secondary evidence`, `Public mention only`, and `Unclear`. Use `Not publicly located` rather than guessing missing fields. A public email must be explicitly published for professional contact; otherwise leave it blank and provide a public contact route or profile.

### Organization schema

Use these fields:

`Organization Record ID`, `Formal Organization Name`, `Public/Brand Name`, `Organization Type`, `Sector Subcategory`, `Official Website URL`, `Primary Geography`, `Mission / Role`, `Relationship to Event`, `Associated Person Count`, `Prominence Tier`, `Prominence Rationale`, `Parent / Affiliate`, `Operating Status`, `Primary Evidence URL`, `Additional Evidence URLs`, `Evidence Checked Date`, `Confidence`, `Research Notes`.

Do not treat an event sponsor, host, employer, publisher, funder, school, agency, or vendor as the same relationship. Preserve relationship labels.

## Coordinator prompt orientation

Use the following as the base coordinator prompt, replacing bracketed values:

> You are the coordinator for a conference presenter roster project. Event or series: [event]. Host: [host]. Sector: [sector]. Geography: [geography]. Time range: [start] through [end]. Research date: [date]. Person inclusion rule: [rule]. Organization inclusion rule: [rule]. Required fields: [schema].
>
> First enumerate editions and run a rapid discovery pass across official event programs, speaker pages, archived agendas, session pages, recordings, transcripts, host announcements, and reputable secondary sources. Create a candidate ledger quickly. Then normalize identities, assign stable IDs, deduplicate conservatively, and partition work into independent batches. Dispatch subagents with exact candidate rows and the common schema.
>
> Require every subagent to distinguish event-time title from current title and to classify the evidence-supported participation type. Do not infer attendance from a biography, social post, registration list, or organizational affiliation alone. Do not guess emails, roles, employers, or identities. Use controlled null/status values and route ambiguity to the exception queue. Validate URLs, dates, identity, duplicate relationships, source coverage, and required fields at each checkpoint. Save progress after discovery and every batch. At completion, reconcile counts and disclose missing editions, unresolved records, limitations, and the percentage of included rows with primary event evidence.

## Subagent prompt orientation

Give each subagent only its assigned batch, the contract, schema, and protocol:

> Research the assigned candidates for [event] during [date range]. Return exactly one structured record per assigned person-event candidate, preserving the supplied Candidate ID. Confirm the person’s identity, event edition/date, organization, title at the event, participation type, session or panel role, sector role, and public contact route when explicitly available. Separate current employment from event-time employment and label the evidence status precisely.
>
> Prefer official event pages, host archives, session pages, videos/transcripts, official organization biographies, and reputable institutional sources. Open sources; do not rely on snippets. Do not guess emails, titles, organizations, attendance, or identity. If evidence supports only a public mention, mark that status rather than upgrading it to presenter or attendee. Return source URLs for every material claim, confidence, research status, and concise notes on ambiguity.

## Contact and privacy rules

Collect only professional contact information that is publicly posted and relevant to the roster. Prefer official staff pages, newsroom contact pages, speaker profiles, organization contact forms, and public professional profiles. Never infer an email from a naming pattern, scrape private data, or present a personal address as verified without explicit publication. If the person’s contact is not public, record `Not publicly located` and retain the organization’s public contact route.

## Exception queue and stopping rule

Maintain `exception_queue.csv` with `Candidate ID`, `Exception Type`, `Evidence Checked`, `Next Best Action`, `Priority`, `Status`, and `Disposition`. Use types such as `IDENTITY CONFLICT`, `EVENT EDITION UNCLEAR`, `ATTENDANCE UNCLEAR`, `TITLE CONFLICT`, `ORGANIZATION CONFLICT`, `MISSING PRIMARY SOURCE`, `DUPLICATE REVIEW`, `SOURCE FAILURE`, and `LIKELY MISSING PROMINENT PERSON`.

Give each exception one targeted second search and one alternative source class. Then mark it `NEEDS_REVIEW`, `DEFERRED`, `EXCLUDED`, or `INCLUDED WITH LIMITATION`. Do not spend unlimited time on one person.

The roster may be described as “evidence-backed coverage of the defined sources and editions” only when every discovered candidate has a disposition, all included rows have evidence, editions were enumerated, duplicates and identity relationships were reviewed, and limitations are disclosed. Never label it “complete” solely because search results were exhausted.

## Resumability and quality metrics

Use statuses `NOT_STARTED`, `IN_PROGRESS`, `VALIDATED`, `NEEDS_REVIEW`, and `COMPLETE`. Preserve candidate order and stable IDs. Track candidate count, included count, verified person-event rows, organization count, editions covered, batches completed, elapsed time, verified rows per hour, exception count, primary-evidence percentage, current-title coverage, and unresolved rate.
