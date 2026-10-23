# Logging Withings weight to the sheet with IFTTT

Every time you step on a Withings scale, IFTTT appends a row to a Google Sheet. Fuel Ledger reads it for your weight trend and base burn calculation.

## Before you start

- **IFTTT writes to a separate file, not to your Fuel Ledger sheet.** IFTTT's "Add row to spreadsheet" action takes a file name and folder in Google Drive and appends to the first sheet of that file; it can't target a tab in an existing file. So you pull the data into a `Withings` tab in your Fuel Ledger sheet with `IMPORTRANGE` (step 4 below).
- On IFTTT's free plan, applets check hourly, so a weigh-in can take up to an hour to show up.
- IFTTT starts a new file every 2,000 rows. That's over five years at one weigh-in a day, but when it happens, point step 4 at the new file.
- **Check the date format and unit yourself once the first row arrives.** These steps are based on the fields the IFTTT Withings trigger provides (`DateAndTime`, `Weight`, `Unit`, `FatMass`, `FatMassPercent`, `LeanMass`), but the author hasn't verified the output in every environment.

## Steps

1. Create a new IFTTT applet. If This = **Withings → New weight measurement**, Then That = **Google Sheets → Add row to spreadsheet**.
2. Set "Formatted row" to the following, so the columns are A = date/time, B = weight, C = body fat %, D = unit:

   ```
   {{DateAndTime}} ||| {{Weight}} ||| {{FatMassPercent}} ||| {{Unit}}
   ```

   Use a recognizable file name such as `withings-log`. The default folder (such as `IFTTT/Withings`) is fine.
3. Step on the scale once and confirm that the file appears in Drive with a first row like `January 15, 2026 at 07:30AM | 65.2 | 18.5 | kg`.
4. In your Fuel Ledger sheet, create a tab named `Withings` and put this in A1, replacing `<URL of the IFTTT file>` with the file from step 3. Click "Allow access" the first time.

   ```
   =IMPORTRANGE("<URL of the IFTTT file>", "A1:D")
   ```

5. Tap ↻ on the Fuel Ledger page and the weight section appears.

## Troubleshooting

- **You changed the column order**: add `withings_col_date`, `withings_col_weight` and `withings_col_fat` to the 設定 tab with the column letters (A, B, …). Leave `withings_col_fat` blank if you have no body fat column.
- **Weights are in pounds**: rows that contain `lbs` are converted to kg automatically. If you didn't include the unit column, switch the Withings app to kg.
- **Dates aren't recognized**: the page reads `2026-01-15 …`, `2026/01/15 …` and `January 15, 2026 at …`. For anything else, build a column that converts the dates to `YYYY-MM-DD` (for example with `=ARRAYFORMULA(...)` instead of a plain IMPORTRANGE) and point `withings_col_date` at it.

## If you already write to the sheet from the Withings API with Apps Script

If your rows look like A = date/time (`2026-01-15 07:30`), B = weight (kg), C = body fat (%), they work as a `Withings` tab as they are.
