from sklearn.metrics import f1_score
import numpy as np

f1_scores = 2 * (precision * recall) / (precision + recall)
best_index = np.argmax(f1_scores)
best_threshold = thresholds[best_index]

print("Best threshold:", best_threshold)
print("Best F1 score:", f1_scores[best_index])
