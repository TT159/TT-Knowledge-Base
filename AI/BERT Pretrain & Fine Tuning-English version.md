
# Intuitive Understanding

**Pretraining** is like a closed-book exam—it teaches the model a broad range of **general** knowledge but lacks specialization.

**Fine-tuning**, on the other hand, is still a closed-book exam, but it’s like attending a targeted prep class beforehand—supplementing knowledge specific to a certain domain.

- **Mastering domain-specific knowledge**: Deep understanding of domain-specific terminology.
    
- **Improved contextual sensitivity**: Better at capturing and interpreting differences and intentions in professional contexts, where the meaning of some terms may vary across domains.
    
- **Integration of expert knowledge**.
    

**RAG (Retrieval-Augmented Generation)** gives the model access to an external knowledge base, turning the process into an **open-book exam**. When answering questions, the model can look things up.

- **On-demand retrieval** of information.
    
- **High scalability**: No need to change or retrain the model itself.
    
- **Meets dynamic needs**.
    

However, it also has drawbacks:

- **Highly dependent on the quality of retrieved content**: Previously, the model relied solely on its own parameters. Now it has to integrate retrieved content, without knowing whether the content is truly relevant. If the knowledge base lacks the necessary information, the retrieved content may be incorrect or unrelated, and the model’s answers will suffer.
    
- **Retrieval requires extra computation**, which may be unsuitable for applications needing ultra-fast response times.
    

---

# Basic Fine-Tuning Workflow

1. Choose a model to fine-tune—start with a pretrained model.
    
2. Prepare a dataset specifically for fine-tuning.
    
3. Perform iterative training.
    
4. Regularly evaluate performance using a validation set.
    
5. Optimize training strategy, e.g., tuning hyperparameters or improving preprocessing.
    
6. Save the fine-tuned model.
    

- **Task-oriented**: Select a pretrained model that aligns well with your target task.
    
- **Model size**: Balance model performance and hardware resource constraints.
    
- **Domain relevance**: Prefer models pretrained on similar domains for higher fine-tuning efficiency and effectiveness.
    

**Data sources**:

- Hugging Face Dataset Hub
    
- Kaggle competition datasets
    
- Academic papers
    
- Easy datasets on GitHub
    

---

# BERT Pretraining & Fine-Tuning

**Bilibili video: "BERT shows you the miracle of pretraining and fine-tuning"**  
**YouTube link**: [https://www.youtube.com/watch?v=gh0hewYkjgo&t=1175s](https://www.youtube.com/watch?v=gh0hewYkjgo&t=1175s)

**BERT from scratch** requires massive resources. Pretraining BERT-base took **16 TPUs and 4 days**.

**Pretraining** BERT involves two main tasks:

- **Masked Language Modeling (MLM)**: The model learns to fill in missing parts of a sentence by understanding its structure.  
    e.g., "There - a - of beautiful flowers." (fill in the missing word)
    
- **Next Sentence Prediction (NSP)**: Determine if one sentence logically follows another.
    
    Examples:
    Sentence 1: I am going outside.   
    Sentence 2: I will be back after 6.   
    Next sentence? YES  
    Sentence 1: I am going outside.   
    Sentence 2: You know nothing about John.   
    Next sentence? NO`
    

These pretraining tasks help the model gain **general-purpose textual understanding**, especially **fill-in-the-blank** capabilities.

**Q**: Can the original pretrained BERT be directly used for QA tasks?  
**A**: Not very well. Because the original BERT was only trained on MLM and NSP, not QA. To perform well on specific tasks like QA, **fine-tuning** on datasets like **SQuAD (QA)**, **NER**, or **MNLI** is necessary.

BERT provides multiple **task heads**, each tailored for fine-tuning on different downstream tasks. A task head acts as an adapter layer.

![[BERT微调.png]]

Today, to evaluate a model, we typically use **multiple tasks**, not just one, to assess overall performance.

---

## GLUE (General Language Understanding Evaluation)

**GLUE** is a benchmark suite introduced in 2018 by researchers at **NYU** and **University of Washington**, designed to evaluate **general NLU (Natural Language Understanding)** abilities.

> Widely used for assessing models like BERT, RoBERTa, GPT - GLUE is a cornerstone of NLP model evaluation.

GLUE includes **9 diverse NLU tasks**, such as classification, sentence similarity, and inference:

|Task|Description|Type|
|---|---|---|
|**MNLI**|Classify sentence pairs as entailment, contradiction, or neutral|3-class|
|**QNLI**|Determine whether a sentence answers a question|Binary|
|**QQP**|Determine if two questions are equivalent|Binary|
|**SST-2**|Sentiment classification (positive/negative)|Binary|
|**CoLA**|Grammaticality judgment|Binary|
|**STS-B**|Semantic similarity scoring (0–5)|Regression|
|**MRPC**|Paraphrase identification|Binary|
|**RTE**|Textual entailment|Binary|
|**WNLI**|Coreference resolution (hard)|Binary (excluded from scoring)|

- Each task uses metrics like **accuracy**, **F1**, or **correlation** (Pearson/Spearman).
    
- **GLUE score** is a weighted average of task performance.
    

GLUE provides a **unified evaluation platform** to test language models’ generality across tasks.

- **SuperGLUE** (2019) is a harder version of GLUE, addressing saturation in GLUE scores.
    
- Includes complex reasoning and multi-hop questions to push model understanding further.
    

---

## SQuAD (Stanford QA Dataset)

**SQuAD (Stanford Question Answering Dataset)** is a well-known **Machine Reading Comprehension (MRC)** dataset created by Stanford University. It is used to train and evaluate natural language processing models on reading comprehension tasks.

- **Name**: SQuAD (Stanford Question Answering Dataset)
    
- **Task Type**: Question Answering (QA)
    
- **Data Source**: English Wikipedia articles
    
- **Published By**: Stanford University
    

The SQuAD dataset consists of multiple samples. Each sample includes the following fields:
{
  "title": "Apollo 11", // Title of a Wikipedia article
  "paragraphs": [
    {
      "context": "Apollo 11 was the spaceflight that first landed humans on the Moon. Commander Neil Armstrong and lunar module pilot Buzz Aldrin formed the American crew that landed the Apollo Lunar Module Eagle on July 20, 1969.", // The passage from which the model must find the answer
      "qas": [  // All questions related to this passage
        {
          "id": "q1", // Unique question ID for evaluation alignment
          "question": "Who was the commander of Apollo 11?", // The question the model needs to answer
          "answers": [  // List of acceptable answers
            {
              "text": "Neil Armstrong", // Correct answer
              "answer_start": 59 // Character index in the context where the answer begins
            }
          ],
          "is_impossible": false // Indicates whether the answer can be found in the context (introduced in SQuAD 2.0)
        }
      ]
    }
  ]
}

The field **`is_impossible`** is set to `false`, indicating that the question has an answer that can be found in the provided context.  
The **`answer_start`** value is `59`, meaning the answer `"Neil Armstrong"` begins at the 59th character of the context string.

### SQuAD 1.1

- **Released**: 2016
    
- **Content**: 536 Wikipedia articles, 100,000+ QA pairs
    
- **Feature**: Every question has an **exact answer**, always a **substring** of the context.
    

### SQuAD 2.0

- **Released**: 2018
    
- **New addition**: 50,000+ **unanswerable questions**, requiring the model to judge if the context contains an answer.
    
- **Goal**: Return "no answer" when appropriate.
    

This makes SQuAD a **text span extraction task**, where the answer is **explicitly present** in the passage (or not), rather than being freely generated.

==The key is to extract the correct answer span from the text.==


### Evaluation Metrics

- **Exact Match (EM)**: Proportion of predictions that exactly match the ground truth.
    
- **F1 Score**: Measures word overlap between prediction and ground truth.
    

SQuAD is **one of the most widely used QA datasets**, utilized by models like BERT, RoBERTa, ALBERT, etc.

---

## Fine-Tuning BERT for QA Tasks

1. Prepare the QA task and dataset.
    
2. Download the original BERT with a QA task head.
    
3. Load SQuAD features into the training pipeline.
    
4. Fine-tune BERT on the SQuAD dataset.
    
5. Use the fine-tuned model to perform inference and answer questions.