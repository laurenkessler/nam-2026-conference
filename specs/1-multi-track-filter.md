# Let attendees filter the schedule by more than one track

## What this changes
On `/schedule` the track pills become toggles: several can be on at once, **All** clears them. A session shows if it is in any selected track and still matches every other filter. The URL carries `?track=a,b`; `GET /api/sessions` accepts `trackSlug=a,b`. One value behaves exactly as today.

## Where
- `server/routes/sessions.js` — the `trackSlug` line in the `GET /` handler: split on commas, drop empties, emit `t.slug IN (?, ?)`; test in `tests/api/endpoints.test.js` (a new test beside the filter tests, using `boot.tracks` slugs and a day from `conferenceDays`-style `boot.days`).
- `src/pages/SchedulePage.jsx` — `filters.track` stays the raw comma string (so `qs()`, `useFetch` deps, `activeKeys` and `BLOCKERS` keep working); the track pill block (~line 163) toggles one slug in/out of that list, writes it back with `set('track', …)`, sets `aria-pressed` on each pill, and marks **All** pressed when none are selected. Test in `tests/schedule.spec.js` using `visit`, `waitForResults`, `API` and `conferenceDays` from `tests/helpers.js` (read-only, no lane needed).

## How it will be proved
| Done when | Check | Layer |
| --- | --- | --- |
| Two tracks list exactly those tracks' sessions; `result-count` matches the API | API: `trackSlug=a,b` equals the union of `a` and `b` ids for the day; spec: click two pills, `result-count` equals the API count | API + spec |
| Removing one track narrows the list again | spec: click one pill off, `result-count` equals the single-track API count | spec |
| Each pill has `aria-pressed` | spec: `aria-pressed` true/false after each click | spec |
| **All** clears the selection | spec: click All, URL has no `track=`, `result-count` equals the unfiltered count | spec |
| Single-track link unchanged, existing API tests pass | API: `trackSlug=a` equals the earlier single-slug result; existing `endpoints.test.js` stays green | API |

## Decisions
- Selecting every track individually leaves `?track=a,b,c…` rather than collapsing to All — OK?
- Unknown slugs in the list simply match nothing (no error) — OK?

## Out of scope
`/speakers` keeps its single-track filter; a track filter still forces list view.
