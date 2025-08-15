


# MoNLI 

> MoNLI contains 2,678 NLI examples in the usual format for NLI datasets like SNLI. In each example, the hypothesis is the result of substituting a single word wp in the premise for a hypernym or hyponym wh. We refer to wh and wp as the substituted words in an example. In 1,202 of these examples, the substitution is performed under the scope of the downward monotone operator not. Downward monotone operators reverse entailment relations: dance entails move, but not move entails not dance. We refer to these examples collectively as NMoNLI. In the remaining 1,476 examples, this substitution is performed under the scope of no downward monotone operator. We refer to these examples collectively as PMoNLI.


MoNLI 数据集包含 2678 个自然语言推理（NLI）样本，格式与常见的 NLI 数据集（如 SNLI）相同。每个样本包括一个前提句（premise）和一个假设句（hypothesis），并带有一个标签（如 entailment或neutral）。这篇文章==只有这两种标签==。没有contradiction这个关系。

每个样本的构造方式是：从前提句中选一个词 `wp`，将其替换为其 **上义词（hypernym）** 或**下义词（hyponym）** `wh`，从而得到假设句。这种替换考察的是词义层次结构对推理关系的影响。

例如：

- 前提Premise S1：She is **dancing**.
    
- 假设Hypothesis（替换 dancing 为 hypernym move） S2：She is **moving**.

"She is dancing" ENTAILS "She is moving"   ->  S1 entails S2

可以从premise推理出hypothesis，则称为entail

这里定义了 `wh`（被替换进来的词）和 `wp`（被替换出去的词）统称为“替换词（substituted words）”。

在 1202 个样本中，这种词的替换发生在一个**向下单调（downward monotone）运算符**（==`not`==）的作用域之下。也就是说，原句中包含否定成分， “not”。

Downward monotone operators reverse entailment relations: dance entails move, but not move entails not dance. 
即这种向下的运算符not会将这个entailment关系反转。
在没有否定时，语义蕴含entailment是**从具体到抽象**：

- “dance” 是 “move” 的一种，因此：
- dance ⊨ move （跳舞→移动）    

但在否定环境下，这种蕴含关系被**反转**了。例如：
S1: She is not dancing
S2: she is not moving
 - 不再是S1 entails S2
 - 而是被反转为S2 entails S1。即she is not moving ENTAILS she is not dancing

这些在否定环境下的替换样本（即存在 `not`）构成了一个子集，称为 **NMoNLI**（Negative Monotonicity NLI）。剩下的 1476 个样本是在**非否定环境**下构造的，称为 **PMoNLI**（Positive Monotonicity NLI），原句中没有 `not` 或类似的否定成分。

这个数据集由人手工构造。

Limitation
查看数据集发现，里面由大量高度相似的句型，句子。如下，这显然是为了减少人工构造数据的成本。但是也可能会带来bias?!

第二个是数据集只包含not这一种negation cues。而否定的形式是有很多种的，仅仅not这一种是不充分的。
{"sentence1": "There is not a single person walking in the city.", "sentence2": "There is not a single debutante walking in the city.", "gold_label": "entailment", "depth": -3, "sentence1_lex": "person", "sentence2_lex": "debutante"}

{"sentence1": "There is not a single debutante walking in the city.", "sentence2": "There is not a single person walking in the city.", "gold_label": "neutral", "depth": 3, "sentence1_lex": "debutante", "sentence2_lex": "person"}

{"sentence1": "There is not a single person walking in the city.", "sentence2": "There is not a single girl walking in the city.", "gold_label": "entailment", "depth": -3, "sentence1_lex": "person", "sentence2_lex": "girl"}

{"sentence1": "There is not a single girl walking in the city.", "sentence2": "There is not a single person walking in the city.", "gold_label": "neutral", "depth": 3, "sentence1_lex": "girl", "sentence2_lex": "person"}



# ScoNe paper

[[2023-ScoNe-Benchmarking Negation Reasoning in Language Models With Fine-Tuning and In-Context Learning]]

在MoNLI基础上再拓展的，也是做negation相关的NLI任务。即也是推理句子蕴含关系的任务。
其依旧也是只考虑not这一种negation cues，但是其更多的关注与negation scope，即否定的范围以及not的个数，考虑了1~2个not。


![[ScoNe.png]]


# CondaQA

[[2022-CondaQA-A Contrastive Reading Comprehension Dataset for Reasoning about Negation]]

original passage -> original sentence + 3 different edits (paraphrase, scope, affirmation)

纯众包人工打造。QA的answer部分也用InstructGPT做过回答，来得到标签，与人工的标注进行对比。

数据集构造
第一阶段，先告诉众包员工如何select passage。然后，针对原始段落，进行3种改写。
第二阶段，针对原始段落，写3个问题。 
>In the second stage of the task, the crowdworker asks at least three questions about the implications of the negated statement in the ==original Wikipedia passage==. We encourage the construction of good questions about implications by providing several examples of such questions. Crowdworkers can group these questions, to indicate questions that are very similar to each other, but admit different answers.

这些问题是与“改写”无关的，完全基于原始 passage 中的“否定句”来构造的。并不是“每个改写生成一个问题”，而是**针对原始的 negated sentence，提出至少 3 个关于其语义影响的问句**。
我的理解是：针对原段落，提出3个问题，这3个问题是根据原段落的句意可以推理得到答案的问题，不是那种直接明了的挖空一样的问题，比如，James didn't attend the meeting. 问题，Did James attend the meeting?不是这种问题。

>Questions should be targeted towards the implications of a negated statement, rather than the factual content of what was or wasn’t negated, to remove common sources of spurious cues in QA datasets

问题应针对否定陈述的含义，而不是被否定或未被否定的事实内容，以消除问答数据集中常见的虚假线索来源。

make three types of modifications to the ==original passage==. 3 types of edits:
(1) they paraphrase the negated statement, 
(2) they modify the scope of the negated statement (while retaining the negation cue), and 
(3) they undo the negation.
### Data Fields
Github link:
https://github.com/AbhilashaRavichander/CondaQA?tab=readme-ov-file#data-fields

- **`sentence1`: passage**  当前的段落（可能经过了修改）
- **`sentence2`: question**
- `label`: answer
- `QuestionID`: unique ID for this question (might be asked for multiple passages)
- `original cue`: Negation cue that was used to select this passage from Wikipedia
- `PassageEditID`: 0 = original passage, 1 = paraphrase-edit passage, 2 = scope-edit passage, 3 = affirmative-edit passage
- **`original passage`**: Original Wikipedia passage the passage is based on (note that the passage might either be the original Wikipedia passage itself, or an edit based on it)
- `SampleID`: unique ID for this passage-question pair
- **`original sentence`**: Sentence that contains the negated statement
- `PassageID`: unique ID for the Wikipedia passage

一个原始段落 -> 1个 original sentence + 3种改写 & 3个问题
所以会有3 x 4 = 12个样本

- `original sentence` 是单句，抓住的是 **否定结构本身**。
- `original passage` 是维基百科的原始段落
- `sentence1` ，是当前的这个passage，可能是原始也可能是经过edit的，提供**上下文**用于推理和问答。

==输入模型的是sentence1与question，从而获得一个答案。==

### 三种改写是基于哪个改？

三种编辑类型如下：

|`PassageEditID`|类型|改写对象|改写目标|是否仅改句子？|
|---|---|---|---|---|
|**1**|**Paraphrase-edit**|`original sentence`|用不同措辞表达同样含义（保持否定）|✅ 是，仅该句子|
|**2**|**Scope-edit**|`original sentence`|改变否定词的语法范围（扩大/缩小 negation）|✅ 是，仅该句子|
|**3**|**Affirmative-edit**|`original sentence`|改为与其语义相反的**肯定句**（去除 negation）|✅ 是，仅该句子|

- 改写操作**不是改整个 passage**，而是基于 `original sentence`。
- 但为了构成一个新的 passage，**系统会将改写后的句子替换回 `original passage` 中的位置**，形成新的 `sentence1`。

换句话说：

> **原始句子是改写的核心单元；原始段落是改写句子被嵌入的上下文环境。**

---

####  举个例子说明整个流程
假设：
- `original sentence`:  
    **"The law does not allow public protests after 10 PM."**
    
- `original passage`:  
    "The law in the city has changed. **The law does not allow public protests after 10 PM.** This rule has been controversial."

现在的三种改写分别如下：

1. **Paraphrase-edit（措辞改写）**
    - 改写为："Public protests are banned by law after 10 PM."
    - 用新句子替换原段落中旧句子，形成新的 passage。

2. **Scope-edit（范围调整）**
    - 改写为："The law does not allow public **gatherings** after 10 PM."
    - 修改被否定的对象，扩大 negation 作用范围。

3. **Affirmative-edit（变为肯定）**
    - 改写为："The law allows public protests after 10 PM."
    - 意图是生成对立的句子（用于探究模型是否能区分 polarity 变化）


换句话说：

> **==original sentence是改写的核心单元==；原始段落是改写句子被嵌入的上下文环境。**

---

- **`sentence1` 就是当前样本使用的那一版本的 passage（段落）**。若 PassageEditID = 0，表示我们使用的是未修改的原文段落。
`sentence1` = `original passage`（如果 PassageEditID = 0）  
或者 = 基于 `original sentence` 改写后嵌入的 passage（如果 PassageEditID ≠ 0）

{"QuestionID": "q10", "original cue": "with the exception of", "**PassageEditID**": 0, "**original passage**": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3682, "label": "YES", "**original sentence**": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "**sentence2**": "Do any bulletins have more than one presenter?", "PassageID": 310, "**sentence1**": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed."}
==**这个例子里PassageEditID=0， 所以sentence1与original passage是一样的。**==

{"QuestionID": "q11", "original cue": "with the exception of", "PassageEditID": 0, "original passage": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3683, "label": "NO", "original sentence": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "sentence2": "Do all of the bulletins have less than two presenters?", "PassageID": 310, "sentence1": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed."}

{"QuestionID": "q12", "original cue": "with the exception of", "PassageEditID": 0, "original passage": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3684, "label": "NO", "original sentence": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "sentence2": "Does more than one bulletin have multiple presenters?", "PassageID": 310, "sentence1": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed."}

{"QuestionID": "q10", "original cue": "with the exception of", "**PassageEditID**": 1, "**original passage**": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3685, "label": "YES", "**original sentence**": "**Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed**.", "**sentence2**": "Do any bulletins have more than one presenter?", "PassageID": 310, "**sentence1**": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). **Each bulletin is read by a single sports presenter, other than Saturday \"Sportsday\", which is double headed.**"}
==**这个例子里PassageEditID=1， 是paraphrase, 所以sentence1是original passage的改写。**==


{"QuestionID": "q11", "original cue": "with the exception of", "PassageEditID": 1, "original passage": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3686, "label": "NO", "original sentence": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "sentence2": "Do all of the bulletins have less than two presenters?", "PassageID": 310, "sentence1": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, other than Saturday \"Sportsday\", which is double headed."}

{"QuestionID": "q12", "original cue": "with the exception of", "PassageEditID": 1, "original passage": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3687, "label": "NO", "original sentence": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "sentence2": "Does more than one bulletin have multiple presenters?", "PassageID": 310, "sentence1": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, other than Saturday \"Sportsday\", which is double headed."}

{"QuestionID": "q10", "original cue": "with the exception of", "**PassageEditID**": 2, "**original passage**": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3688, "label": "YES", "**original sentence**": "**Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.**", "**sentence2**": "Do any bulletins have more than one presenter?", "PassageID": 310, "**sentence1**": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). **Each bulletin is read by a single sports presenter, with the exception of weekend bulletins, which are double headed.**"}

{"QuestionID": "q11", "original cue": "with the exception of", "PassageEditID": 2, "original passage": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3689, "label": "NO", "original sentence": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "sentence2": "Do all of the bulletins have less than two presenters?", "PassageID": 310, "sentence1": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of weekend bulletins, which are double headed."}

{"QuestionID": "q12", "original cue": "with the exception of", "PassageEditID": 2, "original passage": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3690, "label": "YES", "original sentence": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "sentence2": "Does more than one bulletin have multiple presenters?", "PassageID": 310, "sentence1": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of weekend bulletins, which are double headed."}

{"QuestionID": "q10", "original cue": "with the exception of", "PassageEditID": 3, "original passage": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3691, "label": "NO", "original sentence": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "sentence2": "Do any bulletins have more than one presenter?", "PassageID": 310, "sentence1": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, including Saturday \"Sportsday\", which also has just one presenter."}

{"QuestionID": "q11", "**original cue**": "with the exception of", "**PassageEditID**": 3, "**original passage**": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3692, "label": "YES", "**original sentence**": "**Each bulletin is read by a single sports presenter, ==with the exception of== Saturday \"Sportsday\", which is double headed.**", "**sentence2**": "Do all of the bulletins have less than two presenters?", "PassageID": 310, "**sentence1**": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). **Each bulletin is read by a single sports presenter, ==including== Saturday \"Sportsday\", which also has just one presenter.**"}

==**这个例子里PassageEditID=3， 是affirmative, 所以sentence1是original passage的改写。

{"QuestionID": "q12", "original cue": "with the exception of", "PassageEditID": 3, "original passage": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "SampleID": 3693, "label": "NO", "original sentence": "Each bulletin is read by a single sports presenter, with the exception of Saturday \"Sportsday\", which is double headed.", "sentence2": "Does more than one bulletin have multiple presenters?", "PassageID": 310, "sentence1": "Headlines are usually provided at 15 minutes past the hour with a full bulletin after the bottom-of-the-hour headlines. There are also extended sports bulletins per day, entitled \"Sportsday\" or \"Sport Today\" (when simulcasting with BBC World News) broadcast at 00:45, 01:45, 02:45, 03:45, 13:30, 18:30, 19:30 (weekends only), 22:30 (weekdays only). Each bulletin is read by a single sports presenter, including Saturday \"Sportsday\", which also has just one presenter."}


{"QuestionID": "q10", "original cue": "not", "PassageEditID": 0, "original passage": "For example, assume that Gifre has the choice between two alternatives, eating a cookie or not eating anything. Having eaten the first cookie, Gifre could stop eating cookies, which is the best alternative. But after having tasted one cookie, Gifre would freely decide to continue eating cookies until the whole bag is finished, which would result in a terrible stomach ache and would be the worst alternative. Not eating any cookies at all, on the other hand, would be the second-best alternative. Now the question is: should Gifre eat the first cookie or not? Actualists are only concerned with the actual consequences. According to them, Gifre should not eat any cookies at all since it is better than the alternative leading to a stomach ache. Possibilists, however, contend that the best possible course of action involves eating the first cookie and this is therefore what Gifre should do.", "SampleID": 1429, "label": "NO", "original sentence": "According to them, Gifre should not eat any cookies at all since it is better than the alternative leading to a stomach ache.", "sentence2": "Would Gifre have a stomach ache if he followed their advice?", "PassageID": 121, "sentence1": "For example, assume that Gifre has the choice between two alternatives, eating a cookie or not eating anything. Having eaten the first cookie, Gifre could stop eating cookies, which is the best alternative. But after having tasted one cookie, Gifre would freely decide to continue eating cookies until the whole bag is finished, which would result in a terrible stomach ache and would be the worst alternative. Not eating any cookies at all, on the other hand, would be the second-best alternative. Now the question is: should Gifre eat the first cookie or not? Actualists are only concerned with the actual consequences. According to them, Gifre should not eat any cookies at all since it is better than the alternative leading to a stomach ache. Possibilists, however, contend that the best possible course of action involves eating the first cookie and this is therefore what Gifre should do."}

{"QuestionID": "q10", "original cue": "not", "PassageEditID": 1, "original passage": "For example, assume that Gifre has the choice between two alternatives, eating a cookie or not eating anything. Having eaten the first cookie, Gifre could stop eating cookies, which is the best alternative. But after having tasted one cookie, Gifre would freely decide to continue eating cookies until the whole bag is finished, which would result in a terrible stomach ache and would be the worst alternative. Not eating any cookies at all, on the other hand, would be the second-best alternative. Now the question is: should Gifre eat the first cookie or not? Actualists are only concerned with the actual consequences. According to them, Gifre should not eat any cookies at all since it is better than the alternative leading to a stomach ache. Possibilists, however, contend that the best possible course of action involves eating the first cookie and this is therefore what Gifre should do.", "SampleID": 1432, "label": "NO", "**original sentence**": "**According to them, Gifre should not eat any cookies at all since it is better than the alternative leading to a stomach ache.**", "sentence2": "Would Gifre have a stomach ache if he followed their advice?", "PassageID": 121, "**sentence1**": "For example, assume that Gifre has the choice between two alternatives, eating a cookie or not eating anything. Having eaten the first cookie, Gifre could stop eating cookies, which is the best alternative. But after having tasted one cookie, Gifre would freely decide to continue eating cookies until the whole bag is finished, which would result in a terrible stomach ache and would be the worst alternative. Not eating any cookies at all, on the other hand, would be the second-best alternative. Now the question is: should Gifre eat the first cookie or not? Actualists are only concerned with the actual consequences. **According to them, Gifre should avoid eating any cookies at all since it is better than the alternative leading to a stomach ache.** Possibilists, however, contend that the best possible course of action involves eating the first cookie and this is therefore what Gifre should do."}
这种不太适合改造为Mohammed那篇文章里的NSPP任务，原因其不是两句coherent的句子，而是一个句子。不太好通过一个句子来预测下一个句子，因为没有下一个句子。

{"QuestionID": "q10", "original cue": "not", "PassageEditID": 2, "original passage": "For example, assume that Gifre has the choice between two alternatives, eating a cookie or not eating anything. Having eaten the first cookie, Gifre could stop eating cookies, which is the best alternative. But after having tasted one cookie, Gifre would freely decide to continue eating cookies until the whole bag is finished, which would result in a terrible stomach ache and would be the worst alternative. Not eating any cookies at all, on the other hand, would be the second-best alternative. Now the question is: should Gifre eat the first cookie or not? Actualists are only concerned with the actual consequences. According to them, Gifre should not eat any cookies at all since it is better than the alternative leading to a stomach ache. Possibilists, however, contend that the best possible course of action involves eating the first cookie and this is therefore what Gifre should do.", "SampleID": 1435, "label": "NO", "original sentence": "According to them, Gifre should not eat any cookies at all since it is better than the alternative leading to a stomach ache.", "sentence2": "Would Gifre have a stomach ache if he followed their advice?", "PassageID": 121, "sentence1": "For example, assume that Gifre has the choice between two alternatives, eating a cookie or not eating anything. Having eaten the first cookie, Gifre could stop eating cookies, which is the best alternative. But after having tasted one cookie, Gifre would freely decide to continue eating cookies until the whole bag is finished, which would result in a terrible stomach ache and would be the worst alternative. Not eating any cookies at all, on the other hand, would be the second-best alternative. Now the question is: should Gifre eat the first cookie or not? Actualists are only concerned with the actual consequences. According to them, Gifre should not buy any cookies at all since it is better than the alternative leading to a stomach ache. Possibilists, however, contend that the best possible course of action involves eating the first cookie and this is therefore what Gifre should do."}

{"QuestionID": "q10", "original cue": "not", "**PassageEditID**": 3, "**original passage**": "For example, assume that Gifre has the choice between two alternatives, eating a cookie or not eating anything. Having eaten the first cookie, Gifre could stop eating cookies, which is the best alternative. But after having tasted one cookie, Gifre would freely decide to continue eating cookies until the whole bag is finished, which would result in a terrible stomach ache and would be the worst alternative. Not eating any cookies at all, on the other hand, would be the second-best alternative. Now the question is: should Gifre eat the first cookie or not? Actualists are only concerned with the actual consequences. According to them, Gifre should not eat any cookies at all since it is better than the alternative leading to a stomach ache. Possibilists, however, contend that the best possible course of action involves eating the first cookie and this is therefore what Gifre should do.", "SampleID": 1438, "label": "NO", "**original sentence**": "**According to them, Gifre should ==not== eat any cookies at all since it is better than the alternative leading to a stomach ache.**", "**sentence2**": "Would Gifre have a stomach ache if he followed their advice?", "PassageID": 121, "**sentence1**": "For example, assume that Gifre has the choice between two alternatives, eating a cookie or not eating anything. Having eaten the first cookie, Gifre could stop eating cookies, which is the best alternative. But after having tasted one cookie, Gifre would freely decide to continue eating cookies until the whole bag is finished, which would result in a terrible stomach ache and would be the worst alternative. Not eating any cookies at all, on the other hand, would be the second-best alternative. Now the question is: should Gifre eat the first cookie or not? Actualists are only concerned with the actual consequences. **According to them, Gifre should eat many cookies at once, since it is better than the alternative leading to a stomach ache.** Possibilists, however, contend that the best possible course of action involves eating the first cookie and this is therefore what Gifre should do."}


我关注的是original sentence和sentence1 以及 PassageEditID 这3部分。

可以得到一个原始包含negation的句子，且是有coherent的sentence，并不是isolated的句子。

以及，其3个改写。



# AFIN

 The first work on affirmative interpretations was by Sarabi et al. (2019). Hossain et al. (2022b) present AFIN, a corpus of ≈3,000 sentences with negations and their affirmative interpretations.
 Sarabi等人（2019）首次开展了肯定解释的研究。Hossain等人（2022b）提出了AFIN，这是一个包含≈3,000个句子的语料库，这些句子包含否定句及其肯定解释。 These two previous works are limited to generating affirmative interpretations from negations; they do not provide extrinsic evaluations. 这两项工作仅限于从否定中生成肯定解释，它们不提供外部评价。

每对句子包括：
- 一个包含**否定**的句子；
- 该句子的**肯定解释**（即语义等价但不含否定词的句子）；
用途：主要用于研究 **如何从否定句生成肯定解释（affirmative interpretation generation）**。

|局限类型|说明|
|---|---|
|**规模限制**|数据量较小，仅约 3,000 条，限制了在大模型或大任务中训练和泛化能力的研究。|
|**任务限制**|只用于“从否定句生成肯定句”这一**单一生成任务**；**缺乏外部评估（extrinsic evaluation）**，也就是没有验证这些句对对其他下游任务（如 QA、NLI、情感分析等）的帮助程度。|

==AFIN 这篇文章就是提出了一个数据集，但并没有用它去训练模型并在如 CondaQA 这样的 benchmark 上评估它是否真的能增强模型对 negation 的理解。==

AFIN是人工构造的数据集。 AFIN (Hossain et al., 2022b), a **manually** curated corpus that is publicly available. AFIN contains 3,001 sentences with negation and their affirmative interpretations. 作者尝试过用T5等模型自动化构建，但是效果不好，依旧存在挑战性。
 
- AFIN 的目标是**构建一个资源**，用于研究“如何把否定句转成肯定句”；
- 他们的重点在于：提出任务； 收集数据（人工构建，质量较高）；用 T5 等模型进行句子生成；用自动和人工指标评估生成句是否合理；

但他们**没有进一步验证**这个"资源" 是否能：**改善模型对“否定”的理解能力** —— 比如在 CondaQA、MNLI、SST-2 等 benchmark 上提高表现。

此外，  AFIN only considers **verbal negations** (i.e., the negation cues always modify a verb). Here are some examples: 
- It was not formed by a natural process. 
  It was formed by an artificial process. 
- An extinct volcano is one that has not erupted in recent history.
  An extinct volcano erupted in the past
这个是后来实验室的2022年的工作[[2022-Leveraging Affirmative Interpretations from Negation Improves Natural Language Understanding]] 改进的地方。这篇文章提出了Large-AFIN数据集。

### 什么是 “extrinsic evaluation”？（外部评估）

**Extrinsic evaluation** 是指：

> **评估一个资源（如数据集、模型、组件）对某个**实际下游任务**的实际影响。**

这是与 **intrinsic evaluation（内部评估）** 相对而言的：

|类型|说明|示例|
|---|---|---|
|**Intrinsic evaluation**|直接评估资源自身的质量|用 BLEU 或 METEOR 衡量句子生成质量、人工打分|
|**Extrinsic evaluation**|将资源用于某个实际任务中，看其**是否提升了模型效果**|将肯定解释用于 QA、NLI、情感分析，看最终准确率是否上升|

在本文语境中，extrinsic evaluation 意味着：

- 不仅去看模型是否能**正确生成肯定解释**，
- 更要看这些解释是否**提升了模型在自然语言理解任务中的表现**，比如：
    - NLI（自然语言推理）
    - CondaQA（问答）
    - 情感分析（Sentiment Classification）

AFIN 数据集的局限就在于：
> 它只做了 intrinsic evaluation（如人工检查生成句的合理性），**没有做 extrinsic evaluation**，即没有证明其数据能提升下游任务的效果。

---

### 二、在 AFIN/生成任务中的输入与输出是什么？

AFIN 支持的任务是：  
**给定一个包含否定的句子，生成其肯定解释。**

|项目|内容|
|---|---|
|**输入（Input）**|一个否定句，例如：  <br>“It was not formed by a natural process.”|
|**输出（Output）**|一个不包含否定、但语义等价的肯定句：  <br>“It was formed by an artificial process.”|

这可以看作是一个 **文本到文本的生成任务（Text-to-Text Generation）**，类似于机器翻译，只不过是“否定句 → 肯定句”。

在模型层面，通常使用像 **T5** 这样的 encoder-decoder 结构来完成这一任务：
`T5 Input:   "affirmative interpretation: It was not formed by a natural process." T5 Output:  "It was formed by an artificial process."`


# Large-AFIN

[[2022-Leveraging Affirmative Interpretations from Negation Improves Natural Language Understanding]]

实验室2022年的工作。

该数据支持并推动了
### 三个自然语言理解任务：

#### 1.肯定解释生成（Affirmative Interpretation Generation）

- 给定：一个否定句（如 "It is not a bad movie."）
    
- 目标：生成语义等价、但无否定词的肯定句（如 "It is a good movie."）
    

模型示例：使用 T5 进行文本生成。

---

#### 2. 自然语言推理（Natural Language Inference, NLI）

- 给定：一对句子，分别是：
    
    - 含否定的句子（例如：_She did not win the race._）
        
    - 肯定解释（例如：_She lost the race._）
        
- 构造出两组 premise-hypothesis：
    
    - P: 否定句 → H: 肯定句（标为 entailment）
        
    - P: 肯定句 → H: 否定句（标为 entailment）
        

总计生成了约 30 万条新的 entailment 样本，可用于训练更鲁棒的 NLI 模型。

---

#### 3. 情感分析（Sentiment Analysis）

- 给定：一句话（如含否定："It’s not bad"）
    
- 步骤：
    
    1. 用否定检测器判断句子是否包含否定；
        
    2. 若包含，调用预训练的生成器生成肯定解释（如 "It is good"）；
        
    3. 将原句和肯定解释一起输入到分类器（如 RoBERTa），通过 [SEP] 分隔；
        
- 目标：预测情感极性（正面 / 负面）
    

这种做法显著提升了模型在含否定样本上的表现。


==创建了这个数据集，并拓展到3个下游任务上，来评估这个数据集能否对这些任务带来一些性能的提升。==

{"sentence": "With the exception of William H. Seward , Chase was considered the country 's most prominent republicans , and he had done more for the resistance of slavery than any other republicans , but failed to secure the nomination .", "affirmative_interpretation": "With the exception of William H. Seward , Chase was the most prominent Republican in the country and had done more against slavery than any other Republican ."}


这个句子里有两种否定，一个是with the exception of, 一个是后面的failed to

这个数据集确实里面有些样本有问题

 We discovered that 78% are correct, where correct means that the affirmative interpretation satisfies the definition (i.e., no negation and semantically equivalent to the sentence with negation). 我们发现78%是正确的，其中正确意味着肯定的解释满足定义（即，没有否定和语义上等同于带有否定的句子）。