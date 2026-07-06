# Notes: PUT /users/:id

## Plan
Read `tests/update-user.test.js`, `routes/users.js`, and `db/store.js` first to
match the existing GET/POST patterns rather than invent a new style. The route
needed to: validate `name`/`email` are present (400), look up the user via a
new store helper (404 if missing), update and return it (200). Validation
happens before the store lookup, mirroring how `POST /` already validates
before calling `store.createUser`.

## Model
Claude Sonnet 5 (`claude-sonnet-5`), via Claude Code. Chosen because this is a
small, well-specified CRUD addition to an existing pattern (mirror GET/POST) —
no novel architecture or ambiguous tradeoffs that would call for a heavier
reasoning model, and Sonnet is fast enough to iterate through plan → implement
→ test → review in one sitting.

## Commit split
Two commits, in dependency order:
1. `Add updateUser helper to db/store.js` — the store-layer change, independent
   of any route and testable in isolation.
2. `Add PUT /users/:id endpoint` — the route wiring that depends on (1).

Splitting this way keeps each commit reviewable on its own: the first is pure
data-layer logic, the second is just request/response plumbing.

## Review
`npx eslint .` — clean, 0 errors/warnings. `npm test` — all 9 tests pass,
including all 3 cases in `tests/update-user.test.js` (200 update, 404 for
missing id, 400 for missing field).

Ran `/code-review` (8 finder angles: line-by-line scan, removed-behavior
audit, cross-file tracer, reuse, simplification, efficiency, altitude,
CLAUDE.md conventions) against the diff. No correctness bugs surfaced — the
correctness angles (line scan, removed-behavior, cross-file callers) came
back empty: id parsing behaves correctly for `id=0`/non-numeric ids, no
export or invariant was dropped, no caller relies on the old
`updateUser`-shaped return, and handlers are fully synchronous so there's no
race window between the read and the mutation.

Two low-severity findings survived verification, both accepted as-is rather
than fixed:
1. **Reuse** — the `!name || !email` → 400 check in the new PUT handler
   duplicates the identical check in `POST /` verbatim. A shared validator
   would prevent future drift, but with only two call sites it doesn't clear
   this repo's own bar against premature abstraction ("three similar lines is
   better than a premature abstraction").
2. **Conventions** — the new route comment
   (`// PUT /users/:id — update an existing user...`) describes *what* the
   code does rather than a non-obvious *why*, which the global CLAUDE.md
   comment rule technically disallows. It matches the pre-existing
   comment style already on every other route in this file, though, so I
   kept it for local consistency rather than removing it unilaterally.
