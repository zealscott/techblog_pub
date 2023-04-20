+++
title = "基于IMDb数据集的情感分析(机器学习实现)"
slug = "imdb1"
tags = ["Coding"]
date = "2018-05-27T19:33:55+08:00"
description = "使用TF-IDF模型，结合机器学习进行情感分类，能取得较好的准确率"

+++



本文的JupyterNotebook可[参考这里](https://github.com/Scottdyt/MyProject/blob/master/SentimentAnalysis/jupyternotebook/TF-IDF%20with%20ML.ipynb)。

本文介绍NLP的通用方法`TF-IDF`的使用，并且分类准确率能达到`0.95`，进入kaggle排行榜的前100。

# TF-IDF

`TF-IDF（词频-逆文档频率）`算法是一种统计方法，用以评估一字词对于一个文件集或一个语料库中的其中一份文件的重要程度。

**TFIDF的主要思想是：**如果某个词或短语在一篇文章中出现的频率TF高，并且在其他文章中很少出现，则认为此词或者短语具有很好的类别区分能力，适合用来分类。

其计算方法比较简单，这里就不赘述了。本文使用`sklearn`中的`TfidfVectorizer `进行处理。

```python
from sklearn.feature_extraction.text import TfidfVectorizer as TFIV
tfv = TFIV(min_df=3,  max_features=None, 
        strip_accents='unicode', analyzer='word',token_pattern=r'\w{1,}',
        ngram_range=(1, 2), use_idf=1,smooth_idf=1,sublinear_tf=1,
        stop_words = 'english')
```

这里可以设置的参数为`n-gram`，本文经过试验，当n为3或者4时表现良好。

数据清理和预处理的步骤与上一篇博文相似，再此不赘述。

将数据带入进行训练：

```python
X_all = traindata + testdata # Combine both to fit the TFIDF vectorization.
lentrain = len(traindata)

tfv.fit(X_all) # This is the slow part!
X_all = tfv.transform(X_all)

X = X_all[:lentrain] # Separate back into training and test sets. 
X_test = X_all[lentrain:]
```



# 机器学习

## Logistic Regression

首先使用简单的逻辑回归看看向量化的结果如何，这里使用了`GridSearchCV`进行超参数选择：

```python
grid_values = {'C':[30]} # Decide which settings you want for the grid search. 

model_LR = GridSearchCV(LR(penalty = 'L2', dual = True, random_state = 0), 
                        grid_values, scoring = 'roc_auc', cv = 20) 
# Try to set the scoring on what the contest is asking for. 
# The contest says scoring is for area under the ROC curve, so use this.
                        
model_LR.fit(X,y_train) # Fit the model.
```

使用`model_LR.grid_scores_`打印训练集上的结果，出乎意料的是，在训练集上的准确率达到了0.96459！

## MultinomialNB

使用朴素贝叶斯进行训练：

```python
from  sklearn.naive_bayessklearn.  import MultinomialNB as MNB
from sklearn.cross_validation import cross_val_score
import numpy as np
model_NB = MNB()
model_NB.fit(X, y_train)
print "20 Fold CV Score for Multinomial Naive Bayes: ", np.mean(cross_val_score                                                                (model_NB, X, y_train, cv=20, scoring='roc_auc'))
     # This will give us a 20-fold cross validation score that looks at ROC_AUC so we can compare with Logistic Regression.
```

使用20折交叉验证，在训练集上的准确率达到了0.94963。

## SGD 

使用SGD进行训练，该模型使用于大数据集，考虑到我们使用n=3 的TF-IDF模型时，向量维数已经达到了309798，因此此方法也许能更快的找到极值点：

```python
from  sklearn.linear_modelsklearn.  import SGDClassifier as SGD
sgd_params = {'alpha': [0.00006, 0.00007, 0.00008, 0.0001, 0.0005]} # Regularization parameter

model_SGD = GridSearchCV(SGD(random_state = 0, shuffle = True, loss = 'modified_huber'), 
                        sgd_params, scoring = 'roc_auc', cv = 20) # Find out which regularization parameter works the best. 
                        
model_SGD.fit(X, y_train) # Fit the model.
```

训练集上的准确率已经达到了0.96477！对于如此简单的分类算法，这已经相当不错了。说明使用`TF-IDF`模型能很好的模拟数据集，而更高大上的`Doc2Vec`或者`Word2Vec`则拟合较差。



