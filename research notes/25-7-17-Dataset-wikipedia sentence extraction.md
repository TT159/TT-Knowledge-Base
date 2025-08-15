---
date: 2025-07-17
---



## 一、Wikipedia 句子提取的标准流程

### 1. 下载 Wikipedia 原始语料

维基百科文本并不是现成的 txt 格式，你需要先获得：

#### ✅ 推荐方式：使用 [WikiExtractor](https://github.com/attardi/wikiextractor)

- 它从官方的 Wikipedia XML dump 中提取纯文本内容；
    
- 自动去掉模板、链接、元数据等；
    
- 默认按段落/句子输出，适合后续 NLP 处理。

它会生成形如如下 JSON 格式的内容（每篇文章一条）：
```
{   
	"text": "First paragraph.\nSecond paragraph.\n...",   
	"id": 123456,   
	"title": "Example Article" 
}
```

### 2. 切分为句子

将文本(text 字段)切分为句子，可以用 `nltk`


### 3. 匹配 negation cues

准备好你的 cue 列表（例如来自 CondaQA 分的三类），然后匹配句子


## 二、现成替代方案（如你不想处理 Wikipedia dump）

如果你想快速实验，以下两种数据源可以取代 Wikipedia：

|替代语料|说明|
|---|---|
|OpenWebText|模拟 GPT-2 的爬虫语料（已清洗）|
|CC-News + [BooksCorpus]|通用大规模英文文本，适合用来抽句子|
|Wikipedia via HuggingFace|`datasets.load_dataset("wikipedia", "20220301.en")`|

例如 HuggingFace 直接用代码就能拿到 wiki 文本。



## 自己处理 Wikipedia dump vs 用 HuggingFace 现成数据集：有什么区别？

这是你数据采集策略中的一个核心选择。

|比较维度|自己处理 Wikipedia dump|使用 HuggingFace `wikipedia` 数据集|
|---|---|---|
|📦 数据来源|从 Wikipedia 官方 dump 下载|HuggingFace 已处理好的 Wikipedia|
|🧰 是否需要预处理|✅ 需要 WikiExtractor 提取并清洗文本|❌ 无需处理，直接加载 `["text"]` 字段|
|🧠 可控性（如时间点）|✅ 可选任何时间点的 dump|❌ 只能选公开过的几个版本，如 `20220301`|
|💾 本地存储空间需求|非常大（几十 GB）|小得多（分块加载，不占磁盘）|
|💻 处理门槛|高，需要脚本处理、分句、清洗|低，`datasets.load_dataset()` 即可|
|📈 数据完整性（全量）|✅ 可处理全 Wikipedia（甚至多语言）|HuggingFace 提供的是英文为主的版本|
|🧪 推荐使用场景|非英语 / 精准控制抽取 / 构建新数据集|快速实验、英文语料采样、下游任务预训练|



---



Hugging Face 上的数据集确实 **不全是“官方”发布的**，其中绝大多数其实是 **社区用户（包括个人、研究组、公司）上传和维护的**。


## 🧾 一、Hugging Face 上的数据集来源类型

|类型|举例|来源说明|
|---|---|---|
|✅ 官方数据集（HuggingFace Datasets 团队维护）|`wikitext`, `wikipedia`, `glue`, `squad`|高质量、广泛使用，带有官方文档|
|✅ 研究组织或高校上传|`unimelb-nlp/wikiann`, `stanfordnlp/wikitablequestions`|来自论文或项目，通常有论文引用|
|✅ 公司/实验室上传|`microsoft/wiki_qa`, `meta/llama`|有时用于展示内部系统或基准|
|✅ 个人用户上传|`omarkamali/wikipedia-monthly`|可能是个人实验或爬取内容|

---

## 🔍 二、如何判断数据集是否是“官方”或可靠来源？

你可以通过以下几个指标判断：

|判断方法|如何查看|提示|
|---|---|---|
|**用户名/组织名**|看 URL 或顶部展示，例如 `wikimedia/wikipedia`|`huggingface`, `datasets`, `allenai`, `stanfordnlp` 等通常可信|
|**是否有论文引用**|页面中显示 paper（如 arXiv 链接）|有引用说明该数据集用于正式发表|
|**下载量与关注度**|页面右上角显示（如 63K 下载，875 star）|高下载量代表广泛使用|
|**README 是否完整**|是否有 Dataset Card，是否解释字段含义|简洁无说明可能是临时上传的原始数据|
|**是否在 Papers with Code 出现过**|页面右边“Papers with Code”链接|说明有研究成果用过它|

---


有几种方法可以帮你**系统性地获取所有由 Hugging Face 官方支持的、与 Wikipedia 相关的数据集**列表。下面是最有效的 3 种方法，适合你当前对 Wiki 原始语料需求的任务：


## 方法一：使用 Hugging Face 的 Python API（推荐）

你可以通过 `datasets` 库调用 **官方支持的数据集列表**，再用 `"wiki"` 关键词筛选出所有 Wikipedia 相关的官方数据集。

### 示例代码：

```Python
from datasets import list_datasets

# 获取所有官方支持的数据集（不包括社区上传的）
official_datasets = list_datasets(with_community_datasets=False)

# 过滤掉与 Wikipedia 相关的
wiki_datasets = [d for d in official_datasets if "wiki" in d.lower()]

# 打印出来
for name in wiki_datasets:
    print(name)
```



---
## 方法二：Hugging Face 官网搜索 + “仅官方”筛选（半自动）

虽然官网没有一个“官方数据集”的按钮，但你可以这样做：

1. 访问：https://huggingface.co/datasets
    
2. 搜索框中输入关键词：`wiki` 或 `wikipedia`
    
3. 点击右上角 `Add filters` → **License**
    
4. 选择高质量许可（如 `cc-by-sa-3.0`, `apache-2.0`）或官方维护者（如 `huggingface`, `allenai`）
    
5. 手动筛选出来自于 HuggingFace 组织或论文的数据集
    

虽然这种方式不如 API 自动，但更直观。

---



当你打算使用公开数据集进行模型训练、发布论文或开源项目时，**了解数据集的许可证（license）至关重要**。下面我会解释你提到的两个许可证：`cc-by-sa-3.0` 和 `apache-2.0`。

---

## 🔏 一、`cc-by-sa-3.0` 是什么？

### 📘 全称：Creative Commons Attribution-ShareAlike 3.0

这是一个由 Creative Commons 提供的开源内容许可证，常见于 Wikipedia、Wiktionary 等内容类项目。

### ✅ 你可以：

- 免费使用数据（包括训练模型、提取文本、构建新数据集）；
    
- 可以发布你基于这些数据生成的模型或衍生数据。
    

### ❗️但你必须：

1. **署名（Attribution，BY）**：注明原始来源（如 Wikipedia + 链接）；
    
2. **相同方式共享（ShareAlike，SA）**：如果你修改或衍生数据，也必须在同样的 CC-BY-SA 条款下发布。
    

### 📌 应用场景举例：

- Wikipedia 内容使用的是 `cc-by-sa-3.0`；
    
- 你如果用 Wikipedia 文本来训练模型，发布模型时需要注明数据来源；
    
- 如果你发布修改过的 Wikipedia 语料，你也得继续用 `cc-by-sa-3.0` 授权它。
    

---

## 🔐 二、`apache-2.0` 是什么？

### 📕 全称：Apache License 2.0

这是一个 **非常宽松、允许商业用途** 的开源软件许可证，主要用于代码或数据集。

### ✅ 你可以：

- 商业化使用数据或模型；
    
- 修改数据和代码；
    
- 不强制开源派生版本；
    
- 不要求 ShareAlike（不像 CC-BY-SA）
    

### ❗️你只需：

1. **署名**；
    
2. **包含原始 license 文件**；
    
3. **如果你修改了数据或模型，应说明你做了哪些修改**（用于法律透明性）。
    

### 📌 应用场景举例：

- Hugging Face 上很多官方数据集都采用 Apache 2.0；
    
- 你可以用它训练模型，商用发布、不开源都可以；
    
- 最常用于企业和工业界模型。
    

---

## 🧾 三、总结比较

|项目|`cc-by-sa-3.0`|`apache-2.0`|
|---|---|---|
|内容来源|多用于维基百科、文本数据|多用于代码和 HuggingFace 数据|
|商业用途是否允许|✅ 允许（需署名和开源派生）|✅ 允许，无需开源|
|是否必须署名|✅ 是|✅ 是|
|是否强制开源衍生物|✅ 是（必须同样 CC-BY-SA）|❌ 否|
|常见用途|Wikipedia、开源知识共享|企业模型训练、代码库|
|推荐用途|学术、非营利、可接受共享项目|工业、闭源项目、产品模型|



## Wikipedia 数据集

https://huggingface.co/datasets/wikimedia/wikipedia

### Dataset Summary

Wikipedia dataset containing cleaned articles of all languages.

The dataset is built from the Wikipedia dumps ([https://dumps.wikimedia.org/](https://dumps.wikimedia.org/)) with one subset per language, each containing a single train split.

Each example contains the content of one full Wikipedia article with cleaning to strip markdown and unwanted sections (references, etc.).

All language subsets have already been processed for recent dump, and you can load them per date and language this way:

```python
from datasets import load_dataset

ds = load_dataset("wikimedia/wikipedia", "20231101.en")
```

可以使用 Hugging Face 的 `datasets` 库，按指定日期+语言直接加载对应版本的数据。
每种语言是一个子集（subset），比如 `"en"` 是英文，`"fr"` 是法语。  
每个语言子集目前只提供了一个 `"train"` 划分（split），没有 `"test"` 或 `"validation"`。





## 是否每次都需要重新加载 `load_dataset(...)`？是否可以加速？

### ❌ **不是每次都需要重新下载或加载**：

- Hugging Face 的 `datasets` 使用 **本地缓存机制**。
    
- 第一次运行时，它会下载并处理数据，存入本地缓存目录（默认在 `~/.cache/huggingface/datasets/`）。
    
- 后续运行 `load_dataset(...)` 时：
    
    > ⚡ **自动从本地缓存加载，速度非常快（几秒）**，而不会重新下载或解析。
    

所以只要你不删除 cache，下次加载时间会显著缩短（秒级而不是分钟级）。


## 抽取 10K 句子是否需要扫描全部 Wikipedia 数据？

### ❌ 不需要扫描全部文章就能提取 10K 句子（尤其是 Single-word NEG）

- Wikipedia 中包含上百万篇文章；
    
- “not”、“never” 等 high-frequency cue 出现极其频繁；
    
- 实际上你只需处理一小部分文章（几千篇到几万篇）就能提取够 10K 条满足条件的 NEG pairs。




## negation cues 是否会“一直提取不到”？

### 一般来说，不会“完全提取不到”，但 **部分低频 cue 提取困难** 是可能出现的。

### 主要取决于两点：

| 因素                           | 说明                                                                                  |
| ---------------------------- | ----------------------------------------------------------------------------------- |
| **cue 本身在 Wikipedia 中的真实频率** | 比如 `not`, `never` 非常常见；但像 `little reason to` 或 `fail to` 这种 multi-word negation 就很少 |
| **你设置的采样目标是否过高**             | 如果某个 cue 在 Wikipedia 中只出现 500 次，而你要求采样 1000 个句子，就必然失败                               |


## cue 都来自 CondaQA，会不会太罕见？

### 答案：**大部分都不会太罕见，但尾部 cue 会略稀疏**。

- CondaQA 中的 219 个 negation cues 是从真实维基语料中统计来的；
    
- 通常 top 30–50 个 cue 就能覆盖约 90% 的用例（如：not, no, never, none, cannot, fail to, etc.）；
    
- 但是后面那些比如 `have little reason to`, `unlikely`, `barely` 等，在自然语料中的分布会显著稀疏。


## 解决方案（不建议强求每个 cue 都采够）

| 方法                            | 说明                         | 推荐程度         |
| ----------------------------- | -------------------------- | ------------ |
| ✅ 自然频率采样（你当前方案）               | 确保整体数据集分布自然，模型见到的 cue 比例合理 | ⭐⭐⭐⭐⭐ 推荐     |
| ✅ 设置最小采样阈值 MIN_PER_CUE=50–100 | 控制“过于稀疏 cue”完全缺失           | ⭐⭐⭐⭐ 推荐      |
| ❌ 不建议硬性均衡（equal sampling）     | 会放大稀有 cue，数据不自然，模型泛化差      | ⭐ 不推荐        |
| ⚠️ 针对 tail cue 做句子重写 / 数据增强   | 比如人工合成 `barely` 的句子        | ⭐⭐ 可作为未来补充思路 |


## 错误的匹配：

==NLTK被人名中的 . 截断 ==
S1不完整
S1: by Josep Maria Benet i Jornet. 
S2: The film has no male actors, with all roles played by females.

原文：
Actresses (Catalan: Actrius) is a 1997 Catalan language Spanish drama film produced and directed by Ventura Pons and based on the award-winning stage play E.R. by Josep Maria Benet i Jornet. The film has no male actors, with all roles played by females.

S1被截断了，导致语义不完整。这种现象出现在使用`nltk.sent_tokenize()` 自动分句时，尤其是当句号 `.` 出现在**人名或缩略词中间**时，`sent_tokenize()` 会把这个句号当作一个**完整句子的终结符**

==S1的规则是什么==
S1也含有否定词
S1: He believed that retail trade was in this way unnatural. 
S2: Similarly, Aristotle considered making a profit through interest unnatural, as it makes a gain out of the money itself, and not from its use.

==最初用的是单个换行符\n分隔段落，这个冒号后换行了==
还有一种分隔错误，将: 分隔为两个句子了，提取不完整。好像不是因为冒号的问题，而是这个数据集里喜欢冒号后面加换行，所以再一开始nltk以单个\n作为段落分隔符，就会提取不全。可以试试修改下。
S1: The Semitic languages changed significantly between Proto-Semitic and the emergence of Central Semitic languages, particularly in grammar.
S2: Innovations of the Central Semitic languages—all maintained in Arabic—include:

原文：
The Semitic languages changed between Proto-Semitic and the emergence of Central Semitic languages, particularly in grammar. Innovations of the Central Semitic languages—all maintained in Arabic—include:

The conversion of the suffix-conjugated stative formation (jalas-) into a past tense.
The conversion of the prefix-conjugated preterite-tense formation (yajlis-) into a present tense.
The elimination of other prefix-conjugated mood/aspect forms (e.g., a present tense formed by doubling the middle root, a perfect formed by infixing a /t/ after the first root consonant, probably a jussive formed by a stress shift) in favor of new moods formed by endings attached to the prefix-conjugation forms (e.g., -u for indicative, -a for subjunctive, no ending for jussive, -an or -anna for energetic).
The development of an internal passive.

==这种是因为冒号，然后换了一行== 


==采用两个换行符\n来表示不同段落，则出现了把标题给放到一个句子里。==
S1: 
History
The most common use of the term is in the case of English peerage dignities.


==AFF句子的提取逻辑目前好像有点问题。==
即提取了很多(S1, S2), (S2, S3)这样的句子对。
数据样本1：
S1: Linguists still differ as to the best classification of Semitic language sub-groups.
S2: The Semitic languages changed significantly between Proto-Semitic and the emergence of Central Semitic languages, particularly in grammar.

数据样本2：
S1: The Semitic languages changed significantly between Proto-Semitic and the emergence of Central Semitic languages, particularly in grammar.
S2: Innovations of the Central Semitic languages—all maintained in Arabic—include:
 The conversion of the suffix-conjugated stative formation (jalas-) into a past tense.

**spaCy 能减少你遇到的句子截断问题（比如 `S1: by Josep Maria Benet i Jornet.`）的根本原因**，在于它使用了更**复杂、上下文感知的句子分割算法**，而不像 `nltk.sent_tokenize()` 那样主要依赖**规则+正则表达式**。


==S2中含有两个negation cues，其中一个不来自condaQA。无法避免==
spacy解决不了的问题：
S1: He said the Kansas Act had a "declared indifference, but as I must think, a covert real zeal for the spread of slavery.
S2: I cannot but hate it.

S2包含两个否定词。cannot， hate， 但是hate不是condaQA里的negation cue，所以是没有办法的。这种情况肯定会出现。
问chatgpt，它说hate严格说不算是negation cue

S1: According to the Achilleid, written by Statius in the 1st century AD, and to non-surviving previous sources, when Achilles was born Thetis tried to make him immortal by dipping him in the river Styx; however, he was left vulnerable at the part of the body by which she held him: his left heel .

It is not clear if this version of events was known earlier.


None of the sources before Statius make any reference to this general invulnerability.



==7.18号改进后发现的新问题==
S1: Pseudo-reciprocity.
S2: An organism behaves altruistically and the recipient does not reciprocate but has an increased chance of acting in a way that is selfish but also as a byproduct benefits the altruist.
原文：
- Pseudo-reciprocity. An organism behaves altruistically and the recipient does not reciprocate but has an increased chance of acting in a way that is selfish but also as a byproduct benefits the altruist.
看来还得想办法避免这种前句太短的。甚至都不是一个句子的。

## 为什么 spaCy 更准确？

### ✅ 1. **上下文感知的分句模型**

spaCy 使用的是基于机器学习的句子分割器（默认是基于 transition-based parsing），它能根据**上下文结构和语法依赖**判断句子边界。例如：

- 对于像 `"E.R."` 这样的缩写，spaCy 能识别它是人名或作品名的一部分，而不是句末。
    
- 它知道 `"by Josep Maria Benet i Jornet."` 在语法上并不构成完整句子，不应独立分割。
    

### ✅ 2. **依赖句法分析（Dependency Parsing）**

spaCy 在分句时会先做句法分析，它知道哪些 token 构成一个句子的主干（如主谓结构），因此不会轻易把句子在奇怪的地方切断（如作者名字后）。

这比 `nltk` 的规则式分句更智能。

---

## 📉 nltk 的问题来源

### 🔻 规则驱动（rule-based）

nltk 默认用 **PunktTokenizer**，这是基于一些固定规则和句号位置的打分系统。它容易受以下影响：

|情形|nltk 常见误分句表现|
|---|---|
|缩写后跟大写字母|`E.R. by` 被当作句号结尾|
|引号、括号等边界处理|`"He said." She walked.` 拆错|
|名字含句号|`J. R. R. Tolkien` 会被拆|

例如你遇到的句子：

> `Actresses (Catalan: Actrius) is a 1997 Catalan language Spanish drama film produced and directed by Ventura Pons and based on the award-winning stage play E.R. by Josep Maria Benet i Jornet.`

在 `nltk` 中，它会认为 `E.R.` 后是句末，而 **`by Josep Maria...`** 是新句开头，于是你得到了错误的：

- `S1: by Josep Maria Benet i Jornet.` （残句）
    

---

## ✅ 结论

|对比项|`nltk.sent_tokenize()`|`spaCy` 分句 (`doc.sents`)|
|---|---|---|
|技术基础|规则 + 简单模型|依赖句法分析、上下文、实体识别|
|对缩写/专有名识别|较差（容易误判）|较好（知道哪些是缩写/名字）|
|对不完整句处理|可能保留残句|倾向跳过语法不完整的句|
|对真实语料适配性|中等，容易出错|强，尤其适合维基百科这类复杂文本|


但是spacy慢很多。。。

## spaCy vs NLTK 分句性能比较

|特性|`spaCy`|`nltk.sent_tokenize`|
|---|---|---|
|**准确性**|✅ 更高（语法敏感、上下文感知）|⚠️ 较低（基于正则规则）|
|**处理速度**|❌ **较慢**（加载模型 + 解析）|✅ **非常快**（基于规则）|
|**依赖资源**|需要加载 `en_core_web_sm` 等模型（占内存）|只需 punkt tokenizer|
|**解析能力**|支持命名实体识别、词性标注等丰富信息|不支持，只分句|
|**适合任务**|复杂句法敏感任务，避免误切|快速、大规模分句处理|

---

## 🧠 为什么 spaCy 慢？

1. **底层用了统计/神经模型**：spaCy 的分句器基于训练好的模型进行句法解析。
    
2. **加载了整个 NLP pipeline**：即便你只用句子分割，spaCy 默认也会加载词性标注器、NER 等模块，除非显式关闭。
    
3. **精度优先于速度**：它追求更精确的边界识别（如避免把缩写 `Dr.` 当作句末）。

如果你只需要分句而不是 NER、POS，可以使用简化版：
```Python
# 加载时关闭不必要的管道
nlp = spacy.load("en_core_web_sm", disable=["tagger", "parser", "ner", "lemmatizer"])
nlp.add_pipe("sentencizer")  # 仅添加分句器，基于句号规则

def split_into_sentences(text):
    doc = nlp(text)
    return [sent.text.strip() for sent in doc.sents]
```

这种方式可大大提升分句速度，但精度略逊于 spaCy 的 full parser。

**总结建议:**

- 如果你追求**精度**（避免误切），并且数据量不大，建议使用 **spaCy**。
    
- 如果你追求**效率**（如大规模初筛），建议用 **NLTK** 分句 + 后处理清洗异常。


### 1. 为什么用 `nltk` 更合适你的情况？

你提到：

> 我需要提取上百万的句子对

- `spaCy` 的句子分割过程涉及：
    
    - 加载完整模型（语言模型、词性标注、NER 等）；
        
    - 对每一段文字进行**依存句法分析**；
        
    - 这在百万篇文章级别上，会造成**非常显著的时间和内存开销**；
        
- `nltk.sent_tokenize()` 是基于 **规则和标点的轻量级分句**，速度远快于 spaCy；
    
- 实际项目中，如果对分句**鲁棒性要求不是极高**，**nltk 是首选**。
    

### 2. 为何要用 `text.split("\n\n")` 而非 `\n`？

你观察得非常准确：

- Wikipedia 原始文本结构中：
    https://huggingface.co/datasets/wikimedia/wikipedia/viewer/20231101.en?views%5B%5D=_20231101en&row=0
    
    - **两个换行符 `\n\n` 表示段落分隔**；
        
    - 单个换行符通常仅仅表示句中断行或段内句子断开；

使用的huggingface 数据集上显示了原数据集中是用两个换行符表示不同段落。



代码里对aff句子进行补充采样，从而使得neg与aff句子平衡，（因为尽量使得在同一文章中采样相同数目的aff和neg句子对）我设计了一个seen_aff set用于规避在补充采样时，重复采样到之前采样neg时采样过的aff句子。

我之前顾虑，我的数据集是上百万的数量级，会不够，实际上不会。

### **方案合理性分析**

#### ✔ 你现在的逻辑：

- 每提取一个新的 AFF 句子对 `(S1, S2)`，就把它加进 `seen_aff_pairs`（通常是一个 `set`）。
    
- 下一轮判断时，如果这个对已经在 `seen_aff_pairs` 中，就跳过，避免重复。
    

这是**经典的去重方法**，在 NLP 任务中极为常见，尤其是在句子采样、对比学习、负样本挖掘等任务中。

#### ✅ **为什么可行？**

- `set` 查重复杂度是 **O(1)**，所以即使你有上百万条句子对，也能高效判断是否重复。
    
- Python 的 `set` 可以轻松容纳上百万元素，内存负担在现代机器上是可接受的。