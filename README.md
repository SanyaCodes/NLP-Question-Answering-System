# NLP-Question-Answering-System

## Task :

This is a multi-hop questions answering task which requires reasoning over entire paragraph to find the correct answer. In this study I have proposed a novel pipeline for solving multiple choice question which leverages various NLP techniques such as coreference resolution, information retrieval in the form of knowledge graphs, knowledge graph embeddings and a unique answer selection modeule which takes into account the lexical differences between the predicted answer and the given answer choices.

## Dataset :

The study is conducted on the open source SciQ dataset - https://allenai.org/data/sciq
The SciQ dataset contains 13,679 crowdsourced science exam questions about Physics, Chemistry and Biology, among others. The questions are in multiple-choice format with 4 answer options each. For the majority of the questions, an additional paragraph with supporting evidence for the correct answer is provided.

Structure :
<li> Question: The question. </li>
<li> Distractors: The incorrect answer options. </li> 
<li> Correct Answer: The correct answer option. </li> 
<li> Support: A sentence that supports the correct answer option. </li> 
<br/>
<img src="https://github.com/Ssanyachetwani/Closed-Domain-Multiple-Choice-Question-Answering-System-for-Science-Questions/blob/main/rim/SciQ.png" alt="datab" width=350/>

## MY APPROACH : 

**_NOT INCLUDED IN THIS REPOSITORY DUE TO PRIVACY REASONS (UNTIL PUBLISHED)

| Pipeline | Accuracy over SciQ |  
| ------- | --- | 
| Pipeline | 23.6% |
| Modified Pipeline | 62% |
