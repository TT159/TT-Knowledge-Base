---
date: 2025-06-09
tags:
  - NLP
  - Negation
---
# Main Content

==Goal is to evaluate models on their ability to process the contextual implications of negation.==

we present CONDAQA, a QA dataset contains 14,182 examples,  the **first** English reading comprehension dataset which requires reasoning about the implications of negated statements in paragraphs.

Focused on reasoning about negated statements in language.

The CondaQA benchmark includes minimal **pairs** aimed at determining whether models are sensitive to these differences.

pair: 
- origin - paraphrasing
- origin - changing scope
- origin - affirmation

==Notes:== Difference from Paraphrasing in Affirmative Terms article [[2024-Paraphrasing in Affirmative Terms Improves Negation Understanding]] 
CondaQA这篇文章的affirmation，是直接将否定句变为肯定句，将句意反转了。
	e.g., "I don't like this movie" -> "I like this movie"

Affirmative Paraphrasing,是用不含否定词的方式表述原句的意思，即保留了原句的否定含义。
	e.g., "I don't like this movie" -> "This movie is boring"


**Data Collection**
Hire crowd-worker
Extract passages from Wikipedia which contain negated phrases.
3 stages:
- stage 1: Edit original passage
- stage 2: construct questions 
- stage 3: construct answers

Three kinds of edits:
1) paraphrasing (same meaning)
2) changing the **scope** of negation
3) rewriting the passage to include an **affirmative** statement in place of the negated (Opposite meaning)
statement.
![[CondaQA 3 stages.png]]


**Evaluate models**
Baseline performance on CondaQA.

3 strategies：
 We evaluate models that we ==train== either on the entire CONDAQA training data or few examples, as well as zero-shot models. 

 The baseline models that we benchmark on CONDAQA are listed in Table 4. We categorize them based on whether they use (a) all of the training data we provide (**full finetuned**), (b) a small fraction of the available training data (**few-shot**), (c) no training data (**zero-shot**), and on (d) whether they measure dataset artifacts (controls).


**Evaluation metric**
1) accuracy
2) group consistency
	We consider two types of groups:
		   (a) Question-level consistency: each group is formed around a question and the answers to that question for the original Wikipedia passage, as well as the three edited passage instances (**ALL**).
		   (b) Edit-level consistency: each group is formed around a question, the answers to that question for the original Wikipedia passage, and **only one of the edited passages** (PARAPHRASE CONSISTENCY, SCOPE CONSISTENCY, and AFFIRMATIVE CONSISTENCY).


**Results:**

The best performing model is fully finetuned UNIFIEDQA-V2-3B with an accuracy of 73.26%.

There is a gap of ∼40% in consistency between humans and the best model.

Mainstream benchmarks should consider including consistency as a metric to more reliably measure progress on language understanding.

Few- and zero-shot baselines do not match fully finetuned models’ performance, but considerably improve over the **majority** baseline.

UNIFIEDQA-V2-LARGE has a negative correlation with question length.


**Insights**
After full fine-tuning on CondaQA, the performance of UNIFIEDQA-v2-3B is better than /  comparable to UNIFIEDQA-v2-11B.  ->  ==make small model as good as the bigger one.==

the consistency gap is still large, compared to humans.




## Good Words

plausible 
	(possible + reasonable)  看起来合理的，可信的，有道理的
implausible
	看起来不合理的，不可信的