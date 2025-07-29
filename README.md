'''

import numpy as np
import pandas as pd
from sklearn.base import BaseEstimator, TransformerMixin

class MeanTargetEncoder(BaseEstimator, TransformerMixin):
    def __init__(self, cols):
        self.cols = cols
        self.global_mean = None
        self.mapping = {}

    def fit(self, X, y):
        Xy = X.copy()
        Xy["_target"] = y
        self.global_mean = y.mean()
        for col in self.cols:
            # Compute mean target per category
            self.mapping[col] = Xy.groupby(col)["_target"].mean().to_dict()
        return self

    def transform(self, X):
        X_ = X.copy()
        for col in self.cols:
            # Map categories to mean target value, fill unknown with global mean
            X_[col] = X_[col].map(self.mapping[col]).fillna(self.global_mean)
        return X_

numeric_cols = ["price", "volume"]
target_encode_cols = ["sector"]   # columns for target encoding
one_hot_cols = ["region"]

preprocessor = ColumnTransformer([
    ("num", StandardScaler(), numeric_cols),
    ("ohe", OneHotEncoder(handle_unknown="ignore"), one_hot_cols),
    ("target", MeanTargetEncoder(cols=target_encode_cols), target_encode_cols)
])

pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("classifier", XGBClassifier(use_label_encoder=False, eval_metric="logloss"))
])

# Fit model (target encoding computed here using train target)
pipeline.fit(X_train, y_train)

# Predict on test (target encoding applied using learned mapping)
y_pred = pipeline.predict(X_test)

'''
