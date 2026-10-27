# Design rules

## Numbers

- **Sample size beside every average**: "61.8 · 48 calls".
- **"—" for no data, never 0.** A tile with no data says why in its context
  line ("no meetings were recorded").
- **Denominator on every rate** that has more than one honest reading: "42.9%
  of reached leads".
- **Each number traceable**: its tool or formula is visible on hover, or in
  a small "how this is counted" note.
- **Thin values marked**: fewer than 5 records → greyed, never ranked.
- **Labels are the business's words**: "SLA met", "Win rate", "Speed to lead",
  not field names.
- Tabular numerals, thousands separators, money rounded to whole units,
  hours under 1 shown as minutes.

## Charts

| Shows | Use |
|---|---|
| A measure over time | line (rates, scores) or bars (counts) |
| Parts of a whole, ≤ 6 parts | donut with the total in the centre and a side legend |
| Ranking people or reasons | horizontal bars, sorted |
| Outcome mix over time | stacked bars |
| People × qualities | heatmap |
| The team on a per-person chart | a dashed line |

Bars at most about 32 px wide. Tooltips stay inside their card. No 3D, no
pie with more than 6 slices, no dual axes.

## States

- Skeletons while loading; the rest of the page stays usable.
- A failed source: "Failed to load" with a small marker on that card only,
  and a retry. Never a blank card.
- Refreshing: the old values dimmed, not removed.
- An access error (sign-in expired, not granted): one clear message on the
  page with how to fix it ("Reconnect Customer Journey in Settings →
  Connectors").
- Empty selection: "Pick or create a team first", with a button to where that
  happens.

## Look

- Clean and dense: cards with 1 px borders, very little text, sentence case.
- Colours as theme tokens, so light and dark both work.
- Green and red only for better and worse; never the only signal (add an
  arrow or a sign).
- Works at laptop and tablet width; tables scroll inside their card, with the
  header and first column fixed.

## Privacy on the page

No customer emails or phone numbers, no recording links. Records are shown by
name and id. Agents' names and emails are fine.
