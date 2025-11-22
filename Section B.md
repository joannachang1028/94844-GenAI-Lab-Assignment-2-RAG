B.I — Why did you choose this query?

Chosen Query: “What are CMU’s quiet hours?”

I chose this query because “quiet hours” is a clearly defined policy in the student handbook, but the information is distributed across multiple sections and not contained in a single sentence. This makes it a good test case for evaluating retrieval quality. Compared with the other queries, this one strongly depends on the vector store’s ability to capture context beyond sentence boundaries. It also includes specific terminology (“quiet hours,” “quiet policies,” “courtesy hours”), which makes it easier to evaluate whether the top-k retrieved chunks are semantically relevant. Thus, this query offers a meaningful way to compare different chunking methods and k-values for retrieval effectiveness.

B.IV — Analysis for B.1 (Sentence-level chunk, different k)

1. Which k retrieved the best result?

k = 10 retrieved the best overall result.

2. Why?

Because sentence-level chunking breaks the policy into many tiny pieces, most sentences do not contain enough context for the embedding model to understand that they relate to “quiet hours.”
At small k values (k=1 or k=3), the returned sentences are often irrelevant, such as:
	•	“Mid-semester grades are not permanent…”
	•	“These materials are to be kept private…”

These are clearly noise.

At k = 10, more relevant sentences finally appear, including:
	•	“The established quiet hours…”
	•	“Quiet is defined…”
	•	“During finals week, 24-hour quiet hours will be in effect”
	•	“If you have concerns about quiet hours, consult your RA”

Higher k helps compensate for the poor recall introduced by sentence-level chunking, giving the model a greater chance to retrieve the scattered pieces of the quiet-hour policy.

3. What would you recommend to improve result quality?
	•	Use larger chunks (e.g., fixed-size 200 characters or paragraph-level) instead of single sentences
	•	Use a better embedding model (e.g., all-mpnet-base-v2)
	•	Add overlapping windowed chunks to preserve context
	•	Reduce noise by removing grade tables, numeric lists, etc. before embedding

⸻

B.V — Analysis for B.2 (Compare Sentence vs Fixed-size chunking)

1. Which k retrieved the highest quality result?

For fixed-size chunking (200 characters), k = 5 retrieved the best-quality results.

2. Why?

Unlike sentence-level chunking, fixed-size chunking preserves more context within each chunk. As a result, even small k values already retrieve chunks containing partial or full quiet-hours policy content.

Your top-5 results include:
 - Quiet living area designation
 - References to 24-hour quiet hours
 - Policy context surrounding noise standards
 - Residential environment rules

These are significantly more relevant than the sentence-level top-5 results.

Increasing k beyond 5 (e.g., k = 10) tends to add noise, such as:
	•	grading tables
	•	appendix lists
	•	unrelated sanctioning policies

Therefore, k=5 provides the best balance of recall and precision.

3. What would you recommend to improve the quality of the results?
	•	Increase chunk size further (300–500 chars) to capture full policy sections
	•	Use semantic paragraph chunking rather than fixed-size
	•	Experiment with overlapping fixed-size chunks (stride 100)
	•	Use a stronger embedding model for better policy-level semantic matching
