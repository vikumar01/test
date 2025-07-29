'''
import pandas as pd
import joblib

# === 1. Load the saved model ===
model = joblib.load("best_model.pkl")   # path to saved model

# === 2. Load new testing dataset ===
test_df = pd.read_csv("new_test_data.csv")  # path to test csv

# Ensure target column is not included (only features)
# If target exists, drop it
if "execute_trade" in test_df.columns:
    X_test_new = test_df.drop(columns=["execute_trade"])
    y_test_new = test_df["execute_trade"]  # optional, only if you want evaluation
else:
    X_test_new = test_df
    y_test_new = None

# === 3. Make predictions ===
predictions = model.predict(X_test_new)
probabilities = model.predict_proba(X_test_new)[:, 1]  # probability of class 1

# === 4. Show results ===
test_df["Predicted_Label"] = predictions
test_df["Predicted_Prob"] = probabilities

# Save to CSV
test_df.to_csv("predicted_output.csv", index=False)
print("Predictions saved to predicted_output.csv")

# === 5. Optional: If ground truth exists, evaluate ===
if y_test_new is not None:
    from sklearn.metrics import classification_report, recall_score, roc_auc_score
    
    print("\nClassification Report:")
    print(classification_report(y_test_new, predictions))
    print("Recall:", recall_score(y_test_new, predictions))
    print("ROC AUC:", roc_auc_score(y_test_new, probabilities))
    ''' 
