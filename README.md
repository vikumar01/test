'''

class MeanTargetEncoder(BaseEstimator, TransformerMixin):
    def __init__(self):
        self.global_mean = None
        self.mapping = {}

    def fit(self, X, y):
        X = X.copy()
        self.global_mean = y.mean()
        for col in X.columns:
            self.mapping[col] = X.groupby(col).apply(lambda g: y[g.index].mean()).to_dict()
        return self

    def transform(self, X):
        X_ = X.copy()
        for col in X.columns:
            X_[col] = X_[col].map(self.mapping[col]).fillna(self.global_mean)
        return X_
'''
