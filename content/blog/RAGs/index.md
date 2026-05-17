---
title: Introduction to Retrieval Augmented Generation
summary: TBA
date: 2026-05-17

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
#image:
#  caption: 'Image credit: [**Unsplash**](https://unsplash.com)'

cover:
  #image: "https://images.unsplash.com/photo-1557682250-33bd709cbe85?q=80&w=2560"
  position:
    x: 50
    y: 40
  overlay:
    enabled: true
    type: "gradient"
    opacity: 0.4
    gradient: "bottom"
  fade:
    enabled: true
    height: "80px"
  icon:
    name: "✨"

authors:
  - me
  - Ted

tags:
  - LLMs
  - 
  - 

content_meta:
  trending: true
---



{{< toc mobile_only=true is_open=true >}}


### What is RAG?

### When is RAG needed?

 
 #### Indexing Phase
 1. Ingest - Loading the docuemnts
 2. Chunk - Split the ddocuments into small pieces
 3. Embed - Transform the chunks into vectors suing a embedding model
 4. Store - Save the vectors in vector database
   
When there is change in the documents, RAG pipelines can re-index the new documents without retrainign teh models.

#### Query Phase
5. Query - Embed the user question using the same embedding model.
6. Retrieve - Find the most similar chunks via vector similarity search. (Eg. Cosine similarity, Euclidean distance)
7. Rerank - Score and reorder the retrieved chunks using a cross encoder.
8. Generate Response


#### Common Libraries used for Ingestion 

Libraries                                       Data Format
1. LangChain Document Loaders
2. Docling (IBM)
3. PyMuPDF
4. Tessaract OCR
5. Beautiful Soup
6. LlamaIndex Document Connectors
7. Unstructured.io
8. Scrapy
9. FireCrawl

The choice depends on type of document and its content, latency,accuracy and volume.


### Chunking

**What is chunking and why is it needed?** 

Chunking is as the name suggests, breaking down a large piece of text into smaller and more manageable pieces. Chunking is applicable to all modalities of data (e.g images, tables, audio etc.)

But why do LLMs need chunking? Ever tried to listen to someone speak for a long time on a information dense topic and you natually tend to grasp less and less (happens in every lecture halls). LLMs are no expection to this as well [5]. If you want you LLMs to have a precise answer then the information fed to them should be relevant and precise.

The context window of the LLMs are limited. Therefore, chunks should be small enough to fit into the context window while also being large enough to have meaningful information for the task.

The goal of chunking is not to chunk for chunking sake, our goal is to get our data in a format where it can be retrieved for later. It should help the following process of semantic similarity search easier and more meaningful.

Thus chunking helps to place the most informative data with least possible token in the context window so that the LLMs **donot hallucinate** and also saves **token usage**.

#### What about the input format to these chunkers?
The **text splitters** (or chunkers) donot work with raw strings. They work with documents which are objects that hold the text and also additional metadata which makes filtering and manipulation easy. To create document object from a raw text use 
`create_document([text]) # Input is a list of text strings`

#### Chunking Startegies

1. Fixed Size Chunking - Splitting the document into uniform fixed length segments. It is most useful in large amount of random data like the data from the Internet.
    - Character Splitting -Splitting the text into n-character sized chunks. 
    Sliding Window strategy can be used in tandem with other chunking strategies like fixed length chunking. We create sliding windows with some overlap in text. (The start of the chunk #2 is same as the end of the chunk #1). This overlap helps us connect the sentences and establish a relationship. But it is to be noted that the overlap shouldnt be large that it increases the number of chunks, that defeats the whole point.The important parameters are **Chunk Size** , **Chunk Overlap** and the **seperator**. Seperators are character sequence that you would like to split on. These sequences are not included in the chunks.

        Frameworks: 1. CharacterTextSplitter from LangChain
                    2.SentenceSplitter from LlamaIndex.text_splitter

        This strategy is not used mostly in practical applications but is a good starting point.

2. Recursive Chunking - Iteratively splits the text using hierarchical seperators such as paragraphs, sentences, words until the chunk size limit is reached. This makes use of the hierarchichal structure in the document. So to leverage this completely, we must have a structured document and not a huge blob of text.
   
   The chunking follows the foloowing hierarchical seperators:
                    1. ""\n\n" - Double new lines
                    2. "\n" - New line
                    3. " " -Spaces
                    4. "" - Characters

It starts with splitting the input text on double new lines and if the chunk size is strill big then it looks for new line within the section and continues splitting until the chunk doesnot exceed the chunk size. Thus it naturally maintains the structure of the document. You can mention chunk overlap also. When experimenting on finding the best chunking strategy, this is often a good starting point.

3. Document Specific Chunking - Chunking only text files are boring! What about chunking images, code or a PDF?

For different formats of text like markdown, code (Python, Javascript) we can follow similar principles as recursive text splitter but with format specific seperators.

For different formats like JSON,markdown, code, HTML refer to LangChain specific splitters [here](https://docs.langchain.com/oss/python/integrations/splitters). Code based splitters list can be found here [here](https://github.com/langchain-ai/langchain/blob/9ef2feb6747f5a69d186bd623b569ad722829a5e/libs/langchain/langchain/text_splitter.py#L1069)

For e.g.
MarkdownTextSplitter from langchain has 
    \n#{1,6}    - Split by new lines followed by a header (H1 through H6)
    ```\n       - Code blocks
    \n------+\n - Horizontal lines
    \n\n        - Double new line 
    etc.. as seperators

PythonCodeTextSpiltter has \nclass - Classes, \ndef-Functions, \n\tdef- Intended Function etc.. as seperators.

Even though we have clear splitters we need to tune the chunk size inorder to get clean splits where the function or a class is not chunked into multiple parts. We can also deal with various other data formats apart from text like images and tables with [Multimodal RAG] ().

4. Semantic Chunking - Just in the previous section we saw how the chunk size is an important parameter to create meaningful chunks. Chunk size is also a global constant for the text throughout the data which is not very optimal. There has to be a better way *to create boundaries or breakpoints* that is based on the semantic meaning of the content. The goal is to make every chunk cover exactly only one idea.

The idea is based on embedding every single sentence and comparing with the embeddings of following sentences and find where the embedding distance was large and create a breakpoint at that position. We can further improve this idea by using sentence windows where we comnine 3 to 4 overlapping sentences and the position where the distance exceeds the threshold, a chunk is cutoff and a new one begins. Refer to the code in RAG Chatbot notebook

Based on this idea LLamaIndex created the **SemanticSplitterNodeparser** implementing it. The number of sentences considered in the window is given as **buffer size**, **breakpoint percentile threshold** gives the threshold and we  can also specify the **embedding model**.

This definetly increases the indexing latency and for large and changing documents this cost might be too large. 
Second problem being, the threshold sensitivity, now the annoying hyperparameter has just changes its name and form from chunk size to threshold percentile but the even here similar problems exists. The right value depends on your domain, your embedding model, and the density of your documents and you cannot know it without running evaluations.

Thus semantic chunking is worth trying when the input document is very unstructured and other strategies are underperforming.

5. Agentic Chunking - What if we ask the LLM itself to take care of the chunking? The initial chunks are created based on the concept of propositions (atomic, independent verifiable statement or a fact) [paper](https://arxiv.org/pdf/2312.06648) from the document. The generated propositions are feed into a LLM based agent which decides whether the proposition should be added to the current chunk or new chunk should be created. 

### What about PDFs with tables and images? Enter MultiModal RAG!

1. MultiModal Embedding
2. Text Only
3. Combination of Text and Images - This basically decouples the raw information (raw image, tables or even raw text) from the source and the embeddings (by just storing the text summaries of it for easier retrieval). But while sending it to the LLM send it back in the form of  raw information


#### Ingesting Images and Tables with Unstructured.io

Ingestion and Storing Pipeline

Input Document ----> Extract Text | Tables | Images -----> Chunking Text -------> Create a text summary of tables , images and the text ----> Embed the tables, text and images summary ------> Store the embeddings in the Vector Store -----> Store the original documents in a seperate Document store and link the embeddings and the original documents through a document id.

Retrieval Pipeline
User Query -----> Find relevant text embeddings from the vector store ----> Link it to the relevant raw document (text chunk ,image or table) using the doc id ----> Send it as input to the LLM for further processing.


Ingestion & Storing Pipeline

1. Partitioning with Unstructured.io

2. What are base64 encodings? 


1. Splitting into meaningful topic based sections. This involves embedding the chunks into vectors and finding the similar chunks. The similar chunks are then grouped into a single chunk until we reach the maximum chunk size (size of each single chunk) or maximum number of chunks. 



#### Which framework to use?
1. LangChain
2. LlamaIndex


### Embedding Options
It is important to choose a good model, which is trained on language understanding and not in next token prediction.
1. Open AI Embeddings
2. Sentence Transformers
3. Google Embeddings
4. Mistral Embeddings
5. HuggingFace Embeddings
6. Instructor Embeddings
7. Fast Embed

Important things to keep in mind while selecting an embedding model is:
1. **Efficiency Vs Accuracy** - Shallower models (Word2Vec, GloVe) are faster but less context aware compared to deep transformers (BERT, OpenAI)
2. **Context & Nuance** - Transformer models capture deeper contextual meaning enabling better performance on complex queries. 
3. **Dimensionality & Storage** - Higher dimensions offer more expressing power but increase the computational and storage costs.
 When there is little diversity in the text source, then we need a larger dimension to express the differences in the vector space. But when the text is diverse, the inherebnt diversity let us choosse a small dimension in vector space.

#### Model & Typical Usecases

------------------------
Model|Key UseCases| Architecture|Dimension
---|---|---|---
Word2Vec| 1. Semantic Word Relationships <br> 2. Simple NLP tasks | CBOW/Skipgram|50-300
GloVe| 1. Word Similarity Tasks <br> 2. Analogy Tasks| Matric Factorization| 50-300
BERT| 1. Sentence Embeddings <br> 2. Contextual Understanding | Transformer (Encoder Only) | 768 -1024
Open AI Test Embeddings | 1.Retrieval <br> 2. RAG <br> 3.Semantic Search| Transformer|Up to 3072

------------------------

### Similarity Search (Retrieval) Options
1. Cosine Similarity
2. Dot product
3. Euclidean Distance (L2)
4. Manhattan Distance
5. Approximate Nearest Neighbour
6. Product Quantization
7. Maximum Inner Product Search
8. Inverted File Index


### Vector Indexing

#### Vector Search Engines
1. FAISS (Facebook AI Similarity Search)
2. Chroma DB
3. Pinecone
4. Weaviate
5. Elastic Vector Search
6. Redis Vector Search
7. Qdrant

Vector Indexing uses the any one of the above similarity seaching algorithm to retriev the similar chunks from the database.

**LangChain and LLamaIndex are popular frameworks which donot provide their own database for Vector indexing or AI models for embedding. They provide the **glue** to connect the different specialised components into one single, coherent RAG system.**

### TF-IDF (Sparse Search)
Explain with a example


### References
[1] https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/a4570f3c4883eb9b835b0ee18990e62298f518ef/tutorials/LevelsOfTextSplitting/5_Levels_Of_Text_Splitting.ipynb
[2] [Vizuara Context Engineering Bootcamp - Lecture 3 ]
[3] [https://towardsdatascience.com/your-chunks-failed-your-rag-in-production/]
[5]https://arxiv.org/abs/2307.03172