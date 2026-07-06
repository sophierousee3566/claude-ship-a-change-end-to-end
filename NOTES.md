# Notes: PUT /users/:id

## Plan
Read `tests/update-user.test.js`, `routes/users.js`, and `db/store.js` first to
match the existing GET/POST patterns rather than invent a new style. The route
needed to: validate `name`/`email` are present (400), look up the user via a
new store helper (404 if missing), update and return it (200). Validation
happens before the store lookup, mirroring how `POST /` already validates
before calling `store.createUser`.

## Model
Claude Sonnet 5 (`claude-sonnet-5`), via Claude Code.

## Commit split
Two commits, in dependency order:
1. `Add updateUser helper to db/store.js` — the store-layer change, independent
   of any route and testable in isolation.
2. `Add PUT /users/:id endpoint` — the route wiring that depends on (1).

Splitting this way keeps each commit reviewable on its own: the first is pure
data-layer logic, the second is just request/response plumbing.

## Review
Verified manually rather than through an automated review pass:
- `npx eslint .` — clean, 0 errors/warnings.
- `npm test` — all 3 new cases in `tests/update-user.test.js` pass (200 update,
  404 for missing id, 400 for missing field), plus all pre-existing tests
  still pass. The only failures are this file's own existence checks before
  it was written.
- Checked `updateUser` returns `undefined` (not throwing) for an unknown id,
  consistent with `getUserById`'s existing not-found convention, so the route
  could reuse the same `if (!user)` shape as the other endpoints.
