---
title: RNN神经网络
date: 2026-10-10 15:28:57
tags:
---
## RNN介绍

### RNN概念

循环神经网络（Recurrent Neural Network, RNN）是一种==专门处理序列数据的神经网络==。与传统的前馈神经网络不同，RNN具有“**循环**”结构，能够处理和记住前面时间步的信息，使其特别适用于时间序列数据或有时序依赖的任务。

我们要明确什么是序列数据，时间序列数据是指在不同时间点上收集到的数据，这类数据反映了某一事物、现象等随时间的变化状态或程度。这是时间序列数据的定义，当然这里也可以不是时间，比如文字序列，但总归序列数据有一个特点——**后面的数据跟前面的数据有关系**。


{% asset_img 1734745895344.png "Hexo 博客封面示例" %}

{% asset_img Snipaste_2026-10-10_17-11-11.png "Hexo 博客封面示例" %}

{% asset_img Snipaste_2026-10-10_17-13-52.png "Hexo 博客封面示例" %}

基于所提供的循环神经网络（RNN）结构图，以下对序列预测（预测词）的过程及其时间步之间的传递关系进行严谨且通俗的阐释。

#### 一、 核心变量解析

图中展示了一个标准的RNN单元在时间步 `$t$` 的展开状态：
*   **`$x_t$`**：当前时间步的输入。在自然语言处理中，通常代表当前词或当前词的词向量。
*   **`$h_{t-1}$`**：上一时间步（`$t-1$`）传递过来的隐藏状态，承载着历史序列的信息（即“记忆”）。
*   **`$h_t$`**：当前时间步计算出的隐藏状态，将传递给下一时间步。
*   **`$y_t$`**：当前时间步的输出，通常代表对下一个词的预测概率分布。
*   **`$W, V, U, b_h, b_y$`**：网络参数。其中 `$W$` 处理上一隐藏状态，`$V$` 处理当前输入，`$U$` 处理当前隐藏状态以生成输出。

#### 二、 上一次训练（时间步）与下一次训练（时间步）的关系

在RNN中，“上一次”与“下一次”并非指代两个独立的训练轮次（Epoch），而是指**同一序列在不同时间步（`$t-1$` 与 `$t$`）之间的状态传递与参数共享**。其核心关系由公式 `$h_t = \sigma(W h_{t-1} + V x_t + b_h)$` 体现：

1.  **信息传递（记忆机制）**：当前时间步 `$h_t$` 的计算，严格依赖于上一时间步的隐藏状态 `$h_{t-1}$`。`$h_{t-1}$` 浓缩了从序列起始到 `$t-1$` 时刻的所有历史信息。这意味着，网络在处理当前词时，并未遗忘前文。
2.  **参数共享（结构复用）**：无论序列多长，“上一次”与“下一次”计算所使用的权重矩阵（`$W, V, U$`）和偏置项（`$b_h, b_y$`）是**完全共享**的。网络并非在每一步重新学习规则，而是利用同一套规则不断循环处理新的输入。这极大地减少了参数量，并使模型能够泛化到不同长度的序列。
3.  **链式依赖**：从 `$t-1$` 到 `$t$`，隐藏状态 `$h$` 构成了一个连续的链条。上一时刻的输出误差，在反向传播（BPTT）时，会沿着这条链条逆向传递，从而同时更新“上一次”和“下一次”计算中所共用的参数矩阵。

#### 三、 预测词的生成逻辑

预测词的过程分为两个阶段，对应图中的两个公式：

1.  **状态更新阶段**：接收当前输入 `$x_t$` 与历史记忆 `$h_{t-1}$`，通过线性组合与非线性激活函数 `$\sigma$`（如Tanh或ReLU），生成包含当前上下文信息的新隐藏状态 `$h_t$`。
2.  **输出预测阶段**：基于更新后的隐藏状态 `$h_t$`，通过公式 `$y_t = \sigma(U h_t + b_y)$` 进行计算。在词预测任务中，`$\sigma$` 通常为Softmax函数，其输出 `$y_t$` 是一个维度等于词表大小的概率分布。概率值最大的维度所对应的词，即为当前时间步预测出的下一个词。

#### 四、 严谨性补充

图中的是最基础的RNN结构（Vanilla RNN）。在实际的“预测词”训练过程中，损失函数（如交叉熵损失）会衡量 `$y_t$` 与真实下一个词（标签）之间的差异。随后通过时间反向传播算法（BPTT），将误差从序列末端一直回传至序列首端，并利用梯度下降法更新全局共享的参数 `$W, V, U, b_h, b_y$`。

但是该结构在长序列中存在梯度消失或梯度爆炸的数学缺陷，实际应用中常采用LSTM或GRU等变体来优化这种“上一次”向“下一次”的信息传递机制。

### RNN应用场景

- **自然语言处理（NLP）**：文本生成、语言建模、机器翻译、情感分析等。
- **时间序列预测**：股市预测、气象预测、传感器数据分析等。
- **语音识别**：将语音信号转换为文字。
- **音乐生成**：通过学习音乐的时序模式来生成新乐曲。

### 自然语言处理概述

自然语言处理（Nature language Processing, NLP）研究的主要是==通过计算机算法来理解自然语言。==

对于自然语言来说，处理的数据主要就是**人类的语言**，例如：汉语、英语、法语等，该类型的数据不像我们前面接触过的结构化数据、或者图像数据可以很方便的进行数值化。

NLP的目标是让机器能够“听懂”和“读懂”自然语言，并进行有效的交流和分析。

NLP涵盖了从文本到语音、从语音到文本的各个方面，它涉及多种技术，包括语法分析、语义理解、情感分析、机器翻译等。

{% asset_img 1734746204314.png "Hexo 博客封面示例" %}

{% asset_img Snipaste_2026-10-10_17-04-48.png "Hexo 博客封面示例" %}

## 词嵌入层

> 在 **RNN（Recurrent Neural Network）** 中，**词嵌入层（Word Embedding Layer）** 是处理自然语言数据的关键组成部分。它将输入的离散单词（通常是词汇表中的索引）转换为连续的、低维的向量表示，从而使得神经网络能够理解和处理这些词汇的语义信息。

### 词嵌入层作用

词嵌入层的主要目的是**将每个词映射为一个固定长度的向量（将文本转换为向量）**，这些向量能够捕捉词与词之间的语义关系。

传统的文本表示方法（如one-hot编码）无法反映单词之间的相似性，因为在one-hot编码中，每个单词都被表示为一个**高维稀疏向量**，而词嵌入通过**低维稠密向量**表示单词，能够更好地捕捉词汇之间的语义相似性。

词嵌入层首先会根据输入的词的数量构建一个**词向量矩阵**，例如: 我们有 100 个词，每个词希望转换成 128 维度的向量，那么构建的矩阵形状即为: 100*128，输入的每个词都对应了一个该矩阵中的一个向量。


{% asset_img 1734747074293.png "Hexo 博客封面示例" %}

{% asset_img Snipaste_2026-10-10_17-33-26 "Hexo 博客封面示例" %}

**词嵌入层在RNN中的作用**：

- **输入表示**：RNN通常用于处理序列数据。在处理文本时，RNN的输入是由单词构成的序列。由于神经网络不能直接处理离散的单词标识符（如整数索引或字符），因此需要通过词嵌入层将每个单词转换为一个固定长度的稠密向量。这些向量作为RNN的输入，帮助RNN理解词语的语义。
- **降低维度**：词嵌入层将原本高维的稀疏表示（如one-hot编码）转化为低维的稠密向量，减少了计算量，同时保持了词汇之间的语义关系。
- **捕捉语义相似性**：通过训练，词嵌入能够学习到词语之间的关系。例如，语义相似的词（如“猫”和“狗”）在向量空间中会比较接近，而语义不相关的词（如“猫”和“汽车”）则会较为遥远。

###  词嵌入层工作流程

- **初始化词向量**：词嵌入层的初始词向量通常会使用随机初始化或者通过加载预训练的词向量（如Word2Vec或GloVe）进行初始化。

- **输入索引**：每个单词在词汇表中都有一个唯一的索引。输入文本（例如一个句子）会先被分词，然后每个单词会被转换为相应的索引。

- **查找词向量**：词嵌入层将这些**单词的索引**映射为对应的词向量。这些词向量是一个低维稠密向量，表示该词的语义。

- **输入到RNN**：这些词向量作为RNN的输入，RNN处理它们并根据上下文生成一个序列的输出。（词的索引对应的向量，输出是索引，再去查找对应的词）

### 词嵌入层使用

在PyTorch中，我们可以**使用 nn.Embedding 词嵌入层来实现输入词的向量化**。

nn.Embedding 对象构建时，最主要有两个参数:

- num_embeddings：表示**词的数量**

- embedding_dim：表示**用多少维的向量来表示每个词**

```python
#参数1：需要嵌入的词的数量，参数2：嵌入出来的维度（每个词生成embedding_dim个数值）
nn.Embedding(num_embeddings=10, embedding_dim=4)
```

接下来，我们将会学习如何将词转换为词向量，其步骤如下:

1. 先将语料进行分词，构建词与索引的映射，我们可以把这个映射叫做词表，词表中每个词都对应了一个唯一的索引；
2. 然后使用 nn.Embedding 构建词嵌入矩阵，词索引对应的向量即为该词对应的数值化后的向量表示。

例如，我们的文本数据为: "北京冬奥的进度条已经过半，不少外国运动员在完成自己的比赛后踏上归途。"，接下来，我们看下如何使用词嵌入层将其进行转换为向量表示，步骤如下：

```python
import torch
import torch.nn as nn
import jieba # pip install jieba -i https://pypi.mirrors.ustc.edu.cn/simple/


if __name__ == '__main__':
    # 0.文本数据
    text = '北京冬奥的进度条已经过半，不少外国运动员在完成自己的比赛后踏上归途。'

    # 1. 文本分词
    words = jieba.lcut(text)
    print('文本分词:', words)

    # 2.分词去重并保留原来的顺序获取所有的词语
    unique_words = list(set(words))
    print("去重后词的个数:\n",len(unique_words))

    # 3. 构建词嵌入层
    # num_embeddings: 表示词的总数量
    # embedding_dim: 表示词嵌入的维度
    embed = nn.Embedding(num_embeddings=len(unique_words), embedding_dim=4)
    print("词嵌入的结果：\n",embed)

    # 4. 词语的词向量表示
    # enumerate(): 返回列表中每个值 及其对应的 索引.
    for i, word in enumerate(unique_words):
        # 获得词嵌入向量
        word_vec = embed(torch.tensor(i))
        print('%s\t' % word, word_vec)
```

课堂代码：
```python
"""
案例:
    演示词嵌入层的API应用.

RNN介绍:
    全称叫Recurrent neural network, 循环神经网络, 主要处理 序列数据的.
    序列数据: 后边数据对前边的数据有依赖, 例如: 天气预测, 股市分析, 文本生成...

    组成:
        词嵌入层
        循环网络层
        输出层

词嵌入层介绍(作用):
    把词 (或者 词对应的索引) 转成 词向量.
"""

# 导包
import torch
import jieba            # jieba分词器, 需要安装一下, pip install jieba
import torch.nn as nn

# 1.定义函数, 用于演示 词嵌入层的API, 如何把 词(词的索引) -> 词向量
def dm01():
    # 1. 定义一句话
    text = '北京冬奥的进度条已经过半，不少外国运动员在完成自己的比赛后踏上归途。'
    # 2. 使用jiema模块进行分词.
    words = jieba.lcut(text)
    print(f'分词结果: {words}')     # ['北京', '冬奥', '的', '进度条', ...]

    # 3. 创建词嵌入层.
    # 参1: 词表大小(词的个数), 参2: 词向量的维度
    embed = nn.Embedding(len(words), 4)

    # 4. 获取每个词对象的下标索引.
    # i = 0
    # for word in words:
    #     print(i, word)
    #     i += 1

    # enumerate(): 返回列表中每个值 及其对应的 索引.
    for i, word in enumerate(words):    # 效果同上.
        # print(i, word)

        # 5. 把词索引(张量形式) 转成 词向量.
        word_vector = embed(torch.tensor(i))    # 随机的, 每次都不一样, 无所谓.
        print(f'词: {word}, \t\t词向量: {word_vector}')

# 2. 测试
if __name__ == '__main__':
    dm01()
```


**输出结果：**

```python
文本分词: ['北京', '冬奥', '的', '进度条', '已经', '过半', '，', '不少', '外国', '运动员', '在', '完成', '自己', '的', '比赛', '后', '踏上', '归途', '。']
去重后词的个数:
 18
词嵌入的结果：
 Embedding(18, 4)
 北京	 tensor([-1.8043,  1.7860, -0.7821, -0.3167], grad_fn=<EmbeddingBackward0>)
  的	 tensor([-1.4856, -0.5639,  0.2745,  0.1087], grad_fn=<EmbeddingBackward0>)
 冬奥	 tensor([ 0.6969, -0.5615,  1.6524, -0.2651], grad_fn=<EmbeddingBackward0>)
 自己	 tensor([1.7163, 0.9710, 0.0193, 1.4562], grad_fn=<EmbeddingBackward0>)
 不少	 tensor([ 0.8770, -1.0347,  0.4939,  1.6718], grad_fn=<EmbeddingBackward0>)
 归途	 tensor([ 0.0619,  0.0355, -1.6510,  0.8121], grad_fn=<EmbeddingBackward0>)
 比赛	 tensor([-1.7128,  0.0681,  2.4199,  0.2714], grad_fn=<EmbeddingBackward0>)
进度条	 tensor([-0.6706,  0.4089, -0.5371,  0.1093], grad_fn=<EmbeddingBackward0>)
 外国	 tensor([ 0.5665, -0.3168, -0.2065,  0.7682], grad_fn=<EmbeddingBackward0>)
 完成	 tensor([ 1.4640, -0.2800, -0.2242, -0.3790], grad_fn=<EmbeddingBackward0>)
 过半	 tensor([ 1.8026,  0.9831, -0.7177,  0.6779], grad_fn=<EmbeddingBackward0>)
 踏上	 tensor([-1.0745, -0.2610, -1.5350, -0.1767], grad_fn=<EmbeddingBackward0>)
运动员	 tensor([1.8748, 0.2093, 1.9471, 0.3893], grad_fn=<EmbeddingBackward0>)
  ，	 tensor([-1.2841, -0.0675, -0.1263, -0.6892], grad_fn=<EmbeddingBackward0>)
  在	 tensor([ 0.6202, -0.0106,  1.0504,  0.0299], grad_fn=<EmbeddingBackward0>)
 已经	 tensor([2.4461, 0.2113, 0.9387, 0.1680], grad_fn=<EmbeddingBackward0>)
  后	 tensor([-1.1989,  0.8727, -0.9484,  0.0578], grad_fn=<EmbeddingBackward0>)
  。	 tensor([0.4020, 0.7357, 0.3571, 1.2355], grad_fn=<EmbeddingBackward0>)
```

## 循环网络层

文本数据是具有序列特性的，例如: "我爱你", 这串文本就是具有序列关系的，"爱" 需要在 "我" 之后，"你" 需要在 "爱" 之后, 如果颠倒了顺序，那么可能就会表达不同的意思。

为了表示出数据的序列关系，我们需要使用循环神经网络(Recurrent Nearal Networks, RNN) 来对数据进行建模，RNN 是一个具有记忆功能的网络，它作用于处理带有序列特点的样本数据。

###  RNN网络层原理

当我们希望使用循环网络来对 "我爱你" 进行语义提取时，RNN计算过程是什么样的呢？


{% asset_img 01.png "Hexo 博客封面示例" %}

上图中h表示==隐藏状态==，隐藏状态保存了**序列数据中的历史信息**，并将这些信息传递给下一个时间步，从而允许RNN处理和预测序列数据中的元素。

每一次的输入都会包含两个值：**上一个时间步的隐藏状态、当前状态的输入值x**

每一次的输出都会包含两个值：**输出当前时间步的隐藏状态、当前时间步的预测结果y**

**隐藏状态作用：**

- **记忆功能**：隐藏状态就像RNN的记忆，它能够在不同的时间步之间传递信息。当一个新的输入进入网络时，当前的隐藏状态会结合这个新输入来生成新的隐藏状态。

- **上下文理解**：由于隐藏状态携带了过去的信息，它可以用于理解和生成与上下文相关的输出。这对于语言模型、机器翻译等任务尤其重要。

- **连接不同时间步**：隐藏状态通过网络内部的循环连接将各个时间步连接起来，使得网络可以处理变长的序列数据。

上图中，为了更加容易理解，虽然画了 3 个神经元, 但是实际上只有一个神经元，"我爱你" 三个字是重复输入到**同一个**神经元中。


{% asset_img 02.png "Hexo 博客封面示例" %}

我们举个例子来理解上图的工作过程，假设我们要实现文本生成，也就是输入 "我爱" 这两个字，来预测出 "你"，其如下图所示:


{% asset_img 03.png "Hexo 博客封面示例" %}

将上图展开成不同时间步的形式，如下图所示:


{% asset_img 04.png "Hexo 博客封面示例" %}

首先初始化出第一个隐藏状态，一般都是全0的一个向量，然后将 "我" 进行词嵌入，转换为向量的表示形式，送入到第一个时间步，然后输出隐藏状态 h1，然后将 h1 和 "爱" 输入到第二个时间步，得到隐藏状态 h2, 将 h2 送入到全连接网络，得到 "你" 的预测概率。

**RNN神经元内部是如何计算的呢？**

**1. 计算隐藏状态：**每个时间步的隐藏状态`$h_t$`是根据当前输入`$x_t$`和前一时刻的隐藏状态`$h_{t-1}$`计算的。



{% asset_img 1734749643694.png "Hexo 博客封面示例" %}



{% asset_img 05.png "Hexo 博客封面示例" %}

{% asset_img 1734745895344.png "Hexo 博客封面示例" %}

上两图中公式本质相同

{% asset_img 1734749643694png "Hexo 博客封面示例" %}

上述公式中:

- `$W_{ih}$` 表示输入数据的权重
- `$b_{ih}$` 表示输入数据的偏置
- `$W_{hh}$` 表示输入隐藏状态的权重
- `$b_{hh}$` 表示输入隐藏状态的偏置
- `$h_{t-1}$` 表示输入隐藏状态
- `$h_t$` 表示输出隐藏状态


当前输入加权求和，上一次隐藏状态的加权求和
最后对输出的结果使用tanh激活函数进行计算，得到该神经元你的输出隐藏状态。（它底层就是tanh，，其他的深度学习算法一般是ReLU）
再拿这个算出来的ht（当前加权求和的隐藏状态）做一次`$y_t=W_{hy}h_t+b_y$`

**2. 计算当前时刻的输出：**网络的输出`$y_t$`是当前时刻的隐藏状态经过一个线性变换得到的。

$$
y_t=W_{hy}h_t+b_y
$$

- `$y_t$` 是当前时刻的输出（通常是一个向量，表示当前时刻的预测值，RNN层的预测值）
- `$h_t$` 是当前时刻的隐藏状态
- `$W_{hy}$` 是从隐藏状态到输出的权重矩阵
- `$b_y$` 是输出层的偏置项

**3. 词汇表映射**：输出`$y_t$`是一个**向量**，该向量经过**全连接层**后输出得到最终预测结果`$y_{pred}$`，`$y_{pred}$`中每个元素代表当前时刻生成词汇表中某个词的得分（或概率，通过激活函数：如**softmax**）。==词汇表有多少个词，`$y_{pred}$`就有多少个元素值，最大元素值对应的词就是当前时刻预测生成的词。==

下一词预测用的是 **当前步的输出 `$y_t$`**，而传给下一时间步的“上一次隐藏状态”是 **`$h_t$`**，不是 `$y_t$`。

标准 RNN 里：

$$
h_t = f(W_{xh}x_t + W_{hh}h_{t-1}+b_h)
$$

$$
y_t = W_{hy}h_t + b_y
$$

如果 `$y_t$` 是 logits，那么下一词概率是：

$$
p(w_{t+1}\mid w_{\le t}) = \mathrm{softmax}(y_t)
$$

然后选词/采样得到下一词。

而下一时间步的隐藏状态是：

$$
h_{t+1}=f(W_{xh}x_{t+1}+W_{hh}h_t+b_h)
$$


- **下一词预测**：看 `$y_t$`，对它做 softmax 后预测下一个词。
- **下一次隐藏状态**：用 `$h_t$` 传给下一步。
- **`$y_t$` 不是隐藏状态**，它是输出层结果。
- 除非你显式设计“输出反馈”结构，否则不要把 `$y_t$` 当作上一次隐藏状态。


$$
\boxed{\text{预测下一词用 } y_t;\quad \text{下一步隐藏状态用 } h_t}
$$

代码感觉就是：

```python
logits = W_hy @ h_t + b_y
probs = softmax(logits)
next_word = sample(probs)

h_next = rnn_cell(x_next, h_t)   # 这里用的是 h_t，不是 y_t
```

**神经元工作机制总结：**

- **接收输入**：每个RNN神经元接收来自输入数据`$x_t$`和前一时刻的隐藏状态`$h_{t-1}$`。

- **更新隐藏状态**：神经元通过一个加权和（由权重矩阵和偏置项组成）更新当前时刻的隐藏状态`$h_t$`，该隐藏状态包含了来自过去的记忆以及当前输入的信息。

- **输出计算**：基于当前隐藏状态`$h_t$`，神经元生成当前时刻的输出`$y_t$`，该输出可以用于任务的最终预测。


{% asset_img 173540137329.png "Hexo 博客封面示例" %}


**文本生成示例：**

假设我们使用RNN进行**文本生成**，输入是一个初始词语或一段上下文（例如，“m”）。RNN会通过隐藏状态逐步生成下一个词的概率分布，然后根据概率选择最可能的下一个词。

{% asset_img 1735443424646.png "Hexo 博客封面示例" %}

1. 输入：“m” → 词向量输入`$x_1$`（对应“m”）

2. 初始化隐藏状态`$h_0$`，一般初始值为0

3. 隐藏状态更新`$h_1$`，并计算输出`$y_1$`

4. 经过全连接层输出层计算输出`$y_{pred}$`，使用softmax函数将`$y_{pred}$`转换为概率分布

5. 选择概率最高的词作为输出词（例如“a”）

6. 输入新的词“a”，继续处理下一个时间步，直到生成完整的词或句子


**小结：**在循环神经网络中，**词与输出的对应关系**通常通过以下几个步骤建立

1. **词嵌入**：将词转化为向量表示（词向量）。
2. **RNN处理**：通过RNN层逐步处理词向量，生成每个时间步的隐藏状态。
3. **输出映射**：通过线性变换将隐藏状态映射到输出，通常是一个词汇表中的词的概率分布。

###  PyTorch RNN层的使用

- API介绍

  ```python
  RNN = nn.RNN(input_size, hidden_size，num_layers)
  ```

  参数意义是：

  - input_size：输入数据的维度，一般设为词向量的维度
  - hidden_size：隐藏层h的维度，也是当前层神经元的输出维度
  - num_layers: 隐藏层h的层数，默认为1

- 输入数据和输出结果

  将RNN实例化就可以将数据送入其中进行处理，处理的方式如下所示：

  ```python
  output, hn = RNN(x, h0)
  ```
传入**输入x**，和**上一个隐藏状态h0**，传出output和当前层隐藏状态

{% asset_img Snipaste_2026-10-10_20-55-26.png "Hexo 博客封面示例" %}

  - **输入数据：主要包括词嵌入的x 、初始的隐藏层h0**
    - x的表示形式为[seq_len, batch, input_size]，即[句子的长度，batch的大小，词向量的维度]
    - h0的表示形式为[num_layers, batch, hidden_size]，即[隐藏层的层数，batch的大小，隐藏层h的维度]
  - **输出结果：主要包括输出结果output,最后一层的hn**
    - output的表示形式与输入x类似，为[seq_len, batch, input_size]，即[句子的长度，batch的大小，输出向量的维度]
    - hn的表示形式与输入h0一样，为[num_layers, batch, hidden_size]，即[隐藏层的层数，batch的大，隐藏层h的维度]
- API使用

{% asset_img Snipaste_2026-10-10_20-57-06.png "Hexo 博客封面示例" %}

  ```python
  import torch
  import torch.nn as nn
  
  
  #  RNN层送入批量数据
  def test():
      # 词向量维度 128, 隐藏向量维度 256
      rnn = nn.RNN(input_size=128, hidden_size=256)
      # 第一个数字: 表示句子长度,也就是词语个数
      # 第二个数字: 批量个数，也就是句子的个数
      # 第三个数字: 词向量维度
      inputs = torch.randn(5, 32, 128)
      hn = torch.zeros(1, 32, 256)
      # 获取输出结果
      output, hn = rnn(inputs, hn)
      print("输出向量的维度：\n",output.shape)
      print("隐藏层输出的维度：\n",hn.shape)
  
      
  if __name__ == '__main__':
      test()
  ```

  **输出结果：**

  ```python
  输出向量的维度：
   torch.Size([5, 32, 256])
  隐藏层输出的维度：
   torch.Size([1, 32, 256])
  ```

这个配置是 5 个时间步、32 个样本、每个样本每步输入 128 维；RNN 隐藏记忆 256 维，只有 1 层。默认 batch_first=False，所以 batch 在中间那维。

课堂代码：
```python
"""
案例:
    演示RNN层(循环网络层)的API.

循环网络层作用:
    基于 上一次的隐藏状态 + 本次的输入 -> 本次的隐藏状态, 本次的输出.

    公式:
        本次的隐藏状态 = tanh(上次的隐藏状态加权求和 + 本次的输入 加权求和)
        本次的输出 = 本次的隐藏状态加权求和, 有词汇表中所有词的概率, 选概率最大的哪个词作为 最终预测结果.

简单总结下RNN:
    词嵌入层:
        将词(词的索引) 转换为 词向量表示.
    RNN层(循环网络层):
        逐步处理词向量, 生成 每个时间步的 隐藏状态.
    全连接层(输出映射):
        通过线性变换将隐藏状态映射到输出, 通常是1个词汇表中词的概率分布.
"""

# 导包
import torch
import torch.nn as nn

# 大白话: RNN就像是你的大脑, 在看电影的过程中 记住剧情.

# 1. 创建 循环网络层.
# 参1: 词向量的维度, 就像电影里每一帧画面有128个细节(比如: 颜色, 动作, 表情等), RNN(你的大脑)每次看到的就是这128个细节.
# 参2: 隐藏状态向量维度, 你的大脑能记住的剧情信息量有多大, 比如说: 记住角色关系, 前景提要, 值越大, 能存储的剧情记忆就越多.
# 参3: 隐藏层数量, 默认是1, 你的大脑只有1层来处理它, 如果是1, 说明2层大脑接力理解剧情, 第1层处理, 第2层深加工.
rnn = nn.RNN(input_size=128, hidden_size=256, num_layers=1)

# 2. 定义变量, 表示输入的 x
# 参1: 每个句子的词的个数(长度),  假设电影总共有 5帧画面.
# 参2: 句子的数量,              同时有32个人在看这 5帧画面.
# 参3: 词向量的维度.            每一帧画面的细节是128个.
x = torch.randn(size=(5, 32, 128))

# 3. 定义变量, 记录: 上一时刻的隐藏状态.
# 参1: 隐藏层的层数,   RNN的层数, 和num_layers一样, 1层大脑.
# 参2: 句子的数量,     批量大小, 和上述的 x一样, 32个人...
# 参3: 隐藏状态向量维度,  和上述的 hidden_size一样, 256维.
h0 = torch.randn(size=(1, 32, 256))

# 4. 调用RNN处理, 获取到当前时刻的预测值 和 当前的隐藏状态.
# 参1: x, 本次的输入,  参2: h0, 上一次的隐藏状态.

# 返回值1 output: 每个时间步的输出, 包含了所有时间步的隐藏状态.  每看1帧画面, 大脑能记住的剧情 -> 共5帧, 32个人, 256维.
# 返回值2 h1: 最后1个时间步的隐藏状态.                       看完最后1帧画面后, 大脑里的最新剧情记忆 -> 只关注最后1帧.
output, h1 = rnn(x, h0)
print(f'output: {output.shape}')    # [5, 32, 256]
print(f'h1: {h1.shape}')           # [1, 32, 256]
```

rnn = nn.RNN(input_size=128, hidden_size=256, num_layers=1)

{% asset_img Snipaste_2026-10-10_21-07-54.png "Hexo 博客封面示例" %}

{% asset_img Snipaste_2026-10-10_21-11-29.png "Hexo 博客封面示例" %}

{% asset_img Snipaste_2026-10-10_21-13-24.png "Hexo 博客封面示例" %}

{% asset_img Snipaste_2026-10-10_21-14-02.png "Hexo 博客封面示例" %}

{% asset_img Snipaste_2026-10-10_21-15-02.png "Hexo 博客封面示例" %}

## 文本生成案例

文本生成任务是一种常见的自然语言处理任务，输入一个开始词能够预测出后面的词序列。本案例将会使用循环神经网络来实现周杰伦歌词生成任务。


{% asset_img 1734750013375.png "Hexo 博客封面示例" %}

### 导入工具包

```python
import torch
import jieba
from torch.utils.data import DataLoader
import torch.nn as nn
import torch.optim as optim
import time
```

### 数据集

我们收集了周杰伦从第一张专辑《Jay》到第十张专辑《跨时代》中的歌词，来训练神经网络模型，当模型训练好后，我们就可以用这个模型来创作歌词。数据集如下：

```python
想要有直升机
想要和你飞到宇宙去
想要和你融化在一起
融化在宇宙里
我每天每天每天在想想想想著你
这样的甜蜜
让我开始相信命运
感谢地心引力
让我碰到你
漂亮的让我面红的可爱女人
...
```

该数据集共有 5819 行文本。

### 构建词表

在进行自然语言处理任务之前，首要做的就是构建词表。

所谓的词表就是将语料进行分词，然后给每一个词分配一个唯一的编号，便于我们送入词嵌入层。


{% asset_img 1734750711996.png "Hexo 博客封面示例" %}

接下来, 我们对周杰伦歌词的数据进行处理构建词表，具体流程如下：

- 获取文本数据
- 分词，并进行去重
- 构建词表

```python
# 获取数据，并进行分词，构建词表
def build_vocab():
    # 数据集位置
    file_name = 'data/jaychou_lyrics.txt'
    # 分词结果存储位置
    # 唯一词列表
    unique_words = []

    # 每行文本分词列表
    all_words = []

    # 遍历数据集中的每一行文本
    for line in open(file_name, 'r', encoding='utf-8'):
        # 使用jieba分词,分割结果是一个列表
        words = jieba.lcut(line)
        # print(words)
        # 所有的分词结果存储到all_words，其中包含重复的词组
        all_words.append(words)
        # 遍历分词结果，去重后存储到unique_words
        for word in words:
            if word not in unique_words:
                unique_words.append(word)
    # 语料中词的数量
    word_count = len(unique_words)
    # 词到索引映射
    word_to_index = {word: idx for idx, word in enumerate(unique_words)}
    # 歌词文本用词表索引表示
    corpus_idx = []
    # 遍历每一行的分词结果
    for words in all_words:
        temp = []
        # 获取每一行的词，并获取相应的索引
        for word in words:
            temp.append(word_to_index[word])
        # 在每行词之间添加空格隔开
        temp.append(word_to_index[' '])
        # 获取当前文档中每个词对应的索引
        corpus_idx.extend(temp)
    return unique_words, word_to_index, word_count, corpus_idx


if __name__ == "__main__":
    # 获取数据
    unique_words, word_to_index, unique_word_count, corpus_idx = build_vocab()
    print("词的数量：\n",unique_word_count)
    print("去重后的词:\n",unique_words)
    print("每个词的索引：\n",word_to_index)
    print("当前文档中每个词对应的索引：\n",corpus_idx)
```

我们的词典主要包含了:

- unique_words: 存储了每个词

- word_to_index: 存储了词到编号的映射

- corpus_idx: 全为数字（索引）的列表

{% asset_img 1734751188623.png "Hexo 博客封面示例" %}


这段代码是自然语言处理（NLP）中非常经典的**数据预处理步骤**：将人类可读的文本（歌词），转换成机器（神经网络）能够处理的数字索引。

### 🔹 变量逐一解析

1. **`corpus_idx` (Corpus Index, 语料库索引)**
   * **初始值**：`[]`（空列表）。
   * **作用**：这是最终输出的**全局大列表**。它会把所有歌曲转换后的数字索引全部“平铺”装进去，形成一整条超长的数字序列。

2. **`all_words`**
   * **来源**：通常是前面的代码处理好的结果。
   * **形态**：它是一个**二维列表**（列表的列表）。外层列表代表所有的歌，内层列表代表这首歌分词后的所有词。
   * **例如**：`[['今天', '天气', '好'], ['我', '很', '开心']]`。

3. **`words`**
   * **来源**：外层循环 `for words in all_words` 每次迭代取出的元素。
   * **含义**：代表**当前正在处理的一首歌词**（一个包含多个词的列表）。比如第一轮循环，`words = ['今天', '天气', '好']`。

4. **`tmp` (Temporary, 临时变量)**
   * **初始值**：`[]`（空列表）。
   * **作用**：用来**暂时存储当前这一首歌词**转换后的数字索引。每处理完一首歌，它就会被清空重置（因为在循环内部重新赋值 `[]` 了）。

5. **`word`**
   * **来源**：内层循环 `for word in words` 每次迭代取出的元素。
   * **含义**：代表**当前正在处理的单个词**（字符串）。比如 `'今天'`。

6. **`word_to_index`**
   * **来源**：通常是前面构建好的一个**字典 (Dictionary)**。
   * **形态**：键值对映射表，比如 `{'今天': 0, '天气': 1, '好': 2, ' ': 3, ...}`。

---

### 代码逐行逻辑

```python
# 1. 初始化最终存放所有索引的大列表
corpus_idx = []

# 2. 遍历每一首歌 (all_words 是二维列表)
for words in all_words:
    
    # 3. 为当前这首歌创建一个临时列表，存这首歌的索引
    tmp = []
    
    # 4. 遍历这首歌中的每一个词
    for word in words:
        # 5. 去字典 word_to_index 里查这个词对应的数字，加到 tmp 列表中
        tmp.append(word_to_index[word]) # 词 → 索引
        
    # 6. 【关键细节】给这首歌的末尾手动补上一个“空格”的索引
    tmp.append(word_to_index[' ']) # 每行末尾加空格
    
    # 7. 将当前这首歌的索引列表(tmp)，拼接到全局大列表(corpus_idx)的末尾
    corpus_idx.extend(tmp)
```

---


**1. 为什么要在每行末尾加一个空格的索引？**
因为模型是滑动窗口采样（参照你上一张图），如果不加分隔符，上一首歌的最后一个词和下一首歌的第一个词就会被连在一起。
`... ['好', '我'] ...` 模型会学到“好”后面接“我”。但这其实是两首不同的歌。加入空格索引后，序列变成 `... ['好', ' ', '我'] ...`，模型就能学会“空格”代表句子/歌曲的结束，这在生成文本时非常有用（模型知道什么时候该停）。

**2. `append` 和 `extend` 的区别（最容易出错的地方）：**
*   如果写成 `corpus_idx.append(tmp)`，结果会是 `[[0, 1, 2], [5, 6, 7]]`（变成了二维列表，嵌套了）。
*   代码用的是 `corpus_idx.extend(tmp)`，结果会是 `[0, 1, 2, 5, 6, 7]`（打平成一维列表）。这正是构造训练数据 `x = [0, 1, 2, 5, 6, ...]` 所需要的形态。

### 🎯 举个具体数字的例子演练：
假设：
* `all_words = [['我', '爱'], ['你']]`
* `word_to_index = {'我': 0, '爱': 1, '你': 2, ' ': 3}`

执行过程：
1. 第一轮外层循环 `words = ['我', '爱']`
   * `tmp` 变成 `[0, 1]`
   * 末尾加空格：`tmp` 变成 `[0, 1, 3]`
   * `corpus_idx` 变成 `[0, 1, 3]`
2. 第二轮外层循环 `words = ['你']`
   * `tmp` 重置为 `[]`，然后变成 `[2]`
   * 末尾加空格：`tmp` 变成 `[2, 3]`
   * `corpus_idx` 变成 `[0, 1, 3, 2, 3]`

最终，这段代码输出了一条超长的连续数字 `[0, 1, 3, 2, 3]`，随后就会被送去做滑动窗口采样，构造出 `x` 和 `y` 喂给 RNN 模型训练。


    # 6. 遍历每一行的分词结果.
    for words in all_words:
{% asset_img Snipaste_2026-10-10_22-55-15.png "Hexo 博客封面示例" %}

### 构建数据集对象

我们在训练的时候，为了便于读取语料，并送入网络，所以我们会构建一个Dataset对象

构建流程：
一次输入5个词（索引）
{% asset_img Snipaste_2026-10-10_23-13-16.png "Hexo 博客封面示例" %}


```python
class LyricsDataset(torch.utils.data.Dataset):
	def __init__(self, corpus_idx, num_chars):
		# 文档数据中词的索引
		self.corpus_idx = corpus_idx
		# 每个句子中词的个数
		self.num_chars = num_chars
		# 文档数据中词的数量，不去重
		self.word_count = len(self.corpus_idx)
		# 句子数量
		self.number = self.word_count // self.num_chars

	# len(obj)时自动调用此方法
	def __len__(self):
		# 返回句子数量
		return self.number

	# obj[idx]时自动调用此方法
	def __getitem__(self, idx):
		# idx指词的索引，并将其修正索引值到文档的范围里面
		"""
		我 爱你 中国 , 亲爱 的 母亲
		word_count: 7
		num_chars: 2 一个句子由num_chars个词组成
		word_count-num_chars-2: 7-2-1=4  -1:网络预测结果y在x上后移一个词取值-1
		idx=5->start=4
		"""
		start = min(max(idx, 0), self.word_count - self.num_chars - 1)
		end = start + self.num_chars
		# 输入值
		x = self.corpus_idx[start: end]
		# 网络预测结果（目标值）
		y = self.corpus_idx[start + 1: end + 1]
		# 返回结果
		return torch.tensor(x), torch.tensor(y)


if __name__ == "__main__":
	# 获取数据
	unique_words, word_to_index, unique_word_count, corpus_idx = build_vocab()
	# 数据获取实例化
	dataset = LyricsDataset(corpus_idx, 5)
	# 查看句子数量
	print('句子数量：', len(dataset))
	# x, y = dataset.__getitem__(0)
	x, y = dataset[0]
	print("网络输入值：", x)
	print("目标值：", y)
```

**输出结果：**

```python
句子数量： 9827
网络输入值： tensor([ 0,  1,  2,  3, 40])
目标值： tensor([ 1,  2,  3, 40,  0])
```

### 构建网络模型

我们用于实现《歌词生成》的网络模型，主要包含了三个层:

- 词嵌入层: 用于将语料转换为词向量

- 循环网络层: 提取句子语义

- 全连接层: 输出对词典中每个词的预测概率

```python
# 模型构建
class TextGenerator(nn.Module):
    def __init__(self, unique_word_count):
        super(TextGenerator, self).__init__()
        # 初始化词嵌入层: 语料中词的数量, 词向量的维度为128
        self.ebd = nn.Embedding(unique_word_count, 128)
        # 循环网络层: 词向量维度128, 隐藏向量维度256, 网络层数1
        self.rnn = nn.RNN(128, 256, 1)
        # 输出层: 特征向量维度256与隐藏向量维度相同, 词表中词的个数
        self.out = nn.Linear(256, unique_word_count)
        
    def forward(self, inputs, hidden):
        # 输出维度: (batch, seq_len, 词向量维度128)
        # batch：句子数量
        # seq_len： 句子长度， 每个句子由多少个词 词数量
        embed = self.ebd(inputs)
        # rnn层x的表示形式为(seq_len, batch, 词向量维度128)
        # output的表示形式与输入x类似，为(seq_len, batch, 词向量维度256)
        # 前后的hidden形状要一样, 所以DataLoader加载器的batch数要能被整数
        output, hidden = self.rnn(embed.transpose(0, 1), hidden)
        # 全连接层输入二维数据， 词数量*词维度
        # 输入维度: (seq_len*batch, 词向量维度256) 
        # 输出维度: (seq_len*batch, 语料中词的数量)
        # output: 每个词的分值分布，后续结合softmax输出概率分布
        output = self.out(output.reshape(shape=(-1, output.shape[-1])))
        # 网络输出结果
        return output, hidden
    
    def init_hidden(self, bs):
        # 隐藏层的初始化:[网络层数, batch, 隐藏层向量维度]
        return torch.zeros(1, bs, 256)
    
    
if __name__ == "__main__":
	# 获取数据
	unique_words, word_to_index, unique_word_count, corpus_idx = build_vocab()
	model = TextGenerator(unique_word_count)
	for named, parameter in model.named_parameters():
		print(named)
		print(parameter)
```

### 构建训练函数

前面的准备工作完成之后, 我们就可以编写训练函数。训练函数主要负责编写数据迭代、送入网络、计算损失、反向传播、更新参数，其流程基本较为固定。

由于我们要实现文本生成，文本生成本质上，输入一串文本，预测下一个文本，也属于分类问题，所以，我们使用多分类交叉熵损失函数。优化方法我们学习过 SGB、AdaGrad、Adam 等，在这里我们选择学习率、梯度自适应的 Adam 算法作为我们的优化方法。

训练完成之后，我们使用 torch.save 方法将模型持久化存储。

```python
def train():
	# 构建词典
	unique_words, word_to_index, unique_word_count, corpus_idx = build_vocab()
	# 数据集 LyricsDataset对象，并实现了 __getitem__ 方法
	lyrics = LyricsDataset(corpus_idx=corpus_idx, num_chars=32)
	# 查看句子数量
	# print(lyrics.number)
	# 初始化模型
	model = TextGenerator(unique_word_count)
	# 数据加载器 DataLoader对象，并将lyrics dataset对象传递给它
	lyrics_dataloader = DataLoader(lyrics, shuffle=True, batch_size=5)
	# 损失函数
	criterion = nn.CrossEntropyLoss()
	# 优化方法
	optimizer = optim.Adam(model.parameters(), lr=1e-3)
	# 训练轮数
	epoch = 10
	for epoch_idx in range(epoch):
		# 训练时间
		start = time.time()
		iter_num = 0  # 迭代次数
		# 训练损失
		total_loss = 0.0
		# 遍历数据集 DataLoader 会在后台调用 dataset.__getitem__(index) 来获取每个样本的数据和标签，并将它们组合成一个 batch
		for x, y in lyrics_dataloader:
			# 隐藏状态的初始化
			hidden = model.init_hidden(bs=5)
			# 模型计算
			output, hidden = model(x, hidden)
			# 计算损失
			# y形状为(batch, seq_len), 需要转换成一维向量->160个词的下标索引
            # output形状为(seq_len, batch, 词向量维度)
            # 需要先将y进行维度交换(和output保持一致)再改变形状
            y = torch.transpose(y, 0, 1).reshape(shape=(-1,))
			loss = criterion(output, y)
			optimizer.zero_grad()
			loss.backward()
			optimizer.step()
			iter_num += 1  # 迭代次数加1
			total_loss += loss.item()
		# 打印训练信息
		print('epoch %3s loss: %.5f time %.2f' % (epoch_idx + 1, total_loss / iter_num, time.time() - start))
	# 模型存储
	torch.save(model.state_dict(), 'model/lyrics_model_%d.pth' % epoch)

    
if __name__ == "__main__":
	train()
```

**输出结果：**

```python
epoch   1 loss: 1.84424 time 5.75
epoch   2 loss: 0.21154 time 5.91
epoch   3 loss: 0.12014 time 5.85
epoch   4 loss: 0.10625 time 5.73
epoch   5 loss: 0.10226 time 5.58
epoch   6 loss: 0.10009 time 5.65
epoch   7 loss: 0.09942 time 5.66
epoch   8 loss: 0.09783 time 5.66
epoch   9 loss: 0.09663 time 5.75
epoch  10 loss: 0.09568 time 5.77
```

### 构建预测函数

从磁盘加载训练好的模型，进行预测。预测函数，输入第一个指定的词，我们将该词输入网路，预测出下一个词，再将预测的出的词再次送入网络，预测出下一个词，以此类推，知道预测出我们指定长度的内容。

```python
def predict(start_word, sentence_length):
    # 构建词典
    unique_words, word_to_index, unique_word_count, _ = build_vocab()
    # 构建模型
    model = TextGenerator(unique_word_count)
    # 加载参数
    model.load_state_dict(torch.load('model/lyrics_model_10.pth'))
    # 隐藏状态
    hidden = model.init_hidden(bs=1)
    # 将起始词转换为索引
    word_idx = word_to_index[start_word]
    # 产生的词的索引存放位置
    generate_sentence = [word_idx]
    # 遍历到句子长度，获取每一个词
    for _ in range(sentence_length):
        # 模型预测
        output, hidden = model(torch.tensor([[word_idx]]), hidden)
        # 获取预测结果
        word_idx = torch.argmax(output)
        generate_sentence.append(word_idx)
        # 根据产生的索引获取对应的词，并进行打印
    for idx in generate_sentence:
        print(unique_words[idx], end='')
        
        
if __name__ == '__main__':
    # 调用预测函数
    predict('分手', 50)
```

**输出结果：**

```python
分手的话像语言暴力
 我已无能为力再提起 决定中断熟悉
 周杰伦 周杰伦
 一步两步三步四步望著天 看星星
 一颗两颗三颗四颗 连成线背著背默默许下心愿
 看远方的星
```

### 课堂代码：
```python
"""
案例:
    RNN案例, 基于杰伦歌词来训练模型, 用给定的起始词, 结合长度, 来进行 AI歌词生成.

实现步骤:
    1. 获取数据, 进行分词, 获取词表.
    2. 数据预处理, 构建数据集.
    3. 搭建RNN神经网络.
    4. 训练模型.
    5. 模型预测.
"""

# 导包
import torch
import jieba
from torch.utils.data import DataLoader
import torch.nn as nn
import torch.optim as optim
import time


# 1. 获取数据, 进行分词, 获取词表.
def build_vocab():

    # 1. 定义变量, 记录: 去重后所有的词, 每行文本分词结果.
    unique_words, all_words = [], []

    # 2. 遍历数据集, 获取到每行文本.
    for line in open('./data/jaychou_lyrics.txt', 'r', encoding='utf-8'):

        # 2.1 获取到每行歌词, 进行分词.
        words = jieba.lcut(line)  # ['想要', '有', '直升机', '\n']

        # 2.2 所有分词结果记录到  all_words 中.
        all_words.append(words)  # [['想要', '有', '直升机', '\n'], [第2句歌词切词], ......]
       
        # 2.3 遍历分词结果, 去重后, 添加到unique_words中.
        for word in words:
            if word not in unique_words:
                unique_words.append(word)

    # 3. 统计语料中(去重后)词的数量.
    word_count = len(unique_words)  # 5703个词
    
    # 4. 构建词表, 字典形式, key是词, value是词的索引.
    # 例如:  {'想要': 0, '有': 1, '直升机': 2, '\n': 3, ...'冠军': 5701, '要大卖': 5702}
    word_to_index = {word: i for i, word in enumerate(unique_words)}
   
    # 5. 歌词文本用词表索引表示.
    corpus_idx = []

    # 6. 遍历每一行的分词结果.
    for words in all_words:

        # 6.1 定义变量, 记录: 词索引列表.
        tmp = []

        # 6.2 获取每一行的词, 并获取相应的索引.
        for word in words:
            tmp.append(word_to_index[word]) #通过键（真实分出来的文字字段）索引那个列表里面的值（词对应的索引）

        # 6.3 在每行词之间, 添加空格隔开.
        tmp.append(word_to_index[' '])

        # 6.4 获取文档中每个词的索引, 添加到corpus_idx中.
        corpus_idx.extend(tmp)

    # 7. 返回结果: 唯一词列表(5703个词), 词表 {'想要': 0, '有': 1, ... '要大卖': 5702},  (去重后)词的数量, 歌词文本用词表索引表示.
    return unique_words, word_to_index, word_count, corpus_idx


# 2. 数据预处理, 构建数据集.
# 定义数据集类, 继承 torch.utils.data.Dataset
class LyricsDataset(torch.utils.data.Dataset):

    # 1. 初始化词索引, 词个数等...
    def __init__(self, corpus_idx, num_chars):

        # 1.1 文档数据中词的索引
        self.corpus_idx = corpus_idx

        # 1.2 每个句子中词的个数.
        self.num_chars = num_chars

        # 1.3 文档数据中词的数量, 不去重.
        self.word_count = len(self.corpus_idx)

        # 1.4 句子数量
        self.number = self.word_count // self.num_chars

    # 2. 当使用 len(obj)时, 自动调用此方法.
    def __len__(self):
        # 返回句子数量
        return self.number

    # 3. 当使用 obj[index]时, 自动调用此方法.
    def __getitem__(self, idx):
        # idx: 指的是词的索引, 并将其修正索引值 到 文档的范围里边.
        # 3.1 确保索引start在合法范围内, 避免越界, start: 当前样本的起始索引.
        start = min(max(idx, 0), self.word_count - self.num_chars - 1) # 跟ReLU激活函数差不多，样本数减少句子长度是确保输入索引比在范围内，多减一个1，目的是防止预测词索引越界


        # 3.2 计算当前样本的结束索引.
        end = start + self.num_chars

        # 3.3 输入值, 从文档中取出 start ~  end 的索引的词 -> 作为 x（输入）
        x = self.corpus_idx[start:end]

        # 3.4 输出值, 网络预测结果.
        y = self.corpus_idx[start + 1:end + 1] #参考一下给的图片，预测是比输入右移一格的

        # 3.5 返回输入值和输出值 -> 张量形式.
        return torch.tensor(x), torch.tensor(y)


# 3. 搭建RNN神经网络.
class TextGenerator(nn.Module):
    # 1. 初始化方法
    def __init__(self, unique_word_count):      # unique_word_count: 去重的词的数量(5703)
        # 1.1 初始化父类的成员.
        super().__init__()
        # 1.2 初始化词嵌入层: 语料中词的数量, 词向量的维度.
        self.ebd = nn.Embedding(unique_word_count, 128)
        # 1.3 循环网络层: 词向量维度, 隐藏层维度: 256, 网络层数: 1
        self.rnn = nn.RNN(128, 256, 1)
        # 1.4 输出层(全连接层):  特征向量维度(和隐藏向量维度一致), 词表中词的个数.
        self.out = nn.Linear(256, unique_word_count)    # 词表中每个词的概率 -> 选概率最大的哪个词作为 预测结果.

    # 2. 前向传播方法
    def forward(self, inputs, hidden):
        # 2.1 初始化 词嵌入层处理.
        # embd格式: (batch句子的数量, 句子的长度, 词向量维度)
        embd = self.ebd(inputs)
        # print(f'embd.shape: {embd.shape}')

        # 2.2 rnn处理
        # rnn格式: (句子的长度, batch句子的数量, 隐藏层维度)
        output, hidden = self.rnn(embd.transpose(0, 1), hidden)

        # 2.3 全连接, 输入内容必须是二维数据, 即: 词的数量 * 词的维度
        # 输入维度: (seq_len句子数量 * batch, 词向量维度256)
        # 输出维度: (seq_len句子数量 * batch, 词表中词的个数)
        output = self.out(output.reshape(shape=(-1, output.shape[-1])))
        # 2.4 返回结果, 预测结果, 隐藏层.
        return output, hidden

    # 3. 隐藏层的初始化方法.
    def init_hidden(self, bs):      # batch_size
        # 隐藏层初始化: [网络层数, batch, 隐藏层向量维度]
        return torch.zeros(1, bs, 256)


# 4. 训练模型.
def train():
    # 1. 构建词典.
    unique_words, word_to_index, unique_word_count, corpus_idx = build_vocab()
    # 2. 获取数据集.
    lyrics = LyricsDataset(corpus_idx, 32)
    # 3. 初始化(神经网络)模型
    model = TextGenerator(unique_word_count)        # 预测5703个词, 每个词的概率.
    # 4. 创建数据加载器对象.
    # 参1: 数据集对象.  参2: 批次大小(每批5个句子, 每个句子32个词)  参3: 是否打乱数据.
    lyrics_dataloader = DataLoader(lyrics, batch_size=5, shuffle=True)
    # 5. 定义损失函数
    criterion = nn.CrossEntropyLoss()
    # 6. 定义优化器.
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
    # 7. 模型训练.
    # 7.1 定义变量, 记录训练的轮数.
    epochs = 50
    # 7.2 具体的每轮训练动作.
    for epoch in range(epochs):     # epoch: 0, 1, 2, 3...9, 分别表示: 第1轮, 第2轮, ... 第10轮.
        # 7.3 定义变量记录: 本轮开始训练时间, 迭代(批次)次数, 训练总损失.
        start, iter_num, total_loss = time.time(), 0, 0.0
        # 7.4 具体的 本轮 各批次 训练动作.
        # 遍历数据集, 后台会调用 LyricsDataset#__getitem__()方法, 获取到每个样本的数据和标签,
        for x, y in lyrics_dataloader:
            #每次拿5条（batch_size），每条长32个索引

            # 7.5 获取隐藏层初始值.
            hidden = model.init_hidden(5)

            # 7.6 模型计算.
            output, hidden = model(x, hidden)

            # 7.7 计算损失.
            # y的形状: (batch 批次数, seq_len 句子长度, 词向量维度) -> 转成一维向量 -> 每个词的下标索引.
            # output形状为: (seq_len, batch, 词向量维度)
            y = torch.transpose(y, 0, 1).reshape(shape=(-1, ))
            loss = criterion(output, y)

            # 7.8 梯度清零 + 反向传播 + 更新参数.
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

            # 7.9 累计损失 和 迭代次数.
            total_loss += loss.item()
            iter_num += 1

        # 7.10 走到这里, 说明 本轮训练结束, 打印本轮的训练信息.
        print(f'epoch: {epoch + 1}, time: {time.time() - start:.2f}s, loss: {total_loss / iter_num:.4f}')

    # 8. 走到这里, 说明多轮训练结束(模型训练结束), 保存即可.
    torch.save(model.state_dict(), './model/text_generator.pth')


# 5. 模型预测.
def evaluate(start_word, sentence_length):
    # 1. 构建词典.
    unique_words, word_to_index, unique_word_count, corpus_idx = build_vocab()
    # 2. 获取模型.
    model = TextGenerator(unique_word_count)
    # 3. 加载模型参数.
    model.load_state_dict(torch.load('./model/text_generator.pth'))
    # 4. 获取隐藏层初始值.
    hidden = model.init_hidden(1)
    # 5. 将输入的 开始词 转换成 索引.
    word_idx = word_to_index[start_word]
    # 6. 定义列表, 存放: 产生的词的索引.
    generate_sentence = [word_idx]  # 开始词的索引, 是列表的: 第1个值.
    # 7. 遍历句子长度, 获取到每一个词.
    for i in range(sentence_length):
        # 7.1 模型预测.
        output, hidden = model(torch.tensor([[word_idx]]), hidden)
        # 7.2 获取预测结果.   argmax() 从所有结果(5703个词的概率)中, 找最大值对应的索引.
        word_idx = torch.argmax(output)
        # 7.3 把预测结果添加到列表中.
        generate_sentence.append(word_idx)

    # 8. 将索引转成词, 并打印.
    for idx in generate_sentence:
        print(unique_words[idx], end='')



# 6. 测试
if __name__ == '__main__':
    # 1. 获取数据, 进行分词, 获取词表.
    # unique_words, word_to_index, word_count, corpus_idx = build_vocab()
    # print(f'词的数量: {word_count}')         # 去重后, 5703个词
    # print(f'去重后的词: {unique_words}')     # ['想要', '有', '直升机', '\n', '和', '你'...'冠军', '要大卖']
    # print(f'每个词的索引: {word_to_index}')  # 词表: {'想要': 0, '有': 1, '直升机': 2, '\n': 3, '和': 4, '你': 5, ... '冠军': 5701, '要大卖': 5702}
    # print(f'文档中每个词对应的索引: {corpus_idx}')  # [0, 1, 2, 1 3, 40, 0, 4, 5, 6, 7, 8, 3, 40, 0, 4, 5, 9, 10, 11, 3, 40,......]

    # 2. 构建数据集
    # dataset = LyricsDataset(corpus_idx, 5)
    # print(f'句子数量: {len(dataset)}') #由于一行词可能不止num_chars，再加上空格，切分导致产出句子数高于元数据集
    # # 查看下 输入值 和 目标值.
    # x, y = dataset[1]
    # print(f'输入值: {x}')  # [0, 1, 2, 3, 40]    [1, 2, 3, 40, 0]
    # print(f'目标值: {y}')  # [1, 2, 3, 40, 0]    [2, 3, 40, 0, 4]

    # 3. 创建模型对象.
    # model = TextGenerator(word_count)
    # # 查看参数.
    # for name, parameter in model.named_parameters():
    #     print(f'参数名称: {name}, 参数维度: {parameter.shape}')

    # 4. 训练(并保存)模型.
    # train()

    # 5. 测试模型.
    evaluate('星星', 50)
```

### 课堂代码解析
# 继续讲案例代码：从歌词到 AI 生成

你已经理解了 RNN 的 API 和参数，现在我们按代码的执行顺序，把整个案例串起来。

整个案例分 5 步：

```text
1. build_vocab()      -> 把歌词变成数字索引
2. LyricsDataset      -> 把索引切成一个个训练样本
3. TextGenerator      -> 搭建模型（Embedding + RNN + Linear）
4. train()            -> 训练模型
5. evaluate()         -> 用训练好的模型生成歌词
```

---

## 一、`build_vocab()`：把歌词变成数字

### 1. 它要解决什么问题

神经网络只认识数字，不认识文字。所以第一步是：

> 把每个词映射成一个唯一的整数。

### 2. 代码逐段看

```python
unique_words, all_words = [], []
```

- `unique_words`：去重后的词表，比如 `['想要', '有', '直升机', ...]`
- `all_words`：每一行歌词分词后的结果，比如 `[['想要', '有', '直升机'], ['第2句', ...], ...]`

```python
for line in open('./data/jaychou_lyrics.txt', 'r', encoding='utf-8'):
    words = jieba.lcut(line)
    all_words.append(words)
    for word in words:
        if word not in unique_words:
            unique_words.append(word)
```

逐行读歌词，用 `jieba` 分词，然后把没见过的词加进 `unique_words`。

```python
word_count = len(unique_words)  # 5703
word_to_index = {word: i for i, word in enumerate(unique_words)}
```

建立词到索引的映射：

```text
'想要' -> 0
'有'   -> 1
'直升机' -> 2
...
```

```python
corpus_idx = []
for words in all_words:
    tmp = []
    for word in words:
        tmp.append(word_to_index[word])
    tmp.append(word_to_index[' '])
    corpus_idx.extend(tmp)
```

把整份歌词变成一长串索引，每行末尾加一个空格索引作为分隔符。

### 3. 最终产出

```python
unique_words      # 5703 个去重后的词
word_to_index     # {'想要': 0, '有': 1, ...}
unique_word_count # 5703
corpus_idx        # [0, 1, 2, 1, 3, 40, 0, 4, ...] 一长串索引
```

`corpus_idx` 就是整份歌词的数字形式，后面所有处理都基于它。

---

## 二、`LyricsDataset`：把长索引切成训练样本

### 1. 它要解决什么问题

`corpus_idx` 是一长串，不能直接喂给模型。需要把它切成一段一段的。

```python
lyrics = LyricsDataset(corpus_idx, 32)
```

`num_chars=32`，意思是：

> 每个样本是长度为 32 的词序列。

### 2. `__init__`

```python
self.corpus_idx = corpus_idx
self.num_chars = num_chars       # 32
self.word_count = len(corpus_idx) # 总词数
self.number = self.word_count // self.num_chars  # 能切出多少个样本
```

### 3. `__getitem__`：最关键的部分

```python
def __getitem__(self, idx):
    start = min(max(idx, 0), self.word_count - self.num_chars - 1)
    end = start + self.num_chars
    x = self.corpus_idx[start:end]
    y = self.corpus_idx[start + 1:end + 1]
    return torch.tensor(x), torch.tensor(y)
```

假设 `idx=0`，`num_chars=32`：

```text
start = 0
end = 32

x = corpus_idx[0:32]   -> [w0, w1, w2, ..., w31]
y = corpus_idx[1:33]   -> [w1, w2, w3, ..., w32]
```

也就是说：

> 输入是 32 个词，目标是这 32 个词各自的下一个词。

这就是**自回归语言模型**的标准做法：

```text
给定 w0，预测 w1
给定 w1，预测 w2
...
给定 w31，预测 w32
```

### 4. 用 `DataLoader` 打包

```python
lyrics_dataloader = DataLoader(lyrics, batch_size=5, shuffle=True)
```

每次取 5 个样本，所以一个 batch：

```text
x: (5, 32)   -> (batch, seq_len)
y: (5, 32)   -> (batch, seq_len)
```

注意这里是 `(batch, seq_len)`，而 RNN 要的是 `(seq_len, batch, ...)`。  
后面 forward 里会做 transpose。

---

## 三、`TextGenerator`：模型结构

```python
class TextGenerator(nn.Module):
    def __init__(self, unique_word_count):
        super().__init__()
        self.ebd = nn.Embedding(unique_word_count, 128)
        self.rnn = nn.RNN(128, 256, 1)
        self.out = nn.Linear(256, unique_word_count)
```

三层：

| 层 | 作用 | 输入 | 输出 |
|---|---|---|---|
| `Embedding` | 词索引 -> 词向量 | `(batch, seq_len)` | `(batch, seq_len, 128)` |
| `RNN` | 词向量 -> 隐藏记忆 | `(seq_len, batch, 128)` | `(seq_len, batch, 256)` |
| `Linear` | 隐藏记忆 -> 词表分数 | `(N, 256)` | `(N, 5703)` |

### `forward`

```python
def forward(self, inputs, hidden):
    embd = self.ebd(inputs)                        # (5, 32, 128)
    output, hidden = self.rnn(embd.transpose(0, 1), hidden)  # (32, 5, 256)
    output = self.out(output.reshape(shape=(-1, output.shape[-1])))  # (160, 5703)
    return output, hidden
```

三步：

1. **Embedding**：把 `(5, 32)` 的词索引变成 `(5, 32, 128)` 的词向量。
2. **transpose + RNN**：把 `(5, 32, 128)` 换成 `(32, 5, 128)`，喂给 RNN，得到 `(32, 5, 256)`。
3. **reshape + Linear**：把 `(32, 5, 256)` 展平成 `(160, 256)`，再过全连接，得到 `(160, 5703)`。

### `init_hidden`

```python
def init_hidden(self, bs):
    return torch.zeros(1, bs, 256)
```

初始隐藏状态：

```text
(num_layers=1, batch=bs, hidden_size=256)
```

---

## 四、`train()`：训练模型

### 1. 准备阶段

```python
unique_words, word_to_index, unique_word_count, corpus_idx = build_vocab()
lyrics = LyricsDataset(corpus_idx, 32)
model = TextGenerator(unique_word_count)
lyrics_dataloader = DataLoader(lyrics, batch_size=5, shuffle=True)
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
```

### 2. 每一轮训练

```python
for epoch in range(epochs):
    start, iter_num, total_loss = time.time(), 0, 0.0
    for x, y in lyrics_dataloader:
        hidden = model.init_hidden(5)
        output, hidden = model(x, hidden)
        y = torch.transpose(y, 0, 1).reshape(shape=(-1, ))
        loss = criterion(output, y)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        total_loss += loss.item()
        iter_num += 1
    print(f'epoch: {epoch + 1}, time: {time.time() - start:.2f}s, loss: {total_loss / iter_num:.4f}')
```

### 3. 关键点：`y` 为什么要 transpose

```python
y = torch.transpose(y, 0, 1).reshape(shape=(-1, ))
```

- `output` 的顺序是 `(seq_len, batch)` 展平的。
- `y` 原本是 `(batch, seq_len)`。
- 顺序反了，必须先把 `y` 转成 `(seq_len, batch)`，再展平，才能和 `output` 一一对应。

### 4. 每个 batch 的流程

```text
x: (5, 32)
  -> Embedding: (5, 32, 128)
  -> transpose: (32, 5, 128)
  -> RNN: output (32, 5, 256), hidden (1, 5, 256)
  -> reshape: (160, 256)
  -> Linear: (160, 5703)

y: (5, 32)
  -> transpose: (32, 5)
  -> reshape: (160,)

loss = CrossEntropyLoss(output, y)
```

### 5. 训练结束

```python
torch.save(model.state_dict(), './model/text_generator.pth')
```

保存模型参数。

---

## 五、`evaluate()`：生成歌词

```python
def evaluate(start_word, sentence_length):
    unique_words, word_to_index, unique_word_count, corpus_idx = build_vocab()
    model = TextGenerator(unique_word_count)
    model.load_state_dict(torch.load('./model/text_generator.pth'))
    hidden = model.init_hidden(1)
    word_idx = word_to_index[start_word]
    generate_sentence = [word_idx]
    for i in range(sentence_length):
        output, hidden = model(torch.tensor([[word_idx]]), hidden)
        word_idx = torch.argmax(output)
        generate_sentence.append(word_idx)
    for idx in generate_sentence:
        print(unique_words[idx], end='')
```

### 逐步看

1. **构建词表**：和训练时一样，保证索引一致。
2. **加载模型**：用训练好的参数。
3. **初始化隐藏状态**：`init_hidden(1)`，因为只生成一个句子，batch=1。
4. **起始词转索引**：比如 `'星星' -> 1234`。
5. **循环生成**：

```python
output, hidden = model(torch.tensor([[word_idx]]), hidden)
word_idx = torch.argmax(output)
```

每次输入一个词，模型输出 5703 个词的分数，取分数最高的那个作为下一个词。

6. **把索引转回词并打印**。

### 预测时的形状

```text
输入:         (1, 1)        -> (batch=1, seq_len=1)
Embedding:    (1, 1, 128)
transpose:    (1, 1, 128)   -> (seq_len=1, batch=1, 128)
RNN output:   (1, 1, 256)
RNN hidden:   (1, 1, 256)
reshape:      (1, 256)
Linear:       (1, 5703)
argmax:       标量 -> 下一个词的索引
```

然后把新词拼进输入，继续预测下一个，循环 `sentence_length` 次。

---

## 六、整个案例的数据流总结

```text
原始歌词
  -> jieba 分词
  -> 去重，建立词表 word_to_index
  -> 整份歌词转成索引 corpus_idx

corpus_idx
  -> LyricsDataset 切成 (x, y) 样本对
  -> DataLoader 打包成 batch

训练时:
  x: (5, 32)
  -> Embedding: (5, 32, 128)
  -> transpose: (32, 5, 128)
  -> RNN: output (32, 5, 256), hidden (1, 5, 256)
  -> reshape: (160, 256)
  -> Linear: (160, 5703)
  -> 和 y 算 CrossEntropyLoss

预测时:
  输入一个词 (1, 1)
  -> 同样经过 Embedding + RNN + Linear
  -> 得到 (1, 5703)
  -> argmax 取下一个词
  -> 循环生成
```

---

## 七、几个容易忽略的细节

### 1. `LyricsDataset` 里的 `start` 为什么要 clamp

```python
start = min(max(idx, 0), self.word_count - self.num_chars - 1)
```

防止 `idx` 越界。因为 `DataLoader` 可能取到最后一个样本，如果直接 `start + 32` 会超出 `corpus_idx` 长度。

### 2. 为什么每行末尾加 `word_to_index[' ']`

```python
tmp.append(word_to_index[' '])
```

为了让不同歌词行之间有一个分隔符，模型能学到“一行结束”的概念。

### 3. 为什么训练时 `init_hidden(5)`，预测时 `init_hidden(1)`

因为训练时一个 batch 有 5 个样本，预测时一次只生成 1 个句子。  
隐藏状态的 batch 维必须和输入的 batch 维一致。

### 4. 为什么 `output.reshape(-1, 256)` 是 160

```text
seq_len * batch = 32 * 5 = 160
```

因为 RNN 输出是 `(32, 5, 256)`，前两维展平就是 160。

### 5. 为什么 `y` 也要 reshape 成 `(160,)`

因为 `output` 是 `(160, 5703)`，每一行对应一个位置。  
`y` 也必须是 `(160,)`，每个元素是那个位置的真实下一个词。

---

## 八、总结

这个案例的完整链路：

```text
歌词文本
  -> 分词、建词表
  -> 转成索引序列
  -> 切成 (x, y) 样本对
  -> Embedding 把词变成向量
  -> RNN 按时间步处理，产生隐藏记忆
  -> Linear 把记忆映射成 5703 个词的分数
  -> CrossEntropyLoss 训练
  -> 预测时每次取分数最高的词，循环生成
```

核心思想：

> 用前 32 个词预测第 33 个词，训练好之后，给定一个起始词，就能一个接一个地生成歌词。

这就是一个最基础的 **词级 RNN 语言模型**。