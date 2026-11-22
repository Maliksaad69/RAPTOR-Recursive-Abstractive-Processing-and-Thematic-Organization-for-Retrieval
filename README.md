# Corrective RAG

### Retrieval-Augmented Generation with Dynamic Retrieval Correction

Corrective RAG is an advanced **Retrieval-Augmented Generation (RAG)** pipeline that dynamically evaluates the quality of retrieved information and decides whether to trust, refine, or correct it.

Unlike a traditional RAG pipeline that always relies on retrieved documents, Corrective RAG introduces a **retrieval evaluation and correction step**. When the local knowledge base does not provide sufficiently relevant information, the system can rewrite the query, perform a web search, refine the retrieved knowledge, and combine information from multiple sources before generating the final response.

---

## 📌 Overview

A conventional RAG pipeline typically follows:

```text
User Query
    ↓
Vector Search
    ↓
Retrieve Documents
    ↓
LLM
    ↓
Answer
```

Corrective RAG adds an evaluation and correction layer:

```text
                     ┌─────────────────────┐
                     │     User Query      │
                     └──────────┬──────────┘
                                ↓
                     ┌─────────────────────┐
                     │   FAISS Retrieval   │
                     └──────────┬──────────┘
                                ↓
                     ┌─────────────────────┐
                     │ Retrieval Evaluation│
                     └──────────┬──────────┘
                                ↓
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
        High Relevance     Ambiguous        Low Relevance
              ↓                 ↓                 ↓
        Use Retrieved     Combine Local      Web Search
          Knowledge       + Web Results      + Query Rewrite
              │                 │                 │
              └─────────────────┼─────────────────┘
                                ↓
                     ┌─────────────────────┐
                     │ Knowledge Refinement │
                     └──────────┬──────────┘
                                ↓
                     ┌─────────────────────┐
                     │   Response LLM      │
                     └──────────┬──────────┘
                                ↓
                     ┌─────────────────────┐
                     │ Final Answer +      │
                     │ Source Information  │
                     └─────────────────────┘
```

The goal is to reduce the impact of **irrelevant, incomplete, or outdated retrieved documents**.

---

## 🎯 Motivation

Traditional RAG systems can fail when the vector database retrieves documents that are:

- Irrelevant to the user's question
- Only partially relevant
- Outdated
- Missing important information
- Insufficient to answer the query

Corrective RAG addresses this by introducing **dynamic retrieval evaluation**.

Instead of blindly passing retrieved documents to the language model, the system first asks:

> **"Are these retrieved documents actually useful for answering this query?"**

Based on the evaluation, the pipeline dynamically selects the next retrieval strategy.

---

## ✨ Key Features

- 🔎 **Vector-based retrieval** using FAISS
- 🧠 **Retrieval relevance evaluation**
- 🔄 **Dynamic retrieval correction**
- 🌐 **Web search fallback**
- ✍️ **Web-search query rewriting**
- 🧹 **Knowledge refinement**
- 🔗 **Multi-source knowledge combination**
- 🤖 **LLM-based response generation**
- 📚 **Source-aware responses**
- ⚡ **Efficient similarity search**

---

# 🏗️ Architecture

The pipeline consists of several major components.

## 1. User Query

The process begins with a natural-language question from the user.

```text
"What are the latest developments in ...?"
```

The query becomes the input for the retrieval pipeline.

---

## 2. FAISS Retrieval

The system searches a pre-existing knowledge base using **FAISS (Facebook AI Similarity Search)**.

FAISS performs vector similarity search to identify documents that are semantically related to the user's query.

By default, the pipeline retrieves the **top 3 documents**.

```text
User Query
    ↓
Embedding
    ↓
FAISS Similarity Search
    ↓
Top-K Documents
```

---

## 3. Retrieval Evaluation

The retrieved documents are evaluated for their relevance to the original query.

The evaluator produces a relevance score that determines how the pipeline should proceed.

The current decision logic is:

| Relevance Score | Decision |
|---|---|
| `> 0.7` | High relevance → use retrieved knowledge |
| `0.3 – 0.7` | Ambiguous → combine local + web knowledge |
| `< 0.3` | Low relevance → perform corrective web search |

This makes the retrieval process adaptive rather than fixed.

---

# 🔄 Corrective Retrieval Process

## Case 1 — High Relevance

If the retrieved information has a relevance score greater than `0.7`:

```text
Query
 ↓
FAISS Retrieval
 ↓
Relevance > 0.7
 ↓
Use Retrieved Knowledge
 ↓
Generate Response
```

The system trusts the local knowledge base and avoids unnecessary web searching.

### Advantage

This keeps the pipeline efficient when the existing knowledge base already contains useful information.

---

## Case 2 — Ambiguous Relevance

When the relevance score falls between `0.3` and `0.7`, the retrieved information may contain useful context but may not be sufficient on its own.

The system therefore combines local knowledge with web search results.

```text
Query
       ↓
FAISS Retrieval
       ↓
0.3 ≤ Relevance ≤ 0.7
       ↓
Web Search
       ↓
Knowledge Refinement
       ↓
Combine Local + Web Knowledge
       ↓
Generate Response
```

This provides the language model with a broader information context.

---

## Case 3 — Low Relevance

If the relevance score is below `0.3`, the retrieved documents are considered insufficient.

The system then performs corrective retrieval using a rewritten web-search query.

```text
Query
 ↓
FAISS Retrieval
 ↓
Relevance < 0.3
 ↓
Query Rewriting
 ↓
Web Search
 ↓
Knowledge Refinement
 ↓
Generate Response
```

This allows the system to move beyond the limitations of its local knowledge base.

---

# 🧠 Knowledge Refinement

Raw search results can contain unnecessary information, duplicated content, or irrelevant text.

Corrective RAG therefore applies a **knowledge refinement step** before using web results for response generation.

Conceptually:

```text
Web Search Results
        ↓
Extract Relevant Information
        ↓
Remove Unnecessary Content
        ↓
Refined Knowledge
        ↓
Response Generation
```

The objective is to provide the response-generation model with concise, query-relevant information rather than raw search results.

---

# ✍️ Query Rewriting

When the local retrieval process fails to provide useful information, the original user query is rewritten into a more suitable search query.

For example:

```text
Original Query
      ↓
Query Rewriter
      ↓
Optimized Search Query
      ↓
Web Search
```

Query rewriting helps the web search component retrieve information that is more closely aligned with the user's actual information need.

---

# 🤖 Response Generation

After the appropriate knowledge has been collected, the final response is generated by a language model.

The response-generation stage receives:

- Original user query
- Retrieved knowledge
- Refined web information when available
- Source information

Conceptually:

```text
User Query
     +
Retrieved Knowledge
     +
Refined Web Knowledge
     ↓
Language Model
     ↓
Final Response
```

The system can also include source information to improve transparency and allow users to understand where the information originated.

---

# 🔀 Complete Decision Flow

The complete Corrective RAG process can be summarized as:

```text
                         User Query
                             │
                             ▼
                    Generate Query Embedding
                             │
                             ▼
                       FAISS Search
                             │
                             ▼
                       Top-K Documents
                             │
                             ▼
                   Evaluate Relevance
                             │
             ┌───────────────┼───────────────┐
             │               │               │
             ▼               ▼               ▼
        Score > 0.7      0.3–0.7         Score < 0.3
             │               │               │
             ▼               ▼               ▼
       Use Local        Web Search       Rewrite Query
        Knowledge           │               │
             │               ▼               ▼
             │        Refine Knowledge   Web Search
             │               │               │
             │               ▼               ▼
             │        Combine Sources   Refine Knowledge
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                    Response Generation
                             │
                             ▼
                       Final Answer
                             │
                             ▼
                    Source Information
```

---

# 🆚 Traditional RAG vs Corrective RAG

| Feature | Traditional RAG | Corrective RAG |
|---|---:|---:|
| Vector retrieval | ✅ | ✅ |
| Retrieval evaluation | ❌ | ✅ |
| Dynamic correction | ❌ | ✅ |
| Web fallback | Optional | ✅ |
| Query rewriting | Optional | ✅ |
| Knowledge refinement | Limited | ✅ |
| Multi-source knowledge | Limited | ✅ |
| Adaptive retrieval strategy | ❌ | ✅ |
| Source transparency | Depends on implementation | ✅ |

The key difference is that **Corrective RAG does not assume that retrieval is always correct**.

---

# 🧩 Core Components

### FAISS Index

Used for efficient semantic similarity search over the existing knowledge base.

### Retrieval Evaluator

Determines whether retrieved documents are sufficiently relevant to the user's query.

### Query Rewriter

Creates an optimized query for external web search when the local knowledge base is insufficient.

### Web Search

Provides external information when the local vector database cannot adequately answer the query.

### Knowledge Refinement

Processes search results and extracts information relevant to the original question.

### Response Generator

Uses the collected knowledge to generate the final natural-language response.

---

# 📁 Project Structure

The project is implemented as a Jupyter Notebook.

```text
Corrective-RAG/
│
├── Corrective_RAG.ipynb
│
└── README.md
```

> The notebook contains the implementation and demonstrates the complete Corrective RAG workflow.

---

# 🚀 Getting Started

## 1. Clone the Repository

```bash
git clone <your-repository-url>
cd <repository-name>
```

## 2. Create a Virtual Environment

Using Python:

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

On Linux/macOS:

```bash
source .venv/bin/activate
```

---

## 3. Install Dependencies

Install the dependencies required by the notebook.

```bash
pip install -r requirements.txt
```

If the project does not contain a `requirements.txt`, install the packages used by the notebook according to the import cells.

---

## 4. Configure Environment Variables

Create a `.env` file and add the API keys required by the models and search providers used in the notebook.

Example:

```env
OPENAI_API_KEY=your_api_key
```

Do **not** commit API keys or other secrets to GitHub.

---

## 5. Run the Notebook

Start Jupyter:

```bash
jupyter notebook
```

or:

```bash
jupyter lab
```

Open the Corrective RAG notebook and execute the cells sequentially.

---

# 🧪 Example Workflow

A typical query goes through the following process:

```text
User Question
     ↓
Vector Search
     ↓
Retrieve Top-3 Documents
     ↓
Evaluate Relevance
     ↓
 ┌──────────────┬───────────────┬──────────────┐
 │              │               │
High           Medium          Low
 │              │               │
Local          Local +         Web Search
Knowledge      Web             + Query Rewrite
 │              │               │
 └──────────────┴───────────────┘
                ↓
       Knowledge Refinement
                ↓
       Response Generation
                ↓
          Final Answer
```

---

# 📊 Retrieval Decision Thresholds

The retrieval evaluator uses three decision regions:

```text
0.0 ───────────── 0.3 ───────────── 0.7 ───────────── 1.0
       LOW              AMBIGUOUS             HIGH
        │                   │                   │
     Web Search        Local + Web       Local Knowledge
```

### Low: `< 0.3`

The local retrieval is considered insufficient, so the system performs corrective web retrieval.

### Ambiguous: `0.3 – 0.7`

The retrieved information may be useful but is not sufficient to confidently rely on it alone. Local and web knowledge are combined.

### High: `> 0.7`

The retrieved information is considered sufficiently relevant and can be used directly.

> These thresholds are configurable and can be adjusted depending on the evaluation model and application requirements.

---

# 💡 Why Corrective RAG?

The central idea behind Corrective RAG is simple:

> **Retrieval should be evaluated before it is trusted.**

A traditional RAG pipeline can produce poor answers when the retriever returns poor context.

Corrective RAG introduces a feedback mechanism:

```text
Retrieve
   ↓
Evaluate
   ↓
Correct if Necessary
   ↓
Refine
   ↓
Generate
```

This makes the retrieval pipeline more robust for applications where information quality and relevance are important.

---

# ⚡ Benefits

### 1. Dynamic Correction

The system does not blindly trust vector retrieval. It evaluates retrieval quality and changes its strategy when necessary.

### 2. Better Retrieval Robustness

Poor vector-search results can be supplemented or replaced with external information.

### 3. Current Information

Web search allows the pipeline to obtain information that may not exist in the original knowledge base.

### 4. Contextual Responses

Combining multiple knowledge sources can provide the response model with broader context.

### 5. Efficient Retrieval

Web search is only introduced when the local retrieval quality indicates that it may be necessary.

### 6. Source Awareness

The system can preserve source information so that generated responses can be traced back to their supporting knowledge.

---

# ⚠️ Limitations

Corrective RAG improves retrieval robustness, but it does not guarantee that every generated response is correct.

Potential limitations include:

- Relevance evaluation can itself be imperfect.
- Web search results may contain inaccurate information.
- Query rewriting can change the intended meaning of the original query.
- External search introduces additional latency.
- API-based search and language models may introduce usage costs.
- The selected relevance thresholds may require tuning for different datasets.
- Generated answers should still be validated for high-stakes applications.

---

# 🔧 Possible Improvements

The current architecture can be extended in several ways.

### Hybrid Retrieval

Combine vector search with keyword/BM25 retrieval:

```text
Query
 ↓
Vector Search + Keyword Search
 ↓
Result Fusion
 ↓
Relevance Evaluation
```

### Reranking

Introduce a cross-encoder or reranking model after initial retrieval to improve document ordering.

### Adaptive Top-K

Instead of always retrieving three documents, dynamically determine the number of documents required.

### Source Reliability Scoring

Evaluate not only relevance but also the reliability of external sources.

### Multi-Query Retrieval

Generate multiple search queries to improve retrieval coverage.

### Evaluation Framework

Add automated evaluation metrics such as:

- Faithfulness
- Answer relevance
- Context relevance
- Retrieval precision
- Retrieval recall

### Caching

Cache web-search results and embeddings to reduce latency and API costs.

---

# 🛠️ Technologies

The implementation uses technologies associated with modern RAG systems, including:

- **Python**
- **Jupyter Notebook**
- **FAISS**
- **Vector Embeddings**
- **Large Language Models (LLMs)**
- **Web Search**
- **Retrieval Evaluation**
- **Query Rewriting**
- **Knowledge Refinement**

The exact model and API configuration can be found in the notebook.

---

# 📚 Related RAG Concepts

This project demonstrates several important concepts in modern Retrieval-Augmented Generation:

- Vector similarity search
- Semantic embeddings
- Retrieval evaluation
- Corrective retrieval
- Query rewriting
- Knowledge refinement
- Multi-source retrieval
- Context-aware generation
- Grounded response generation

---

# 🔮 Future Scope

The architecture can be further developed into a production-grade adaptive RAG system with:

- Agentic retrieval
- Hybrid search
- Cross-encoder reranking
- Knowledge graphs
- Long-term conversational memory
- Source credibility analysis
- Automatic hallucination detection
- RAG evaluation pipelines
- Observability and tracing
- Streaming responses
- Production API deployment

---

# 📖 References

This implementation is based on the broader concept of **Corrective Retrieval-Augmented Generation (CRAG)** and related research in retrieval-augmented language models.

A useful starting point for further study is the paper:

**Corrective Retrieval Augmented Generation**

> Corrective RAG introduces a mechanism for evaluating retrieved documents and taking corrective actions when retrieval quality is insufficient.

---

# 👨‍💻 Author

**Malik Saad Ahmed**

AI & DevOps Engineer

- GitHub: `Maliksaad69`
- LinkedIn: `malik-saad-ahmed-93404a318`

---

# ⭐ Support

If you find this project useful for learning about **RAG, retrieval evaluation, or corrective retrieval**, consider giving the repository a ⭐ on GitHub.

Contributions, improvements, and experimentation with different retrieval strategies are welcome.
