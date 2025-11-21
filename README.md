# 94844-GenAI-Lab-Assignment-2-RAG

Assignment 2: Retrieval Augmented Generation (RAG) with LLMs
Group 3: Joanna Chang and Siqi Yu

## Project Overview

This repository contains the implementation and analysis for Assignment 2, which focuses on building a RAG (Retrieval Augmented Generation) system using vector stores. The project involves:

- Loading and chunking the CMU Student Handbook PDF
- Creating embeddings using sentence transformers
- Building a FAISS vector store
- Querying the vector store with various queries
- Analyzing retrieval performance

## Files Structure

- `HW2_RAG.ipynb` - Main Jupyter notebook with implementation
- `Section A.md` - Complete analysis and answers for Part A
- `rag_hw02_partA_results.xlsx` - Query results for Part A (5 queries with k=5)
- `rag_hw02_submission_template.xlsx` - Submission template
- `rag-requirements-1.txt` - Python package dependencies
- `CMU Student Handbook 2023-24.pdf` - Source document

## Setup Instructions

1. Install dependencies:
```bash
pip install -r rag-requirements-1.txt
```

2. Set up environment variables:
   - Create a `.env` file with your API keys:
   ```
   OPENAI_API_KEY=your_key_here
   HF_TOKEN=your_token_here
   ```

3. Open `HW2_RAG.ipynb` in Jupyter Notebook and run the cells sequentially.

## Section A: Experimenting with Vector Store Query Design

### A.I. Similarity Metric Rationale
- Complete analysis and answers are documented in `Section A.md`

### A.II. Retrieved Results into the Spreadsheet
- Part A query results are saved in `rag_hw02_partA_results.xlsx`

### A.III. Qualitative Analysis of Query Results

**Query Performance Summary:**

| Query | Status | Description |
|-------|--------|-------------|
| Query 1: Academic integrity policy statement | ❌ Unsuccessful | Retrieved related content but not the actual policy statement |
| Query 2: Cheating definition | ❌ Unsuccessful | Retrieved general misconduct content but not specific cheating definition |
| Query 3: Improper/illegal communications policy | ❌ Unsuccessful | Retrieved general misconduct content but not specific communications policy |
| Query 4: CMU's quiet hours | ⚠️ Partially Successful | Retrieved definitions and references but not actual times |
| Query 5: Where pets allowed | ⚠️ Partially Successful | Retrieved some location information but incomplete |

- Complete analysis and answers are documented in `Section A.md`

## Technologies Used

- **FAISS** - Vector similarity search
- **Sentence Transformers** - Embedding generation (`bert-base-nli-mean-tokens`)
- **PyMuPDF (fitz)** - PDF processing
- **LangChain** - Text chunking utilities
- **Pandas** - Data manipulation
- **NLTK** - Sentence tokenization