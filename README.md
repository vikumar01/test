'''
# === Save Logistic Regression Coefficients (if best model is LogisticRegression) ===
if best_model_name == "LogisticRegression":
    classifier = best_pipeline.named_steps["classifier"]
    preprocessor = best_pipeline.named_steps["preprocessor"]

    # Get feature names after preprocessing
    feature_names = []
    for name, transformer, cols in preprocessor.transformers_:
        if name == 'num':
            feature_names.extend(cols)
        elif name == 'cat':
            ohe = transformer
            feature_names.extend(ohe.get_feature_names_out(cols))

    # Coefficients for logistic regression
    coefs = classifier.coef_[0]
    coef_df = pd.DataFrame(
        pd.Series(coefs, index=feature_names).sort_values(ascending=False),
        columns=["Coefficient"]
    )

    coef_df.to_csv("logisticregressionresults.csv")
    print("\nLogistic Regression coefficients saved: logisticregressionresults.csv")
