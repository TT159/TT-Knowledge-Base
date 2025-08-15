---
date: 2025-06-30
tags:
  - Negation
  - affirmative
---

2022年 emnlp paper - 出自实验室


> common single-tokens that are not lexicalized negations are the most frequent. These sentences with negations and their affirmative interpretations come from our collection (Section 3) and include errors. For example, the affirmative interpretation in the second to last example includes ends, a lexicalized negation, because our cue detector did not identify it.

 Lexicalized negations usually take the form of a noun (e.g., lack, dismissal) or verb (e.g., prevent, avoid)

 Affixal negations are surprisingly common (30.15%) and include both prefixes (e.g., untouched) and suffixes (useless).

 a few negations (2.58%) are multitoken (e.g., no longer, not at all)

“词汇化的否定”（**lexicalized negation**）和“非词汇化的否定”（**non-lexicalized negation**）的区别，本质上在于：

> ✅ 否定信息是否**直接内嵌在单词本身的含义中**，而不是通过语法结构（如 not、never）表达出来。

---

## 🧠 一句话理解：

| 类型                                         | 定义                   | 示例                                                                 | 否定在哪？  |
| ------------------------------------------ | -------------------- | ------------------------------------------------------------------ | ------ |
| **词汇化否定**  <br>(lexicalized negation)      | 单词本身就含有否定含义          | **nothing**, **never**, **nobody**, **without**, **lack**, **end** | 单词内部   |
| **非词汇化否定**  <br>(non-lexicalized negation) | 否定通过其他语法结构表达（通常是添加词） | **not**, **n't**, **fail to see**, **rarely wins**                 | 单词外部结构 |

---

## 🧪 举例对比：

|原句|是否词汇化|原因|
|---|---|---|
|She said **nothing**.|✅ 词汇化|“nothing” 本身就表示否定，无需额外标记|
|She **didn't** say **anything**.|❌ 非词汇化|否定通过 `didn't` 实现，`anything` 是中性词|
|He **never** smiled.|✅ 词汇化|“never” 是带否定语义的时间副词|
|He **rarely** smiled.|❌ 非词汇化|“rarely” 是低频副词，不是绝对否定，但表达否定倾向|
|She **lacked** the courage.|✅ 词汇化|“lack” 这个动词表达否定含义|
|She **didn't have** the courage.|❌ 非词汇化|用助动词和 not 表达否定|

---

## 🔍 为什么要区分？

在构造或分析否定语料（如 CondaQA、¬ATOMIC）时：

1. **词汇化否定更难检测**：它不像 `not` 那样容易分词定位。比如模型看到 "nothing"，不一定能明确它表达了否定含义。
    
2. **句法转换更复杂**：比如将“nothing”转换成肯定句时，不是简单删除，而是需要“something”或“a thing”等替换。
    
3. **影响否定识别与生成任务**：特别是在构造**肯定/否定句对（affirmative ↔ negative）**、或训练模型识别否定语义时，要精细地区分这两类。



 The corresponding affirmative interpretation is never just the original sentence with negation after removing the negation cue—doing so results in a sentence that is not semantically equivalent.