import pandas as pd
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import LabelEncoder

# Sample data with missing in both 'abc' and 'name'
df = pd.DataFrame({
    'abc': ['cat', 'dog', np.nan, 'cat', 'dog', np.nan, 'cat', np.nan],
    'name': ['A', 'B', 'A', np.nan, 'A', 'B', np.nan, np.nan]
})

# Drop rows where 'name' is missing
df_clean = df.dropna(subset=['name']).reset_index(drop=True)

# Label encode 'abc' and 'name'
le_abc = LabelEncoder()
le_name = LabelEncoder()

# Fit abc encoder only on non-missing abc values
abc_notnull = df_clean['abc'].dropna()
le_abc.fit(abc_notnull)

# Encode abc, keep NaN as np.nan
df_clean['abc_encoded'] = df_clean['abc'].map(lambda x: le_abc.transform([x])[0] if pd.notna(x) else np.nan)
df_clean['name_encoded'] = le_name.fit_transform(df_clean['name'])

# Train linear regression on rows where abc is not missing
train = df_clean[df_clean['abc_encoded'].notna()]
X_train = train[['name_encoded']]
y_train = train['abc_encoded']

model = LinearRegression()
model.fit(X_train, y_train)

# Predict abc for rows where abc is missing
missing_abc = df_clean['abc_encoded'].isna()
X_missing = df_clean.loc[missing_abc, ['name_encoded']]
preds = model.predict(X_missing)

# Round and convert predicted labels back to original categories
preds_rounded = np.round(preds).astype(int)
df_clean.loc[missing_abc, 'abc_encoded'] = preds_rounded

df_clean['abc_imputed'] = le_abc.inverse_transform(df_clean['abc_encoded'].astype(int))

print(df_clean[['abc', 'name', 'abc_imputed']])
