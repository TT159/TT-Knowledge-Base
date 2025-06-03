---
date: 2025-05-18
tags:
  - NLP
  - EAE
aliases:
  - BEMEAE
---
# 主要内容

这篇文章是2025年NACL的文章，由Enfa Fane, Md Nayem Uddin, Oghenevovwe Ikumariegbe, Daniyal Kashif, Eduardo Blanco, Steven R. Corman 完成。

主要针对[[Event Argument Extraction (EAE) Task]]任务，解决该任务中常用的evaluation matric exact span match (ESM)的局限性问题。

提出了一种新的评估方式BEMEAE：introduce a novel ==evaluation metric== that addresses the limitations of ESM by incorporating deterministic components and a semantic matcher.

在9个不同的EAE models (four classification-based models and five generation-based models) 以及两个datasets RAMS & GENEVA 上进行了评估。

同时还通过automatic semantic matching来评估了。evaluate various semantic matching methods, including GPT-4。无需人工标注。

A key advantage of our approach is its adaptability to any event argument extraction framework, without the need for additional costly human assessments or extensive retraining.

与其他已经提出的ESM的替代方式相比，BEMEAE does not require additional human annotations.

---
## BEMEAE

BEMEAE. This metric introduces (1) a series of deterministic components to account for textual variations that do not affect the correctness of an argument and (2) a semantic matching component to identify additional correct candidate arguments.

---
### BEMEAE的创新：
#### 1. Deterministic Components

##### Same text, not just spans
exact span match incorrectly classifies identical text that refer to the same entity appearing in different positions as incorrect. 当完全相同的文本（span）在文本中出现在不同位置时，即使它们指的是同一个实体，ESM 也会错误地将其判为错。

例如：
The convoy was hit. Later, the convoy was found to be undamaged.
如果人工标注的是第一次出现的 "the convoy"，而模型预测的是第二次出现的 "the convoy"，那 **在文本内容上是完全一致的**，语义也对。

但 ESM 只看 **文本起始和结束位置**，所以位置对不上就算错。

改进：
To address this issue, we compare **candidate** and **reference** arguments (excluding pronouns) based on their uncased textual content rather than their offsets.

改用“只比较文本内容是否一致”，忽略 offset，即位置。

这是一种改良的匹配方法，也称为：Textual content match。

与下面的uninformative variations的关联。
- “放弃 offset” → 是为了只看内容，不看位置
- “消除不重要的变异” → 是为了内容匹配时更宽松
- 第一点（offset → text） 是宏观维度的匹配标准改变； 
- 第二点（uninformative variation） 是微观维度的匹配容错处理；  
- 两者是递进关系，不是重复，而是层层放松约束。

---
##### Uninformative Variations
虽然两个文本span在表面形式上不同，但语义完全一致，仅仅是因为一些“无意义的细节”造成了差异。但 ESM 会判断为错 ，因为它要求 **完全的 span 匹配（文本、位置、边界）**。

这些“无意义的差异”包括：
- **冠词**：a, an, the
- **限定词（determiners）**：this, those, my, their
- **标点符号**：逗号、句号等
- **Saxon genitive（’s）**：所有格形式，如 “John’s car” vs “the car of John”

改进：
To partially address this limitation, we identify and remove uninformative tokens that do not alter the meaning of the span.
**先“清洗”掉无意义的token，再做文本比较**。
1) **先清洗 reference 和 candidate spans**
- - 删除以下无信息词：
        - `a`, `an`, `the`（冠词）
        - `this`, `my`, `their` 等限定词
        - 所有标点符号
        - `'s`（如 "John’s car" → "John car"）

2) **再进行文本内容比较（textual match）**
- 不要求起始位置或长度一样
- 只要“去除杂质后的文本”一样，就判定为匹配.

| 原始 Reference      | 原始 Candidate      | 清洗后文本    | 是否匹配               |
| ----------------- | ----------------- | -------- | ------------------ |
| "the convoy"      | "convoy"          | convoy   | ✅ YES              |
| "John’s car"      | "the car of John" | john car | ❌ NO（语序不一样，尚未考虑语义） |
| "their vehicles." | "vehicles"        | vehicles | ✅ YES              |

---
##### Breaking Down (and Aggregating) Arguments
**(ESM)** 的又一个关键局限 → **它无法处理 "granularity"（粒度）问题**，特别是在面对并列成分时。

在事件参数抽取（EAE）中，人工标注可能将多个概念**作为一个整体 span 进行标注**，比如：

> Reference（人工标注的 span）：**"weapons and explosives"**

但模型可能会更细致地抽取：
- Candidate 1：**"weapons"**
- Candidate 2：**"explosives"**

从语义上讲，两者是**完全等价的**，甚至模型抽得更细致，但：

> **Exact Span Match 不认这个结果，因为它不是一个完整的 span。**


例子：
> “The police found weapons and explosives in the building.”
- **参考标注（reference）**：`"weapons and explosives"` （一个 span，表示 role=Item）
- **模型预测（candidate）**：
    - `"weapons"`（一个 span）
    - `"explosives"`（另一个 span）

人类判断：是正确答案，只是粒度更细。
ESM 判断：错误！因为它们 **不是 exact span match**。

改进：
作者提出了一个**对模型预测进行结构调整的策略**，试图让模型预测结果在评估时能被视为“等效”的。we either break them into separate arguments or aggregate separate arguments using “and” as needed to check for a match with the reference.

方案1：**分拆（Breaking Down）**
- 将 `weapons and explosives` 这种并列项拆成两个单独的 arguments
- 用于对比时更宽容地匹配 reference 中的多个部分

 方案 2：**合并（Aggregating）**
- 如果模型分别预测了 `"weapons"` 和 `"explosives"`，则将它们合并为 `"weapons and explosives"`
- 然后再与 reference 进行匹配

**检查是否语义等价，不再只拘泥于 span 的边界**。

---
##### Informative Variations

**Informative variation（有信息的变体）** 是指：

> 模型预测的 argument 比参考答案多了一些**有用的上下文修饰语**，**虽然更长，但并没有错，反而更准确。**

📌 也就是说，模型预测的是一个更“具体”、更“丰富”的 span。

例子：
> “The armed men entered the building.”
- **参考标注**（reference argument）：`men`
- **模型预测**（candidate argument）：`armed men`

人类判断：模型预测的更具体，是正确的。
但：**ESM 会认为这是错误的**，因为 `"armed men"` ≠ `"men"`（span 不一致）

改进：
在参考答案中“加回”修饰语，再进行匹配。we generate modified versions of single-token reference arguments by adding relevant modifiers

关键想法：
- 如果 candidate 比 reference 多了一些合理的修饰成分，不应该立即判错。
- 那么，我们可以：  
    → 对 reference 做一些增强，加回可能的修饰语，看看是否可以和 candidate 对得上。

怎么找“修饰语”？（dependency parsing）：他们使用 **spaCy** 的依存句法分析（不是自己提出的方法，已有的方法），找到以下类型的修饰词（都是有用的修饰成分）

| 类型        | 依存标签（dependency label） | 示例                            |
| --------- | ---------------------- | ----------------------------- |
| **形容词修饰** | `amod`                 | "armed men" 中 "armed"         |
| **同位语修饰** | `appos`                | "John, the CEO"               |
| **名词修饰**  | `nmod` / `nounmod`     | "school building"             |
| **数字修饰**  | `nummod`               | "3 soldiers"                  |
| **所有格修饰** | `poss`, `possessive`   | "their weapons", "John’s car" |

例子：
> “Three armed men were seen near the site.”

- Reference argument：`men`
- Candidate argument：`three armed men`

改进后：
- 用 spaCy 识别出 `amod` = "armed"，`nummod` = "three"
- 把它们附加回 `"men"`，构造更丰富的参考版本 `"three armed men"`
- 与 candidate 比较后，✅ 认为匹配成功


作者说：We manually analyzed 100 candidate arguments initially flagged as errors by exact span matching but deemed correct by this step. Our analysis showed **100% accuracy**.
说明：这个依存分析 + 修饰增强的方法是非常有效的，极大地弥补了 ESM 的不足。

---

#### 2. Semantic Matching Component

> We sampled 500 candidate arguments from RAMS and GENEVA that were still identified as errors after applying our deterministic steps (确定性的规则). 

Although our deterministic components improve upon ESM, they cannot capture all semantically equivalent reference-candidate pairs that ESM misidentified as errors.

在应用了上面的第一种改进策略后，依旧没有做对的，再进行下面这个部分的改良优化。

##### Human Assessments

Each candidate argument was reviewed by two human annotators, who were provided with the relevant document and asked to answer three specific questions.
作者引入**人工评估**来进一步判断这些预测是否其实是“正确”的。

Inter-annotator Agreement: 作者测量了两位人工标注者之间的一致性（Cohen's kappa）
设置了3种标签： correct, partial， incorrect

| 数据集        | κ 值  | 说明                     |
| ---------- | ---- | ---------------------- |
| **RAMS**   | 0.84 | 接近完美一致（Almost perfect） |
| **GENEVA** | 0.95 | 几乎完全一致                 |
人类对这些判错预测的判断非常一致，说明这不是“主观模糊”的任务，而是自动评估确实“错怪了”正确预测。

作者列出了一些例子，说明自动方法的不足：
例如： 缩略词vs全称
预测使用了缩略词，参考用了全称。 比如， U.S. vs United States

作者最后指出：
**即使改进了确定性规则判断，仍然漏掉大量“语义正确”的预测结果**。
> These examples highlight the importance of incorporating a **semantic matching** component to ensure accurate evaluation.

也就是说，仅靠 ESM + 硬规则还远远不够，未来需要：

- 引入语义层面理解（例如使用语言模型 Embedding、文本相似度）
- 进行更“人类式”的判定：不是只看 span，而是看 **是否表达了正确的实体和角色**
---

##### Automatic Assessments

>While human judgment is the most reliable, it is impractical to apply it to every prediction due to time and cost constraints.
  Therefore, we assessed the following methods to approximate human assessments and analyzed their agreement with human annotators.
  
**为了克服人工评估成本高的问题，研究者设计了一些自动评估方法，用来模拟人类判断**，并分析它们与人类的评价结果之间的一致性。

虽然人工评估最准确，但是成本高，无法覆盖所有预测结果，所以用自动方法模拟人类判断，尽量拟人化打分。

###### a) Cosine Similarity + Logistic Regression
- 把 reference span 和 candidate span 分别转换为向量（embedding）
- 取它们的平均词向量（mean embedding）
- 计算 **cosine similarity**
- 用这个相似度作为输入特征，训练一个**逻辑回归模型**，预测标签（Correct / Partial / Incorrect）

用了三种 embedding 方法：
- **word2vec**
- **GloVe**
- **sentence-BERT**

缺点：
- 粒度太粗，不能捕捉结构语义
- 受限于 embedding 表达能力（特别是 word2vec、GloVe 只考虑词，不考虑上下文）

这是直接整个文本或者每个span的平均embedding，计算余弦相似度，看这cosine这一个单个指标。无法识别span语句里词与词之间的对应关系。span对span的匹配，遗漏了词之间的相似性，词的相似性被平均稀释了，难以被精准捕捉到。

而下面的方法b) BertScore是用 BERT 输出的 token embedding，逐 token 计算最大对齐分数。细粒度更高。token 对 token（多对多 token 匹配），计算Precision / Recall / F1（多个得分）。可识别近义词、同义结构，如 “feline” ≈ “cat”。更强的语义捕捉能力，但成本高。

---

###### b) BERTScore
- 用 DeBERTa 模型生成 token 级别的 contextual embedding
- 计算 reference 和 candidate 之间的 **BERTScore**（是一种 token 对 token 的最大相似度匹配得分）

**BERTScore** 是一种**评估两个文本相似度的指标方法**，它用 BERT 的 token embedding 来计算文本之间的 **token-level 语义匹配质量**。

优点：
- 考虑上下文（更强的语义表示）
- 评估粒度更细致（token-level matching）
    
缺点：
- 只看文本表层相似性，可能不能准确区分语义角色是否匹配

给定两个句子（假设是模型输出的文本和参考答案）：
- 参考句子 (Reference)：`"a black cat"`
- 预测句子 (Candidate)：`"the dark feline"`

然后：
1. **用 BERT 编码两个句子**
2. 得到每个 token 的 embedding（上下文感知）
3. **对每个预测 token，找到参考句中最相似的 token**
    - 通过 **余弦相似度**
4. 对这些相似度取平均，就是最终的 BERTScore
    
有三个分数：
- **Precision**：预测的 token 能多好地匹配参考
- **Recall**：参考的 token 是否都能被预测命中
- **F1**：上面两者的调和平均

**BERT 是一个嵌入模型，而 BERTScore 是一个评估指标方法**，它利用 BERT 的 token embedding，通过计算两个句子中 token 对 token 的语义相似度来衡量整体文本的匹配程度，**比传统的 BLEU / ROUGE 更语义敏感、更接近人类判断标准**。

上述两种方法的对比：
**a) Cosine 方法**：把整段文本变成一个向量，只比较整体语义“是否大致相近”；  
**b) BERTScore 方法**：比较每个词的语义是否能被对方句子里的词“对得上”，所以更细致、更接近人类理解。

---

###### c) GPT-4o 模型评估
- 使用 GPT-4o 和 GPT-4o mini 进行**自然语言评分**
- 设计不同的 prompt 配置来控制模型判断的标准

四种设置（实验变量）：

| 配置项              | 选项                                      |
| ---------------- | --------------------------------------- |
| 🌐 Prompt 类型     | Expert instruction vs. Intuitive（更自然表达） |
| 📄 是否包含reference | With vs. Without                        |

> **GPT-4o with expert instruction + reference included** 给出与人类判断一致性最高的结果（Cohen's κ 最大）

表明大模型能够以人类方式理解语义等价  
而且可以理解复杂语境下的抽取是否正确

**直接“把判断任务交给大模型去做”**，利用它的语言理解和推理能力，让它**像人一样阅读上下文、参考答案和模型预测结果，然后判断是否正确**。

这个方式本质上是一种 **LLM-as-a-Judge（用大模型作为评估器）** 的策略，近年来在 NLP 评估领域越来越流行，因为它能：
- 跨越 span 的边界限制
- 理解语义角色
- 捕捉模糊但“人类觉得没问题”的表达

基本流程如下：
1. **准备一个 Prompt（提示词模板）**
    - 里面包含：
        - **上下文句子**（含事件）
        - **事件触发词**（trigger）
        - **模型预测的 argument（candidate）**
        - **人工标注的 argument（reference）**
        - 一个问题指引 GPT 判断预测是否正确
        
2. **调用 GPT-4o 或 GPT-4o mini**
    - 让模型“扮演评审员”，阅读输入，输出标签（Correct / Partial / Incorrect）
        
3. **将 GPT 判断结果与人工标注对比**
    - 计算一致性（Cohen's κ）

---


## 实验 Experiment

### 数据集 Dataset

数据集对比：
RAMS uses news as source texts and annotates **one event per document** (length: 4.7 sentences). In contrast, GENEVA uses text from several domains and annotates **two events per document** (length: one sentence). 
GENEVA considers less event types but over three times the amount of argument types (65 vs. 220);


将数据集进行了5 splits和 split 1的对比
1）**“5 splits”**：
指的是将数据集 **划分为 5 组不同的训练/验证/测试子集（data splits）**，然后：

- 每个模型分别在这 5 个数据划分上训练和测试；
- 最后报告 **平均的 F1 得分** 和 **平均排名**。

2）**“Split 1”**：
training and testing using only Split 1.
指的是只使用数据划分中的**第1个 split**，进行训练和测试：

- 不再重复 5 次；
- 减少了训练运行的次数，**节省大量计算资源**。

#### 为何要做 5 splits 而不是简单的仅仅只用一个？

因为自然语言任务中的模型表现会因为训练/测试划分的不同而有波动：

- 某些 split 可能 easier（测试集中简单样本多）；
- 有些 harder（测试集中罕见样本多）；
- 所以 **平均多个 splits 可以更公平地评价模型的泛化能力**，减少偶然性

但在本文里，他们发现这两种方式在 RAMS 数据集上 split 的差异很小。模型在 5 splits 平均值 和 split 1 上的 F1 分数几乎一样。因此，他们后续分析中选择只用 split 1，这样可以**减少 80% 的计算开销**，因为不用训练5次了。

这可能是因为RAMS 数据集：
1. **数据集划分方式较稳定**：不同 splits 的事件类型分布较均匀；
2. **样本数量足够大**：可以让每个 split 都足够代表整体；
3. **事件类型较集中、场景一致性强**：如新闻领域，事件表达形式重复率高。
> 所以在这种数据集中，模型在不同 splits 上的波动较小，平均和单次差别不大。



## 好词好句

1) off-the-shelf: 现成的，买来不用改就用的
	The semantic component uses either human judgments or off-the-shelf methods, enabling effective evaluation without the need for extensive additional resources.

2) Unless specified otherwise 除非另有规定
	Unless specified otherwise, in this paper, “F1 score” refers to the ESM F1 Score.
