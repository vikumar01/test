✅ 1️⃣ Load the data
import pandas as pd

# Load your file (change filename/path and sheet name if needed)
df = pd.read_excel("transactions.xlsx")  # or pd.read_csv("transactions.csv")

# Ensure column names match exactly
df.columns = [col.strip().lower() for col in df.columns]

# Rename to standard names if needed
df = df.rename(columns={
    'accountnumber': 'account',
    'correspondentaccountnumber': 'corr_account',
    'valuedate': 'valuedate'
})

✅ 2️⃣ Convert date column and add time buckets
df['valuedate'] = pd.to_datetime(df['valuedate'])

df['date'] = df['valuedate'].dt.date                    # Day-level
df['week'] = df['valuedate'].dt.to_period('W').apply(lambda x: x.start_time.date())
df['month'] = df['valuedate'].dt.to_period('M').apply(lambda x: x.start_time.date())

✅ 3️⃣ Get total distinct time buckets in the entire data
total_days = df['date'].nunique()
total_weeks = df['week'].nunique()
total_months = df['month'].nunique()

✅ 4️⃣ Group by account pairs and compute distinct buckets
daily_counts = (
    df.groupby(['account', 'corr_account'])['date']
      .nunique()
      .reset_index(name='days_with_transfer')
)

weekly_counts = (
    df.groupby(['account', 'corr_account'])['week']
      .nunique()
      .reset_index(name='weeks_with_transfer')
)

monthly_counts = (
    df.groupby(['account', 'corr_account'])['month']
      .nunique()
      .reset_index(name='months_with_transfer')
)

✅ 5️⃣ Merge results and calculate percentages
final_df = daily_counts.merge(weekly_counts, on=['account', 'corr_account'], how='outer') \
                       .merge(monthly_counts, on=['account', 'corr_account'], how='outer')

final_df['days_with_transfer'] = final_df['days_with_transfer'].fillna(0)
final_df['weeks_with_transfer'] = final_df['weeks_with_transfer'].fillna(0)
final_df['months_with_transfer'] = final_df['months_with_transfer'].fillna(0)

final_df['total_days'] = total_days
final_df['total_weeks'] = total_weeks
final_df['total_months'] = total_months

final_df['pct_daily'] = final_df['days_with_transfer'] / total_days * 100
final_df['pct_weekly'] = final_df['weeks_with_transfer'] / total_weeks * 100
final_df['pct_monthly'] = final_df['months_with_transfer'] / total_months * 100

✅ 6️⃣ View the top recurring pairs
print(final_df.head(20))

# Example: pairs with 100% daily recurrence
daily_pairs = final_df[final_df['pct_daily'] == 100]

# Example: pairs with 100% weekly recurrence
weekly_pairs = final_df[final_df['pct_weekly'] == 100]

# Example: pairs with ≥80% monthly recurrence
monthly_pairs = final_df[final_df['pct_monthly'] >= 80]

print("Daily:", daily_pairs)
print("Weekly:", weekly_pairs)
print("Monthly (80%+):", monthly_pairs)

✅ 7️⃣ Export the result (optional)
final_df.to_excel("recurrence_patterns.xlsx", index=False)
