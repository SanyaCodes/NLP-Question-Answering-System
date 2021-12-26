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

## MY APPROACH : 

### 1. Data Pre-Processing :
The SciQ dataset contains .json files that are processed for obtaining ques- tion and answer in a required format. Later stages of the pipeline require that each query file must contain questions along with their respective topic entities and correct answers. This topic entity must also be present in the list of entities generated wile extracting the KB tuples from corpus. Thus SpaCy library is used for noun parsing to obtain noun entities in each question, and then, using fuzzy match and WordNet, we select those entities that are present in the KB entity list.

### 2. SpanBERT for Coreference Resolution :
The model is supplied with corpuses that contain relevant information for solving solving the SciQ question dataset. The major challenge with using these corpuses is that they contain numerous occurrences of pronouns referring to objects and subjects mentioned previously in the passage. This indicates that not only is the model required to find the answer spread over two or three sentences but it is also expected to understand the links connecting these individual sentences, i.e. the pronouns. Therefore, Coreference Resolution is applied as an indispensable step of the pipeline. By employing SpanBERT, an enhanced version of BERT(Bidirectional Encoder Repre- sentations from Transformers), which provides improved representation and predictions of spans of textual data, the system component is able to replace all pronouns with their respective noun.

### 3. IMoJIE for Information Extraction :
Various models for Information extraction have benchmarked their results on domain specific datsets as well as open domain sentences collected from Wikipedia and other sources. Each model reports its shortcomings like redundancy, wrong extractions, non-uniformity, etc. We compare the scores achieved by each model and choose the model based on its correctness of deriving one or more tuple(s) from a given sentence. IMoJIE is chosen for this task and fed with the coreferenced paragraphs to extract three-tuples of the form <entity1, relation, entity2>. 

### 4. LibKGE for Knowledge Graph Embedding :
The next step requires converting the obtained Knowledge Graph into embeddings for which an open-source PyTorch Library - LibKGE is used. Word or vector embeddings is a common sub-task performed in NLP and IR. It involves conversion of words into low-dimensional forms i.e. into vectors. Hence, the words which are similar in meaning are located closer to each other in the vector space. The LibKGE library includes common KGE algorithms like RESCAL, TransE, ComplEx and RotatE. In this study, the ComplEx model is employed owing to its extremely high compatibility with many datasets including SciQ. It can handle a large variety of symmetric and antisymmetric relations and outperforms many KGE models with a fairly simple approach towards the task. Since this method of using embeddings is not only able to overcome several restrictions posed by multi-hop QA, but also eliminates the problem of knowledge graph sparsity, we translate the obtained knowledge graph into its embeddings.

### 5. EmbedKGQA for Answer Prediction :
After converting KG into embeddings, the EmbedKGQA model is used to convert the question and its topic entity into embeddings. It uses RoBERTa as the question embedding module. After obtaining the question, topic entity and answer, it uses the ComplEx scoring function to train the model and predicts direct links between the question and answer entity. In this way, the restriction posed by multi-hop questions are resolved and the model gets trained in such a way that, provided a question and its topic entity, it gives the predicted answer directly in the form of an embedding.

### 6. Answer Selection Module :
The last step of the pipeline is based on selecting an answer from a list of given choices. For this, an answer selection module is introduced. After predicting the answer embedding by EmbedKGQA, all given answer choices are also converted to embedding form by employing the RoBERTa model. Euclidean distance is calculated for each set of predicted answer and corresponding options. The lesser the distance, the closer is the option, hence indicating their similarity. Therefore, depending upon their closeness, the option with least euclidean distance is chosen as the final answer by the model. Since grammar and vocabulary play a major role in QA tasks, there are instances where predicted answer is close to the correct answer but differs lexically. Therefore, using WordNet, four synonyms are assigned to each answer choice. Once all the original choices and their respective synonyms are converted into embeddings, their closeness to the predicted answer embed- ding helps in choosing the option to which the closest embedding belongs. This ensures the model can detect the correct answer despite the lexical differences.
