# 1. Deduplicate dfB — keep first occurrence (change to 'last' if needed)
df_b_dedup = df_b.drop_duplicates(subset='Y', keep='first')

# 2. Merge to replace X
df_a = df_a.merge(df_b_dedup, left_on='X', right_on='Y', how='left')

# 3. Replace X with Z where available
df_a['X'] = df_a['Z'].fillna(df_a['X'])

# 4. Drop helper columns
df_a = df_a.drop(columns=['Y', 'Z'])

print(df_a)
