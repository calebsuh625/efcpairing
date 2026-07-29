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
  summer and 16 in winter, which changes real answers — on the sample roster,
  9 of 17 students get a different best match in January than in August.
- **Weekends that straddle a DST transition are flagged**, since Pacific clocks
  change at 02:00 Sunday while Beijing does not move.
- **Transition edge cases are handled explicitly.** Pacific Sunday 01:00–04:00 is
  2 real hours in March and 4 in November. The nonexistent 02:30 (spring forward)
  shifts forward; the doubled 01:30 (fall back) resolves to the first occurrence.

Because a recurring "Saturday 9am" block means a different real instant depending on
the date, overlap is always computed against an explicit **reference weekend** you
pick in the header.

## Matching

Suggestions are ranked out of 100:

| Factor | Weight | Notes |
| --- | ---: | --- |
| Schedule overlap | 35 | **Hard gate** — zero overlap is never suggested |
| Subject / goal fit | 25 | shared focus areas |
| Level appropriateness | 20 | gated, with a marginal-suggestions override |
| Shared interests | 10 | free-text, e.g. basketball, piano |
| Tutor capacity | 10 | prefers tutors below max, balances load |

Every score comes with its reasoning — what makes it good and where it's fragile:

> **3 h of overlap across 2 blocks:** Sat 09:00–11:00 Beijing · Fri 18:00–20:00 PDT
> **Both listed:** conversation, reading
> **Tutor has 1 of 3 slots open**
> ⚠ Only one shared time block — fragile if either side cancels
> ⚠ 2 other unassigned students also rank this tutor first, but only 1 slot remains

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
- **CSV import** for tutor and student rosters; **CSV export** of the final
  pairing list, with overlap hours and both timezones per pair.

Export before closing the tab, or you'll lose the session.

## Self-tests

The header runs 26 assertions on every load, covering DST boundaries to the minute,
date-line crossings, the nonexistent and doubled hours, weekday labelling on both
sides, and the scoring gates. The panel expands itself if anything fails.

## Out of scope

No accounts, passwords, email, notifications, attendance tracking, hour logging, or
analytics. WeChat already handles all of that.
