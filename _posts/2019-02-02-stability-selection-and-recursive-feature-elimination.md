Here I would like to introduce two common methods that are used in feature selection in machine learning: stability selection and recursive feature elimination (RFE), and both can be considered as wrapper methods. They build on top of other model based selection methods such as regression or SVM, extracting rankings from the aggregations.

**Stability Selection:**

Stability selection is a relatively novel method for feature selection, based on subsampling in combination with selection algorithms (which could be regression, SVMs or other similar method). The high level idea is to apply a feature selection algorithm on different subsets of data and with different subsets of features. After repeating the process a number of times, the selection results can be aggregated, for example by checking how many times a feature ended up being selected as important when it was in an inspected feature subset. We can expect strong features to have scores close to 100%, since they are always selected when possible. Weaker, but still relevant features will also have non-zero scores, since they would be selected when stronger features are not present in the currently selected subset, while irrelevant features would have scores (close to) zero, since they would never be among selected features.

And scikit-learn has implemented stablility selection in RandomizedLasso and RandomizedLogisticRegression. e.g.:

```python
from sklearn.linear_model import RandomizedLasso

rlasso = RandomizedLasso(alpha=0.025)
rlasso.fit(X_train, y_train)

print "Features sorted by their score:"
print sorted(zip(map(lambda x: round(x, 4), rlasso.scores_), 
                 names), reverse=True)
```
X_train and y_train are your dataset. And if features have score of 1.0, which means they are always selected as useful features. As you can see in the use of RandomizedLasso, a regularization paremter alpha is in place, and score could change when this regularization parameter is changed. In general, the score will drop smoothly compared with pure lasso or random forest. Therefore, it could be useful for both pure feature selection to reduce overfitting.

**Recursive Feature Elimination:**

Recursive feature elimination is based on the idea to repeatedly construct a model and choose either the best or worst performing features (e.g. based on coefficients), setting the feature aside, and then repeating the process with the rest of features. This process keeps going until all features in the dataset are exhausted. Features are ranked according to when they are eliminated. So it is kind of a greedy optimization for finding the best performing subset of features.

The stability of RFE depends heavily on the type of model that is used for feature ranking at each iteration. Just as non-regularized regression can be unstable, so can RFE when utilizing it, while using ridge regression can provide more stable results.

```python
from sklearn.feature_selection import RFE
from sklearn.linear_model import LinearRegression

# use linear regression as the model
lr = LinearRegression()
# continue the elimination until the last one
rfe = RFE(lr, n_features_to_select=1)
rfe.fit(X_train,y_train)

print "Features sorted by their rank:"
print sorted(zip(map(lambda x: round(x, 4), rfe.ranking_), names))
```

And alternatively, there is a very powerful package developed by scikit-learn which is call BorutaPy, this is Python implementation of Boruta R package.
```python
from boruta import BorutaPy
from sklearn.ensemble import RandomForestClassifier

rfc = RandomForestClassifier(n_estimators=200, n_jobs=4, class_weight='balanced', max_depth=6)
boruta_selector = BorutaPy(rfc, n_estimators='auto', verbose=2)

boruta_selector.fit(X, y)
```

And the subset of feature will be selected. You can then select those columns and then only train on them instead of whole features, like:
```python
selected_cols = X_train.columns[boruta_selector.support_]
```



