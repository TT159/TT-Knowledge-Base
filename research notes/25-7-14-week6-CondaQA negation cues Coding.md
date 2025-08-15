---
date: 2025-07-14
---

There are 219 unique negation cues, which is consistent with what is said in the article.

The frequency statistics show that they are extremely unbalanced. For example, 'not' appears 1390 times, while 'unhappily' only appears 6 times.

|方案|优点|缺点|
|---|---|---|
|**保留真实分布（不平衡）**|模型学习符合语言实际使用的频率；提升主流用法的泛化能力|模型可能忽略低频 cue，对少见否定不敏感，泛化差|
|**均衡所有 cue 的频率**|增强模型对所有 negation cue 的覆盖；更公平、全面|偏离现实语言分布，可能削弱主流 cue 的泛化能力，过拟合稀有构造|

这也许是一个empirical 问题？我们可以做对比实验，看效果。

实验一：**保留自然分布为基础训练集**；即不平衡


实验二：对低频 negation cues 做 **额外增强**， 提高其可见性；即让所有的negation cues 频次平等。

看看怎么样，会产生什么样的效果。



## affixal 定义

 1   `absence`

- 来源：拉丁语 _absentia_ ← _ab-_（away）+ _esse_（to be）
    
- 词根结构中带有前缀 **ab-**，表示**“远离”或“不存在”**。
    
- 然而，这里的否定意义并**不是由现代英语常见的否定词缀（如 un-, in-, dis-）构成**。
    

2      `absent`

- 同样来源于拉丁 _absens_ ← _ab-_（away）+ _esse_（to be）
- 表达“缺席”，但结构上并不包含我们通常认定的否定前缀。


### ❗ 关键区别：是否构成现代 **affixal negation**

| 条件                              | 是否满足                         |
| ------------------------------- | ---------------------------- |
| 含有现代可分解的否定前缀（un-, in-, dis- 等）？ | ❌ 否                          |
| 是否可以通过去掉前缀还原为一个**正向词**？         | ❌ 否：`sent` / `sence` 不是正向对应词 |
| 否定语义是否来自词缀构造？                   | ❌ 否，而是整体意义从源语中继承             |


|词汇|是否为 affixal negation|原因|
|---|---|---|
|`absence`|❌ 否|虽有“ab-”前缀，但不是现代英语中的否定构词，不能拆解为可对比的正向词|
|`absent`|❌ 否|同上，否定含义不是通过现代词缀构成的|


属于什么类型？这两个词属于：

- **Lexical negation** 或 **inherent negative concepts**：它们**整体表达否定意义**，但不是靠词缀构成；
    
- 在一些文献中，它们不被归入 affixal negation，也不算 canonical single-word negation（如 `not`、`never`）；
    
- 可以单独标为一类（如“inherent negation noun/adjective”）。

`absent` 中的 `sent` 是一个合法词，它在词典（如 NLTK 的 `words.words()`）里是存在的。但是！这并不意味着我们应该把 `absent` 判断为 affixal negation。

#### 那为什么 **不能** 把 `absent` 当成 affixal？

虽然 `sent` 是一个合法单词，但：

- `absent` 并不是由 **否定前缀 `a-` 或 `ab-` + sent** 构造出来的；
    
- `sent` 的意思是 “to send 的过去分词”，它与 `absent` 中的语义**毫无关联**；
    
- `absent` 的词源来自拉丁文 _absens_（_ab-_ "away" + _esse_ "to be"），意思是“being away”，不是 "not-sent"。


## 一、什么是 **Lexical Negation / Inherent Negative Concepts**？

这是指一些单词**整体上就表达否定意义**，但它们的否定含义**不是靠现代词缀构造出来的**，而是词本身就包含“否定”概念。

---

### 🧠 举例说明：

|单词|否定含义|构成方式|是否 affixal|
|---|---|---|---|
|`absent`|缺席 → 表示“没有在”|整体词就是这个意思，不靠前缀|❌ 不是 affixal|
|`deny`|否认 → 表示“不是”|不靠否定前缀或后缀|❌|
|`refuse`|拒绝 → 表示“不要”|整体构词，不拆分|❌|
|`loss`|损失 → 表示“失去”|不由 `un-`、`in-` 等构成|❌|
|`avoid`|回避 → 表达否定选择|同上|❌|

这类词的特点：

- 不能通过“去掉前缀”还原成一个正向词；
    
- 否定含义并不来自 `un-`, `in-`, `dis-` 这些常见前缀；
    
- 它们的否定性是 **“内在固有的”（inherent）**，不是派生构词。
    

---

## ✅ 二、什么是 **现代否定词缀（modern affixes）**？

这是语言学中对英语中活跃、生产性强的**标准否定前缀或后缀**的统称。它们用于构造新的否定词，是典型的 **affixal negation**。

### 常见现代否定词缀（你可以记下来）：

|前缀|含义|示例|
|---|---|---|
|`un-`|not|unhappy, unkind|
|`in-` / `im-` / `il-` / `ir-`|not / without|invalid, impossible, illegal, irregular|
|`dis-`|opposite of / not|dislike, disconnect|
|`non-`|not|nonhuman, nonviolent|
|`a-`|without / not (古希腊借入)|amoral, atypical, asexual|
|`-less`|without（后缀）|powerless, hopeless|

这些词缀具有：

- **规则性（可以广泛组合）**
    
- **否定语义显著**
    
- **可以与大量词根组合构词**

其中，否定后缀远比前缀少。

**`abnormal`** 并不被广泛归为 **现代英语中的典型 affixal negation**，因为 **`ab-` 本质上不是一个否定前缀**，它表示的是 **"away from", "off", "separate"**，而不是直接表达 “否定” 含义。

`ab-` 的含义：
- 来源：拉丁 _ab-_，表示 **"away from"**, **"off"**, **"separation"**；
    
- 常见于词根派生中表达 **偏离常规**，但不是否定一个事物本身的存在或真值。

---

## ✅ 总结一下两者区别

|类型|例子|是否靠词缀|否定性来源|
|---|---|---|---|
|**Affixal Negation**|`unhappy`, `insecure`, `disagree`, `asexual`|✅ 是|前缀或后缀构成|
|**Lexical Negation / Inherent Negation**|`deny`, `refuse`, `loss`, `absent`, `avoid`|❌ 否|词本身含否定概念，不靠词缀|



