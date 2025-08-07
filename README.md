from sklearn.linear_model import LinearRegression

not_null = df[df[target_col].notnull()]
X_train = not_null[top_features]
y_train = not_null[target_col]

model = LinearRegression()
model.fit(X_train, y_train)

missing = df[df[target_col].isnull()]
df.loc[df[target_col].isnull(), target_col] = model.predict(missing[top_features])
