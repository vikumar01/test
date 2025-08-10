# Save the mapping to decode later
abc_mapping = dict(enumerate(df['abc'].astype('category').cat.categories))

# Split data into known and missing target
not_null = df[df['abc'].notnull()]
missing = df[df['abc'].isnull()]

# Prepare training data
X_train = not_null[['name_encoded']]
y_train = not_null['abc_encoded']

# Train classifier
model = RandomForestClassifier(random_state=42)
model.fit(X_train, y_train)

# Predict for missing rows
X_missing = missing[['name_encoded']]
preds = model.predict(X_missing)

# Fill in the missing 'abc' values with decoded predictions
df.loc[df['abc'].isnull(), 'abc'] = [abc_mapping[p] for p in preds]
