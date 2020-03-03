---
layout:     post
title:      使用word2vec和xgboost寻找Quora上的相似问题
subtitle:   
date:       2018-09-20
author:     IceGomic
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - 机器学习
    - 自然语言处理
    - 人工智能
---


> 本文首次发布于 [IceGomic Blog](http://icegomic.github.io), 作者 [@IceGomic](http://github.com/icegomic) ,转载请保留原文链接.
JanusGraph使用[Gremlin Server](https://tinkerpop.apache.org/docs/3.4.5/reference/#gremlin-server)引擎作为服务器组件来处理和响应客户端查询。 当打包在JanusGraph中时，Gremlin Server称为JanusGraph Server。

原作者 Susan Li

Changing the world, one article at a time. Sr. Data Scientist, Toronto Canada. Opinion=my own.

[http://www.linkedin.com/in/susanli/](http://www.linkedin.com/in/susanli/)

使用word2vec和xgboost寻找Quora上的相似问题

备注：Quora是一个国外的问答网站，可以理解为外国的知乎。

网址 [https://www.quora.com/](https://www.quora.com/) 当然 是需要翻墙的

![image.jpeg](https://upload-images.jianshu.io/upload_images/100763-9ff90dbbbb98fd82.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上周，我们探索了几种不同的去重技术，尝试使用BOW，TFIDF，Xgboost方法来识别相似文档。我们发现使用传统的TFIDF方法可以解决一些比较明显的问题。这可以解释为什么谷歌在搜索领域长期使用TFIDF方法来判断一个单词对于一个页面的重要程度。

为了深入研究和提升能力，我们来探索一些新的方法来解决类似的匹配和去重问题，首先我们把去重问题引申为一个分类问题，然后再去解决它。

数据

这个任务的目标是鉴别Quora中的一对问题是不是表达同样的意思，在数据中，每一组数据包含两个问题，以及人类专家（难道不是运营）标注的这俩问题是否属于同一个意思的标签。不过需要注意的是，这个标注过程是很主观的，对于同一对问题是否表述同一个意思，不同的专家可能有不同的意见。所以这个标签算是一种参考，它不是100%准确的。
```
df = pd.read_csv('quora_train.csv')

df = df.dropna(how="any").reset_index(drop=True) 

a= 0

for i in range(a,a+10):

    print(df.question1[i])

    print(df.question2[i])

    print()
```
![image.png](https://upload-images.jianshu.io/upload_images/100763-063e2678277a3057.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

计算词移距离（WMD：word mover’s distance）

词移距离是一个能让我们使用“距离”评估两个文档相似度的方法，这种方法不关心是不是有相同的单词。因为它使用了word2vec的向量进行计算。它的主要思想是利用文档中词的embedded向量，来计算一篇文档“游走”到另一篇文档的最小距离来衡量两篇文章的差异性。我们来看一个例子。下面是一对已经标注重复的数据：
```
question1 = 'What would a Trump presidency mean for

current international master’s students on an F1

visa?'

question2 = 'How will a Trump presidency affect the

students presently in US or planning to study in US?'

question1 = question1.lower().split()

question2 = question2.lower().split()

question1 = [w for w in question1 if w not in

stop_words]

question2 = [w for w in question2 if w not in

stop_words]
```

上面的代码做了两个操作，一个是转换大小写，一个是去除停用词，算是初步清洗样本吧。

我们预先用google news的语料训练了Word2vec模型。使用了genim的word2vec算法包。
```

import gensim

from gensim.models import Word2Vec

model =

gensim.models.KeyedVectors.load_word2vec_format('./wor

d2Vec_models/GoogleNews-vectors-negative300.bin.gz',

binary=True)
```
下面开始计算两个问题的WMD距离。注意，这俩文本表达了同样的意思，并被标注为重复Quora数据。
```
distance = model.wmdistance(question1, question2)

print('distance = %.4f' % distance)
```
结果是 distance = 1.8293

结果的值看起来很大，我们需要把它进行标准化。

 标准化word2vec向量

在使用wmd方法时，首先去标准化word2vec向量，这是有好处的，这样他们就有一样的长度了。
```
model.init_sims(replace=True)

distance = model.wmdistance(question1, question2)

print('normalized distance = %.4f' % distance)

normalized distance = 0.7589
```
标准化之后，这个值就小多了。

我们再来做一对实验，这次用的是两个不重复的问题。
```
question3 = 'Why am I mentally very lonely? How can I

solve it?'

question4 = 'Find the remainder when [math]23^{24}

[/math] is divided by 24,23?'

question3 = question3.lower().split()

question4 = question4.lower().split()

question3 = [w for w in question3 if w not in

stop_words]

question4 = [w for w in question4 if w not in

stop_words]

distance = model.wmdistance(question3, question4)

print('distance = %.4f' % distance)

distance = 1.2637

model.init_sims(replace=True)

distance = model.wmdistance(question3, question4)

print('normalized distance = %.4f' % distance)

normalized distance = 1.2637 、
```
这一次 ，标准化之后的值没有发生变化。WMD方法认为这一组数据不如第一组那么相似，看起来很有效果不是吗。

FuzzyWuzzy

在前面的一篇文章中，我们已经了解过Python中模糊字符匹配的方法[https://towardsdatascience.com/natural-language-processing-for-fuzzy-string-matching-with-python-6632b7824c49](https://towardsdatascience.com/natural-language-processing-for-fuzzy-string-matching-with-python-6632b7824c49)

快速了解一下FuzzyWuzzy工具在处理我们的重复问题上能起什么作用。
```
from fuzzywuzzy import fuzz

question1 = 'What would a Trump presidency mean for

current international master’s students on an F1

visa?'

question2 = 'How will a Trump presidency affect the

students presently in US or planning to study in US?'

fuzz.ratio(question1, question2)

53

fuzz.partial_token_set_ratio(question1, question2)

100

question3 = 'Why am I mentally very lonely? How can I

solve it?'

question4 = 'Find the remainder when [math]23^{24}

[/math] is divided by 24,23?'

fuzz.ratio(question3, question4)

28

fuzz.partial_token_set_ratio(question3, question4)
```
37

从上面的结果看起来，FuzzyWuzzy工具也不认为第二组数据相似。这很好，因为人类专家也这么认为。

特征工程

首先，我们先实现几个函数——计算WMD，标准化WMD，word2vec表达
```
def wmd(q1, q2):

    q1 = str(q1).lower().split()

    q2 = str(q2).lower().split()

    stop_words = stopwords.words('english')

    q1 = [w for w in q1 if w not in stop_words]

    q2 = [w for w in q2 if w not in stop_words]

    return model.wmdistance(q1, q2)

def norm_wmd(q1, q2):

    q1 = str(q1).lower().split()

    q2 = str(q2).lower().split()

    stop_words = stopwords.words('english')

    q1 = [w for w in q1 if w not in stop_words]

    q2 = [w for w in q2 if w not in stop_words]

    return norm_model.wmdistance(q1, q2)

def sent2vec(s):

    words = str(s).lower()

    words = word_tokenize(words)

    words = [w for w in words if not w in stop_words]

    M = []

    for w in words:

    try:

        M.append(model[w])

    except:

        continue

    M = np.array(M)

    v = M.sum(axis=0)

    return v / np.sqrt((v ** 2).sum())
```
我们需要构建一些新的特征：

1.单词个数

2.字符个数

3.问题1和问题2中相同单词的个数

4.问题1和问题2中不同单词的个数

5.问题1和问题2的向量余弦距离

6.问题1和问题2的向量曼哈顿距离

7.--杰卡德距离

8.--兰氏距离

9.--欧氏距离

10.--闵可夫斯基距离

11.--布雷柯蒂斯距离

12.峰度和偏度

13.词移距离

14.标准化词移距离

所有以上距离计算公式都可以在scipy.spatial.distance中找到。
```
1\. df['len_q1'] = df.question1.apply(lambda x: len(str(x))

2\. df['len_q2'] = df.question2.apply(lambda x: len(str(x))

3\. df['diff_len'] = df.len_q1 - df.len_q2

4\. df['len_char_q1'] = df.question1.apply(lambda x: len(''.join(set(str(x).replace(' ',''))))

5\. df['len_char_q2'] = df.question2.apply(lambda x: len(''.join(set(str(x).replace(' ',''))))

6\. df['len_word_q1'] = df.question1.apply(lambda x: len(str(x).split())

7\. df['len_word_q2'] = df.question2.apply(lambda x: len(str(x).split())

8\. df['common_words'] = df.apply(lambda x: len(set(str(x[‘question1’]).lower().split()).intersection(set(str(x[‘question2’]).lower().split()))), axis=1)

9\. df['fuzz_ratio'] = df.apply(lambda x: fuzz.ratio(str(x[‘question1’]), str(x[‘question2'])),axis=1)

10.df['fuzz_partial_ratio'] = df.apply(lambda x: fuzz.partial_ratio(str(x[‘question1’]), str(x['question2'])), axis=1)

11.df['fuzz_partial_token_set_ratio'] = df.apply(lambda x: fuzz.partial_token_set_ratio(str(x['question1']), str(x['question2'])), axis=1)

12.df[‘fuzz_partial_token_sort_ratio'] = df.apply(lambda x: fuzz.partial_token_sort_ratio(str(x['question1']),str(x['question2'])), axis=1)

13.df['fuzz_token_set_ratio’] = df.apply(lambda x: fuzz.token_set_ratio(str(x[‘question1’]), str(x['question2'])), axis=1)

14.df[‘fuzz_token_sort_ratio’] = df.apply(lambda x: fuzz.token_sort_ratio(str(x[‘question1’]),str(x[‘question2'])), axis=1)
```


word2vec模型

前面说了，我们使用预先训练好的google news 语料的Word2vec模型。我下载下来并保存在word2Vec_models文件夹里面。我们用gensim的模块加载这个模型。
```
model =

gensim.models.KeyedVectors.load_word2vec_format('./wor

d2Vec_models/GoogleNews-vectors-negative300.bin.gz',

binary=True)

df['wmd'] = df.apply(lambda x: wmd(x['question1'],

x['question2']), axis=1)
```
标准化word2vec模型
```
norm_model =

gensim.models.KeyedVectors.load_word2vec_format('./word2Vec_models/GoogleNews-vectors-negative300.bin.gz',

binary=True)

norm_model.init_sims(replace=True)

df['norm_wmd'] = df.apply(lambda x:

norm_wmd(x['question1'], x['question2']), axis=1)
```
好了，到这里，我们构建的所有特征都已经有了，计算出他们的距离。
```
question1_vectors = np.zeros((df.shape[0], 300))

for i, q in enumerate(tqdm_notebook(df.question1.values)):

    question1_vectors[i, :] = sent2vec(q)

question2_vectors = np.zeros((df.shape[0], 300))

for i, q in enumerate(tqdm_notebook(df.question2.values)):

    question2_vectors[i, :] = sent2vec(q)

df['cosine_distance'] = [cosine(x, y) for (x, y) in zip(np.nan_to_num(question1_vectors), np.nan_to_num(question2_vectors))]

df['cityblock_distance'] = [cityblock(x, y) for (x, y) in zip(np.nan_to_num(question1_cectors). np.nan_to_num(question2_vectors))]

df['jaccard_distance'] = [jaccard(x, y) for (x, y) in zip(np.nan_to_num(question1_cectors). np.nan_to_num(question2_vectors))]

df['canberra_distance'] = [canberra(x, y) for (x, y) in zip(np.nan_to_num(question1_cectors). np.nan_to_num(question2_vectors))]

df['euclidean_distance'] = [euclidean(x, y) for (x, y) in zip(np.nan_to_num(question1_cectors). np.nan_to_num(question2_vectors))]

df['skew_q1vec'] = [skew(x) for x in np.nan_to_num(question1_vectors)]

df['skew_q2vec'] = [skew(x) for x in np.nan_to_num(question2_vectors)]

df[‘kur_q1vec’] = [kurtosis(x) for x in mp.nan_to_num(question1_vectors)]

df[‘kur_q2vec’] = [kurtosis(x) for x in mp.nan_to_num(question2_vectors)]
```
使用Xgboost进行分类模型训练
```
df.drop(['question1', 'question2'], axis=1, inplace=True)

df = df[pd.notnull(df['cosine_distance'])]

df = df[pd.notnull(df['jaccard_distance'])]

X = df.loc[:, df.columns != 'is_duplicate']

y = df.loc[:, df.columns == 'is_duplicate']

X_train, X_test, y_train, y_test = train_test_split(X, y,test_size=0.3, random_state=0)

import xgboost as xgb

 model = xgb.XGBClassifier(max_depth=50, n_estimators=80, learning_rate=0.1, colsample_bytree=.7, gamma=0, reg_alpha=4, objective=’binary:logistic’, eta=0.3,silent=1, subsample=0.8).fit(X_train, y_train.values.ravel))

prediction = model.predict(X_test)

cm = confusion_matrix(y_test, prediction)

print(cm)

print(’Accuracy’, accyracy_score(y_test, prediction))

print(classification_report(y_test, prediction))
```
![image.png](https://upload-images.jianshu.io/upload_images/100763-40933b752a9a3179.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用我们构建的特征，Xgboost分类准确率为0.77，如果融合Xgboost+TFIDF可以达到0.8

同时，我们的召回从0.67提升到了0.73

Jupyter notebook can be found on Github. 周末加油鸭！

Reference:

[https://www.linkedin.com/pulse/duplicate-quora-question-abhishek-](https://www.linkedin.com/pulse/duplicate-quora-question-abhishek- "Page 9")thakur/
