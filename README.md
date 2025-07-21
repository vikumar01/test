model_configs = {
    "LogisticRegression": {
        "model": LogisticRegression(max_iter=1000),
        "params": {
            "classifier__C": [0.01, 0.1, 1.0, 10.0]
        }
    },
    "RandomForest": {
        "model": RandomForestClassifier(random_state=42),
        "params": {
            "classifier__n_estimators": [100, 150],
            "classifier__max_depth": [5, 10, None]
        }
    },
    "SVM": {
        "model": SVC(probability=True),
        "params": {
            "classifier__C": [0.1, 1.0, 10],
            "classifier__kernel": ["rbf", "linear"]
        }
    },
    "XGBoost": {
        "model": xgb.XGBClassifier(use_label_encoder=False, eval_metric="logloss", random_state=42),
        "params": {
            "classifier__n_estimators": [100, 150],
            "classifier__max_depth": [3, 6],
            "classifier__learning_rate": [0.05, 0.1]
        }
    }
}

# === 4
