# blinkup-activations

Admin/config prototypes for the BlinkUp Connect SDK — the screens a team uses to set up fan
activations and season-long rewards. These are **front-end prototypes**: they render the full
admin UI and publish their configuration as JSON to the browser's `localStorage`. That JSON is
the **data contract** we want the backend to persist and serve.

Live: https://wlbott.github.io/blinkup-activations/

## Pages

| File | What it is | Reads | Writes |
|------|-----------|-------|--------|
| `ActivationBuilder.html` | Game-day activation builder (Bar of the Game, Stat Trigger Deal, Fan of the Game, Time-Based Prizing) | `blinkup_activations` | `blinkup_activations` |
| `GameDay.html` | Operator console — fires the configured activations on game day | `blinkup_activations` | — |
| `SeasonRewards.html` | Season-long **Grand Prize** builder (points-based, season-long) | `blinkup_grand_prize` | `blinkup_grand_prize` |

To inspect any payload live: open the page, hit **Save & Publish**, then in dev tools →
Application → Local Storage read the key listed above.

## Data contracts

### `blinkup_grand_prize` (SeasonRewards.html)

Single object.

```json
{
  "seasonStart": "2025-10-10",
  "seasonEnd": "2026-04-15",
  "seasonLabel": "Season 2025",
  "sponsorName": "Bud Light",
  "primaryColor": "#241773",
  "accentColor": "#9E7C0C",
  "name": "Season Grand Prize",
  "prize": "Two Tickets to a Ravens Game",
  "whatsIncluded": ["2 premium game tickets", "$100 dining credit"],
  "points": [
    { "key": "stadium", "label": "Attend a game",        "points": 500, "showOnCard": true },
    { "key": "bar",     "label": "Attend a watch party", "points": 200, "showOnCard": true },
    { "key": "home",    "label": "Home watch party",     "points": 100, "showOnCard": true }
  ],
  "winnerMode": "drawing",
  "entrants": 100,
  "drawnWinners": 1,
  "outrightWinners": 10,
  "announceAt": "2026-04-16T12:00"
}
```

Notes:
- **`winnerMode`** — `"drawing"`: top `entrants` fans get entered, `drawnWinners` are drawn at
  random. `"outright"`: top `outrightWinners` fans win directly. Only the fields for the selected
  mode are relevant.
- **`points[].key`** is the stable identifier (`stadium` / `bar` / `home`); `label` is fan-facing
  display text; `showOnCard` controls whether the row appears in the app's "How to Earn Points" list.
- **`primaryColor` / `accentColor`** drive the fan card (primary = headers/score card/badges,
  accent = card background/highlights).

### `blinkup_activations` (ActivationBuilder.html → GameDay.html)

Array of activation objects. Common fields: `id`, `typeId`, `name`, `prize`, `isInstant`,
`scheduledTime`. Type-specific fields:

| `typeId` | Activation | Extra fields |
|----------|-----------|--------------|
| `botg` | Bar of the Game | `barMethod` (`random` / `manual` / `usage`) |
| `stat` | Stat Trigger Deal | `eligibleStadium`, `eligibleBars` (always instant) |
| `fotg` | Fan of the Game | `fanMethod` (`random` / `invitations` / `redemptions`) |
| `time` | Time-Based Prizing | `winnerCount` **or** `scheduledTime`, `eligibleStadium`, `eligibleBars` |

Only enabled activations are written. `GameDay.html` reads this array to drive the operator console.

## Out of scope for these payloads

- **Game schedule** — the season "My Season" punch-card (opponent + home/away + game order) should
  come from the existing **Events** data, not from `blinkup_grand_prize`.
- **Runtime/computed values** — fan rank, points totals, leaderboard, check-in counts, and progress
  bars are all computed server-side/at runtime; none of it lives in these configs.
