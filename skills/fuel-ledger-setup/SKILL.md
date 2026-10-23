---
name: fuel-ledger-setup
description: Use when someone asks to set up or update Fuel Ledger (燃料帳), e.g. "Set up Fuel Ledger", "Update Fuel Ledger", 「燃料帳をセットアップして」; checks the connectors and the spreadsheet, fills in settings, and publishes the page as the user's own private claude.ai Artifact.
---

# Setting up Fuel Ledger

Fuel Ledger is a page where Claude estimates nutrition from meal photos and notes and logs it to a Google Sheet, and that shows a daily energy balance from Strava activities and Withings weight. Artifacts that use connectors can't be shared, so **each person has to publish their own copy**. This skill walks through that.

The page itself is `app/fuel-ledger.html` in the same repository as this skill. If it isn't available locally, get it from https://github.com/hirokitnk/fuel-ledger (`git clone`, or save `https://raw.githubusercontent.com/hirokitnk/fuel-ledger/main/app/fuel-ledger.html`).

The page's on-screen text is English; the sheet's tab names and stored values stay Japanese. Talk to the user in whatever language they use, and ask whether they want Claude's estimates written in English or Japanese (`log_language` = `en` or `ja` on the 設定 tab).

## 1. Check prerequisites

1. Check that this session's tool list includes Google Sheets and Strava connector tools. They appear as names like `mcp__<id>__get_values`, `mcp__<id>__append_values` and `mcp__<id>__list_activities`. Read the `artifact-capabilities` skill to see which id belongs to which connector.
   - If they're missing, tell the user: "Add Google Sheets and Strava under claude.ai Settings → Connectors, then reopen this session." Then stop. The Strava connector requires a paid Strava subscription.
2. Check that the Artifact tool is available (Claude Code or the Claude desktop app). If it isn't, say so and stop.
3. Read the `artifact-capabilities` skill and confirm that `mcp` and `sample` are among the available capabilities.

## 2. Check the spreadsheet

1. Ask the user to copy the sheet first, using the "Copy the template" link in the README, and then ask for the URL of their copy.
2. Take the spreadsheet ID from the URL: the part between `/d/` and the next `/`.
3. Read `食事ログ!A1:H2` and `設定!A1:J200` with `get_values`.
   - If a tab can't be read, tell the user which one is missing and ask whether to create it. If they agree, create it with the Google Sheets connector's add-tab tool (`update_spreadsheet` or similar; read the tool's description to choose), then write the headers and defaults from `template/README.md` with `update_values`.
   - The 食事ログ headers are `日付, 区分, 内容, エネルギー(kcal), たんぱく質(g), 脂質(g), 炭水化物(g), 確度`.
4. Also try reading `Withings!A1:E5`. If it doesn't exist, explain that it's optional and point to the steps in `docs/withings-ifttt.md`.

## 3. Fill in settings

If `age`, `sex`, `height_cm`, `weight_kg` or `body_fat_pct` on the 設定 tab are blank, ask the user for them. Show what you'll write (cells and values) and **get their consent before** writing with `update_values`. Mention that if there is a Withings tab, its most recent 7-day average weight and body fat % take priority. Optionally ask for `goal_note` (e.g. a target race).

Tell them the base burn that results from the calculation (Mifflin-St Jeor × activity factor), and that they can override it with `base_office_kcal` / `base_home_kcal` if it doesn't feel right.

## 4. Publish the page

1. Read `app/fuel-ledger.html` and write a copy to a working directory such as your scratchpad, with `__SPREADSHEET_ID__` in `const SPREADSHEET_ID = "__SPREADSHEET_ID__";` replaced by the ID from step 2. Don't modify the original in the repository. Keep the file name `fuel-ledger.html`.
2. Check that the tool names the page calls (`get_values`, `append_values`, `list_activities`) match this session's connector tool names. If they differ, change the page's `callTool(SHEETS, "...")` / `callTool(STRAVA, "...")` and the declaration below to match, and tell the user what you changed. Also check the argument names (`spreadsheetId`, `range`, `values`, `first`, `range_start`, `range_end`) against the tools' input schemas.
3. Publish with the Artifact tool. On the first publish, pass `icon: "food"` and declare these capabilities:

   ```json
   {
     "mcp": {"servers": [
       {"server": "Google Sheets", "tools": ["get_values", "append_values"]},
       {"server": "Strava", "tools": ["list_activities"]}
     ]},
     "sample": {"images": true}
   }
   ```

   `server` must be the connector's display name exactly as the `artifact-capabilities` skill shows it.
4. Once published, give the user the link. Tell them that when they open the page they'll be asked, in turn, to allow Google Sheets, Strava and Claude.

## 5. Explain how to use it on an iPhone

- Open the published link in **Safari on the iPhone** and sign in to claude.ai. Connectors probably don't work inside the Claude iPhone app.
- Tap Share → "Add to Home Screen" to open it like an app.
- The first time, approve the prompts for Google Sheets, Strava and Claude.
- Tap "＋ Log" at the bottom, take a photo or pick one from the library, tap Estimate, check the numbers, then tap Save to sheet.

## 6. Updating

When `app/fuel-ledger.html` in the repository changes, publish again as in step 4, **passing the URL of the previously published Artifact as `url`**. If you don't know the URL, find it with the Artifact tool's `action: "list"`. The link stays the same, so the Home Screen icon keeps working. If the capabilities haven't changed, you can omit `capabilities` (the previous declaration carries over).
