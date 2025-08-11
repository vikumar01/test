'''
import pandas as pd
from rapidfuzz import process, fuzz

# Example messy data
df = pd.DataFrame({
    'office': [
        'New York Office',
        'NY Office',
        'NY',
        'N.Y.',
        'NYC Branch',
        'London Office',
        'LDN Headquarters',
        'Londn',
        'Ldn'
    ]
})

# Step 1: Lowercase for matching
df['office_lower'] = df['office'].str.lower()

# Step 2: Get unique names
unique_names = df['office_lower'].unique()

# Step 3: Build fuzzy match groups
name_map = {}
processed = set()

for name in unique_names:
    if name in processed:
        continue
    # Find similar names (score >= 80)
    matches = [n for n, score, _ in process.extract(name, unique_names, scorer=fuzz.token_sort_ratio) if score >= 80]
    processed.update(matches)
    # Pick most frequent representative from matches
    representative = df[df['office_lower'].isin(matches)]['office'].mode()[0]
    for m in matches:
        name_map[m] = representative

# Step 4: Apply mapping
df['office_standard'] = df['office_lower'].map(name_map)

# Step 5: Drop helper column
df = df.drop(columns=['office_lower'])

print(df)

'''

