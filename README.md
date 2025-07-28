# **SVO-triplet constrained, unsupervised QA synthesis pipeline with hybrid dual-scale retrieval for technical knowledge indexing**
## **Overview**
This system transforms unstructured technical documentation (product specs, expert 
articles, manuals) into a query-optimized knowledge base. By integrating linguistic 
structure extraction, constrained generation, and multi-strategy retrieval, it enables 
precise technical question answering while minimizing computational costs. 

## **Target application**
Ideal for knowledge bases, chat-bots, and product search.

## **Key Components**
### Semantic-aware chunking
- After coarse rule based chunking with overlap an LLM splits the larger texts into smaller coherent chunks, preserving technical context and logical flow.
- Benefit: Maintains explanatory relationships between concepts within chunks, avoiding arbitrary fragmentation.

### SVO triplet extraction
- Each sentence in a chunk is processed to extract Subject-Verb-Object (SVO) triplets (e.g., "The algorithm [S] optimizes [V] performance [O]").
- Design decision: Triplets enforce structured representation of core relationships, reducing ambiguity and halucination.

### SVO-constrained QA generation
- An LLM generates QA pairs using SVO triplets as constraints, with the full chunk as context.
- Key Innovations:
    - Cost efficiency: SVO direction allows smaller/cheaper LLMs (e.g., 7B-parameter models) without sacrificing precision.
    - Detail preservation: Questions target granular technical elements (e.g., "What does the algorithm optimize?") while retaining broader context.

### Validation & Filtering
- A “roberta” based model distilled for Extractive QA scores each generated pair:
    - Answer relevance to the question.
    - Contextual alignment with the source chunk.
- Low-scoring QAs are discarded via a threshold.
- Benefit: Ensures factual accuracy and eliminates hallucinated content.

### Redundancy reduction
- Question embeddings are clustered using Affinity Propagation to identify semantic redundancy.
- Design decision: Affinity Propagation adapts to unknown cluster counts, automatically retaining representative questions.

### Dual-scale Embedding
- Final QA pairs are embedded jointly (question + answer as one vector). for precise, detail-oriented queries.
- Whole chunks for broad-context retrieval.

### Hybrid embedding:
- Dense retrieval: Semantic matching via dense embedding
- Sparse retrieval: Term-attention based matching with IDF calculation for keyword relevant queries
- Late Interaction (ColBERT): Contextualized token-level relevance scoring

## **Benefits**
### Cost-effective scalability
- Indexing: Smaller LLMs handle SVO-triplet-constrained generation, reducing indexing costs vs. large-model approaches.
- Retrieval: Only uses embedding, vector-search and a re-ranking. No LLM or NER model.

### Precision-recall balance
- SVO anchoring captures atomic details, while chunk context supports conceptual queries.

### High signal-to-noise ratio
- Validation + clustering typically removes 30-40% of generated QAs, retaining only novel, high-value pairs.

### Optimized retrieval
- Joint QA embeddings enable direct matching of questions to answer-bearing content.
- Dual embeddings support both specific fact lookup and exploratory searches.
- Hybrid (dense+sparse) embedding increase retrieval precision

## **Requirements**
1. This notebook uses Cloudflare Workers AI for different models. It also runs some model locally that require some free memory (1-2GB)
2. Qdrant is used as a Vector DB for testing retrieval. A Qdrant cluster is required. Free managed cluster is available that is more than enough for testing.

**Important**: Text generation nodel needs to support structured output (JSON). Even if it supports sometimes it fails to generate a propper JSON output. Therefore the code is written that long running (multiple) generations can be resumed. e.g. question generation

*Huggingface inference (optional) is implemented but the 'json_schema' for output_format is limited and this notebook heavily relies on that feature.
It could still work with 'json_object' as output format but not tested. HF would be an option to test RAG results with a wide range of models.*

## **Important Notes**
- This notebook uses services. They should fit to free tiers but always verify pricing before providing API keys.
- JSON files are provided as data source. There is code to load them and skip long running task.
- The code has been made for demonstrational purposes. Validation, error handling, etc. omitted for better understanding. It is not made for production.

## **Use a virtual environment**
### Steps to create with conda:
1. Create myenv  
`conda create -n myenv`  
*or with python version*  
`conda create -n myenv python=3.12`
2. Activate  
`conda activate myenv`
3. Install ipykernel  
`conda install ipykernel`
4. Add venv to Jupyter with name myenv  
`python -m ipykernel install --user --name=myenv`

## **Installation**
Install packages from requirements.txt (recommended)  
`pip install -r requirements.txt`  
Or use this command  
`pip install haystack-ai dotenv huggingface_hub cloudflare ipywidgets spacy qdrant-client fastembed accelerate`  
Install claucy  
`pip install git+https://github.com/mmxgn/spacy-clausie.git`  
Download the model for spacy  
`python -m spacy download en_core_web_trf`  

## **API Access**
Set the following environment variables to use services  
Cloudflare workers AI  
`CLOUDFLARE_ACCOUNT_ID` - Cloudflare account ID  
`CLOUDFLARE_API_TOKEN` - Cloudflare API Token  
Qdrant  
`QDRANT_URL` - Qdrant cluster URL  
`QDRANT_API_KEY` - Qdrant API key  
Hugging Face (optional)  
`HF_TOKEN` - Hugging Face API Token   
