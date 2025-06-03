---
date: 2025-06-01
tags:
  - Negation
  - InformationRetrieve
---

# Main Content

**How to design contrastive datasets:**
Our dataset follows previous work in designing contrastive evaluation datasets (Kaushik et al., 2019; Penha et al., 2022; MacAvaney et al., 2022)

They **construct a straightforward benchmark (NevIR)**: asking IR models to rank two documents that differ only by negation.

1) Contrastive documents
	Collecting pairs of documents that differ as minimally as possible but include negation, using the **CondaQA** (Ravichander et al., 2022) dataset as a starting point.
	
2) Contrastive queries


Step 1:
NevIR builds off of existing work in negation (Ravichander et al., 2022) by using 2556 instances of contrastive **document pairs** that differ only with respect to a crucial negation.

Step 2:
Then crowdsource **query annotations** for the two documents in pair, where each query is only relevant to one of the respective documents and is irrelevant to the other document.

这些亚马逊上的众包annotators被要求：针对每对文档（Paragraph1 vs. Paragraph2），各写一条问题（query），用来检索相应的文档。



**Goal:**
Test whether models correctly rank the documents when accounting for the negation.


They test negation in neural IR using **a contrastive evaluation framework**, which has shown great utility in understanding neural models
通过对比评估框架在神经信息检索中进行测试否定，这一方法在理解神经模型方面展现了显著的实用性。

Contrastive evaluations can provide important insight into understanding and improving neural models.


## Background

As modern IR models use LMs as the backbone of their architectures, it is intuitive that negation will pose problems to IR systems as well.

To the best of our knowledge, there is little to no published work on negation for neural models.

## Experiment

### Matric 
In early investigations we observed that IR models tended to rank one document above the other for both queries. 

This motivates our usage of a **pairwise accuracy score** to avoid score inflation when models don’t actually understand the negation.

由于模型老是给出一样的排序（不管问题有没有否定），如果只用单个问题的准确率来评估，模型可能依然得高分。
引入了 **pairwise accuracy** 来评估：要求模型必须根据问题（特别是是否有否定）**切换排序顺序**，否则就算错。

if the model has correctly ranked the documents for both queries (flipping the order of the ranking when given the negated query) we know that the model has correctly understood the negation and the pair is marked as correct.
如果模型能在两次排序中翻转排序顺序（即在肯定问题里Doc A排第一，在否定问题里Doc B排第一），就算模型理解了“否定”。

## Questions

In early investigations we observed that IR models tended to rank one document above the other for both queries.
无论问题是否包含否定，IR模型往往都会把同一篇文档排在另一篇文档的上面。
- 也就是说，如果你有两个文档（Doc A和Doc B），不管query是“否定版”还是“肯定版”，模型可能一直把Doc A排在前面（或者一直把Doc B排在前面）。
- 这说明模型没有真正理解“否定”导致的语义变化。

why? 为什么呢？如果是不理解doc中的否定，那应该是一种随机排序呀？怎么会出现总是将一个排在前面的情况呢


## Additional Knowledge

### Non-neural Methods

#### BM25

#### TF-IDF

### IR models are not able to scale to larger LMs as easily

- 现代信息检索模型普遍使用语言模型（LM）作为它们的核心架构，因此可以预期否定问题也会对IR系统造成挑战。
- 这里的LM指的是像BERT、T5这样的预训练Transformer模型，它们用于将查询和文档表示为向量（嵌入）以进行相似度计算。
- 之前已经在NLP中证明LM对否定的理解能力较差（比如BERT可能会忽略“not”），因此IR系统（本质上是用LM去做语义匹配）也会继承这种弱点。

IR场景和其他NLP任务（比如分类、生成）在**系统需求**和**工程限制**上有明显不同。

“scale to larger LMs”指的就是**把大模型直接放到信息检索的实时系统里**，而不像NLP一些离线任务那样可以直接加载大型模型。为什么呢？

#### (1) IR系统的实时性要求高
- 在用户发起查询时，IR系统通常需要在几百毫秒甚至更短的时间内返回搜索结果。
- 如果用大型LM（比如T5-3B或者GPT-3）去对每个文档都做一次query-document交互计算（比如cross-encoder），对上百万甚至上亿文档来说，这是不可行的。

#### (2) 大模型推理成本高

- 大模型（尤其是参数量达到数亿、数十亿的模型）不仅推理速度慢，还需要更多显存、CPU/GPU资源，直接对每个文档做推理会导致系统崩溃或响应超时。

#### (3) Neural IR一般分为两个阶段

- **第一阶段（高召回）**：通常使用稀疏检索（BM25）或轻量的神经模型（bi-encoder）快速筛选大量文档（通常是top-k，比如top 1000）。
    
- **第二阶段（重排序）**：对第一阶段召回的少量文档（比如top 100）进行更精细的排序，这时才有机会用cross-encoder或大型LM（如MonoT5）。
    
- 但即便如此，真正要把大模型直接嵌入到整个IR pipeline（尤其是第一阶段）中进行大规模文档比较，几乎不可能。

#### (4) IR vs. NLP其他任务的差别
- 在NLP任务（如问答、情感分析）中，往往只对单条输入（一个句子或段落）做推理，因此可以直接用大型LM。
    
- 但IR需要在一条查询和成千上万条文档之间做配对检索，计算量极其庞大。


IR模型确实在架构上使用了LM作为骨干（比如bi-encoder、cross-encoder），但由于IR场景需要在海量文档中实时检索结果，因此无法像NLP某些离线任务那样轻松地使用更大的LM做推理。这就导致：即便更大的LM在否定问题上可能更好，也难以直接投入到大规模IR任务中。

---

### IR Model vs Generative Language Model

#### 🔹 IR系统

- 主要目标：**在一个大型文档库中找到最相关的文档，并将其返回给用户**。
- 应用场景：
    - 搜索引擎（Google、Bing）
    - 企业内部文档检索
    - 法律/医学/金融等领域的知识库搜索
    - FAQ检索（面向客户服务）

#### 🔹 ChatGPT等生成式大模型

- 主要目标：**根据用户的自然语言输入直接生成一个完整、连贯的回答**。
- 应用场景：
    - 对话问答
    - 代码生成
    - 文本摘要、写作辅助
    - 解释或推理复杂问题

#### 🔹 IR系统流程

1. **两阶段检索（典型架构）**：
    - 阶段1（初步召回）：快速过滤大量文档（通常使用BM25或轻量的神经模型，如bi-encoder），比如从几百万文档里筛出Top-1000。
    - 阶段2（重排序）：用更复杂的神经模型（如cross-encoder）对Top-1000文档进行精细排序。
2. **输出**：返回Top-k文档（有时带有简短片段或摘要）。
3. **特点**：
    - 需要高吞吐（比如一次查询就要匹配上千万条文档）。
    - 每次推理成本低（尤其在第一阶段）。
    - 结果是文档或文档片段，而不是直接“生成”完整回答。

感觉有点类似推荐系统。

#### 🔹 ChatGPT等生成式大模型流程

1. 直接将用户问题输入到一个端到端的生成模型（GPT-4、Claude等）。
2. 模型一次性生成一个完整的回答。
3. **特点**：
    - 不需要预先存储文档库，也不依赖倒排索引。
    - 不涉及大规模文档筛选，而是直接生成文本。
    - 可能调用知识（参数中记忆的知识），但不一定引用或返回真实文档。

IR系统计算成本更低，（高并发可用），尤其在第一阶段。而ChatGPT通常一次推理需要更多显存/内存/算力。延迟上，IR系统第一阶段毫秒级，第二阶段可以稍慢，但整体可优化到用户可接受的响应时间。而ChatGPT，单次推理显著更慢，尤其是长对话或多轮对话场景。

✅ **IR系统**：文档检索（信息查找）+ 高效率 = 适合大规模文档库。  
✅ **ChatGPT**：直接生成答案（基于模型内部的知识）


IR是在已有的人类建立的知识库/文档库里进行检索，找出最相关的文档或句子作为输出，而不会自己去生成句子，不存在大模型所存在的hallucination的问题

- 经典的IR模型（BM25、DPR、ColBERT等）就是在一个**已经存在的文档库（knowledge base** 里检索，返回与用户查询最相关的文档、段落或句子。
- 它们不会“凭空”编造信息，而是直接引用库里的现有文本。
- 这种检索的局限是：如果文档库没有包含最新或直接相关的答案，IR就只能给出相似的“最接近”的段落，但未必能真正回答问题（例如“2023年世界杯冠军是谁？”如果库里没有，模型就找不到）。

而ChatGPT这类**属于生成式模型（Generative Model）。**

- ChatGPT（GPT-3、GPT-4等）本质上是一个大规模语言模型（LM），通过在海量互联网数据上进行训练，学习了大量的语言模式和世界知识。
- 它的输出是直接“生成”的，即根据输入query+上下文，用概率模型预测下一个单词，从而拼出完整的回答。
- 它的回答基于**训练时看到的数据**，所以如果问题超出了训练数据（比如问2023年世界杯冠军，而模型只训练到2022年），它就只能**猜测或编造一个答案(hallucination)** ，并不一定与事实一致。

**IR的答案可能不是最新的，因为文档库可能不更新，但其输出的文档是真实存在的；而ChatGPT的回答可能是胡编乱造。**

---
### RAG

Retrieval-Augmented Generation确实是**生成模型与IR结合**的一种典型架构。

**RAG的核心思路：**
1. **检索（IR模块）**：
    - 给定一个用户查询，从文档库里检索出Top-K最相关的文档（段落）。
    - 这部分就像DPR或BM25等做的事情。

2. **生成（LM模块）**：
    - 把用户查询和检索到的文档拼接在一起，喂给生成模型（例如T5、GPT）。
    - 让模型在“检索到的上下文”基础上生成回答。

3. 这样既结合了文档的可追溯性，又借助了大模型的语言生成能力。

🔎 **RAG vs. ChatGPT**

- **ChatGPT** 是一个单纯的生成模型，回答只依赖模型训练时的知识。
- **RAG** 则在生成前先做检索，用检索结果丰富上下文，回答更可追溯，也减少了hallucination的风险。
可以把RAG理解为ChatGPT加了“外部检索器”，先从数据库找一圈相关信息再生成回答。

ChatGPT是生成模型（Generative Language Model）。它直接基于训练数据生成文本，没有“检索”步骤。

- 如果没有接入外部知识库，它就只能根据自己“记忆中的世界”生成答案。
- 如果你看到一些产品将ChatGPT和外部检索结合起来（比如搜索插件或RAG），那其实是在ChatGPT基础上加了IR功能，属于“Retrieval-Augmented Generation”范式。

#### IR & ChatGPT & RAG 对比总结

|方面|IR系统|ChatGPT|RAG|
|---|---|---|---|
|数据源|现有的文档库|训练时的语料记忆|文档库+生成模型|
|回答方式|直接返回最相关的文档片段|生成一个回答（可能hallucination）|先检索文档，再基于上下文生成答案|
|最新性|取决于文档库更新时间|训练数据截止时间|同时依赖文档库（可更新）|
|是否可追溯|可以（返回真实文档）|不可以（直接生成）|可以（检索部分可追溯）|

---
### BEIR Datasets

The most similar area in IR is that of argument retrieval (Wachsmuth et al., 2018; Bondarenko et al., 2022), also included in the BEIR dataset, whose aim is to find a counterargument for the given query.

(However, these datasets specifically don’t include negation in the query)

So although argument retrieval datasets contain a larger amount of negations compared to standard IR datasets like MSMarco (Nguyen et al., 2016), negation is not a conscious choice in the design of either the documents or the queries and is confounded by the implicit task definition.
因此，尽管论点检索数据集包含的否定词比标准信息检索数据集（如MSMarco，Nguyen等人，2016)更多，但否定并不是文档或查询设计中的有意选择，而是由隐含的任务定义所导致的。相比之下，我们明确地提供了否定对文档和查询的影响，并对其进行了测量。

因为这些数据集是用于来找关于用户query的反论点/反驳点的，所以其任务定义上潜函了否定的含义，但是这些数据集并不是专门被设计来帮助模型理解用户输入的document或者query里所包含的否定的，这些数据集里的用户输入的query里也并不包含否定。

**BEIR**（**Benchmarking IR**）是一个大规模的、标准化的**信息检索（IR）评测数据集集合**，全称：**BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of Information Retrieval Models**。  
它由Nils Reimers等人（2021年）在论文《BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of Information Retrieval Models》中提出。

在神经IR模型迅猛发展的背景下，很多模型在单一的数据集（如MS MARCO）上测试结果，但在实际场景中，IR模型需要面对各种任务（如事实性问答、辩论检索、法条检索、句子检索等），数据分布差异大。  
BEIR的目标是：
- 为IR模型提供统一的、多样化的基准，方便**跨任务、跨领域评测**。
- 支持**zero-shot检索**：即模型在没有针对某个任务专门训练的情况下的泛化能力。

#### BEIR数据集包含哪些任务？

BEIR数据集覆盖了18个不同的IR任务，包括：

- **问答检索（Question Answering）**：如NQ（Natural Questions）、HotpotQA。
- **句子检索（Sentence Retrieval）**：如CQADupStack（社区问答去重）。
- **事实核查（Fact Checking）**：如FEVER。
- **辩论检索（Argument Retrieval）**：如ArguAna、Touché-2020。
- **对话检索（Dialog Retrieval）**：如Quora。
- 以及其他领域的文档检索、维基百科检索等。

其中，“**Argument Retrieval（辩论检索）**”任务包括：

- 给定一个问题或陈述，检索相反观点（反方论据）或相关论据。
- 例如：查询“Is climate change real?”，系统需要找到支持/反对的论据。


---

### MS MARCO Dataset
#### 1️⃣ MS MARCO 是什么？

**MS MARCO** 是 **Microsoft Machine Reading Comprehension** 的缩写。  
它最初是微软为了训练和评测机器阅读理解（MRC）系统而构建的，但后来演化成了**开放域信息检索(ODQA)** 的一个重要基准数据集。

🔹 **主要任务**：
- **检索任务（Passage Ranking）**：给定一个自然语言查询（query），在大规模文档集合中检索最相关的文档（通常是段落级别）。
- **阅读理解任务**：给定查询和检索到的段落，预测答案（生成答案或抽取答案）。

在IR社区，最常用的部分是 **MS MARCO Passage Ranking** 任务。

#### 2️⃣ MS MARCO 数据结构

以 Passage Ranking 任务为例：

- **查询（query）**：大部分是真实的用户搜索查询（往往比较短、口语化、语法不完整）。
    
    - 例如：“what is the fastest animal in the world?”
        
- **候选段落（passages）**：从Bing搜索引擎爬取的网页段落（包含答案或不包含答案）。
    
- **标签**：对候选段落的二分类（相关/不相关），由人工标注。

典型的配置是：每个查询对应数十到数百个段落，标注其中哪些是相关段落。

#### 3️⃣ 为什么说 MS MARCO 是“单一的”？

这句话的背景是和 BEIR 这样的多任务、多领域 benchmark 进行对比时提出来的。

🔍 MS MARCO 的特点：  
✅ 主要是**检索网页段落**，大多是**开放域常识问题**（比如旅游、健康、科技等）。  
✅ 训练集、开发集和测试集都是**同一种问题类型**，数据来源统一，查询风格也比较相似。  
✅ 主要任务是**passage ranking**。

相比之下：  
🚫 不涉及多个领域（法律、医疗、科学论文、对话、观点辩论等）。  
🚫 不涉及复杂的跨领域、多语言、多任务的泛化问题。  
🚫 不含结构化知识、图表等复杂场景。

所以在论文里，当我们说“MS MARCO是单一的”，其实就是指它：

- 主题单一（几乎都是真人搜索引擎问题）
- 数据来源单一（主要来自Bing搜索）
- 检索任务单一（passage ranking）
- 语言风格相似（用户query+网页段落）

这和像 BEIR 这样的benchmark（包含18个不同子任务、不同来源、不同领域、不同难度）相比，它的多样性就显得有限了。

MS MARCO数据里否定问题的比例非常低（因为它是web搜索数据，绝大多数问题都是常规检索），因此无法很好地衡量模型对否定的理解能力。

NevIR专门设计了**对比文档对**（例如包含否定词和不包含否定词的文档），这就能专门评测模型能否分辨这种细微的语义差异。

---
### Development Set

在这篇论文（NevIR）里的 Table 1（你贴出来的那几张表格），结构大致是这样的：

|Statistic|Train|Dev|Test|
|---|---|---|---|
|# Pairs|948|225|1383|
|Question 1 Length|...|...|...|
|Document 1 Length|...|...|...|

这是一种很常见的**机器学习/信息检索任务的划分表格**。

#### 什么是 DEV？

**DEV** 是 **Development Set**（开发集）的缩写，在机器学习实验中通常指的是：

- 用于**模型调参**、**早停**、**超参数选择**或者**中间验证**的数据。
- 不是用来训练模型，也不是用来做最终测试，而是在开发阶段用来辅助模型的“内部评估”。

#### DEV在IR实验中的用途

在信息检索（IR）或者NLP任务中：  
✅ 训练集（Train）：用于训练模型的权重。  
✅ 开发集（Dev）：用于模型开发时的验证，如：

- 监控模型的效果（比如pairwise accuracy）。
- 选择最优超参数（比如学习率、batch size）。
- 避免过拟合。  
    ✅ 测试集（Test）：只在最后一次评估模型性能时使用，不能用于模型开发或调参。

#### 这篇论文的使用

在NevIR数据集中，他们把数据按照文档对（pairs）分为Train, Dev, Test：

- **Train**（948对）：用来训练模型（比如Fine-tune）。
    
- **Dev**（225对）：用来验证模型的性能（比如pairwise accuracy），选最优超参数。
    
- **Test**（1383对）：最后报告模型的性能，不参与任何训练或调参。

**DEV列表示开发集**，用来调参和监控模型的性能。  
在信息检索、NLP等实验里是非常常见的标准做法，保证模型性能评估的客观性和泛化能力。





## 好词好句

	spur /spe'r/  n. 马刺；鞭策；激励；（公路或铁路的）支线 v. 鼓动； 激励；使更快发生；策马前进
		We hope that our analysis will spur increased attention to the problem of negation in information retrieval and provide a dataset for IR training and evaluation. 我们希望我们的分析将刺激人们对信息检索中的否定问题更多的关注，并提供一个IR训练和评估的数据集。
	counterarguments	n. 反论点，反驳
	surge n. 汹涌；激增；大量；奔涌向前 v. 汹涌；使强烈地感到；激增；飞涨
		there has been a surge of interest in retrieval augmented language models 人们对检索增强型语言模型的兴趣激增
	intertwine v. 缠结在一起
		as LMs and IR systems become more intertwined and used in production, understanding and improving their failure cases (such as negation) becomes crucial for both companies and users. 随着LM和IR系统越来越相互交织并被用于生产，理解和改进它们的故障案例（如否定）对企业和用户来说都至关重要。
	lexical adj. 词汇的；具词典性质的，词典的
		including the ability to go beyond simple lexical matches to encode the semantic similarity of the natural language text. 包括超越简单的词汇匹配来编码自然语言文本的语义相似性。
	prominent adj. 突出的，杰出的；突起的；著名的
	compound adj. 混[复]合的 v. 混合，组合
		this problem is compounded as IR models are not able to scale to larger LMs as easily, due to efficiency and latency constraints on processing large amounts of documents in real-time. 由于处理大量文档的实时效率和延迟限制，IR模型无法轻松地扩展到更大的LMs，从而使这个问题变得更加复杂。
	halluci'nation  n.c 幻觉
	para'digm  /pere' dai/ n.c.范例，模式，样式。
		Our work provides both zero-shot (no model fine-tuning) and standard train/test splits to accommodate both paradigms. 我们的工作提供了零样本（无需模型微调）和标准的训练/测试分割，以适应这两种范式。
	To the best of our knowledge  据我们所知
	conscious adj. 意识到的；神志清醒的；慎重的；关注的，有意的
	confound  adj. 糊涂的，困惑的，讨厌的（轻微的诅咒用语） v. 击败；使惊惶；使困惑惊讶；搞乱
	axiom /eksim/ n. 公理；自明之理；原理；格言
		The axiom that supply equals demand 供给等于需求的公理
	verbatim adj.& adv. 一字不差的（地）；逐字的（地）
		we found that annotators would typically quote verbatim from the passage. 我们发现，注释者通常会逐字引用文章内容
	pilot n. 驾驶员；引航员；试验性的方案；试播节目；常燃小火 v. 驾驶；引导；试验；顺利通过 adj. 试验性的
		a series of initial pilot HITs 这里翻译成试验性的。在众包平台（尤其是 Amazon Mechanical Turk, 简称MTurk）里，HIT 是 Human Intelligence Task 的缩写，也就是人类智能任务
	inflation n. 膨胀；通货膨胀；夸张；自命不凡
		score inflation 分数虚高