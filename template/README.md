# Spreadsheet layout

For people who build the sheet by hand instead of copying the template. Tab names must match exactly (including full-width/half-width characters).

## 食事ログ tab (meal log, required)

Row 1 is the header; data starts on row 2. The page reads `食事ログ!A2:H5000` and appends one row when you save.

| Column | Header | Contents |
|---|---|---|
| A | 日付 | Date, `YYYY-MM-DD` (plain text recommended) |
| B | 区分 | Meal: 朝 (breakfast) / 昼 (lunch) / 夕 (dinner) / 間食 (snack) / 運動中 (during exercise) |
| C | 内容 | Items and amounts joined with 「、」 |
| D | エネルギー(kcal) | Number |
| E | たんぱく質(g) | Protein, number |
| F | 脂質(g) | Fat, number |
| G | 炭水化物(g) | Carbs, number |
| H | 確度 | Confidence: 高 (high) / 中 (medium) / 低 (low) |

If Google Sheets auto-converts column A to dates, they may display as `2026/01/15` and the page can no longer match them. Select column A and choose Format → Number → Plain text.

## 設定 tab (settings, recommended)

Keys in column A, values in column B, descriptions in column C. The page reads `設定!A1:J200`. Missing keys and blank values fall back to the defaults.

| Key | Example | Default | Description |
|---|---|---|---|
| `age` | 45 | 40 | Age |
| `sex` | male | male | male / female |
| `height_cm` | 172 | 170 | Height |
| `weight_kg` | 68 | 65 | Weight (replaced by the 7-day average if there is a Withings tab) |
| `body_fat_pct` | 18 | 20 | Body fat % (replaced by the 7-day average if there is a Withings tab) |
| `activity_office` | 1.5 | 1.5 | Activity factor on office days |
| `activity_home` | 1.4 | 1.4 | Activity factor on home days |
| `base_office_kcal` | (blank) | — | Base burn on office days. Overrides the calculation when set |
| `base_home_kcal` | (blank) | — | Base burn on home days. Overrides the calculation when set |
| `deficit_kcal` | 300 | 300 | Daily deficit target |
| `protein_g_per_kg` | 1.8-2.0 | 1.8-2.0 | Protein target range per kg of body weight |
| `fat_max_g` | 70 | 70 | Fat ceiling |
| `carb_g_per_kg_easy` | 4 | 4 | Carbs per kg on light days |
| `carb_g_per_kg_hard` | 5 | 5 | Carbs per kg on days with ≥ 400 kcal of exercise extra |
| `ea_kcal_per_kg_ffm` | 30 | 30 | Energy availability floor per kg of fat-free mass |
| `goal_note` | Some Marathon 2027 | (blank) | Goal shown in the header and included in the estimate prompt |
| `withings_col_date` | A | A | Withings tab column holding the date/time |
| `withings_col_weight` | B | B | Withings tab column holding the weight |
| `withings_col_fat` | C | C | Withings tab column holding body fat % (blank if none) |

### Frequently eaten products (same tab, from column E)

Put a header row with `品名` in E1, then one product per row below it. These are inserted into Claude's estimate prompt as known values.

| E 品名 (name) | F 量 (amount) | G kcal | H たんぱく質 (protein) | I 脂質 (fat) | J 炭水化物 (carbs) |
|---|---|---|---|---|---|
| be LEGEND WEIGHT DOWN | 30g | 114.5 | 21.8 | 0.9 | 4.9 |
| オイコス プレーン砂糖不使用 | 1個 | 72 | 13 | 0 | 5 |
| マルトデキストリン | 20g | 80 | 0 | 0 | 20 |
| 納豆（たれ込み） | 1パック | 95 | 8 | 5 | 6 |

## Withings tab (optional)

The page only reads `Withings!A1:H5000`; it never writes. Header rows and unreadable rows are skipped. If you weigh in more than once a day, the lower (later) row wins.

Supported formats:

- **Apps Script format**: A = date/time `2026-01-15 07:30`, B = weight (kg), C = body fat (%), D = muscle mass, E = grpid
- **IFTTT format**: A = date/time `January 15, 2026 at 07:30AM`, B = weight, C = body fat %, D = unit (`kg` / `lbs`). See [docs/withings-ifttt.md](../docs/withings-ifttt.md)

If any cell in a row says `lb` or `lbs`, the weight is converted to kg. If your columns are in a different order, set `withings_col_*` on the 設定 tab.
