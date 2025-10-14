Data & assumptions

I'll assume your transactions table (or CSV) has at least:

from_acct (sending account)

to_acct (receiving account)

ts (timestamp or datetime of transfer)

amount (optional; useful to require similar amounts)

You have 6 months of data.

2) High-level plan

Normalize timestamps (floor to date, week, month depending on frequency).

Group by (from_acct, to_acct) and count distinct time-buckets (days/weeks/months).

Compare counts to total number of buckets in the period to detect strict recurrence (e.g., sent on every day / week / month) or use proportion for fuzzy recurrence (e.g., ≥ 80% of weeks).

Optionally filter by amount stability (similar amounts) to strengthen matching.

Visualize with a heatmap/calendar for human validation.

3) SQL queries (examples)

Assume table name transactions(from_acct, to_acct, ts, amount) and ts is a timestamptz / datetime.

a) Helper: total number of distinct days/weeks/months in the dataset
-- Date range
SELECT 
  MIN(date_trunc('day', ts))::date AS start_date,
  MAX(date_trunc('day', ts))::date AS end_date,
  (MAX(date_trunc('day', ts))::date - MIN(date_trunc('day', ts))::date) + 1 AS total_days,
  COUNT(DISTINCT date_trunc('week', ts)) AS total_weeks,
  COUNT(DISTINCT date_trunc('month', ts)) AS total_months
FROM transactions;

b) Pairs that have transfers on every day in the period
WITH buckets AS (
  SELECT DISTINCT date_trunc('day', ts)::date AS d
  FROM transactions
),
pair_days AS (
  SELECT from_acct, to_acct, COUNT(DISTINCT date_trunc('day', ts)::date) AS days_with_transfer
  FROM transactions
  GROUP BY from_acct, to_acct
),
tot AS (
  SELECT COUNT(*) AS total_days FROM buckets
)
SELECT p.from_acct, p.to_acct, p.days_with_transfer, t.total_days
FROM pair_days p CROSS JOIN tot t
WHERE p.days_with_transfer = t.total_days;

c) Pairs that have transfers in every week
WITH pair_weeks AS (
  SELECT from_acct, to_acct, COUNT(DISTINCT date_trunc('week', ts)) AS weeks_with_transfer
  FROM transactions
  GROUP BY from_acct, to_acct
), tot AS (
  SELECT COUNT(DISTINCT date_trunc('week', ts)) AS total_weeks FROM transactions
)
SELECT p.from_acct, p.to_acct, p.weeks_with_transfer, t.total_weeks
FROM pair_weeks p CROSS JOIN tot t
WHERE p.weeks_with_transfer = t.total_weeks;

d) Fuzzy recurrence (e.g., appears in at least 80% of weeks)
WITH pair_weeks AS (
  SELECT from_acct, to_acct, COUNT(DISTINCT date_trunc('week', ts)) AS weeks_with_transfer
  FROM transactions
  GROUP BY from_acct, to_acct
), tot AS (
  SELECT COUNT(DISTINCT date_trunc('week', ts)) AS total_weeks FROM transactions
)
SELECT p.from_acct, p.to_acct, p.weeks_with_transfer, t.total_weeks,
       ROUND(100.0 * p.weeks_with_transfer / t.total_weeks, 2) AS percent_weeks
FROM pair_weeks p CROSS JOIN tot t
WHERE (p.weeks_with_transfer::numeric / t.total_weeks) >= 0.80
ORDER BY percent_weeks DESC;

e) Require similar amount each time (optional)
-- e.g., coefficient of variation threshold per pair
WITH stats AS (
  SELECT from_acct, to_acct,
         COUNT(*) AS n,
         AVG(amount) AS avg_amt,
         STDDEV(amount) AS std_amt
  FROM transactions
  GROUP BY from_acct, to_acct
)
SELECT from_acct, to_acct, n, avg_amt, std_amt, (std_amt/NULLIF(avg_amt,0)) AS cv
FROM stats
WHERE (std_amt/NULLIF(avg_amt,0)) < 0.10; -- low variation → likely recurring fixed amount

4) Python / pandas approach (recommended for exploratory work & visualization)

Below is example code — you can run it on your CSV or database export.

import pandas as pd
import numpy as np

# load
df = pd.read_csv('transactions.csv', parse_dates=['ts'])
# ensure columns: from_acct, to_acct, ts, amount

# add date/week/month columns
df['date'] = df['ts'].dt.date
df['week'] = df['ts'].dt.to_period('W').apply(lambda r: r.start_time.date())
df['month'] = df['ts'].dt.to_period('M').apply(lambda r: r.start_time.date())

# helper to compute recurrence for a bucket column
def recurrence_by_bucket(df, bucket_col, freq_name):
    # total number of distinct buckets in the whole dataset
    total_buckets = df[bucket_col].nunique()
    g = (df.groupby(['from_acct','to_acct'])[bucket_col]
           .nunique()
           .reset_index(name='buckets_with_transfer'))
    g['total_buckets'] = total_buckets
    g['pct'] = g['buckets_with_transfer'] / total_buckets
    g = g.sort_values(['pct','buckets_with_transfer'], ascending=False)
    return g

daily = recurrence_by_bucket(df, 'date', 'daily')
weekly = recurrence_by_bucket(df, 'week', 'weekly')
monthly = recurrence_by_bucket(df, 'month', 'monthly')

# examples: strict daily recurrence
strict_daily = daily[daily['buckets_with_transfer'] == daily['total_buckets']]

# fuzzy example: >= 80% of weeks
fuzzy_weekly = weekly[weekly['pct'] >= 0.8]

# show top candidates
print("Top daily candidates:\n", daily.head(10))
print("Top weekly candidates (>=80%):\n", fuzzy_weekly.head(20))

Visual check: calendar heatmap per pair

You can pivot per pair into a matrix of buckets (rows=pair, cols=bucket) and visualize as an image/heatmap; or for a single pair show a calendar-style heatmap.

# example for one pair
pair = ('acctA','acctB')
mask = (df['from_acct']==pair[0]) & (df['to_acct']==pair[1])
s = df[mask].copy()
s['date'] = pd.to_datetime(s['date'])
daily_index = pd.date_range(s['date'].min(), s['date'].max(), freq='D')
presence = s.groupby('date').size().reindex(daily_index, fill_value=0)
# plot
import matplotlib.pyplot as plt
plt.figure(figsize=(12,2))
plt.plot(daily_index, presence.clip(0,1))  # 1 if any transfer that day
plt.title(f'Presence by day for pair {pair}')
plt.ylabel('Transfer present (0/1)')
plt.show()


(If you want nicer calendar heatmaps I can add code to make them.)

5) Advanced checks / periodicity detection

If you want to detect regular periodicity (e.g., every 7 days but not always same weekday), use:

Autocorrelation of the binary time series per pair (presence/absence). Peaks at lag 7, 14 indicate weekly periodicity.

Fourier transform (FFT) on presence series to find dominant periods.

Sequence mining (Apriori) if you want patterns like “if A→B happens, then B→C happens two days later” (more complex).

Here's a short example of autocorrelation for one pair:

from statsmodels.tsa.stattools import acf
presence_vals = (presence > 0).astype(int).values
ac = acf(presence_vals, nlags=60, fft=True)
# check if ac[7] is high → weekly periodicity

6) Practical considerations & thresholds

Strict recurrence (every day/week/month) is rare; prefer thresholds (≥70–90%) depending on your tolerance.

Time zone / daylight: make sure timestamps use consistent timezone.

Multiple transfers on same day: collapse to presence (binary) or use counts if frequency matters.

Amount stability: requiring amounts similar (CV < 0.1) reduces false positives (e.g., payroll or subscriptions).

Business days vs calendar days: if transfers happen only on weekdays, treat business-day buckets.

Gaps at start/end: consider only full buckets (e.g., exclude the first/last partial month) or count only weeks fully inside the 6 months.

7) Output you might want

Table: (from_acct, to_acct, bucket_type, buckets_with_transfer, total_buckets, pct, avg_amount, cv_amount)

Flag: recurring_daily, recurring_weekly, recurring_monthly (true/false by chosen thresholds)

Visualization per suspicious pair (calendar heatmap + amount time series)

-- Date range
SELECT 
  MIN(date_trunc('day', ts))::date AS start_date,
  MAX(date_trunc('day', ts))::date AS end_date,
  (MAX(date_trunc('day', ts))::date - MIN(date_trunc('day', ts))::date) + 1 AS total_days,
  COUNT(DISTINCT date_trunc('week', ts)) AS total_weeks,
  COUNT(DISTINCT date_trunc('month', ts)) AS total_months
FROM transactions;

