'''
# === Feature Importance Utility ===
def plot_feature_importance(model, X, pipeline):
    # Get feature names after preprocessing
    cat_features = pipeline.named_steps['preprocessor'].transformers_[1][1].get_feature_names_out(cat_cols)
    feature_names = np.concatenate([num_cols, cat_features])
    
    if hasattr(model, "feature_importances_"):
        importances = model.feature_importances_
    elif hasattr(model, "coef_"):
        importances = np.abs(model.coef_).flatten()
    else:
        print("⚠️ Feature importance not available for this model.")
        return

    feature_importance_df = pd.DataFrame({
        "Feature": feature_names,
        "Importance": importances
    }).sort_values("Importance", ascending=False)

    # Plot top 15
    plt.figure(figsize=(8, 6))
    sns.barplot(x="Importance", y="Feature", data=feature_importance_df.head(15), palette="viridis")
    plt.title("Top 15 Feature Importances")
    plt.tight_layout()
    plt.savefig("feature_importance.png")
    plt.show()

    print("\nTop 10 Important Features:")
    print(feature_importance_df.head(10))
    '''
