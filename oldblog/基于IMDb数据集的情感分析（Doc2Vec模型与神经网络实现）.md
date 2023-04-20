+++
title = "基于 IMDb 数据集的情感分析（Doc2Vec）"
slug = "imdb2"
tags = ["Coding"]
date = "2018-05-27T14:46:57+08:00"
description = "使用Doc2Vec模型参加Kaggle的NLP比赛，最终score达到0.97，前1%"

+++




本文所有的代码都可以在[我的github上找到](https://github.com/Scottdyt/MyProject/tree/master/SentimentAnalysis)。

在上一篇[博文](../imdb1)中，我们使用了`TF-IDF`，准确率达到了`0.95`，已经进入前100，但还不够，我们试试使用更加高大上的`Doc2Vec`结合神经网络模型，其准确率能否再次提升。

# 数据介绍

- 本数据来源于[IMDB电影评论集](http://ai.stanford.edu/~amaas/data/sentiment/)，是Kaggle上一个入门的[项目](https://www.kaggle.com/c/word2vec-nlp-tutorial)。
- 在Kaggle上详细使用了`word2vec`进行向量化，本文主要介绍`Doc2Vec`模型的使用，并使用神经网络模型提高准确率。
- 数据包括
  - **测试数据（testData)：**25000条
  - **未标注数据（unlabelData）**：5000条
  - **训练数据（trainData）**：25000条（正负情感各一半）
  - 每个电影的评论都不超过30条（测试集电影与训练集电影不相同）
  - 我们使用所有数据来作为`doc2vect`模型的语料
- 本文主要使用python中的`pandas`、`nltk`、`gensim`、`TensorFlow`库进行数据清理和分析。
- 注意，在kaggle上，针对此题的评分标准是按照`ROC`曲线（AUC） ，并不是硬分类，参考[Wikipedia](https://en.wikipedia.org/wiki/Receiver_operating_characteristic)。

# 数据预处理

此部分的`JupyterNotebook`可[参考这里](https://github.com/Scottdyt/MyProject/blob/master/SentimentAnalysis/jupyternotebook/preprocess.ipynb)。

对于拿到的电影评论数据，我们需要进行数据清理以后才能使用`doc2vec`进行向量化。

本文采用`pandas`库进行数据预处理，好处是可以使用`apply`函数对其进行并发操作，提高效率。

NLP的数据预处理一般包括以下几个部分：

1. 如果是网页内容，首先需要去掉`Html tag`
2. 将文档分割成句子。
3. 将句子分割成词语。
4. 纠正拼写错误（可选）、替换缩写词（可选）。
5. `Lemmatizer`、`stemming`
   - 词型还原或词干提取，两者的差别可[参考这里](https://blog.csdn.net/march_on/article/details/8935462)
6. 去除停顿词，去除标点、转化为小写字母。
7. 去掉长度过短的词语。

本项目没有纠正拼写错误（口语化词语太多）、没有去处数字（实测会提高精度）最后将训练集分开（`pos`和`neg`）。

# Doc2vec

此部分的`JupyterNotebook`可[参考这里](https://github.com/Scottdyt/MyProject/blob/master/SentimentAnalysis/jupyternotebook/Doc2Vec.ipynb)。

`Doc2Vec`模型比较复杂，相对于`word2vec`模型，它可以直接得到每个文档的向量，省略了将词向量转换为段向量的过程。

由于其直接是段向量，因此考虑了词之间的顺序，具有较好的语义信息。

而传统的`word2vec`模型，使用平均词向量或聚类的方式得到的段向量没有考虑词的顺序。

## 输入数据

- `test.txt`：25000 条评论用于测试
- `train-neg.txt`: 12500 条negative 评论用于训练
- `train-pos.txt`: 12500 条positive 评论用于训练
- `train-unsup.txt`: 50000 条未标注的评论用于训练`doc2vec`模型

## 格式化数据

我们需要将每句话以`LabeledSentence`类的形式传入`gensim`的`Doc2Vec`模型中：

```python
[['word1', 'word2', 'word3',..., 'lastword'], ['label1']]
```

其中，`label`标签是指这一段话的唯一标签，在官方文档中这个标签可以有多个（本文只使用一个），可以用标签提取当前模型训练出来的段向量。

传统的`LabeledSentence`类只能接受一个文件，而我们需要将多个文件一起进行训练，因此我们对其进行简单的封装，生成我们的`LabeledLineSentence`类，再使用`gensim`的`TaggedDocument`函数转化为合适的格式。

其中，`LabeledSentence.sentences_perm()`是为了我们之后训练时能够**随机打乱段落顺序，这能有效提高训练准确率。**

现在，我们只需要传入一个字典（数据路径+数据前缀）即可，请记住，数据的前缀必须要唯一,例如：

```python
sources = {'CleanData\\test.txt': 'TEST', 'CleanData\\train-neg.txt': 'TRAIN_NEG','CleanData\\train-pos.txt': 'TRAIN_POS', 'CleanData\\train-unsup.txt': 'TRAIN_UNS'}
sentences = LabeledLineSentence(sources)
```

## 训练模型

训练模型的过程比较简单，分为`模型构建`、`构建词汇`、`模型训练`、`模型序列化`，主要代码为：

```python
self.model = Doc2Vec(min_count=40, window=15, size=400, sample=1e-4, negative=5, workers=7)

self.model.build_vocab(self.sentences.to_array())
     self.model.train(self.sentences.shuffle(),total_examples=self.model.corpus_count, epochs=self.model.iter)

self.model.save(filename)
```

- 在设置参数时，将`mini cout`设置为40，`window`为10-20，`size`为400或者更大，训练出来的模型会比较准确（具体训练参数参考代码`Doc2Vec.py`）。

## Inspecting Model

- 查找给定词语中最相似的词语：
  - ```python
    model.wv.most_similar('good')
    [('decent', 0.7674194574356079),
     ('great', 0.7440007328987122),
     ('fine', 0.7338722944259644),
     ('bad', 0.7250358462333679),
     ('solid', 0.6936144232749939),
     ('nice', 0.6750378012657166),
     ('well', 0.6653556823730469),
     ('fantastic', 0.6441545486450195),
     ('terrible', 0.6301878690719604),
     ('excellent', 0.6274154186248779)]
    ```

- 对给定词语中最不匹配的词：

  - ```python
    model.wv.doesnt_match("man woman child kitchen".split())
    'kitchen'
    ```

- 我们也可以使用`label`标签查看训练的段落向量：

  - ```python
    model["TEST_24810"]
    ```


### TensorBoard

我们可以利用google提供的TensorBoard很方便的对模型进行可视化，来帮助查看我们训练的好坏。

其可视化代码可参考我的[Github](https://github.com/Scottdyt/ClassProject/tree/master/IntroduceToAI/SentimentAnalysis)上`Visualize.py`。

- 展示界面如下：

![imdb1](/images/coding/project/imdb1.png)

- 我们想观察与`suck`关联度最高的词

  ![imdb2](/images/coding/project/imdb2.png)

## 得到段向量

至此，我们已经成功将文本向量化，只需要从模型中把需要的向量取出，即可进行常规的数据分析：

```python
train_arrays = np.zeros((25000, 100))
train_labels = np.zeros(25000)
for i in range(12500):
    prefix_train_pos = 'TRAIN_POS_' + str(i)
    prefix_train_neg = 'TRAIN_NEG_' + str(i)
    train_arrays[i] = model[prefix_train_pos]
    train_arrays[12500 + i] = model[prefix_train_neg]
    train_labels[i] = 1
    train_labels[12500 + i] = 0
```

这里只显示了训练数据的向量，实际上，我们需要将所有数据取出，为了方便，可使用`pickle`保存到磁盘。

# 机器学习

本节代码见[这里](https://github.com/Scottdyt/MyProject/blob/master/SentimentAnalysis/ML.py)。

首先我们用传统的机器学习算法进行训练，使用`GridSearchCV`调参，查看在训练集上的`ROC`，与[使用TF-IDF](zealscott.com/posts/26189)中的训练方式类似，不再赘述。

最后，在逻辑回归上的训练集准确率有`0.92`，在SVM上准确率有`0.94`，还不错。

# 神经网络

此部分的JupyterNotebook可[参考这里](https://github.com/Scottdyt/MyProject/blob/master/SentimentAnalysis/jupyternotebook/LSTM.ipynb)。

对于`NLP`领域来说，最常用的神经网络模型是LSTM，因为它能很好的考虑到词向量之间的关联。

这里需要对输入向量进行修改，我们要将每个文档提取`feature`，每个`feature`用一个向量表示，我使用了文档中前500个词作为`feature`，每个词拿到之前训练好的`Doc2Vec`中提取向量，因此每个文档是`（500,400）`的二维向量（400是训练的模型向量长度）。

由于矩阵过于庞大，我们需要先保存到磁盘，而传统的使用`pickle`库效率很低，我们使用了`numpy`自带的`np.save`保存，避免内存不够用。

`LSTM`具体原理可见附录，这里为了方便起见，使用`Keras`提供的high level API进行训练。其中，有4层卷积层，一层LSTM，最后是全连接层。

最后训练出来，在测试集上有`0.97`左右的准确率，挺不错的。

# 收获

- 注意题目要求，题目说了`Submissions are judged on area under the ROC curve. `，所以这个题并不是要求我们硬分类，只是需要我们提供概率即可（衡量分类器的好坏）。关于`ROC`的介绍和讨论参考附录。
- `kears`没有提供`ROC`的性能评价，需要我们自己实现，可以参考[这里](https://stackoverflow.com/questions/41032551/how-to-compute-receiving-operating-characteristic-roc-and-auc-in-keras)，[Keras](https://keras.io/metrics/#custom-metrics)上也有简短的介绍（使用`Tensorflow`的评价指标）。
- 序列化大数据时，使用`np.save()`性能好于`pickle`。
- 当对数据进行向量化后，先进行线性回归等简单的分类器，来衡量向量化好坏。
- 熟悉了`keras`框架和LSTM的使用，锻炼了调参过程。


# Reference

## 主要参考资料

1. `Doc2Vec`部分主要参考：[Sentiment Analysis Using Doc2Vec](http://linanqiu.github.io/2015/10/07/word2vec-sentiment/)

2. `word2vec`可以参考kaggle上的toturial：[kaggle tutorial](https://www.kaggle.com/c/word2vec-nlp-tutorial#part-2-word-vectors)

3. `LSTM`可参考[The Unreasonable Effectiveness of Recurrent Neural Networks](http://karpathy.github.io/2015/05/21/rnn-effectiveness/)

4. [How to compute Receiving Operating Characteristic (ROC) and AUC in keras?](https://stackoverflow.com/questions/41032551/how-to-compute-receiving-operating-characteristic-roc-and-auc-in-keras)

5. [机器学习和统计里面的auc怎么理解？](https://www.zhihu.com/question/39840928?from=profile_question_card)


## 文本预处理

1. [正则提取出HTML正文](https://blog.csdn.net/pingzi1990/article/details/41698331)
2. [replacer](https://github.com/PacktPublishing/Natural-Language-Processing-Python-and-NLTK/blob/master/Module%203/__pycache__/replacers.py)
3. [RegexReplacer](https://groups.google.com/forum/#!topic/nltk-users/BVelLz2UNww)
4. [词干提取与词性还原](https://blog.csdn.net/march_on/article/details/8935462)
5. [pos tag type](https://stackoverflow.com/questions/15388831/what-are-all-possible-pos-tags-of-nltk?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
6. [Stemming and Lemmatization](https://www.jianshu.com/p/22be6550c18b)
7. [IMDB电影评论集](http://ai.stanford.edu/~amaas/data/sentiment/)

## Doc2Vec

- 利用Doc2Vec的改进
  - [Sentiment Analysis Using Doc2Vec](http://linanqiu.github.io/2015/10/07/word2vec-sentiment/)
- Kaggle针对Word2vector
  - [kaggle tutorial](https://www.kaggle.com/c/word2vec-nlp-tutorial#part-2-word-vectors)
- Gensim发明者写的
  - [Doc2vec tutorial](https://rare-technologies.com/doc2vec-tutorial)
- Github上的一篇，但没太看懂
  - [Gensim Doc2vec Tutorial on the IMDB Sentiment Dataset](https://github.com/RaRe-Technologies/gensim/blob/develop/docs/notebooks/doc2vec-IMDB.ipynb)
- Kaggle上的讨论
  - [Using Doc2Vec from gensim.](https://www.kaggle.com/c/word2vec-nlp-tutorial/discussion/12287)
- gensim官方参数文档
  - [Deep learning with paragraph2vec](https://radimrehurek.com/gensim/models/doc2vec.html)
  - [IMDB](https://github.com/RaRe-Technologies/gensim/blob/develop/docs/notebooks/doc2vec-IMDB.ipynb)
- 关于参数`negative sampling`
  - [Negative Sampling](http://mccormickml.com/2017/01/11/word2vec-tutorial-part-2-negative-sampling/)
- [关于window的调参](https://stackoverflow.com/questions/22272370/word2vec-effect-of-window-size-used)   ​

## LSTM

- 使用`TFlearn`
  - [从代码学AI ——情感分类(LSTM on TFlearn)](https://blog.csdn.net/hitxueliang/article/details/77550819?locationNum=5&fps=1)
  - [tflearn中lstm文本分类相关实现](https://blog.csdn.net/luoyexuge/article/details/78243107)
- [使用Keras训练LSTM](https://github.com/danielmachinelearning/Doc2Vec_CNN_RNN)

## 优秀代码

1. [只用机器学习](http://nbviewer.jupyter.org/github/jmsteinw/Notebooks/blob/master/NLP_Movies.ipynb)


