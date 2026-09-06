# What is in here, and what to do with it

Everything drops into your existing app repo. Nothing replaces the app.

```
supabase/migrations/          19 SQL files      -> commit only, do not run
supabase/functions/beta-feedback/index.ts       -> replaces the old one
supabase/diagnostic-bank-connections.sql        -> read-only helper
src/ui/tokens.js             new palette        -> replaces the old one
src/index.css                new type + colours -> replaces the old one
```

Already done: the security fix (`0019`) was run in the SQL editor and verified.
The waitlist is closed. Nothing below is urgent.

---

## 1. Commit the migrations

Safest first. Copy `supabase/migrations/` into the repo, commit, push.

**Do not run them.** They are already applied. This is only so a new laptop or a
staging project can rebuild the database from source, which it cannot do today.

## 2. The feedback function

First search the codebase for `beta-feedback`. Nothing in `App.jsx` calls it, so
if nothing else does either, **skip this step**. There is no point deploying a
fix to something no one calls.

If something does call it:

```bash
supabase functions deploy beta-feedback --no-verify-jwt
```

Then Dashboard -> Edge Functions -> beta-feedback -> Settings, and check
**Verify JWT is OFF**. The flag is sometimes ignored when updating an existing
function, and if the toggle stays on, the function returns 401 without ever
running.

The old version ran on the service role and read `user_id` from the request
body, so anyone could file feedback under someone else's account. This one takes
identity from the signed-in session instead, and adds the CORS handling the old
one was missing.

## 3. The new look

On its own branch, separate from the two above.

Copy in `src/ui/tokens.js` and `src/index.css`, then one find and replace in
`App.jsx`:

- find `color: P.brass`
- replace `color: P.brassText`

62 places. **Leave `background: P.brass` alone**, those are buttons and they are
correct as they are.

Why the split: the old single token measured 2.04:1 as text on the light
surface, where 4.5 is the minimum. Every small brass label was close to
unreadable in daylight. `brass` is now the fill, `brassText` is the text.

Then `npm run build`, look at it in both themes, merge.

---

## Not in here

The design prototype. It is a mockup with sample data and no database, and it
must never be deployed to `/app`. It lives in its own repo as a reference to
build against.
