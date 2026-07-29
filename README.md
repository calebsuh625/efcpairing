# efcpairing

Pairing system for tutors/students based on interest, skill, and scheduling, prev done manually but this is inefficient and there have already been efforts to fix this. I propose a new system that only aims to fix what is currently not optimal about English Tutoring

## Usage

Open `tutor-match.html` in a browser. That's the whole install — one file, inline
CSS and JS, no build step, no backend, no network calls.

Click **Load sample data** for a demo roster of 12 tutors and 20 students that
exercises the tricky cases.

The tool is admin-facing. There are no logins. It *suggests* pairings and never
assigns them — you make the call with one click (human-in-the-loop)

## The timezone problem

Tutors enter availability in **US Pacific**. Students enter it in **Beijing time
(UTC+8)**. These do not line up in any intuitive way:

| Tutor (Pacific) | Student (Beijing) |
| --- | --- |
| Friday 17:00–17:30 | **Saturday** 08:00–8:30 |
| Saturday 21:00–21:30 | **Sunday** 12:00–12:30 |
| Sunday 17:00–20:00 | **Monday** 08:00–11:00 — useless it's a school day |

Saturday morning in China is Friday afternoon in California, so the weekday label
differs on each side of every match. Every overlap is displayed in **both**
timezones with the correct weekday for each.

- **Overlap is computed in absolute UTC**, not by shifting clock times, so date-line
  crossings fall out naturally instead of being special-cased.
- **US daylight saving is handled; China has no DST.** The offset is 15 hours in
  summer and 16 in winter, so the same session reads an hour earlier on the Pacific
  side all winter.
- **Weekends that straddle a DST transition are flagged**, since Pacific clocks
  change at 02:00 Sunday while Beijing does not move.
- **Transition edge cases are handled explicitly.** Pacific Sunday 01:00–04:00 is
  2 real hours in March and 4 in November. The nonexistent 02:30 (spring forward)
  shifts forward; the doubled 01:30 (fall back) resolves to the first occurrence.

## Session slots

Sessions are fixed **30-minute slots from 07:00 to 13:00 Beijing time**, Saturday and
Sunday — 24 slots a week, the last being 12:30–13:00. Both sides tick boxes on the same
grid; each just sees it on their own clock:

| | Student picks (Beijing) | Tutor sees (Pacific) |
| --- | --- | --- |
| Summer (PDT) | Sat / Sun 07:00–13:00 | **Fri / Sat** 16:00–22:00 |
| Winter (PST) | Sat / Sun 07:00–13:00 | **Fri / Sat** 15:00–21:00 |

The grid is anchored in Beijing time because China has no DST, so a slot always means
the same real moment. Two things follow from that:

- Overlap becomes an exact **set intersection** of slot IDs rather than interval
  arithmetic, so it cannot silently go wrong across the date line.
- Pairings stay **stable across the US clock change** — only the Pacific label moves,
  not who matches whom.

The one rough edge it creates: over winter the earliest Saturday slot lands at 15:00
Friday Pacific, during US school hours. The tool flags that as a weakness rather than
hiding it.

Because the Pacific label of a slot depends on the date, everything is rendered against
an explicit **reference weekend** you pick in the header.

## Matching

Suggestions are ranked out of 100:

| Factor | Weight | Notes |
| --- | ---: | --- |
| Shared slots | 35 | **Hard gate** — no shared slot is never suggested |
| Subject / goal fit | 25 | shared focus areas |
| Level appropriateness | 20 | gated, with a marginal-suggestions override |
| Shared interests | 10 | free-text, e.g. basketball, piano |
| Tutor capacity | 10 | prefers tutors below max, balances load |

Every score comes with its reasoning — what makes it good and where it's fragile:

> **10 shared 30-min slots (5 h):** Sat 08:00–11:00 Beijing · Fri 17:00–20:00 PDT
> **Both listed:** conversation, reading
> **Shared interests:** basketball, movies
> **Tutor has 1 of 2 slots open** (1 already assigned)
> ⚠ This is the tutor's last open slot
> ⚠ 1 other unassigned student also ranks this tutor first, but only 1 slot remains

Existing pairings are preserved by default. Only unmatched students get proposals.

## Problems it surfaces

- Students with **no viable tutor**, and specifically why — no overlap at all, no
  tutor at their level, or the only fits are at capacity
- Tutors with **open capacity but no viable student**
- **Existing pairs whose availability no longer overlaps** (e.g. after a DST shift
  or an availability edit)

## Saving your work

State is held in memory only — nothing is written to browser storage.

- **Export JSON** is your save file. **Import JSON** restores it.
- It round-trips everything: both rosters, selected slots, and existing pairings.

Export before closing the tab, or you'll lose the session.

## Self-tests

The header runs 27 assertions on every load, covering DST boundaries to the minute,
date-line crossings, the nonexistent and doubled hours, the shape of the session grid,
slot intersection, and the scoring gates. The panel expands itself if anything fails.

## Out of scope

No accounts, passwords, email, notifications, attendance tracking, hour logging, or
analytics. WeChat already handles all of that.
