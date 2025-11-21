# Section A: Vector Store Query Design

## A.I. Similarity Metric Rationale

**Metric Chosen:** Euclidean Distance (L2) - using `faiss.IndexFlatL2`

### Rationale for Choosing Euclidean Distance (L2)

I chose Euclidean Distance (L2) for the vector store because it provides an intuitive and computationally efficient way to measure similarity between embeddings. Euclidean Distance measures the straight-line distance between two points in high-dimensional space, where smaller distances indicate greater similarity. FAISS offers highly optimized implementations for L2 distance, enabling fast similarity searches even with large vector stores. Additionally, when embeddings are normalized (as is common with models like `bert-base-nli-mean-tokens`), L2 distance correlates well with semantic similarity while still considering both the direction and magnitude of vectors, providing a balanced approach to measuring similarity.

### One Advantage of Using Euclidean Distance

**Efficient computation with optimized implementations**: FAISS provides highly optimized implementations for L2 distance, enabling fast similarity searches even with large vector stores containing thousands of embeddings. This computational efficiency is crucial for real-time query processing in RAG systems.

### One Difference Between Euclidean Distance and Other Similarity Metrics

**Euclidean Distance vs. Cosine Similarity and Dot Product**: Euclidean Distance measures distance (where smaller values indicate greater similarity), while both Cosine Similarity and Dot Product measure similarity (where larger values indicate greater similarity). Additionally, Euclidean Distance considers both the direction and magnitude of vectors, whereas Cosine Similarity only considers the angle between vectors (ignoring magnitude), and Dot Product is highly sensitive to vector magnitude, which can cause large-magnitude vectors to dominate the similarity score regardless of their direction.

---

## A.III. Qualitative Analysis of Query Results

### Did the queries retrieve the information you were expecting to obtain? Why or why not?

The queries produced mixed results, with varying degrees of success:

**Query 1: "What is the policy statement for the academic integrity policy?"**
- **Unsuccessful**: The retrieved responses include related content about academic integrity (reporting forms, appeals process) but do not contain the actual policy statement itself. Responses mention "Academic Integrity Reporting Form" and "appealing an academic integrity violation," which are related but not the core policy statement. Some responses (Responses 3-5) are about intellectual property, fair use, and confidential information, which are tangentially related but not directly answering the query.

**Query 2: "What is the policy violation definition for cheating?"**
- **Unsuccessful**: The responses contain general misconduct definitions and related policy content, but not the specific definition of "cheating" as a policy violation. Response 2 mentions "misconduct" definitions, and Response 3 mentions "circumventing system security," which relates to cheating but is not the explicit definition. Responses 4-5 are about hazing violations, which are not relevant to cheating.

**Query 3: "What is the policy statement for improper or illegal communications?"**
- **Unsuccessful**: The responses retrieved general misconduct and violation content but not the specific policy statement for improper or illegal communications. Response 4 mentions "Violate Federal or State criminal law," and Response 5 references "Misuse and Inappropriate Behavior," which are somewhat related but do not directly answer the query about communications policy.

**Query 4: "What are CMU's quiet hours?"**
- **Partially Successful**: This query retrieved partially relevant information. It seems the search tried to capture relted keywords, but didn't really answer the question. Response 2 states "The established quiet hours stated above are the minimums for every residential area," and Response 4 provides a definition: "Quiet is defined as being unable to hear any noise at a distance of 10 feet from a room with a closed door." However, the actual quiet hours times are not explicitly stated in the retrieved responses (Response 2 references "quiet hours stated above," suggesting the information may be in a previous chunk).

**Query 5: "Where are pets allowed on CMU?"**
- **Partially Successful**: This query retrieved partially relevant information. Response 4 directly answers: "Pets are permitted outside on campus grounds when leashed and properly attended." Responses 2-3 provide additional relevant information about emotional support animals in housing facilities, while response 1 directs to the Housing Policies website for more information about keeping pets at housing. 

### Why do you think the queries were successful / unsuccessful in retrieving the information you expected or needed?

**Partially successful queries (Queries 4 and 5):**
- **Specific terminology match**: Both queries used specific terms ("quiet hours," "pets allowed") that appear in the source document, enabling some semantic matching. However, the retrieval was incomplete - Query 4 retrieved definitions and references to quiet hours but not the actual times, and Query 5 retrieved some location information but may have missed comprehensive details.
- **Clear, focused queries**: These queries ask for concrete information rather than abstract policy statements, which helped the embedding model find some relevant matches. However, the sentence-level chunking may have fragmented the complete answer across multiple chunks.
- **Context fragmentation**: Even for these more concrete queries, sentence-level chunking broke information across chunks (e.g., Query 4's Response 2 references "quiet hours stated above," indicating the actual times are in a previous chunk that wasn't retrieved).

**Unsuccessful queries (Queries 1, 2, and 3):**
- **Abstract query structure**: Queries asking for "policy statements" or "definitions" are more abstract and do not match well with the actual text structure in the document, which may present information differently than expected.
- **Context fragmentation**: The sentence-level chunking broke policy statements that span multiple sentences, making it difficult to retrieve complete policy statements.
- **Semantic mismatch**: The embedding model did not effectively capture the semantic relationship between abstract query terms (e.g., "policy statement") and the actual content format in the document.
- **Multi-concept queries**: Query 3 combines multiple concepts ("improper," "illegal," "communications"), which was challenging for the embedding model to match accurately.
- **Terminology variations**: The document uses different terminology than the queries (e.g., "misconduct" vs. "cheating"), causing semantic mismatches that prevented accurate retrieval.

### What features or components of the vector store design do you think affected the success or failure of retrieving information that was relevant to your queries to the vector store?

**1. Chunking Strategy (Sentence-level)**
- **Impact on partial success**: Sentence-level chunking partially worked for queries seeking specific facts (quiet hours, pet locations), but even these queries suffered from context fragmentation. For example, Query 4 retrieved a reference to "quiet hours stated above" but not the actual times, indicating that related information was split across chunks.
- **Impact on failure**: For queries seeking policy statements or definitions that span multiple sentences, sentence-level chunking fragmented the context, making it impossible to retrieve complete answers. This particularly affected Queries 1, 2, and 3, which sought comprehensive policy statements. Even for the partially successful queries, chunking limitations prevented complete information retrieval.

**2. Embedding Model (bert-base-nli-mean-tokens)**
- **Impact on partial success**: The model captured some semantic relationships for concrete queries with specific terminology (e.g., "quiet hours," "pets"), enabling partial matches. However, it did not consistently retrieve all relevant information, suggesting limitations in semantic understanding.
- **Impact on failure**: The model struggled significantly with abstract queries seeking "policy statements" or "definitions," as these abstract concepts don't align well with how the model represents semantic similarity. The model also had difficulty with multi-concept queries (Query 3) that combine several related but distinct terms, and with terminology variations (e.g., "misconduct" vs. "cheating").

**3. Similarity Metric (Euclidean Distance / L2)**
- **Impact**: The Euclidean Distance metric functioned as expected, measuring vector distances effectively. However, the metric's performance is dependent on the quality of embeddings produced by the model. Since the embedding model struggled with abstract concepts and semantic relationships, these limitations were reflected in the distance calculations, resulting in poor ranking of relevant documents for abstract queries.

**4. k Parameter (k=5)**
- **Impact on partial success**: Using k=5 provided multiple candidate responses, which helped retrieve some complementary information for Queries 4 and 5 (e.g., quiet hours definition and pet location information). However, even with k=5, the complete answers were not retrieved, suggesting that relevant chunks may have been ranked lower or that information was too fragmented.
- **Impact on failure**: For unsuccessful queries, k=5 retrieved multiple irrelevant responses, suggesting that even the top 5 matches were not highly relevant. This indicates deeper issues with semantic matching - the embedding model and similarity metric were not effectively identifying relevant content, so increasing k would not have helped.

**5. Query Structure and Terminology**
- **Impact**: Queries using specific, concrete terms that match document terminology (e.g., "quiet hours," "pets") performed better than queries using abstract terms (e.g., "policy statement," "definition") or combining multiple concepts. However, even queries with concrete terminology only achieved partial success, indicating that the embedding model and chunking strategy have fundamental limitations in capturing complete answers, regardless of query structure.

**Overall Assessment**: The vector store design achieved only partial success even for the best-performing queries, and complete failure for abstract queries. The sentence-level chunking strategy consistently fragmented context, preventing complete information retrieval. The embedding model's limitations with abstract concepts, terminology variations, and multi-concept queries were the primary factors causing retrieval failures. The similarity metric and k parameter functioned as designed but were constrained by the quality of embeddings and chunking strategy.

