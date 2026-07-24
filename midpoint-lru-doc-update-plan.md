# Documentation Update Plan: Midpoint-LRU in Manage the OpenEdge Database

## Feature Summary

The feature described in `green.htm` adds midpoint-LRU behavior to the general LRU buffer chain (GNLRU) to reduce scan pollution. It introduces three startup parameters, new observability surfaces, and lifecycle logging:

- `-lruColdPercent`
- `-lruColdTimeThreshold`
- `-lruColdAccessThreshold`

The feature is runtime behavior, not a database creation format change. It applies when a database is started with the new parameters and remains off by default. It does not change database creation, copy, conversion, or deletion procedures themselves.

Working set refers to the data blocks that users and applications are actively using often enough that the database benefits from keeping them in memory. Protecting the working set helps the database avoid extra disk reads and maintain faster, more consistent performance.

## Documentation Goal

Update the Create and Delete Databases subtree so users who create, copy, or convert a database understand all of the following:

- Midpoint-LRU is configured at database startup, not during database creation.
- Any database created or copied by supported methods can use the feature when started with the new parameters.
- The feature is limited to GNLRU and does not apply to `-B2` or `-Bp` behavior.
- `-znolru` disables the feature.
- Detailed parameter syntax and observability details belong in startup, monitoring, and reference topics, with cross-links from the Create/Delete subtree.

## In-Scope Pages in the Create and Delete Databases Subtree

| Page | Action | Planned update |
| --- | --- | --- |
| Create and Delete Databases | Revise | Add a short overview paragraph that distinguishes database creation tools from post-creation startup tuning. State that midpoint-LRU is a startup-controlled cache-management feature available after a database is created, copied, or converted. Add a cross-link to the startup-parameter topic once available. |
| Ways to create an OpenEdge database | Revise | Add a note that the database creation method does not enable midpoint-LRU. Clarify that databases created with PROSTRCT CREATE, PRODB, PROCOPY, Data tools, or conversion workflows can use midpoint-LRU only when the database is started with the new startup parameters. |
| Create a database with PROSTRCT CREATE | Revise | Add a post-procedure note: after creating the database structure and loading schema, midpoint-LRU can be enabled only at database startup. Keep the page procedural; do not add parameter tables here. |
| Create a database with the PRODB utility | Revise | Add a note that PRODB copies structure and contents, but does not persist midpoint-LRU behavior into the database. Clarify that the feature is enabled only by startup parameters when the copied database is opened. |
| Create a database with a Data tool | Revise | Add a short note that Data Dictionary, Data Administration, and Database Administration Console creation workflows do not configure midpoint-LRU. Point readers to the startup configuration topic for enabling the feature after creation. |
| Copy a database | Revise | Add one paragraph explaining that copying a database does not change midpoint-LRU behavior because the feature is not a stored database-format option. A copied database can use midpoint-LRU when started with the required parameters. |
| Copy a database using PROCOPY | Revise | Add a post-task note parallel to the PRODB note: copied databases are eligible for midpoint-LRU at startup, but PROCOPY does not enable or configure the feature. |

## Pages in the Subtree That Should Stay Unchanged

These pages do not need midpoint-LRU content unless the team decides to add a broad, repetitive note everywhere in the subtree:

- Database creation and large file support
- PRODB maintains pathname convention
- Convert an OpenEdge Release 12 Database to OpenEdge Release 13
- Convert an OpenEdge Release 11 Database to OpenEdge Release 12
- Convert an OpenEdge Release 10 database to OpenEdge Release 12
- Database conversion utilities
- Use the Schema Mover
- Delete a database

Rationale: the feature does not alter file layout, pathname behavior, conversion mechanics, schema movement, or deletion behavior. Adding midpoint-LRU detail to those pages would dilute topic focus without improving task completion.

## Recommended Messaging Pattern for the Create/Delete Pages

Use a consistent short note pattern instead of duplicating detailed reference content:

> Midpoint-LRU is an optional database startup feature that helps protect the working set from large scans. It is not enabled when you create, copy, or convert a database. To use it, start the database with the midpoint-LRU startup parameters. The feature applies only to the general LRU buffer chain and remains disabled when `-znolru` is in effect.

Keep the subtree content at the conceptual level. Put parameter definitions, valid ranges, defaults, examples, and monitoring outputs in the runtime and reference topics.

## Required Follow-On Updates Outside the Create/Delete Subtree

Because the design adds startup parameters and observability, the Create/Delete updates should link to follow-on topics in the same documentation set:

- Start Up and Shut Down overview: add a forward reference that some database buffer-management features, including midpoint-LRU, require a restart because they are startup parameters.
- Monitoring or reference topics for VST, promon, and `mybkfiles`: document the new counters, enablement fields, and diagnostic switches.
- Database log messaging topic, if one exists in this bundle: document the shutdown summary entries written to the `.lg` file when midpoint-LRU is enabled.

## Required Updates in the Database Startup Parameters Subtree

Only the startup-parameter pages that either summarize performance parameters or provide the authoritative per-parameter entries need midpoint-LRU updates.

| Page | Action | Planned update |
| --- | --- | --- |
| Database server performance parameters | Revise | Add the three new midpoint-LRU startup parameters to the quick-reference performance table with concise one-line descriptions. Position them near existing buffer and LRU-related parameters such as `-B`, `-B2`, `-lruskips`, and `-lru2skips`. |
| Alphabetical listing of database startup parameters | Revise | Add entries for the three new parameters so they appear in the chapter's detailed alphabetical reference. |
| Individual parameter pages for `-lruColdPercent`, `-lruColdTimeThreshold`, and `-lruColdAccessThreshold` | Add | Create the three detailed parameter topics with syntax, usage category, default, valid range, enablement rules, `-znolru` interaction, and GNLRU-only scope. |

## Database Startup Parameters Pages That Should Stay Unchanged

These child pages do not need midpoint-LRU edits unless engineering later decides the feature belongs in a different parameter classification:

- Issue startup parameters
- Startup parameter usage categories
- Database server-type parameters
- Database server internationalization parameters
- Database server statistics parameters
- Database server consistency check parameters
- Database server TLS connection parameters
- Database server network parameters

Rationale: midpoint-LRU is a database server performance feature. It does not introduce a new usage category, and it does not belong to server-type, internationalization, statistics, consistency-check, TLS, or network parameter groupings.

## Engineering Clarifications Required Before Publishing

The design text contains several value and unit inconsistencies. Resolve these before any doc text is finalized:

| Question | Conflicting details in source | Doc impact |
| --- | --- | --- |
| What is the valid range for `-lruColdPercent` when enabled? | One section says `[5, 95]` when set; another defines min `0`, max `95`, with default `0`. | Needed for parameter table and validation text. |
| What is the default for the access-threshold parameter? | One section says default promote count is `1`; another says `DBPRM_LRU_COLD_ACCESS_THRESHOLD_DEFAULT = 3`. | Needed for examples and tuning guidance. |
| What are the units for `-lruColdTimeThreshold`? | One section uses milliseconds with default `1000` and cap `3600000`; another uses seconds with default `1` and max `3600`; field descriptions also say seconds. | Needed for syntax, examples, and monitoring descriptions. |
| What exact customer-facing term should docs standardize on? | The design uses midpoint-LRU, cold/hot, OLD/YOUNG, promote-to-MRU, and GNLRU split terminology. | Needed for consistent terminology across all topics. |

## Proposed Authoring Tasks

1. Update the overview page and creation-method pages listed in scope with a consistent startup-feature note and cross-links.
2. Update the Database Startup Parameters subtree by revising the performance quick-reference page, the alphabetical listing, and the three new individual parameter topics.
3. Update monitoring/reference topics for VST, promon, `mybkfiles`, and `.lg` lifecycle logging.
4. Validate terminology and numeric defaults against engineering before review.
5. Run a final consistency pass to ensure every creation and copy workflow uses the same midpoint-LRU positioning statement.

## Acceptance Criteria for the Documentation Update

- Users can tell that midpoint-LRU is not configured by database creation or copy utilities.
- Every relevant creation and copy topic points readers to the startup configuration path.
- No out-of-scope pages are cluttered with unnecessary midpoint-LRU detail.
- Parameter values, units, defaults, and restrictions are consistent across all updated topics.
- Observability references exist for VST, promon, `mybkfiles`, and `.lg` logging.
