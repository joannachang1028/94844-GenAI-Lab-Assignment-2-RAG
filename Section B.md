# Section B: Vector Store Query Design

## B.I. Rationale for Selected Query

**Chosen query:** “What are CMU’s quiet hours?”

I selected this query for three main reasons:

1. **Well-defined but fragmented concept.**  
   Quiet hours are a clearly defined policy, but the relevant content is scattered across multiple sections of the handbook. This makes the query a good test of whether the vector store can retrieve related information spread over different parts of the document.

2. **Clear, domain-specific terminology.**  
   The query uses specific terms such as *“quiet hours,” “quiet policies,”* and *“courtesy hours”*. This makes it straightforward to evaluate whether a retrieved chunk is semantically relevant or not.

3. **Useful for comparing designs.**  
   Because the answer is relatively short and factual, it provides a meaningful basis for comparing both:
   - different chunking strategies (sentence-level vs. fixed-size chunks), and  
   - different values of the `k` parameter (1, 3, 5, 10)
   in terms of precision, recall, and noise in the retrieved results.

---

## B.IV. Analysis of k Parameters for B.1 (Sentence-Level Chunking)

### 1. Which ‘k’ parameter retrieved the highest quality / best result?

For the sentence-level chunking setup in B.1, I believe **k = 5** produced the best overall result.

- When **k = 1**, the top result was a sentence about mid-semester grades, which is completely unrelated to quiet hours. The very top hit was off-topic, so precision at 1 was poor.
- When **k = 3**, one sentence finally became relevant:

  > “The established quiet hours stated above are the minimums for every residential area.”

  However, the other sentences were still about privacy and academic materials, not quiet hours.
- When **k = 5**, the retrieved set included both:
  - “The established quiet hours stated above are the minimums for every residential area.”
  - “‘Quiet’ is defined as being unable to hear any noise at a distance of 10 feet from a room with a closed door.”

  Together, these sentences define what “quiet” means and how quiet hours are enforced, which aligns closely with the query.
- When **k = 10**, additional results introduced more noise, such as:
  - sentences about short-term accommodations,
  - a sentence about consent in sexual activity, and
  - a sentence about not requiring substantial use of university resources.  

  These are mostly off-topic. Increasing `k` from 5 to 10 slightly improves recall but clearly reduces precision.

Overall, **k = 5** provides the best trade-off: it captures multiple relevant sentences about quiet hours without being overwhelmed by unrelated policy text.

### 2. Why was this parameter the best for the query?

I consider **k = 5** the “best” mainly because it balances **relevance** and **noise**:

- At low `k` (1 or 3), the system either misses the core answer entirely (`k = 1`) or only partially covers it (`k = 3`).
- At higher `k` (10), the system returns many sentences that are clearly unrelated to quiet hours, making it harder for a downstream QA model or a human reader to identify key information.
- At `k = 5`, the result set consistently includes:
  - a sentence explicitly mentioning “quiet hours” and “minimums for every residential area”, and
  - a sentence giving a concrete operational definition of “quiet” (cannot hear noise at 10 feet with a closed door).

This combination is concise enough to be readable, but rich enough to reconstruct a correct and complete answer.

### 3. How could the result quality be improved?

Beyond tuning `k`, several changes could improve retrieval quality:

1. **Hybrid retrieval (semantic + keyword).**  
   Combine embedding-based similarity with keyword/BM25 filtering on terms like *“quiet hours”*, *“noise”*, or *“residential area”*. This would downweight obviously irrelevant sentences about grades, consent, or accommodations, even if they are semantically close in the embedding space.

2. **Better chunking for policy sections.**  
   Sentence-level chunking sometimes isolates a sentence that refers to missing context (e.g., “the quiet hours stated above”) without including the actual time window. Using paragraph-level chunks or small overlapping windows of 2–3 sentences would help keep definitions and their local context together.

3. **Query reformulation or expansion.**  
   Reformulate the query to:

   > “What are CMU’s quiet hours, including the specific times and the definition of quiet in residential areas?”

   or add synonyms such as *“noise policy”* or *“residence hall quiet time”*. This can help the embedding model align the query more precisely with the handbook’s wording.

4. **Second-stage re-ranking.**  
   After retrieving the top-k sentences, a re-ranking step (using a small LLM or rule-based scoring) could prioritize chunks that explicitly mention “quiet hours” and demote generic policy sentences, improving the final ranked list.

---

## B.V. Analysis of k Parameters for B.2 (Fixed-Size 200-Character Chunking)

### 1. Which ‘k’ parameter retrieved the highest quality / best result?

For the alternative chunking method in B.2 (fixed-size 200-character chunks), **k = 1** produced the highest quality result.

With this chunking strategy, the top-1 result already contains highly relevant information:

> “particularly quiet and unobtrusive environment. This designation includes a provision for an extension of the university’s current quiet hours policy to 24 hours. As such, residents will be responsib…”

This chunk:

- directly mentions **quiet hours**,  
- introduces the idea of a **24-hour quiet policy**, and  
- clearly describes the context of a quiet living environment.

When `k` increases to 3, 5, or 10, the additional chunks include:

- text about **user directory privacy**,  
- text about **social events and small gatherings**,  
- text about **community standards processes**, and  
- large blocks of unrelated content: grading tables, sanctions, health center information, and accommodation policies.

These extra chunks add some weakly related context (e.g., social events in quiet areas) but also introduce significant noise that is not needed to answer “What are CMU’s quiet hours?”. From an answer-precision perspective, **k = 1**, which returns the single most relevant chunk, is the best choice.

### 2. Why was this parameter the best for the query?

Under fixed-size character chunking, each chunk tends to contain multiple sentences and policy ideas, making each retrieved chunk very dense. Because of this:

- The **top-1 chunk** already contains the key idea: a quiet living area with extended quiet hours (up to 24 hours), which directly responds to the query.
- As `k` increases, subsequent chunks bring in unrelated policy topics (grading, sanctions, health center responsibilities, etc.). These are semantically close to “policy” but not to “quiet hours.”
- Unlike sentence-level chunking, where a larger `k` can bring together complementary sentences, here increasing `k` mostly causes **topic drift** instead of improving coverage of quiet-hours details.

Since the first hit is strongly on-topic and later hits are increasingly noisy, **k = 1** gives the best signal-to-noise ratio.

### 3. How could the result quality be improved?

The fixed-size chunking experiment shows that the model can retrieve roughly the right area of the text, but each chunk may still contain a mix of relevant and irrelevant information. To improve this setup:

1. **Use semantic units for chunking instead of raw character counts.**  
   Rather than blind 200-character windows, use:
   - paragraph-level chunks (split on blank lines or headings), or  
   - sliding windows of 2–3 sentences.  

   This keeps chunks more coherent and reduces the chance of mixing quiet-hours text with unrelated grading policies in the same embedding.

2. **Introduce overlap in fixed-size chunks.**  
   If character-based chunking is required, use overlapping windows (e.g., 200 characters with 50–100 characters of overlap). This increases the likelihood that the key quiet-hours sentence appears in at least one chunk in a complete form.

3. **Apply a lightweight keyword-based post-filter.**  
   After retrieving top-k chunks, apply a simple filter that checks for terms like “quiet hours,” “quiet living area,” or “noise.” Chunks without any of these terms can be downweighted or removed before passing them to a reader model.

4. **Adapt k to chunk size.**  
   With longer, denser chunks, smaller `k` values (e.g., 1–3) are often sufficient and help maintain precision. A simple heuristic would be:
   - For **short, sentence-level chunks**: use larger `k` (e.g., 5–10).  
   - For **long, fixed-size chunks**: use smaller `k` (e.g., 1–3).

Overall, these adjustments would make the fixed-size chunking strategy more competitive with sentence-level chunking and would reduce the amount of off-topic policy content returned for a focused query like “What are CMU’s quiet hours?”.