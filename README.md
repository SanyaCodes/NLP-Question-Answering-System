# Closed-Domain-Multiple-Choice-Question-Answering-System-for-Science-Questions

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
In the initial step of the pipeline, it targets every occurrence of pronouns in the corpus using the Coreference Resolution technique. By replacing all pronouns with the entity they refer to, we make each sentence in the corpus independent. Following this, the task is to construct or build a knowledge base in the form of knowledge graph tuples. Using Information Extraction technique, each sentence is converted into one or more possible tuples of the form <entity1, relation, entity2>. The pipeline makes the use of embeddings to reach to the correct answer and also counters the problem of knowledge graph sparsity. For this purpose the extracted tuples, questions and answer choice data are converted into their respective embeddings. Once the complete dataset has been embedded, direct links are predicted between the topic entity of questions and all entities present in the knowledge graph. The model predicts an answer based on the distance between entities and the predicted answer is thus obtained. The model compares the predicted answer with given answer choices choosing the option that is the closest through an answer selection module. 

### Pipeline :
<img src="https://github.com/Ssanyachetwani/Closed-Domain-Multiple-Choice-Question-Answering-System-for-Science-Questions/blob/main/rim/pipeline.png" alt="datab" width=500/>

### Knowledge Graph :
Knowledge Graphs are a simplified representation of a large amount of interconnected data in the form of a graph, with the nodes representing entities i.e. people, places, objects etc. and their connecting edges representing the relationship between two (neighbouring) entities.

<img src="https://github.com/Ssanyachetwani/Closed-Domain-Multiple-Choice-Question-Answering-System-for-Science-Questions/blob/main/rim/KG.png" alt="datab" width=500/>

### 1. Data Pre-Processing :
The SciQ dataset contains .json files that are processed for obtaining question and answer in a required format. Later stages of the pipeline require that each query file must contain questions along with their respective topic entities and correct answers. This topic entity must also be present in the list of entities generated wile extracting the KB tuples from corpus. Thus SpaCy library is used for noun parsing to obtain noun entities in each question, and then, using fuzzy match and WordNet, we select those entities that are present in the KB entity list.

### 2. SpanBERT for Coreference Resolution :
The model is supplied with corpuses that contain relevant information for solving solving the SciQ question dataset. The major challenge with using these corpuses is that they contain numerous occurrences of pronouns referring to objects and subjects mentioned previously in the passage. This indicates that not only is the model required to find the answer spread over two or three sentences but it is also expected to understand the links connecting these individual sentences, i.e. the pronouns. Therefore, Coreference Resolution is applied as an indispensable step of the pipeline. By employing SpanBERT, an enhanced version of BERT(Bidirectional Encoder Repre- sentations from Transformers), which provides improved representation and predictions of spans of textual data, the system component is able to replace all pronouns with their respective noun.

<img src="https://github.com/Ssanyachetwani/Closed-Domain-Multiple-Choice-Question-Answering-System-for-Science-Questions/blob/main/rim/ReferencePara.png" alt="datab" width=350/>
<img src="https://github.com/Ssanyachetwani/Closed-Domain-Multiple-Choice-Question-Answering-System-for-Science-Questions/blob/main/rim/spanbert.png" alt="datab" width=350/>

### 3. IMoJIE for Information Extraction :
Various models for Information extraction have benchmarked their results on domain specific datsets as well as open domain sentences collected from Wikipedia and other sources. Each model reports its shortcomings like redundancy, wrong extractions, non-uniformity, etc. We compare the scores achieved by each model and choose the model based on its correctness of deriving one or more tuple(s) from a given sentence. IMoJIE is chosen for this task and fed with the coreferenced paragraphs to extract three-tuples of the form <entity1, relation, entity2>. 

<img src="https://github.com/Ssanyachetwani/Closed-Domain-Multiple-Choice-Question-Answering-System-for-Science-Questions/blob/main/rim/IE.png" alt="datab" width=350/>

### 4. LibKGE for Knowledge Graph Embedding :
The next step requires converting the obtained Knowledge Graph into embeddings for which an open-source PyTorch Library - LibKGE is used. Word or vector embeddings is a common sub-task performed in NLP and IR. It involves conversion of words into low-dimensional forms i.e. into vectors. Hence, the words which are similar in meaning are located closer to each other in the vector space. The LibKGE library includes common KGE algorithms like RESCAL, TransE, ComplEx and RotatE. In this study, the ComplEx model is employed owing to its extremely high compatibility with many datasets including SciQ. It can handle a large variety of symmetric and antisymmetric relations and outperforms many KGE models with a fairly simple approach towards the task. Since this method of using embeddings is not only able to overcome several restrictions posed by multi-hop QA, but also eliminates the problem of knowledge graph sparsity, we translate the obtained knowledge graph into its embeddings.

### 5. EmbedKGQA for Answer Prediction :
After converting KG into embeddings, the EmbedKGQA model is used to convert the question and its topic entity into embeddings. It uses RoBERTa as the question embedding module. After obtaining the question, topic entity and answer, it uses the ComplEx scoring function to train the model and predicts direct links between the question and answer entity. In this way, the restriction posed by multi-hop questions are resolved and the model gets trained in such a way that, provided a question and its topic entity, it gives the predicted answer directly in the form of an embedding.

### 6. Answer Selection Module :
The last step of the pipeline is based on selecting an answer from a list of given choices. For this, an answer selection module is introduced. After predicting the answer embedding by EmbedKGQA, all given answer choices are also converted to embedding form by employing the RoBERTa model. Euclidean distance is calculated for each set of predicted answer and corresponding options. The lesser the distance, the closer is the option, hence indicating their similarity. Therefore, depending upon their closeness, the option with least euclidean distance is chosen as the final answer by the model. Since grammar and vocabulary play a major role in QA tasks, there are instances where predicted answer is close to the correct answer but differs lexically. Therefore, using WordNet, four synonyms are assigned to each answer choice. Once all the original choices and their respective synonyms are converted into embeddings, their closeness to the predicted answer embed- ding helps in choosing the option to which the closest embedding belongs. This ensures the model can detect the correct answer despite the lexical differences.

## Robustness against Knowledge Base Variation :
The curated knowledge base tuples of the Aristo Tuple KB Dataset are used in this work, during the experimentation phase, to gauge the correctness of the pipeline. Aristo Knowledge Base is a set of 294,000 highly precise tuples, that spread over a range of domains in the field of science and reports an accuracy of 77.4% in answering questions of the SciQ dataset.<br/> <br/>
Carrying out all the steps sequentially on the collected reference material M, the pipeline is able to obtain an accuracy of 23.6%, but interestingly, when the curated tuple KB is used the accuracy significantly rises to about 62%. Hence we can infer that pipeline can detect correct answers, provided an accurate underlying KB is fed to the model. We also observe that while building the knowledge base from reference set M, the information extractor generally tends to break the sentence into three major parts. So the entity1 and entity2 contain more than one word and this diverts the attention of the system to other words of the phrase, that are less important. As mentioned earlier, Information Extraction models also do not extract rule-based relations between entity1 and entity2. They are highly dependent on the grammar and vocabulary of the sentence, which creates a set of varied relations. 

## Results :
Each component of the pipeline playa a vital role, and its individual accuracy can dramatically affect the overall result of the pipeline the accuracy and precision of the KB tuples can affect the results significantly. Tuples obtained by Information Extraction (IE) methods prove to be inefficient especially when compared to curated, high-precision and domain specific KB, in this case, Aristo Tuple KB. The dependency of IE techniques on lexical aspects of a sentence prove to have a great impact on the results. With the tuple entities namely entity1 and entity2, being a span of the sentence, and the relations not following a certain schema, the uniformity of KB is greatly compromised. These issues are resolved when the Aristo KB is introduced since it has been built on rule-based relations and contains one or two word entities. Hence, there is a strong need for the knowledge base to incorporate these features, as the variation in relations and entities with each sentence, ultimately affects the training of subsequent components. 

| Pipeline | Accuracy over SciQ |  
| ------- | --- | 
| Pipeline with Information Extraction step | 23.6% |
| Pipeline with Curated KB Tuples | 62% |
