# Verify before handing over

Do not say the dashboard is finished until every step has run. Report what
passed, what failed and what could not be checked, plainly.

## 1. Every data source, once

Run each data source at the default people and period. Every one returns
data or an honest empty. Fix a failing call and run it again. A refused
filter name: fix it from the list the error gives.

## 2. The page itself

After each change, read the platform's check result if it has one. Fix
script errors, `NaN`, `undefined`, blank charts, and labels cut off or
overlapping.

## 3. The flow, end to end

- Change the period: every tile and chart updates, and none keeps an old value.
- Change the people, including down to one person, and to none (empty state,
  no calls made).
- Open every tab and sub-tab.
- Click through every drill-down: a person, a bar, a row.
- Every link between pages carries what it should.
- Exports produce a file, or the stated fallback.

## 4. Numbers against the source

Pick 3–4 headline numbers and get each with one direct tool call of your own,
same people and period. They must match exactly. Always include:
- one count (calls, leads);
- one rate with a tricky denominator (SLA met, connect rate);
- one value combined across people (team win rate);
- one per-person value.

A mismatch is a bug in the page, not rounding. Find it before handing over.

## 5. The catalog's traps

Check the page against [data-rules.md](data-rules.md): medians not averages
for hours, connected calls only for call quality, rates from counts, the
win-rate cohort named, time series with gaps filled, "—" not 0, no empty list
sent.

## 6. What could not be checked

Steps that need the viewer (approving a model call, a download, a sign-in)
are named as not verified, with what the user should try.
