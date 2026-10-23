# Fuel Ledger (燃料帳)

Fuel Ledger is a personal nutrition log for endurance athletes that runs as a private claude.ai Artifact. Snap a photo of your meal, Claude estimates calories and macros, and you append the result to your own Google Sheet. Strava activities and (optionally) Withings weigh-ins are combined into a daily energy balance, energy availability and a weight trend. It is designed to be used from Safari on an iPhone.

> The sheet's tab names (食事ログ, 設定) and the values written to it (meal types such as 朝, confidence 高/中/低) stay in Japanese for compatibility. Set `log_language` on the 設定 tab to choose whether Claude writes descriptions in English (`en`, the default) or Japanese (`ja`).

## What it does

<!-- Add screenshots here -->
<!-- ![Daily balance](docs/images/day.png) -->
<!-- ![Logging a meal](docs/images/add.png) -->

- Claude estimates kcal, protein, fat and carbs from one or more photos plus a short note. You review and edit the numbers, then append one row to your sheet
- "Intake − Burn = Balance", plus how much protein, fat and carbs you have left for the day
- Strava activities are converted to the extra burn on top of your resting burn, and added to the day
- A warning when energy availability (intake − exercise) drops below your floor
- A 14-day balance chart
- Withings 7-day average weight and weekly fat mass (optional)

## What you need

- A **paid Claude plan** (Pro, Max, Team or Enterprise). Photo estimates **use your own Claude usage**
- A paid **Strava** subscription (required by the Strava connector on claude.ai)
- **Google Sheets** (a Google account)
- **Claude Code** or the **Claude desktop app** (only for publishing the page)
- A Withings scale (optional)

## Setup

### 1. Copy the template sheet

Open [Copy the template](https://docs.google.com/spreadsheets/d/1X_fhxmI3Py8gC3MUcaeueAt_wIkLuNCgnepKfR4oQUw/copy) and click "Make a copy". You get a sheet with two tabs, 食事ログ (meal log) and 設定 (settings), in your own Drive. Keep the tab names exactly as they are; the page looks them up by name.

To build the sheet by hand instead, follow the column layout in [template/README.md](template/README.md).

### 2. Connect Google Sheets and Strava on claude.ai

On claude.ai, go to **Settings → Connectors**, add **Google Sheets** and **Strava**, and sign in to each.

### 3. Install the skill and ask Claude to set it up

1. Clone this repository.

   ```bash
   git clone https://github.com/hirokitnk/fuel-ledger.git
   ```

2. Install the skill (Claude Code).

   ```bash
   mkdir -p ~/.claude/skills && cp -r fuel-ledger/skills/fuel-ledger-setup ~/.claude/skills/
   ```

3. In Claude Code or the desktop app, open the cloned folder and say "Set up Fuel Ledger" (or 「燃料帳をセットアップして」). Claude asks for your sheet URL from step 1 and, if they are blank, your age, height and so on. At the end you get a link to your own private page.

### 4. Open it in Safari on your iPhone and add it to the Home Screen

1. Open the link from step 3 in **Safari on your iPhone** and sign in to claude.ai
2. Tap Share → **Add to Home Screen**
3. The first time, approve the prompts that let the page use Google Sheets, Strava and Claude

Day to day: tap **"＋ Log"** at the bottom → take a photo or pick one from your library → add a note on amounts (e.g. "rice 130 g, the fried item is chicken breast") → **Estimate** → check the numbers → **Save to sheet**.

### Manual setup (without the skill)

1. Do steps 1 and 2
2. Copy `app/fuel-ledger.html` and replace `__SPREADSHEET_ID__` in `const SPREADSHEET_ID = "__SPREADSHEET_ID__";` near the top with your sheet's ID (the part of the URL between `/d/` and the next `/`)
3. Fill in age, sex, height, weight and body fat % on the 設定 tab
4. In Claude Code or the desktop app, ask Claude to "publish this HTML as an Artifact with these capabilities" and give it this declaration:

   ```json
   {
     "mcp": {"servers": [
       {"server": "Google Sheets", "tools": ["get_values", "append_values"]},
       {"server": "Strava", "tools": ["list_activities"]}
     ]},
     "sample": {"images": true}
   }
   ```

5. Open it on your iPhone as in step 4

### Updating

Run `git pull`, then ask Claude to "update Fuel Ledger". It republishes to the URL of the Artifact you published before, so **your link stays the same**.

## Adding Withings weight data

This is optional. If your sheet has a tab named `Withings`, the page shows your weight trend and weekly fat mass, and uses your 7-day average weight and body fat % to calculate your base burn.

The standard route is an IFTTT applet (Withings → Google Sheets) that writes to a separate file, which you pull into the `Withings` tab with `IMPORTRANGE`. See [docs/withings-ifttt.md](docs/withings-ifttt.md). Sheets filled by an Apps Script from the Withings API (A = date/time, B = weight, C = body fat %) also work as they are.

## Calculation rules (summary)

- **Base burn** = basal metabolic rate (Mifflin-St Jeor) × activity factor (1.5 on office days, 1.4 on home days), rounded to the nearest 100 kcal. You can override it on the 設定 tab
- **Exercise extra**: run = Strava × 0.93, swim = minutes swimming × 8.5 kcal/min, Zwift = × 0.8, bike = × 0.6, strength = × 0.7, anything else = × 0.7
- **Intake target** = base burn + exercise extra − daily deficit (300 kcal by default)
- **Energy availability floor** = fat-free mass × 30 kcal
- **Protein** = body weight × 1.8–2.0 g, **fat** ≤ 70 g, **carbs** = body weight × 4 g (× 5 g on days with ≥ 400 kcal of exercise extra)
- Nuts are estimated about 15% below food-table values (20% for almonds) to account for incomplete absorption

Details and sources: [docs/calculation-rules.md](docs/calculation-rules.md).

## Limitations

- **Connectors (Google Sheets, Strava) probably don't work inside the Claude iPhone app.** Open the claude.ai link in Safari instead
- **Artifacts that use connectors can't be shared with other people.** A link someone else published won't work for you, so everyone sets up their own copy
- The office/home toggle is **remembered per device (browser)**, so your iPhone and computer keep separate settings
- **Edit or delete logged rows in the spreadsheet.** The page only appends
- Change personal settings on the 設定 tab, then tap ↻ on the page to reload them

## Disclaimer

- Fuel Ledger is **not a substitute for medical or nutritional advice.** If you have a medical condition, are pregnant, or have been told to restrict your diet, talk to a doctor or registered dietitian
- Estimates from photos **are approximate**, especially for restaurant food or dishes whose amounts are unclear. Treat low-confidence entries as rough guides
- **Strava data is used only for calculations inside the page and is never sent to Claude** (per Strava's terms). Claude receives your meal photos and notes, your profile from the 設定 tab (age and so on), and your list of frequently eaten products

## License

[MIT](LICENSE)
