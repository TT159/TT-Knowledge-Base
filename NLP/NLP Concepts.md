---
date: 2025-05-21
tags:
  - NLP
---
## 什么是**平行语料库（Parallel Corpus）**？

**定义：**  
平行语料库是指**成对对齐的文本数据/片段** (通常是句子对)，通常是**两种语言之间的对应句子对**，也可以是**一种语言中具有语义对齐的不同表达方式**。这些片段在语义上等价，但可能在语言、表达方式或风格上不同。

### 最常见的形式：双语平行语料库

**“双语平行语料库”**（bilingual parallel corpus）——是最标准的平行语料形式。

> **定义**：在两个不同语言之间，每个句子在语义上相同，互为翻译。

 **例子：**

| 英文                                  | 中文         |
| ----------------------------------- | ---------- |
| I love natural language processing. | 我喜欢自然语言处理。 |
| She didn't go to school today.      | 她今天没有去上学。  |

这类数据对机器翻译训练非常重要。例如，Google 翻译、DeepL 等模型就是用海量的这类平行语料来训练的。

---

### 🔄 二、除了翻译，还有哪些形式的“平行”？

虽然双语对是最常见的形式，但平行语料其实可以**不限于不同语言**，也可以是**同一种语言中的不同表达方式对齐**，比如：

### 1. **同义改写平行语料**（monolingual paraphrase corpus）：

用于训练模型学会“换句话说”。

|原句（英语）|同义改写|
|---|---|
|He didn't attend the meeting.|He missed the meeting.|
|The weather is not good.|The weather is bad.|

这种在同一语言内部生成的平行语料也被广泛用于**语义理解、问答、生成式任务等模型训练**。

---

### 2. **风格变换语料（Style transfer parallel corpus）**

这种语料对在语义不变的前提下进行风格转换，比如：

|原句（普通风格）|转换后（正式风格）|
|---|---|
|Gonna go out now.|I am going out now.|
|What’s up?|How are you?|

这种语料可以用来训练模型做风格迁移，如把口语变正式、把现代文改成古文等。

---

### 3. 同义改写 affirmative paraphrase

比如[[2024-Paraphrasing in Affirmative Terms Improves Negation Understanding]] 这篇文章里的改写就也是一种parallel， 它将显式含有否定词的句子用不含否定词的句子来表示，同义改写，即文章中说的affirmative paraphrase. 

例如：
I don't like this movie.  <=> This movie is boring.

### 📚 三、平行语料库的常见来源

|来源方式|举例|
|---|---|
|**人工翻译文档**|欧盟议会记录（Europarl）、联合国文件等，多语言版本|
|**字幕对齐**|TED演讲、电影字幕（OpenSubtitles）|
|**圣经、法律文件**|多语言圣经版本、法律文本（如 JRC-Acquis）|
|**Wikipedia 双语条目**|不同语言的维基百科词条可以进行自动对齐|
|**机器生成**|用机器翻译自动生成（低资源语言常用这种方法）|

---

### 四、平行语料库的用途总结

| 应用场景       | 说明                          |
| ---------- | --------------------------- |
| 📘 机器翻译训练  | 最核心用途，训练 Transformer、T5 等模型 |
| ✍️ 同义句识别   | 判断两个句子是否语义一致                |
| 🗣️ 风格迁移训练 | 改变句子风格，保持语义不变               |
| 🔍 自然语言推理  | 判断两个句子是否 entail/contradict  |
| 📊 多语言语义比较 | 研究不同语言表达相同意思的方式             |

---

### 总结回顾：

|名称|是否同一种语言|示例|用途|
|---|---|---|---|
|**双语平行语料库**|❌ 否|英语 ↔ 中文翻译|机器翻译|
|**同义改写语料库**|✅ 是|英语原句 ↔ 英语改写|语义理解、问答等|
|**风格变换语料库**|✅ 是|口语句 ↔ 正式语句|风格迁移、文本生成|

---

## 什么是**Paraphrase**

### 1️⃣ 什么是 NLP 里的 paraphrase？

在 NLP 中，“**paraphrase**”指的是：

> **对同一意思的不同表达方式。用不同的词汇、语法、句式表达相同的语义。

- 也就是说，**两个句子虽然词汇或结构不同，但语义是相同的**。
    
- 例如：
    
    - 原句：“John bought a car yesterday.”
        
    - 改写： “Yesterday, John purchased a vehicle.”
        
- 这两个句子虽然用的词不同（buy → purchase, car → vehicle），但是它们的意思是一样的。
    

在 NLP 里，paraphrase 通常用来训练模型的句子相似性、同义句检测、生成式改写任务等。


包括很多种形式：

- 同义词替换（例如：“buy” → “purchase”）
    
- 句型转换（例如：“He bought a car.” → “A car was bought by him.”）
    
- 结构调整（例如：“Yesterday, John purchased a car.” → “John bought a car yesterday.”）

---

## 什么是**回译（Backtranslation）**？

**定义：**  
回译是指一种**自动数据生成方法**：  
先将一个句子**翻译成另一种语言**，然后再**从这门语言翻译回原始语言**，从而得到一个**语义接近但表达方式不同的句子**。

#### ✅ 举个例子

1. 原句（英文）：
    
    > _I didn't like the movie._
    
2. 翻译成法语：
    
    > _Je n'ai pas aimé le film._
    
3. 再翻译回英文（回译）：
    
    > _I didn’t enjoy the movie._
    

➡️ 得到一个**语义相似但词句不同的表达方式**，可用于==生成同义句、扩充数据==等。

---

### 🧠 回译的常见用途：

|用途|举例|
|---|---|
|数据增强|为模型生成更多多样表达的训练数据|
|同义改写|制作 paraphrase 对|
|消除偏见|通过多轮翻译让语言更中性|
|自动生成平行语料|特别是在只有一种语言的原始数据时|

---

### ✅ 总结：

|概念|解释|
|---|---|
|**平行语料库**|成对对齐的、语义相同的句子集合（可跨语言，也可单语）|
|**回译（Backtranslation）**|先翻译到其他语言，再翻回来，从而生成同义句或新表达方式|


## 什么是coreference？

**coreference**（共指消解）是自然语言处理（NLP）中的一个核心任务。

**Coreference** 指的是：

> **文中两个或多个表达，指的是同一个实体（object/person/place/idea）**。

处理这个任务的过程叫做：**coreference resolution（共指消解）**。

### 📌 举个例子：

> **Mary went to the store. She bought some milk.**

- “**Mary**” 和 “**She**” 指的是 **同一个人**；
- 所以，“Mary” 和 “She” 之间就是一个 **coreference link**。

再比如：

> **The president gave a speech. The speech was well received.**

- “**a speech**” 和后面的 “**The speech**” 是 **重复指向**，也构成一种 coreference。

---

### ✅ Coreference 与 NLP 应用的关系：

在你给的句子中提到的：

> **event-argument structures** 对 **coreference** 有帮助。

这是因为：

- 在处理一个事件结构时，我们往往需要知道事件中的参与者（arguments）是谁；
    
- 如果不能识别这些参与者在不同句子中是否是同一个人，就无法准确地理解事件之间的关系；
    
- 比如在多句新闻中，“the suspect”、“he”、“the man”可能指的都是同一个人，coreference resolution 就能把这些对应起来。
    

---

### 📘 应用场景包括：

- **新闻摘要**（summarization）：如果不解决 coreference，生成的摘要可能重复或混淆主体；
    
- **问答系统**（QA）：用户可能问“他做了什么”，模型需要知道“他”指的是谁；
    
- **信息抽取**：从多个句子中识别出同一个实体参与了哪些事件。