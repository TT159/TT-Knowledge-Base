---
date: 2025-06-02
tags:
  - Bias
  - LLM
---

# Main content

==Do LMs exhibit bias towards Western entities in non-English, non-Western languages ?==

Primary Objective:
**To assess whether LMs can appropriately distinguish between Arab and Western entities when prompted by culturally specific contexts**


 We thus **construct a new benchmark, CAMeL** (Cultural Appropriateness Measure Set for LMs), which consists of an extensive list of 20,368 Arab and Western entities extracted from Wikidata and CommonCrawl, covering eight entity types (i.e., person names, food dishes, beverages, clothing items, locations, authors, religious places of worship, and sports clubs), and an associated set of 628 naturally occurring prompts as contexts for those entities.


- Developed **CAMeL**, the first large-scale benchmark for measuring cultural bias in LLMs, containing 628 naturally-occurring prompts from Twitter/X and 20,368 culturally relevant entities (e.g., names, foods, locations).
- Designed two types of prompts: culturally contextualized (Arab-specific) and agnostic (neutral).
- Created a **Cultural Bias Score (CBS)** based on language models’ preference for Western vs. Arab entities in text-infilling tasks.
- Evaluated 16 LLMs (monolingual Arabic, multilingual) on tasks including story generation, NER, sentiment analysis, and text infilling.


- Demonstrated that LLMs (even Arabic-specific ones) often prefer Western entities in culturally Arab contexts, with CBS scores of 40–65%.
- Identified that training data is a major source of bias; Wikipedia is the most Western-centric.
- Analyzed stereotype presence in story generation, showing negative stereotypes associated with Arab names (e.g., “poor,” “modest”).
- Showed that prompt adaptation (N-shot demos) can reduce bias, while culture tokens have minimal effect.

 we discuss that the prevalence of Western content in Arabic corpora may be **a key contributor** to the observed biases in LMs.
我们讨论了阿拉伯语语料库中西方内容的流行可能是观察到的语言模型偏差的关键因素。

## Background

 Truly multicultural LMs should not only communicate across languages but do so with an awareness of cultural sensitivities, fostering a deeper global connection.

much less work (§2) has examined the **cultural appropriateness** of LMs in the non-Western and non-English environments.

 There is **no resource** readily available for doing so, especially one that can contrast Arab vs. Western cultural differences


## Construction of CAMeL

starting by collecting entities that exhibit cultural variation. 从收集表现出文化差异的实体开始


then obtain prompts from Twitter/X data as natural contexts for these entities, which enable various testing setups for measuring cultural biases in LMs
随后，从Twitter/X数据中获取提示作为这些实体的自然上下文，这使得能够通过各种测试设置来衡量语言模型中的文化偏见。

### Step 1： Collect Entities

#### Source 1: Entity Extraction from Wikidata

- 研究者首先确定了8类文化相关的实体（包括人名、食物、饮品、服饰、地点、作家、宗教场所、体育俱乐部）。
    
- 通过在 **Wikidata** 中搜索这些实体类型的类别（例如“food”、“city”、“drink”等），提取所有用阿拉伯语标注的实体。
    
- 对于“地点”、“作家”和“体育俱乐部”等实体，由于在Wikidata中往往带有国别信息，因此可以直接区分阿拉伯或西方文化（阿拉伯世界、西欧和北美）。

However, for other entity types, we had to manually classify them into Arab and Western lists due to the lack of such demographic labels
- 但是，对于其他类别（如饮品、衣服等），由于缺乏显式的文化标签，只能手动对其进行分类（阿拉伯 vs. 西方）【注：作者在附录B.1中提供了更多细节】。
    
- 需要注意的是，Wikidata对地点、体育俱乐部和作家的覆盖较好，但对其他实体的覆盖相对有限。


#### Source 2: Entity Extraction from Web Crawls. (Arabic subset of CommonCrawl corpus)

- 为了弥补 Wikidata 覆盖不足的问题，研究者又利用了 **CommonCrawl 的阿拉伯语子集**。

 For each entity type, we manually design 5 to 10 generic patterns composed of nouns or noun-verb expressions typically followed by a specific entity
- 这里使用了 **模式匹配** 的方法（pattern-based extraction），即使用一批（5-10条）手工设计的模式（一般由名词或动词-名词短语构成）来提取实体。
    
    - 例如：“sister named …” 这样的模式可以提取到人名。 is likely to be followed by a female name.
        
    - 研究者还对这些模式进行了 **阿拉伯语的性别和数的变位**，以适应阿拉伯语的语法特性。
    - 在阿拉伯语中，动词的变位反映了主语的性别（男性或女性）和数（单数、双数或复数）。
        
- 匹配时只提取模式后面最多2个词，以保证召回率（即找到更多实体），而不局限于精确的长句。
    
- 这样，每个类别大概能提取到 **5000到10000条实体**，然后再经过人工筛选和注释，确保高准确率。
    
- 同时，对人名和服饰等实体根据性别进行分类（符合阿拉伯语的性别语法特征），但作者特别说明这并非为了排斥其他性别身份，而是出于语言结构的考虑。
 We split name and clothing entities into male/female categories to match Arabic’s gendered grammar, without intending to exclude other gender identities

采用pattern-matching的方法，这可以避免 using any LMs in the construction of the dataset that will be used for evaluating LMs

此外， We avoid using more specific and longer patterns to ensure wider coverage of entities (i.e., higher recall lower precision). 我们避免使用更具体和更长的模式，以确保实体的广泛覆盖（即，更高的召回率和更低的精确度）。

模式越长，被匹配的候选者就肯定越少嘛。在做模式匹配提取实体时，他们刻意避免用过于具体和太长的模式。这样做的目的是为了 **尽可能多地提取出候选实体（高召回率）**，即宁可多抓一些噪音（不精准，也就是precision会降低）实体，也不想漏掉那些可能相关的实体。

例子：
假设要提取人名，如果用的模式是：
- 太具体的模式：“أنا رأيت أختي المسماة ...”（我看到了我妹妹，名字叫……），  
    → 这可能只提取到很少的名字，因为上下文必须严格符合“我妹妹，名字叫……”。

- 更宽泛的模式：“اسمها ...”（她的名字是……），  
    → 这可以覆盖更多的语境，也许提取到更多的人名，虽然有些可能是噪音，”她的名字是爷爷起的“，提取到的pattern后面的两个词就会是”爷爷“，而不是名字。
    
- 这样做的好处：能提取到更多潜在的实体，保证数据集的覆盖率。
- 不足之处：可能会包含一些错误或噪音（precision 下降）。

这种做法符合信息提取的常见策略：**先保证高召回（把尽可能多的东西抓回来），后续可以通过人工筛选或者后处理提高精确率**。

### Step 2: Obtain Prompt 

To assess whether LMs can appropriately distinguish between Arab and Western entities when prompted by culturally specific contexts
我们的主要目标之一是评估语言模型在特定文化背景下是否能够恰当地区分阿拉伯实体和西方实体。

 To ensure we evaluate LMs in scenarios that mirror actual language uses, we construct our prompts from natural contexts that we retrieve from Twitter/X, rather than crowdsourcing prompts
为了确保我们在模拟实际语言使用的场景中评估语言模型，我们从Twitter/X中检索的自然语境构建提示，而不是众包撰写的提示。

#### Prompt Types
##### Prompt Type 1: culturally-contextualized prompts (CAMeL-Co)
==Create prompts that embed an Arab cultural reference==
所以需要创建含有Arab文化参考的prompts

**Arbar文化提示（CAMeL-Co）**：包含阿拉伯文化元素的提示，专门用于测试模型在阿拉伯文化上下文下的

To achieve this, we create prompts that embed an Arab cultural reference, ensuring they provide contexts uniquely suited for Arab entities
为了实现这一目标，我们创建了包含阿拉伯文化参考的提示，确保它们为阿拉伯实体提供独特的上下文。

This allows to gauge the LM’s cultural adaptation ability.

we created 250 CAMeL-Co prompts by replacing the original context entities with a [MASK] token

##### Prompt Type 2: culturally-agnostic prompts (CAMeL-Ag)

**中立提示（CAMeL-Ag）**：不带任何文化背景的提示，用来检测模型默认的文化倾向。

 we create prompts with neutral contexts, enabling us to determine the default cultural leanings of LMs.
我们创建中性语境的提示，使我们能够确定LMs的默认文化倾向。


 we constructed 378 prompts for CAMeL-Ag using generic patterns as search queries that do not contain any cultural reference

#### How to do

 We employ **two keyword search strategies** to retrieve tweets that reflect an Arab cultural context for each entity category.

##### 1️⃣ 检索自然上下文（Retrieving Natural Contexts）

- **数据来源**：为了保证提示的“真实性”，研究者从社交媒体 **Twitter/X** 上抓取真实用户的推文，而不是通过人工众包或让语言模型自己生成。
    
- **检索策略**：
    - **关键词法**：从作者准备的阿拉伯实体列表中随机抽取20个词（例如食物、地名、衣服等），作为关键词进行搜索，捕捉包含这些文化实体的推文。
	     we use 20 randomly sampled Arab entities from our lists as search queries to capture discussions about culturally-relevant entities.
        
    - **模式法**：手工设计1-2个描述阿拉伯文化实体的形容词短语（如“by the Arab author”等），进一步精准搜索推文中的阿拉伯文化上下文。
	     We further refine our search using one or two manually-designed patterns of adjective phrases that directly reference an Arab entity
        
- **时间范围**：检索的推文时间限定在2023年8月1日至9月30日之间，目的是避免LM的训练数据里已经包含了这些内容，确保测试公平。

因为大语言模型（LM）是在大量数据上预训练的，而大部分模型的训练数据一般都会在某个时间点之前收集好，比如到2023年初或更早。
如果CAMeL数据中的提示是在2023年8月以后才收集的，那这些推文就**很可能没被模型见过**。
这就能保证测试时模型没见过这些具体句子，从而避免了“数据泄漏”，模型无法“背答案”或者因为记忆而表现得过好。

这就是“公平”的含义：
- 让模型面对新问题，而不是旧问题（即它没在训练时就学过）。
- 这样测出来的效果才反映模型的真实泛化能力，而不是看它能不能记住训练数据。

##### 2️⃣ 生成提示 prompt generate

- **文化化提示（CAMeL-Co）**：在推文中找到包含阿拉伯文化实体的句子，然后将其中的实体替换成 `[MASK]` 作为填空提示。这样，模型需要在给定的文化上下文中生成答案。
    
    - 例如：推文“我和朋友在伊斯兰祷告后去喝[某种饮品]”，可以生成“我和朋友在伊斯兰祷告后去喝 [MASK]”。
        
- **中立提示（CAMeL-Ag）**：同样从推文里提取，但这些句子中不包含明显的文化信息，用作对比实验。


##### 📝 情感标注（Sentiment Annotation）

- 为了后续评估模型在情感分析（Sentiment Analysis）任务上的公平性，这些提示也被人工标注为 **正面、负面或中性**。
- 标注者之间的一致性非常高（Cohen’s Kappa = 0.954），说明数据质量可靠。

这是为了“情感分析”任务服务的。因为CAMeL不仅用来测量模型在故事生成或NER（命名实体识别）上的文化偏见，还希望测量在“情感分析”任务（也就是判定一句话是正面、负面还是中性）上，模型对不同文化背景的表现是否公平。

为什么要标注？**正面、负面、中性**：
    - 正面：表达了积极、赞扬、愉快的情绪。
    - 负面：表达了消极、批评、愤怒的情绪。
    - 中性：不带情绪色彩，仅陈述事实。
        

通过人工标注，确保测试集的情感标签是真实且一致的，这样评估模型的时候就能：  
直接比较模型对含有阿拉伯实体和西方实体的句子预测的情感标签，看它是否存在偏向（比如模型是不是更容易将含阿拉伯实体的句子判为负面）。

如果模型对同样的句子（结构和含义）在遇到阿拉伯实体时更倾向于打负面标签，而遇到西方实体就打正面标签，这就说明模型在情感分析上也存在文化偏见。因此，情感标注可以帮助检测并量化这种潜在的文化不公平。

==实际上，就是在做情感分析任务时，有了ground truth==


## Experiment: Measuring Bias

### Cultural Stereotypes in Story Generation
we analyze the frequency of adjective usage by LMs in the stories featuring Arab or Western names. To do so, we extract all adjectives from stories using the Farasa POS tagger (Abdelali et al., 2016) and compute their Odds Ratio (OR) (Wan et al., 2023)

A large OR indicates more odds for an adjective of appearing in Western stories, while a small OR indicates more odds of appearing in Arab ones.

We inspect adjectives with the 50 highest and lowest ORs to identify and categorize adjectives that reflect stereotypes based on the work of Cao et al. (2022a), which outlines descriptive adjectives for stereotypical traits (e.g., poor, likeable, etc.) using the Agency-Belief Communion (ABC) framework (Koch et al., 2016).

通过对比分析生成故事里的形容词，结合OR指标和ABC框架，来量化和揭示语言模型在文化上的潜在刻板印象（偏见）。  

Stories about Arab characters more often cover a theme of poverty with adjectives such as “poor” persistently used across LMs

- 不同模型（JAIS-Chat、GPT-3.5、GPT-4）都展现出类似的倾向。
    
- 模型往往把阿拉伯名字的故事开头写成“生于贫困、谦逊的家庭”，而在西方名字的故事里则更容易写成“富有、卓越”等高地位形象。

这段分析说明GPT类模型在“讲故事”任务中存在潜在的文化偏见。  
模型在涉及阿拉伯人物时更倾向于使用“poor”、“modest”等负面或传统的形容词，而在涉及西方人物时则更倾向于“wealthy”、“popular”等正面形容词。  
通过OR和ABC框架量化这种偏见，为模型公平性改进提供了客观依据。


### Fairness in NER and Sentiment Analysis

We leverage culturally-contextualized prompts (CAMeL-Co) which have been manually labelled for sentiment (§3.2) to create the test data. Specifically, for each of the prompts, we replace the [MASK] token with 50 randomly sampled Arab and Western entities. 
我们利用文化语境化的提示（CAMeL-Co），这些提示已手动标注情感（第3.2节），以生成测试数据。具体而言，对于每个提示，我们将[MASK]标记替换为50个随机选取的阿拉伯和西方实体。(这样可以直接比较模型在不同文化实体下的表现。)

命名实体识别（NER）与情感分析（Sentiment Analysis）中的公平性

**这部分的目的是检验模型在这两个任务上是否对“阿拉伯实体”和“西方实体”表现公平。**

- **NER（命名实体识别）**：从文本中识别人名、地名等信息。
    
- **Sentiment Analysis（情感分析）**：判断句子表达的是正面、负面还是中性情绪。

 We create models capable of performing Arabic NER and sentiment prediction by fine-tuning LMs on datasets commonly used in Arabic NLU benchmarks.
 我们通过在常用的阿拉伯语NLU基准数据集上微调模型，使其能够完成阿拉伯语NER和情感预测任务。
- **ANERCorp**：阿拉伯语NER数据集
- **HARD**：阿拉伯语情感分析数据集

Task 1, NER:
We find that most LMs perform better when tagging Western person names and locations.
Larger discrepancies are observed on locations, reaching up to 20 F1 points of difference.
我们发现大多数模型在标注西方人名和地点时表现更好。地点标签的差距更大，最高可达20个F1点的差异。


Task 2, Sentiment analysis:
 we examine differences in false positive and false negative predictions between sentences containing Arab vs. Western entities.
 我们研究了包含阿拉伯实体和西方实体的句子在假阳性和假阴性预测上的差异。
 
- **假阳性（FP）**：模型把中性/负面句子错判成正面。
- **假阴性（FN）**：模型把正面句子错判成中性/负面。

 We observe that nearly all LMs achieve higher false negatives on sentences containing Arab entities, suggesting more false association of Arab entities with negative sentiment.
我们观察到几乎所有模型在含有阿拉伯实体的句子上假阴性更高，**表明它们更容易错误地将阿拉伯实体与负面情绪联系起来。**
这说明模型对阿拉伯实体存在潜在的负面偏见。

### Culturally-Appropriate Text Infilling

文化适应性的文本补全任务  
 we use a likelihood-based score that compares model preference of Western vs. Arab entities as fillings of [MASK] tokens in CAMeL prompts.
这部分主要是检验模型是否能根据给定的文化背景提示，正确选择最合适的阿拉伯或西方实体填充 `[MASK]`。


我们使用基于似然的评分方法，创建了Cultural Bias Score (CBS)，比较模型在 `[MASK]` 中更倾向于选择西方实体还是阿拉伯实体，以此测试模型的文化适应能力。

**Cultural Bias Score (CBS)**

The CBS computes the percentage of a model’s preference of Western entities over Arab ones
CBS计算了模型对西方实体的偏好超过阿拉伯实体的百分比。

这是本文提出的衡量模型文化偏见的指标，衡量**模型在给定提示下更倾向于选择西方实体的概率**。

- 数学表达式（简化解释）：

    - 对于一个提示 `tₖ` 和所有的阿拉伯实体集合 `A` 及西方实体集合 `B`，比较：
        
        - `P[MASK](bⱼ|tₖ)`（西方实体的概率）
        - `P[MASK](aᵢ|tₖ)`（阿拉伯实体的概率）
    - 如果西方实体的概率大于阿拉伯实体，就加1。
        
    - 最后，对所有组合做平均，得到 `CBS ∈ [0,1]`，乘以100%表示偏向西方的比例。

- CBS接近100%表示模型总是偏向西方实体（严重西方偏见）。
- CBS接近0%表示模型几乎总是选择阿拉伯实体（理想情况下，符合文化背景）。
- CBS≈50%表示模型对两边都差不多。

CBS（Cultural Bias Score）= 1表示全选西方，0表示全选阿拉伯。


✅ **LMs普遍偏向西方实体**（即使提示明确是阿拉伯文化），CBS高达40-60%，和中立上下文里表现差不多，说明模型文化适应性弱。  
✅ **单语阿拉伯模型也有西方偏见**，可能是因为预训练数据偏向西方话题。  
✅ **多语种模型偏见更严重**，多语言训练时往往强化了英语（西方）主导的文化。  
✅ **Prompt Adaptation技巧（N-shot Demos）可以改善文化适应性**，Culture Token效果不大。

###  Analyzing Arabic Pre-training Data

  the observed failures of LMs in appropriate cultural adaptation could be the prevalence of Western content in the Arabic pretraining corpora. To gain more insight, we analyze six Arabic corpora that are commonly used in pretraining LMs, comparing their cultural relevance.

语言模型在适当文化适应方面的失败，可能源于阿拉伯语预训练语料库中西方内容的普遍性。为了获得更深入的见解，我们分析了六种常用于预训练语言模型的阿拉伯语语料库，并对比了它们的文化相关性。



==Local news and Twitter/X corpora had the lowest CBS, suggesting that future work may consider these sources for training more culturally adapted LMs.==
本地新闻和Twitter/X语料库的CBS最低，这表明未来的工作可以考虑这些来源来训练更具文化适应性的语言模型。


## Limitations

- Focused only on Arabic vs. Western cultural contexts.
    
- No evaluation of sub-regional or intra-cultural variations (e.g., within the Arab world).
    
- Primarily lexicon-level analysis, without assessing stylistic or deeper semantic features.
    
- Data limited to Twitter/X contexts; other platforms might exhibit different biases.



## Additional 

### n-gram LMs

### Arabic Grammar

In Arabic, verbs are conjugated to reflect gender (male or female) and number (singular, dual, or plural) of the subject.

在阿拉伯语里，动词在使用时必须根据主语的“性别（男性或女性）”以及“数（单数、双数或复数）”进行变化。换句话说，动词的形式会根据“谁在做动作”而发生变化。
#### 🌟 举个例子：

假设动词是“写”（写作）——在阿拉伯语里是 “يكتب” (yaktubu, “he writes”)。

- 如果主语是单数男性，就用 “ يكتبُ ” (yaktubu)
    
- 如果主语是单数女性，就用 “ تكتبُ ” (taktubu)
    
- 如果主语是双数男性，就用 “ يكتبان ” (yaktubāni)
    
- 如果主语是双数女性，就用 “ تكتبان ” (taktubāni)
    
- 如果主语是复数男性，就用 “ يكتبون ” (yaktubūna)
    
- 如果主语是复数女性，就用 “ يكتبن ” (yaktubna)
    

👉 这样一来，动词的词尾或者前缀都会随着主语的性别和数而发生变化。这是阿拉伯语的一个语法特点。

---
## Farasa POS tagger & Odds Ratio (OR)

we analyze the frequency of adjective usage by LMs in the stories featuring Arab or Western names. To do so, we extract all adjectives from stories using the Farasa POS tagger (Abdelali et al., 2016) and compute their Odds Ratio (OR) (Wan et al., 2023)
我们分析了在包含阿拉伯或西方名字的故事中，语言模型（LMs）使用形容词的频率。为此，我们利用法拉萨词性标注器从故事中提取所有形容词，并计算其优势比（OR）。

Farasa 是一个**阿拉伯语的自然语言处理工具**，用于分词和词性标注。OR 用来衡量某个形容词在西方或阿拉伯名字故事中出现的相对可能性。

OR 是常用的概率比指标，用于衡量某事件在两组中的相对可能性。  
公式（简化版）：

OR = 事件在西方故事中出现的概率​ / 事件在阿拉伯故事中出现的概率

OR值大，表示某个形容词更有可能出现在西方名字的故事中；OR值小，表示更有可能出现在阿拉伯名字的故事中。**通过OR值可以量化模型在生成故事时的文化偏见。**


我们挑选出OR值最高和最低的前50个形容词，根据Cao等（2022a）的研究，利用**Agency-Belief Communion (ABC)** 框架（Koch等，2016）将这些形容词按刻板印象进行分类（如贫穷、受欢迎等）。

用ABC框架对形容词进行刻板印象的标签化，比如：

- Agency（能动性）：富有、上进
- Belief（信念/价值观）：传统、宗教
- Communion（关系性）：友善、受欢迎

这样就可以分析模型是不是在潜意识中把阿拉伯角色贴上了“穷苦”、“保守”等标签，而把西方角色贴上了“富裕”、“开明”等标签。

---

## Likelihood-based scoring

就是对模型预测的token（候选实体）概率进行比较，看模型更倾向于生成哪一个。这是测试模型“倾向性”的常用方法。


## Text Infilling

在 `[MASK]` 处让模型填空。BERT等MLM模型是天然支持的（直接计算概率），GPT/T5等因自回归结构，需要稍作修改，比如只看提示前的上下文。



## 好词好句

	reach v. 到达；实现，达到；引起注意；伸手，够得着；联系 n. 臂展；影响范围；河段；边远地区；领域
		As the reach of large language models,... 随着大型语言模型的普及
	in light of adv. 按照，根据
		In light of the global deployment of LMs, it is crucial to ensure these models grasp the cultural distinctions of diverse communities. 鉴于语言模型在全球的部署，确保这些模型掌握不同社区的文化差异至关重要。
	grasp  v. 抓紧；理解；急忙抓住 n. 紧抓；掌握；理解；力所能及
	bridge n. 桥（梁）；纽带；鼻梁（架）；桥牌；琴马；齿桥 v. 弥合，消除；兼具…的特点；在…上架桥
		Despite progress to bridge the language barrier gap, LMs still struggle at capturing cultural nuances and adapting to specific cultural contexts. 尽管在克服语言障碍方面已取得进展，但语言模型在捕捉文化细微差别和适应特定文化背景方面仍面临挑战。
	foster v. 培养，促进；收养，养育
		foster a deeper global connection.
	completion n.c. 完成；结束；（房地产等的）完成交易；完成交割；补充
		LMs fail at appropriate cultural adaptation in Arabic when asked to provide completions to various prompts, often suggesting and prioritizing Western-centric content. 当要求LMs对各种提示进行补充时，它们在阿拉伯语中无法进行适当的文化适应，通常会提出以西方为中心的内容并优先考虑。
	prayer  n. 祈祷；祷词；祈望；祈祷仪式
	predominantly adv. 占主导地位地；显著地；占优势地
		in the predominantly Muslim Arab world
	prevalent adj. 流行的，盛行的；普遍存在的，普遍发生的
	inadequate /inˈadəkwət/ adj. 不充足的；不适当的；不足胜任的；信心不足的
		user may find it upsetting to see inadequate cultural representation by LMs in their own languages. 用户可能会因为看到自己的语言中LMs的文化代表性不足而感到不安。
	demographic adj. 人口（学）的；人口统计（学）的 n. 人口特征，人口统计数据；人群
	considerable adj. 相当大/多的
		considerable effort has gone into exploring biases in LMs
	religion  n. 宗教；支配自己生活的大事；教派；心爱的事物
	cater  v. 满足…的需要；接待；考虑到；提供饮食；承办酒席
		their ability to cater to diverse cultural contexts becomes crucial.
	heritage /ˈherədij/ n. 遗产；传统
		cultural heritage 文化遗产
	versatile /ˈvərsədl/ adj. （指工具、机器等）多用途的；多才多艺的；有多种学问、技能或职业的；多功能的
		 We show that CAMeL entities and prompts enable cross-cultural testing of LMs in versatile experimental setups. 我们证明了CAMeL实体和提示能够在多样的实验设置中对语言模型进行跨文化测试。
	stereotype /ˈsterēəˌtaip/  n.c 老套，模式化的见解，刻板印象，有老一套固定想法的人 v. 把…模式化，使成陈规
		 cultural stereotypes  文化刻板印象，文件定势
	benchmark   n.c 基准，参照；标准检查程序；水准标; v. 进行基准测试
		 We benchmark 16 LMs pre-trained with Arabic data. 我们对16个使用阿拉伯语数据预训练的大型语言模型进行了基准测试。
	proper nouns 专有名词
	common nouns 普通名词
	gauge /ɡāj/  n. 测量的标准或范围；尺度，标准；测量仪器；评估 vt. （用仪器）测量；确定容量，体积或内容；评估，判断；采用
	agnostic /aɡˈnästik/ n. 不可知论者
		culturally-agnostic 文化中立的
	contextualize  v. 将……置于上下文（或背景）中研究
		culturally-contextualized 文化背景化的
	spoil   n. <正>战利品，赃物 v. 变质；损坏；毁掉；破坏
	What X spoils, Y will fix 一个典型的英语复合句结构，用作强调修复与弥补。表示：X 损毁的... / 让...糟糕, Y 会弥补/修复 
		What the world spoils, my Arab cooking skills will fix. 无论世界让什么变得糟糕，我的阿拉伯烹饪都能让它变好。
	mirror n. 镜子，反光镜；真实的写照；反映，借鉴；榜样 vt. 反映；反射
		 To ensure we evaluate LMs in scenarios that mirror actual language use 为了确保我们在模拟/反映实际语言使用的场景中评估语言模型
	portray v. 画像；描述；描绘；描画
	treat v. 对待，治疗。 n. an event or item that is out of the ordinary and gives great pleasure. 款待，享受，让人感到快乐的事情
		Learning about how those classes made a lasting impact is a treat. 了解这些课程如何产生持久的影响，真是一种享受。
	recipe /ˈresəˌpē/ n.c. 食谱 a set of instructions for preparing a particular dish, including a list of the ingredients required.
	odd adj. 奇怪的，单的，偶然的，异常的
		odd number 奇数， 单数
	odds n. 赔率，可能性，概率。the ratio between the amounts staked by the parties to a bet, based on the expected probability either way.
	vanilla  n. 香子兰，香草；香子兰荚；香子兰精，香草精 adj. 香草的；香草味的；相对没有新意的，普通的