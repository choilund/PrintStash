# Library search and browsing clarity

The library keeps typing and Enter in the current view. Enter flushes the query,
closes suggestions and preserves collection, tags and other filters. The explicit
**Search with AI** suggestion opens the existing search route; Enter on that route
submits another AI query. Result cards retain navigation, previews and subject
labels while diagnostic evidence remains available only in the API.

Collection navigation stays visible. Advanced filters open on demand, with active
sections expanded for shared URLs and saved views. Removable filter chips remain
above results. Library tools contains organization commands; active selection has
its own count and Done action. Detail tabs use a two-row grid, and Similar cards
switch layout according to their panel width.

## Retrieval policy

Name recovery is a separate lexical signal: a single normalized token of at least
five characters can differ by one insertion, deletion, substitution or adjacent
transposition. Indexed vocabulary prefix probes and authorized posting queries
have fixed candidate caps. Exact names precede recovered spelling matches.
Candidate retrieval, final materialization and cursor context preserve visibility,
structured filters and live/trashed restrictions. Leading-character recovery uses
Latin letters, digits and characters in the query; it is not a complete Unicode
spellchecker.

A small domain vocabulary finds functional metadata through indexed conjunctive
phrases (for example a holder's cradle or a gear's toothed wheel). This does not
supply visual labels or replace the embedding models. Semantic native scores must
clear their own threshold before rank fusion; multiple weak legs cannot bypass
that threshold. Administrator per-space overrides remain authoritative.

The calibrated policy applies to single-word text queries using the pinned BGE
small English encoder and CLIP B/32 thumbnail encoder. Text uses 0.67 and thumbnail
CLIP uses 0.27. Other models, multiword queries, multiview, point-cloud, uploaded
images and related-Model queries retain their existing independently measured
policies. Index identities, public API contracts and visual aggregation are
unchanged. These bounded fixtures do not establish universal relevance across
arbitrary libraries or languages.

## Reproducing quality measurements

`backend/tests/fixtures/search/clarity-corpus.json` freezes four calibration objects,
four absent calibration concepts, seven held-out objects and six validation
queries. Geometry is original or repository-owned analytic geometry. The goose has
an opaque name and no description; the holder's description expresses function
without the word “holder”. Distractors include misleading filenames. No private
library geometry, descriptions or screenshots are fixtures.

`clarity-vectors.json` contains real measured CPU embeddings, model revisions,
render hashes, corpus/generator hashes and inference timings. Regenerate explicitly
with the pinned local model exports:

```sh
cd backend
.venv/bin/python -m tests.fakes.search_clarity_measure \
  --bge-dir /path/to/pinned-bge --clip-dir /path/to/pinned-clip \
  --output tests/fixtures/search/clarity-vectors.json
.venv/bin/python -m tests.fakes.search_clarity_calibrate
.venv/bin/pytest tests/integration/modules/search/retrieval/test_clarity_quality.py -q -s
```

Calibration uses only calibration rows: take the maximum unrelated cosine,
including cross-object and absent-concept negatives, add 0.01 and round upward to
0.01. Text's maximum is 0.65416; thumbnail CLIP's is 0.25917. Native positive recall
on calibration objects is respectively 1/4 and 3/4; functional lexical retrieval
recovers gear and bolt without accepting their weak semantic similarities. The
held-out assertions do not choose these thresholds.

The existing text benchmark and visual benchmark remain unchanged. Visual quality
covers thumbnail and multiview styles, point-cloud retrieval and an attributed
photograph of a physical Benchy. Uploaded-image and related-Model behavior is
verified separately from text-to-appearance retrieval.

The baseline replays the retrieval implementation from `4b9afeb9` against the same
frozen database and measured vectors. All six new acceptance assertions fail at
that baseline; all pass with the calibrated implementation.

| Metric | Base | Implementation |
|---|---:|---:|
| Positive recall@5 (4 queries) | 4/4 | 4/4 |
| Precision among returned top-five results | 4/20 | 4/4 |
| Absent-concept abstention | 0/2 | 2/2 |
| Relevant Spectre rank (`specter`) | 1 | 1 |
| Relevant holder rank | 3 | 1 |
| Median positive-query replay wall time | 640 ms | 1,140 ms |

Wall times are single observations under shared-host CPU contention, replaying
measured embeddings rather than running inference. They establish neither a speed
improvement nor a stable performance regression. The committed vector file retains
separate actual CPU-inference timings. Exact spelling recovery also passes with AI
disabled, so finding Spectre no longer depends on a semantic match.

Local verification so far: 37 final lexical/relevance integration tests plus two
visibility cases passed. Core coverage ran 2,105 tests and all five floor checks:
99.28% combined statement/branch coverage; both new helpers have 100% coverage.
The new PostgreSQL cases passed. The full PostgreSQL file exposed an existing
rollback test that omitted the base branch's deferred projection worker; explicitly
advancing that worker restored its intended assertion (verified separately).
Existing text, visual and sparse quality files passed. Frontend format/lint/typecheck
and backend/core lint/typecheck passed. The security diff scan
of `4b9afeb9..b91ab3ef` reviewed all 20 source inventory entries and found no reportable
issue. Broad coverage and browser gates are still in progress; early or interrupted
runs are not passed gates.

## Acceptance coverage

Status denotes final verified evidence, not merely a test's presence.

| # | Behavior | Category | Input | Asserted outcome | Tier | Status |
|---|---|---|---|---|---|---|
| 1 | Enter preserves library filtering | Happy | Query plus active filters | Library URL retains filters | Frontend unit | ❌ Final run pending |
| 2 | Explicit AI action opens results | Happy | Ready AI, nonempty query | Dedicated search route | Frontend unit | ❌ Final run pending |
| 3 | Clearing preserves other filters | Edge | Filtered library | Only query removed | Frontend unit/browser | ❌ Final run pending |
| 4 | Late responses stay stale | Edge | Replaced/cleared query | No stale suggestions | Frontend unit | ❌ Final run pending |
| 5 | Cards omit explanations | Happy | Evidence-bearing results | No explanation disclosure | Frontend unit | ❌ Final run pending |
| 6 | Advanced filters start collapsed | Happy | Default library | Collection navigation visible | Frontend unit | ❌ Final run pending |
| 7 | Active filters are discoverable | Edge | Shared URL/saved view | Expanded sections and chips | Frontend unit/browser | ❌ Final run pending |
| 8 | Clear all resets advanced filters | Edge | Family and ordinary filters | Default filtering, preserved sort | Frontend unit | ❌ Final run pending |
| 9 | Secondary actions retain permissions | Error | Read-only collection | Restricted actions disabled | Frontend unit | ❌ Final run pending |
| 10 | Tabs fit narrow panels | Edge | 400px detail panel | No horizontal overflow | Playwright | ❌ Final run pending |
| 11 | Similar names remain readable | Edge | Narrow desktop panel | Name width and reachable Compare | Playwright | ❌ Final run pending |
| 12 | Specter finds Spectre | Happy | Spelling variants/distractors | Intended result first | Backend integration | ✅ 37-test final retrieval run |
| 13 | Goose matches appearance | Happy | Opaque name, original geometry | Relevant result in top five | Backend integration | ✅ 37-test final retrieval run |
| 14 | Holder matches function | Happy | Indirect name, functional metadata | Relevant result in top five | Backend integration | ✅ 37-test final retrieval run |
| 15 | Weak matches are rejected | Edge | Absent concept | No strong matches | Backend integration | ✅ 37-test final retrieval run |
| 16 | Search respects visibility | Error | Private/trashed/filtered candidates | No unauthorized results | Backend integration/PostgreSQL | ✅ Both candidate paths and PostgreSQL cases |
| 17 | Semantic failure preserves keywords | Error | Inference failure | Keyword results, accurate status | Backend integration | ❌ Final run pending |
| 18 | Revised search works end to end | Happy | Real backend/local index | Library → explicit AI results | Real-backend Playwright | ❌ Final run pending |
| 19 | Selection always offers Done | Edge | Grouped Family view, keyboard selection | Count and Done visible outside tools | Frontend unit | ❌ Final run pending |
