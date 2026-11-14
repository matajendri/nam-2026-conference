# Filter the schedule by more than one track

## What this changes
On `/schedule` the track pills toggle independently: an attendee can turn on
Cloud and AI together and see the sessions of both, and nothing else. Every
other filter (day, venue, level, format, topic, search) still narrows on top.
Turning off the last track returns to "All". The selection is in the URL
(`?track=cloud,ai`), so it survives a reload and a shared link; a one-track
link reads exactly as before.

## Where
- `server/routes/sessions.js` — `trackSlug` is split on commas (empty parts
  dropped) into `t.slug IN (?, …)`. One value is the same query as today.
  `speakers.js` reads its own `trackSlug` and is not touched.
- `src/lib/api.js` — unchanged: `qs()` already passes `trackSlug=a,b` through.
- `src/pages/SchedulePage.jsx` — `selectedTracks` is `params.get('track')`
  split on commas; `filters.track` stays a string (`all` or `a,b`) so the
  `useFetch` deps, `activeKeys`, `BLOCKERS` and Clear all keep working
  unchanged. A track pill toggles its slug in or out; the URL lists slugs in
  the bootstrap `tracks` order so the same selection is always the same link.
  "All" clears the key. Every pill carries `aria-pressed`.
- `tests/api/endpoints.test.js` — a two-track request in `filtered()`, and a
  new describe proving the union.
- `tests/schedule.spec.js` — one browser test (desktop and mobile).
- `context/engineering-context/schedule.md` — the `trackSlug` row and the
  rail description now say several tracks.

## How it will be proved
| Done when | Check | Layer |
| --- | --- | --- |
| two tracks list both tracks' sessions and no others | `trackSlug=a,b` on one day returns exactly the ids of `a` plus the ids of `b` (false today: `t.slug = 'a,b'` matches nothing) | API |
| other filters still apply | `day` + `trackSlug=a,b` returns only that day's sessions, and adding `level` returns a subset in which every session has that level | API |
| one-track link unchanged | `trackSlug=a` equals today's single-track result, every session in `a` | API |
| selecting, deselecting, back to All, in the URL, survives reload | spec: pick A → `result-count` = API count for A; add B → count = API count for A,B, URL `track=a,b`, both `aria-pressed=true`; reload keeps both pressed; drop A → count = B's; drop B → All pressed and no `track` in the URL (false today: clicking B replaces A) | browser |
| grid stays unavailable with a track filter | unchanged code path (`BLOCKERS` reads `filters.track`); the existing grid tests and the spec's `view=list` load cover it | existing |

## Decisions
- Matching is any selected track (OR), confirmed when the ticket was written.
- Order in the URL follows the track list, not click order, so a selection is
  one link however it was built.

## Out of scope
Venue, level, format and topic stay single-choice. The speakers page's
`trackSlug` filter is unchanged.
