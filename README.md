'''

import statsmodels.api as sm
from statsmodels.stats.outliers_influence import variance_inflation_factor

# --- Transform training data using same pipeline (without classifier) ---
X_train_transformed = preprocessor.fit_transform(X_train, y_train)

# Get feature names after preprocessing
feature_names = []
for name, transformer, cols in preprocessor.transformers_:
    if name == "num":
        feature_names.extend(cols)
    elif name == "ohe":
        feature_names.extend(transformer.get_feature_names_out(cols))
    elif name == "target":
        feature_names.extend(cols)

# Convert to DataFrame
X_train_df = pd.DataFrame(X_train_transformed, columns=feature_names)

# --- Add constant for intercept ---
X_train_df_const = sm.add_constant(X_train_df)

# --- Fit statsmodels Logit for inference ---
logit_model = sm.Logit(y_train, X_train_df_const)
result = logit_model.fit(disp=False)

# --- Extract coefficients and inference metrics ---
summary_table = result.summary2().tables[1]
summary_table = summary_table.rename(columns={
    'Coef.': 'Coefficient',
    'Std.Err.': 'Std_Error',
    'P>|z|': 'P_value'
})

# --- Compute Odds Ratios ---
summary_table['Odds_Ratio'] = summary_table['Coefficient'].apply(lambda x: np.exp(x))

# --- Compute VIF ---
vif_data = pd.DataFrame()
vif_data['Feature'] = X_train_df.columns
vif_data['VIF'] = [variance_inflation_factor(X_train_df.values, i)
                   for i in range(X_train_df.shape[1])]

# Merge VIF into summary table
summary_with_vif = summary_table.merge(vif_data, left_index=True, right_on='Feature', how='left')

# Save to CSV
summary_with_vif.to_csv("logisticregression_detailed_results.csv", index=False)
print("\nDetailed Logistic Regression results saved: logisticregression_detailed_results.csv")
