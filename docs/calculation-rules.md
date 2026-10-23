# Calculation rules and rationale

Fuel Ledger's numbers come from a simple model meant to give you a rough guide. They are not a substitute for medical or nutritional advice.

## Daily balance

```
Balance       = Intake − Burn
Burn          = Base burn (office day / home day) + Exercise extra
Intake target = Burn − Daily deficit (deficit_kcal, 300 kcal by default)
```

The balance turns green when the deficit is more than half your target, yellow within ± half, and red above that.

## Base burn

Everything you burn outside exercise: basal metabolism, daily activity and the thermic effect of food.

```
BMR       = 10 × weight (kg) + 6.25 × height (cm) − 5 × age + 5     (men)
BMR       = 10 × weight (kg) + 6.25 × height (cm) − 5 × age − 161   (women)
Base burn = BMR × activity factor   (rounded to the nearest 100 kcal)
```

- The BMR formula is **Mifflin-St Jeor** (Mifflin et al., 1990). It is considered close to measured values in healthy adults and is the equation recommended in the Academy of Nutrition and Dietetics' evidence review.
- The default activity factors are 1.5 on office days and 1.4 on home days: the difference between a day with a walking commute and a day spent mostly sitting. Exercise is added separately from Strava, so it is not included here.
- Example: man, 45, 175 cm, 70 kg → BMR 1,573.75 kcal → 2,400 kcal on office days, 2,200 kcal on home days.
- If there is a `Withings` tab, weight and body fat % come from the most recent 7-day average.
- If the result doesn't match how you feel or what you measure (your weight trend), override it with `base_office_kcal` / `base_home_kcal` on the 設定 tab. Comparing two to three weeks of cumulative balance with the change in your 7-day average weight (1 kg of fat ≈ 7,200 kcal) gives you a basis for adjusting it.

## Exercise extra (fixed in the page)

Strava's calorie figures include what you would have burned at rest anyway. To avoid counting that twice against your base burn, each activity type is multiplied by a factor so that only the extra burn remains.

| Type (Strava `sport_type`) | Extra | Why |
|---|---|---|
| Run (Run, TrailRun, VirtualRun) | Strava × 0.93 | Running is intense, so the resting share is under 10% |
| Swim (Swim) | Minutes swimming × 8.5 kcal | Strava's swim figures vary a lot. Moderate front crawl (about 8.3 METs) at around 65 kg is roughly 9 kcal/min gross, or 7.9 kcal/min after subtracting 1 resting MET; 8.5 splits the difference |
| Zwift (VirtualRide) | Strava × 0.8 | Based on power meter data, so fairly accurate; the resting share is subtracted |
| Bike (Ride, EBikeRide) | Strava × 0.6 | Commutes are low intensity with lots of stops, so the resting share is large; estimates without power tend to run high |
| Strength (WeightTraining) | Strava × 0.7 | Long rests, and heart-rate-based estimates tend to run high |
| Anything else | Strava × 0.7 | Errs on the low side |

Exercise data is used only for calculations inside the page and is never included in the estimate prompt sent to Claude (per Strava's terms).

## Nutrition targets

| Item | Calculation | Setting | Rationale |
|---|---|---|---|
| Protein | Body weight × 1.8–2.0 g | `protein_g_per_kg` | A guide for preserving muscle in endurance athletes during weight loss (ISSN and ACSM position stands give 1.6–2.2 g/kg) |
| Fat | Ceiling | 70 g (`fat_max_g`) | What's left after securing carbs and protein. The fat share of calories is also shown (green at 30% or less) |
| Carbs | Body weight × 4 g (× 5 g on days with ≥ 400 kcal of exercise extra) | `carb_g_per_kg_easy` / `_hard` | A guide for days with about an hour of moderate training; lower than the ACSM/IOC range of 5–7 g/kg because you're in a deficit |

## Energy availability (EA)

```
EA    = Intake − Exercise extra
Floor = Fat-free mass × 30 kcal   (fat-free mass = weight × (1 − body fat % / 100))
```

Staying below 30 kcal per kg of fat-free mass for long periods is associated with effects on hormones, bones and immunity (relative energy deficiency in sport, RED-S). To keep you above it even in a deficit, the page shows the minimum you still need to eat. Change the factor with `ea_kcal_per_kg_ffm`.

## Estimate adjustments

- Nuts are estimated about 15% below food-table values (20% for almonds on their own). Some of the fat is held inside cell walls and isn't absorbed, and studies have measured less metabolizable energy than labels show (Novotny et al., 2012; Baer et al., 2016).
