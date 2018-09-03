[app_creation]: {{ site.baseurl }}/images/blog-2018-04-03.png

After working for a while, i think this might be a good opportunity to wrap up the general metrics used for evaluation of machine learning models. Because model evaluation is essential, different metrics may give different results.

1. Accuracy:  

This is the ratio of number of correct predictions to the total number of predictions made. The problem of accuracy arises when the dataset is imbalanced, because the cost of misclassification of minor class samples will be very high. So we should never use accuracy as the metric when target variable classes in the data are a majority of one class. 

For example, if the size of sample is 100, and 95 is A, and 5 is B, however, if our model is bad, and predict everything as A, we still get 95% accuracy although the model is bad.

2. Log loss: 

This is also known as logarithmic loss, which works by penalising the false classification and is often used for multi-class classification. Unlike accuracy, loss is not a percentage. It is a summation of the errors made for each example in training or validation sets. It’s calculated as below:

![app_creation]

N is the number of total number of samples
M is the total number of classes
y_ij is whether sample i is in class j or not
p_ij is the probability of sample i belonging to class j
The closer log loss is, the higher accuracy the model has. The bigger log loss is, the lower accuracy it has.

3. Confusion matrix: 

The confusion matrix is very intuitive metrics used for finding correctness and accuracy of the model. Summarily, true/false means the model predicts correctly/incorrectly, and positive/negative refers to the prediction is 1 or 0.

True positive (TP): true positives are cases when actual class of the data point is 1 and the predicted value is also 1.
True negative (TN): true negatives are cases when actual class is 0 and prediction is also 1.
False positive (FP): false positives are cases when actual is 0 and prediction is 1. 
False negative (FN): false negatives are cases when actual class is 1 and the prediction is 0.

When trying to evaluate it, depending on business needs and the context of problem, we should choose different criteria. And since true means prediction is correct, thus we should just minimise FN and FT. 

In cases like disease detection problem, since we want to capture all disease cases, we might end up with a classification that the patient is healthy while prediction is wrong (and with further examination, the patient will be ensured healthy.), therefore, in this case, we prefer to minimise FN. In cases like spam email detection,  it will be pretty bad if the model classifies an important email as spam, so making FN can be accepted, therefore, in such case, minimising FP is more important.

4. Precision:

Precision is the ability of a model to identify only the relevant objects. It is the percentage of correct positive predictions, so it’s calculated as: 

Precision = TP / (TP + FP)

5. Recall or Sensitivity:

Recall is the ability of a model to find all the relevant cases. It is the percentage of true positive detected among all relevant true cases:

Recall = TP / (TP + FN)

As we see from the definition of precision and recall, precision is more about being precise, but recall is more about capturing cases. 

6. Specificity:

Specificity is exactly opposite of recall:

Specificity = TN / (TN + FP)

7. Area under curve (AUC)

AUC is widely used for evaluation and generally used for binary classification problem. AUC of a classifier is equal to the probability that the classifier will rank a randomly chosen positive example higher than a randomly chosen negative example. And it’s composed of two rates: true positive rate (recall) and false positive rate (specificity).

TPR (y-axis) refers to the proportion of positive data points that are correctly considered as positive, with respect to all positive data points. while FPR (x-axis) corresponds to the proportion of negative data points that are mistakenly considered as positive, with respect to all negative data points.

And the range of value of AUC is [0, 1], and the bigger AUC, the better the model is.

8. F1 score

Literally, F1 score is the harmonic mean between precision and recall, and it kinds of trying to find balance between precision and recall. It tells how precise the classifier is as well as how robust it is (meaning that it does not miss a number of cases). If one number of precision or recall is really small, the F1 Score kind of raises a flag and is closer to the smaller number than the bigger one, giving the model an appropriate score rather than just an arithmetic mean. High precision but low recall will give a very good accuracy but will mean the model miss a large number of instances that are difficult to classify. 

F1 Score = 2 * Precision * Recall / (Precision + Recall)

Therefore, the greater the F1 score is, the better the model is.

9. Mean squared error (MSE) or Root mean squared error (RMSE)
The MSE takes the average of the square of the difference between original and predicted values, and it’s easier to compute the gradient. Since it takes square, the effect of large errors will become more pronounced, and the model should focus on large errors. 

And RMSE is the root of MSE.
