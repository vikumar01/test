from sklearn.metrics import roc_curve

fpr, tpr, roc_thresholds = roc_curve(y_val, y_probs)
j_scores = tpr - fpr
j_best_index = np.argmax(j_scores)
j_best_threshold = roc_thresholds[j_best_index]

print("Threshold using Youden’s J:", j_best_threshold)


---
y_pred_new = (y_probs >= best_threshold).astype(int)

from sklearn.metrics import classification_report
print(classification_report(y_val, y_pred_new))
