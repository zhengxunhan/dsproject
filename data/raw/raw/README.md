# Hop In - Dataset

**Hop In** is a (fictional) ride-sharing app in Charlottesville.
You have every ride request from **Sep 1, 2025 to Feb 28, 2026**.

> Do not edit anything in `data/raw/`. Your scripts read from here and save results to `data/processed/`.

## Files

| File | Rows (approx.) | One row is... |
|---|---|---|
| `rides.csv` | ~15,000 | one ride request |
| `drivers.csv` | ~300 | one driver |
| `zones.csv` | 20 | one area of town |

`rides.driver_id` links to `drivers.driver_id`. `rides.pickup_zone` links to `zones.zone_id`
(merging once on `pickup_zone` is enough for the main questions; `dropoff_zone` also links to `zones.zone_id` if you want it).

## Data dictionary

### rides.csv

| Column | Description |
|---|---|
| `ride_id` | Ride ID |
| `driver_id` | Driver who took the request |
| `request_time` | When the rider asked for a ride |
| `pickup_zone` | Where the ride started |
| `dropoff_zone` | Where the ride ended |
| `ride_type` | `Standard`, `XL` (bigger car), or `Shared` (shared with other riders) |
| `distance_miles` | Trip distance (miles). All rides are inside Charlottesville - the longest real trip is under 20 miles |
| `duration_min` | Trip time (minutes) |
| `eta_min` | Wait time the app promised (minutes) |
| `wait_min` | Real wait until the driver arrived (minutes) |
| `surge` | Surge pricing multiplier (1.0 = normal price, 1.5 = 50% more) |
| `weather` | Weather at the time |
| `fare` | Price the rider paid, before tip (USD) |
| `tip` | Tip (USD) |
| `rating` | Rider's rating of the ride, 1-5 |
| `lost_item` | Did the rider report leaving something in the car? |
| `status` | `Completed` or `Cancelled` (cancelled rides pay a $5 fee) |
| `arrived_late` | Did the driver arrive 5+ minutes later than promised? |

### drivers.csv

| Column | Description |
|---|---|
| `driver_id` | Driver ID |
| `join_date` | Date the driver joined |
| `vehicle_year` | Car model year |
| `vehicle_make` | Car brand |
| `vehicle_model` | Car model |
| `vehicle_type` | Sedan, SUV, or Minivan |
| `is_full_time` | Drives full time? |
| `hourly_earnings` | Average earnings per hour this period (USD) |

### zones.csv

| Column | Description |
|---|---|
| `zone_id` | Zone ID |
| `zone_name` | Area name |
| `zone_type` | Campus, Downtown, Shopping, Residential, or Airport |

Home football games this season: Sep 6, Sep 13, Oct 4, Oct 18, Nov 1, Nov 15, Nov 22. Winter break: Dec 15 - Jan 13.

## Finding and fixing the 8 problems

There are **exactly 8 problems** hidden in this data, one of each kind below.
The table tells you *how to spot* each kind and *what to look up* to fix it - not where it is. Finding it is your job.

| # | Kind of problem | How to spot it | What to look up to fix it |
|---|---|---|---|
| 1 | A number stored as text (e.g. `"$4.50"`, `"20 min"`) | `df.info()` says `object` for a column that should be a number; `df.head()` shows a symbol or unit | `.str.replace()`, then `pd.to_numeric()` |
| 2 | Labels with inconsistent capitalization or extra spaces | `df[col].value_counts()` shows the same label several times (`Cafe`, `cafe`, `CAFE `) | `.str.strip()`, `.str.title()` |
| 3 | Duplicate rows | `df.duplicated().sum()` is not 0 | `df.drop_duplicates()` |
| 4 | Missing values that *mean something* | `df.isna().sum()`; then ask *why* it is missing (read the data dictionary!) | Usually **keep** them and explain why. Only drop / fill when your question needs it |
| 5 | Dates written in different formats | `df[col].head(20)` shows different styles (`2025-09-03`, `09/03/2025`, ...) | `pd.to_datetime(df[col], format="mixed")` |
| 6 | IDs that don't match across tables | after a merge, rows disappear (or columns are all NaN); `value_counts()` on the ID shows different styles | `.str.upper()`, `.str.replace()`, `.str.zfill()` - fix **before** merging |
| 7 | The same thing written in different ways (e.g. `Y` / `yes` / `1`) | `df[col].value_counts()` or `df[col].unique()` | write a dictionary and use `df[col].map(my_dict)` |
| 8 | Values that are impossible | `df.describe()` - look at `min` and `max`. Compare with the limits written in the data dictionary | decide a rule (e.g. "drop rows above X") and filter: `df = df[df[col] <= X]` - explain your rule |

**Tips**

- Check **every** column with `value_counts()` (text columns) or `describe()` (number columns). Problems hide in boring columns too.
- After every fix, check that it worked (run `info()` / `value_counts()` / `describe()` again).
- Write down each problem in the issues table in `main/step1_prepare.py`: column, problem, how you fixed it, **why**.
- Never overwrite the files in `data/raw/`. `step1_prepare.py` saves ONE cleaned, merged file: `df.to_csv("../data/processed/clean.csv", index=False)`. Run the scripts from inside `main/`.

## Step-by-step checklist

Work through the three starter files in `main/` in order: `step1_prepare.py` (sections 1-4) -> `step2_eda.py` (section 5) -> `step3_model.py` (sections 6-10). Tick each box before you move on. Your TA's check-in sheet has the full, detailed version.

**1. Problem definition**
- [ ] Pick ONE main question (see "Question ideas" below - you may also make your own).
- [ ] Write down your target `y`: which column? Is it a category (-> classification) or a number (-> regression)?

**2. Load and first look**
- [ ] Load **every** file with `pd.read_csv("../data/raw/<file>.csv")`. Print `.shape`, `.head()`, `.info()`, `.describe()` for each.
- [ ] Say in words what one row is in each file (see the Files table above).

**3. Cleaning**
- [ ] Find all 8 problems (table above) and fill in the issues table in the template.
- [ ] Fix them. Re-check after each fix.
- [ ] Clean the IDs **before** you merge.

**4. Merge and summarize**
- [ ] Merge the tables using the keys written under "Files" above.
- [ ] Print the number of rows **before and after** each merge. With `how="left"` it should stay the same. If it changes, something is wrong (duplicates in the small table? IDs not cleaned?).
- [ ] Check for new NaN columns after the merge (`isna().sum()`): many NaN = the IDs did not match.
- [ ] Use `groupby(...)[...].mean()` / `.count()` to answer at least one sub-question.
- [ ] Save `../data/processed/clean.csv` at the end of step1. step2 and step3 read it with `pd.read_csv(..., parse_dates=[<date columns>])`.

**5. EDA**
- [ ] At least 3 plots of different types, each with a title, axis labels, and one sentence of insight.
- [ ] Make new columns you need, e.g. `df["hour"] = df["time_col"].dt.hour`, `df["month"] = df["date_col"].dt.month`,
      `df["weekday"] = df["date_col"].dt.dayofweek` (0 = Monday), or a difference of two dates: `(df["b"] - df["a"]).dt.days`.

**6. Modeling**
- [ ] Choose features `X` that you would know **before** the outcome happens.
- [ ] Turn text columns into numbers with `pd.get_dummies(...)`. Drop or fill NaN in the columns you use.
- [ ] Classification: report accuracy **and compare it with the baseline** (always guessing the most common class:
      `y_test.value_counts(normalize=True).max()`). Also report recall if one class is rare.
- [ ] Regression: report R^2 and MAE, and explain MAE in real units ("we are off by about 4 minutes on average").

**7-8. Conclusion and limitations**
- [ ] Answer your main question in 3-5 sentences, using numbers from your analysis.
- [ ] Name at least one limitation (missing data, rows you dropped, things the data can't tell you).

**Common mistakes**

- Merging before cleaning the IDs -> rows silently disappear or columns become NaN.
- Forgetting `format="mixed"` -> error, or wrong dates.
- Dropping every row with a NaN (`dropna()` on the whole table) -> you can lose most of your data. Only drop NaN in the columns you use.
- Using a column that *is* (or gives away) the answer as a feature -> "99% accuracy" that means nothing.
- Reporting accuracy without the baseline.
- Plots with no title / labels, or a plot with no sentence explaining it.

## Question ideas

You may use these or make your own. These are *starting points* - the answers are for you to find.

**Explore (Q1) - questions to answer with groupby and plots**

- When and where is surge pricing highest (hour, weekday, weather, game days)?
- When does the driver arrive later than the app promised? What happens to cancellations and ratings then?
- When do riders leave things in the car?
- Do full-time drivers earn more per hour? (drivers.csv)

**Predict (Q2) - possible targets (`y`)**

| y | Type | Notes |
|---|---|---|
| `surge` | regression | Try hour, weekday, weather, game day, pickup zone type (merge zones). |
| `arrived_late` | classification | Try weather, surge, hour, game day, pickup zone type, eta_min. |
| `fare` | regression | Completed rides. Try distance, duration, surge, ride type. |
| `lost_item` | classification | Harder: very few rides have a lost item, so look at recall, not just accuracy. |

Every row you predict should be one row of the table (after merging). You do **not** need to build a new
table with one row per customer / player / driver.
