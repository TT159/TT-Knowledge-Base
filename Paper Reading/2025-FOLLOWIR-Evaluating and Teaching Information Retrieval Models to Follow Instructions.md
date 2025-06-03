---
date: 2025-05-31
tags:
  - InformationRetrieve
---


# Main Content

**search with instruction** / **instructions for retrieval**


retrieve with instructions would enable support for more complex information needs.


## Background

Modern Language Models (LMs) are capable of following long and complex instructions that enable a large and diverse set of user requests. 
While Information Retrieval (IR) models use these LMs as the backbone of their architectures, virtually none of them allow users to provide detailed instructions alongside queries, 
thus limiting their ability to satisfy complex information needs.

许多现代LMs（如GPT）可以很好地跟随用户指令。然而，IR社区虽然大量使用LMs，但大多数只用作“相似度计算器”，没有利用LM的指令跟随能力。现实中的信息需求复杂，需要能够理解和遵循复杂指令的IR系统。

现有IR模型大多只处理简短或形式化的检索指令（甚至忽视指令），缺乏评估检索系统是否真正“遵循指令”的基准测试。

1) Recent work has started to move towards search with instructions, but this **topic is still understudied** with only a handful of papers. 

2) we find their **use of instructions to be narrow**: instructions are typically short (fewer than 10 words) and repetitive (only one instruction per dataset)

3) Further, these works **lack of evaluation datasets** that explicitly measure instruction following—instead focusing on standard ad-hoc retrieval benchmarks.


==Our key intuition is to leverage instructions developed for professional annotators of IR systems in order to study the capabilities of instruction-following IR models.==
我们的关键直觉是利用为IR系统专业标注员开发的指令(instruction / narratives)来研究遵循指令的IR模型的能力。

if annotators can use these TREC instructions to annotate document relevance, so should instruction-following retrieval models
在TREC这样的检索比赛里，每一个查询都有对应的 narrative 来指导标注员如何判断相关性。这给了作者一个机会：把这些 narratives 作为指令，让IR模型也来尝试按照它们执行任务。


## Contributions

1) FOLLOWIR: a benchmark for evaluating  instruction following in retrieval (on top of three already highly-judged corpora: TREC Robust 2004, TREC Common Core 2017, TREC News 2021)

2) build **a training data/set** for teaching retrieval models to follow instructions along with **a new open-sourced IR model, FOLLOWIR-7B,** that can handle long instructions in IR.

3) analysis of why current models fail to understand instructions. 
   develop a new evaluation framework, p-MRR. measuring rank-wise score changes of documents given a pair of different instructions with the same query.





















## Additional knowledge

### ad-hoc ?

ad-hoc  一个拉丁文常用短语。形容词/副词， “特别的”“临时的”“专门的”。指的是根据具体情况临时制定、执行的工作任务，通常没有预先制定的计划或标准操作程序。

在信息检索（IR）领域，**ad-hoc retrieval**（临时检索、即席检索）指的是：  
给定一个查询（query），在一个静态文档集合（corpus）中找出与查询最相关的文档。
- 这里的“ad-hoc”强调查询的**临时性**，即模型并不知道用户的查询内容，也没有为这个查询做过特殊的定制化训练，而是需要模型在运行时动态处理。
- 这和“静态的、预定义的问题（比如问答系统）”相对，模型需要针对每一个新查询临时构建答案。

例子：
用户查询：2024年全球变暖的主要原因有哪些？
模型需要根据预先准备好的新闻文章或科学论文集合，找出与该问题最相关的文档。


ad-hoc search system

ad-hoc retrieval benchmarks


### narratives

在这篇论文和信息检索（IR）领域中，**narrative**（叙述、指令）指的是——  
**由TREC等共享任务（shared tasks）组织者为人工标注员撰写的、用于指导标注员判断文档是否与查询相关的详细说明**。

这些 Instruction / narratives 通常比简短的用户查询 Query（比如“影响气候变化的因素是什么？”）要**更详细**，(论文 Figure 1) 包含了：

- 具体的背景信息（例如对查询主题的解释）。
- 什么样的文档应该判定为“相关”。
- 什么样的文档应该判定为“不相关”（例如“只讨论气候变化对动物影响的文档可能不相关”）。
- 可能包括“排除条件”（例如“如果只提到交通政策，不涉及碳排放，则不相关”）。

用途：
**给人看的**：这些 narrative 最初是写给TREC等评估者的，帮助他们对成千上万篇文档做一致的相关性标注。  
**作为模型输入**：在这篇论文中，作者把这些 narratives 当作“指令（instructions）”提供给IR模型，用来测试模型是否能像人类评估员一样理解和遵循这些指令。


在本论文中的用法：
> Fortunately, the IR field is rich with such data, as these instructions—**also known as narratives**—are created for all queries in any well-constructed IR dataset.

这句话的意思是，IR领域里有很多这样的数据（narratives），因为在TREC这样的检索比赛里，每一个查询都有对应的 narrative 来指导标注员如何判断相关性。
这给了作者一个机会：把这些 narratives 作为指令，让IR模型也来尝试按照它们执行任务。

|术语|解释|
|---|---|
|**narrative**|TREC等任务中用于指导标注员判断文档与查询相关性的详细指令说明。既包含背景信息，也包含正向和负向的相关性标准。|
|**在论文中**|narrative 被用作IR模型的“指令”，测试模型能否像人类一样理解并按照指令完成检索任务。|

---


### Text REtrieval Conference (TREC)

TREC数据集的构建基于一系列信息检索会议（Text REtrieval Conference, TREC），这些会议自1992年以来每年举办一次。数据集的构建过程包括收集大量的文本数据，涵盖新闻文章、网页内容、问答系统输入等多种类型。每个文档都经过严格的标注和分类，以确保数据的高质量和多样性。此外，TREC数据集还包含了大量的查询任务，这些任务由专家设计，旨在模拟真实世界中的信息检索需求。

这些数据集通常包括：
1. **文档集合（corpus）**：比如新闻文章、网页等。
2. **查询（topics）**：真实或模拟的用户查询问题。
3. **人工标注的文档相关性判断（qrels）**：即每个文档与每个查询的相关/不相关标签（由人工标注员根据指令进行判定）。

因此，TREC更像是一个**数据集合集 + 比赛任务平台 + 标注流程的集合**。TREC collections

Each year TREC sponsors many tracks, or shared tasks, on a given dataset.
在TREC会议中，主办方会发布各种主题（track），并针对每个track提供相应的**检索任务和数据集**。研究者通常会把TREC中的某个任务（track）对应的数据集合称为“一个TREC数据集”。
- **TREC Robust 2004**：一个面向新闻检索的任务，包含新闻文章和相关查询。
- **TREC Common Core 2017**：基于纽约时报语料库的检索任务。
- **TREC News 2021**：包含华盛顿邮报新闻文章的数据集。

在论文中，作者提到使用了这些TREC数据集，包括文档集合、查询和“指令（narrative）”，这些指令是专业标注员在评估文档相关性时用到的、用于解释查询背景和判定标准。

TREC数据集以其广泛的主题覆盖和高质量的标注著称。数据集中的文档和查询任务涵盖了从新闻报道到科技文献的多个领域，为研究者提供了丰富的实验材料。此外，TREC数据集的多样性体现在其包含了多种类型的**查询任务**，如精确检索、模糊检索和问答系统测试等，这使得该数据集在信息检索领域的研究中具有极高的实用价值。

Typically, this is done by pooling (汇集) a set of results (runs) from a variety of retrieval models and then annotating them in rank order until funding runs out.
这个标注过程通常是：
1. **Pooling**：先把很多检索模型（不同团队提交的，把多种模型结果合并）跑出来的文档结果合并到一个大列表（即runs）。
2. **Rank order**：对这些文档按模型的排名顺序（即检索模型的排序）排列。
3. **Annotating them in rank order**：然后从上到下（从最可能相关到不太可能相关）让人工标注员进行标注。
4. **Until funding runs out**：一直标注到资金用完为止（因为人工标注很贵，不可能标注完所有文档）。

📌 这种方法保证了大部分**高分文档都能被标注到**，因为这些文档更有可能是相关的。

如果要计算每个查询的完整召回率（recall），需要对整个语料库中每篇文档都打标记（相关/不相关），这对于几百万个文档的集合来说显然不现实。  

recall error is tested using post-hoc sampling and annotation.
因此，研究者会用post-hoc sampling 和 annotation的方法来估计召回率，而不是全部标注。
虽然不能穷尽每个查询-文档的组合，但总体上，TREC的召回率还是比较高的（因为通过pooling保证了大部分相关文档都会被检索出来并被标注到）

**Recall**（召回率）在信息检索中表示模型找到了多少真正相关的文档。

Recall = 模型找到的相关文档数量 / 所有真正相关的文档数量​

如果有10篇文档真正与查询相关，而你的模型只找到了其中7篇， recall = 7 / 10 = 0.7

TREC用的是**pooling**方法（把多种模型结果合并）：
- 不只是你一个模型的结果，而是所有参赛者的top结果都合并起来，然后进行人工标注。
- 这样可以确保几乎所有潜在的“相关文档”都至少被一个模型捞到过。
- 这就大幅减少了遗漏文档的风险，保证了标注集合的“相关文档”召回率很高（即使没全标，也几乎不会漏掉太多）。

“post-hoc”来源于拉丁文，意思是“事后”。在这里，它指：

- 不是预先全量标注，而是在模型检索后，再来做额外的标注（补充采样标注）。
- 因为数据量巨大，TREC不可能一开始就对所有文档都标注一遍，只能在模型结果出来后进行采样标注。

**post-hoc sampling 是怎么做的？**
一般步骤：
1. pooling：先从各模型结果里挑出文档（top-k）。
2. 标注员先按排名标注一批。
3. 如果需要估计召回，就从“没标注到的文档”中随机抽样一部分，进行额外的人工标注。
4. 通过这部分样本的相关性比例，来估计剩余文档中可能存在的相关文档数量。

📌 举个例子：
- 你标注了1000篇文档，发现其中100篇是相关的。
- 你随机抽取了100篇未标注的文档，发现其中1篇是相关的。
- 那么可以估计剩余的9000篇未标注文档里大约还有90篇相关文档。
- 这样可以补偿最初标注时可能漏掉的相关文档。


**TREC数据集主要用于信息检索系统的评估和开发**。研究者可以通过使用该数据集来测试和优化他们的检索算法，评估其在不同查询任务上的表现。此外，TREC数据集还可以用于训练和验证机器学习模型，特别是在自然语言处理和信息检索领域。使用TREC数据集时，研究者需要遵循其提供的标准评估协议，以确保结果的可比性和公正性。

**在信息检索领域，TREC（Text REtrieval Conference）数据集被广泛用于评估和比较不同检索系统的性能。该数据集包含了大量的文本数据和查询任务，使得研究人员能够系统地测试和优化检索算法。**

---


## 好词好句

	virtually   adv. 实际上，实质上，事实上，几乎；无形中；无形
	virtual     adj. 实质上的，事实上的；（计算机）虚拟的
	alongside   prep. 在…旁边；沿着…的边；与…一起/同时
		virtually none of them allow users to provide detailed instructions alongside queries 几乎没有允许用户在查询的同时提供详细说明
	exclusively adv.  唯一地；专门地，特定地；专有地；排外地
		However, the vast majority of these systems are fine-tuned to operate exclusively as text spans similarity estimators. 然而，这些系统中的绝大多数都经过了微调，专门用于作为文本跨度相似性估计器运行。
	venue    n. （事件或活动的）发生地；举办地点；场地
	understudy n. 预备演员，替角，代替之人 v. 练习做临时演员, 未被充分研究的
		this topic is still understudied with only a handful of papers. 这个课题目前研究较少，只有少数几篇论文
	explicitly  adv. 明白地，明确地，明显地
	thorough  adj. 详细的；仔细的；完全的
	minute detials 细微细节
		These instructions are thorough and complex, including minute details about what makes a document relevant vs not-relevant. 这些说明非常全面且复杂，包括关于使文档相关或不相关的细微细节
	if   除了表示“如果...就...”， 也可以表示“既然....”
		Thus if annotators can use these TREC instructions to annotate document relevance, so should instruction-following retrieval models. 因此，既然人工标注者可以使用这些 TREC 指令来标注文档的相关性，那么依照指令进行检索的模型也应该能够做到。
	nascent  adj. 初期的； 初生的；开始形成的；发生中的
	incorporae v 组成公司；包含；使混合；使具体化
	flurry  n. 一阵风、雨或雪；爆发，骚动
		Despite this flurry of activity, these efforts do not have an explicit instruction-related retrieval benchmark to evaluate on. 尽管有这些活动，但这些努力并没有一个明确的指令相关检索基准来评估。
	concurrent adj. 同时发生的  In work concurrent to ours