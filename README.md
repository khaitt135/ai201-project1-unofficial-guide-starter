# The Unofficial Guide — Project 1

## Domain

Student experiences and advice around Northeastern University's co-op program. This knowledge is valuable because it reflects real student outcomes — application volumes, interview strategies, and early career decisions — that are not captured in official university documentation. The co-op process is brutal and competitive, and the most useful guidance comes from students who have been through it, not from NEU's official pages.

---

## Document Sources

| # | Source | Type | URL or file path |
|---|--------|------|-----------------|
| 1 | Reddit r/NEU | Forum thread | https://www.reddit.com/r/NEU/comments/1pl27a1/coop_search_stats/ |
| 2 | Reddit r/NEU | Forum thread | https://www.reddit.com/r/NEU/comments/1sf3qem/coop_search_results/ |
| 3 | Reddit r/NEU | Forum thread | https://www.reddit.com/r/NEU/comments/1i057k8/how_did_you_get_a_coop/ |
| 4 | Reddit r/NEU | Forum thread | https://www.reddit.com/r/NEU/comments/1sdx6ne/how_is_this_coop_differentbetter/ |
| 5 | Reddit r/NEU | Forum thread | https://www.reddit.com/r/NEU/comments/1thsny7/sometimes_i_forget_how_insane_northeasterns_coop/ |
| 6 | Hacker News | Forum thread | https://news.ycombinator.com/item?id=30619899 |
| 7 | Hacker News | Forum thread | https://news.ycombinator.com/item?id=47860627 |
| 8 | Hacker News | Forum thread | https://news.ycombinator.com/item?id=47832297 |
| 9 | Blind | Forum thread | https://www.teamblind.com/post/early-career-better-pay-or-better-companies-mac2eu5l |
| 10 | Blind | Forum thread | https://www.teamblind.com/post/early-years-of-my-career-and-prep-culture-is-a-nightmare-uth1spzx |

---

## Chunking Strategy

**Chunk size:** 300 characters

**Overlap:** 50 characters

**Why these choices fit your documents:** Sources are short informal forum posts and Reddit comments. Most individual thoughts fit within 300 characters. Overlap of 50 characters ensures that ideas split across chunk boundaries are still partially captured in adjacent chunks. Before chunking, documents were cleaned to remove navigation menus, timestamps, upvote counts, ads, and sidebar boilerplate using regex and a manual phrase blocklist.

**Final chunk count:** 265 chunks across 10 documents

---

## Embedding Model

**Model used:** all-MiniLM-L6-v2 via sentence-transformers, running locally with no API key or rate limits.

**Production tradeoff reflection:** For a production deployment, I would consider all-mpnet-base-v2 which handles up to 384 tokens and scores higher on retrieval benchmarks. OpenAI's text-embedding-3-large would improve semantic accuracy but adds per-call API cost and network latency. For a student-facing tool with informal, slang-heavy text ("coop grind", "nuworks", "OA"), a model fine-tuned on career or forum data would likely outperform a general-purpose model. Multilingual support is not a concern here since all sources are in English.

---

## Grounded Generation

**System prompt grounding instruction:**
Answer the question using ONLY the provided documents. If the documents do not contain enough information to answer, say "I don't have enough information on that." Do not use any outside knowledge.

**How source attribution is surfaced in the response:** Source filenames are collected from ChromaDB metadata for each retrieved chunk and appended programmatically to every response in a separate Sources field in the Gradio UI. Attribution is not left to the LLM — it is extracted directly from retrieval results.

---

## Evaluation Report

| # | Question | Expected answer | System response (summarized) | Retrieval quality | Response accuracy |
|---|----------|-----------------|------------------------------|-------------------|-------------------|
| 1 | show me typical co-op stats | 68-100+ applications, high ghost rate, 1 offer | "I don't have enough information" | Partially relevant | Inaccurate |
| 2 | strategy to secure co-op | LeetCode, networking, applying early, referrals | Be confident in interviews, use NUSource and LinkedIn | Relevant | Partially accurate |
| 3 | biggest mistake in one's first job | Not negotiating, staying too long at low-growth company | Not joining big tech sooner | Relevant | Partially accurate |
| 4 | tips on finding jobs | LeetCode, cold email, resume review, career fairs | Check resume, practice interviews, use LeetCode and NUSource | Relevant | Accurate |
| 5 | is it better high pay unknown company or lower pay brand name | Brand name early for exit opportunities | Consensus leans brand name, but 40% raise may be worth it | Relevant | Accurate |

**Important note on query phrasing:** The system works well with short keyword queries that mirror the informal register of the source documents. Long, formal evaluation-style questions (20+ words) consistently failed to retrieve relevant chunks even when the same question phrased as short keywords succeeded. For example, "What do early-career professionals (2-5 years post-grad) say was the biggest mistake they made in their first co-op or job?" returned no results, while "biggest mistake in one's first job" retrieved relevant Blind and HN content. All 5 evaluation questions above were tested using their short keyword form.

### Retrieved chunks per question

**Q1 — show me typical co-op stats**
- [Sometimes I forget how insane NEU's co-op is] "overall there is just a much heavier emphasis on co-op here than other schools"
- [Co-op Search Stats] "these coop cycles are kind of brutal; I know people who have applied to 100+ both times and got little to no interviews"
- Root cause: Sankey chart data (68 applications, 40 ghosted, 1 offer) was embedded as an image in the PDF. pdfplumber extracts text only, so the numbers were never stored as chunks.

**Q2 — strategy to secure co-op**
- [Sometimes I forget...] "the co-op program is well-integrated and seamless"
- [How did you get a co-op] "Use NUSource and LinkedIn to reach out to alumni at top-tier tech companies"
- [How is this co-op different/better] "university staff are much more helpful in the process"

**Q3 — biggest mistake in one's first job**
- [How did you get a co-op] "if I had joined big tech 10 years earlier I would have hit my retirement number"
- [Early years of my career - Blind] "prep culture is a nightmare"
- [Ask HN - grad school] career regret discussion

**Q4 — tips on finding jobs**
- [Sometimes I forget...], [Co-op Search Stats], [How did you get a co-op], [Co-op Search Results]
- All 4 sources directly discuss job search strategies for co-op students

**Q5 — is it better high pay unknown company or lower pay brand name**
- [Early career Blind] "Sometimes it's just getting closer to a better long-term lane. I wouldn't leave purely for higher pay."
- Single source, highly relevant, accurate response

---

## Failure Case Analysis

**Question that failed:** Long-form version: "What do early-career professionals (2-5 years post-grad) say was the biggest mistake they made in their first co-op or job?"

**What the system returned:** "I don't have enough information on that."

**Root cause (tied to a specific pipeline stage):** This is a retrieval failure caused by query phrasing mismatch. The embedding model matches semantic similarity between the query vector and stored chunk vectors. Long, formal questions with parenthetical qualifiers produce a different embedding than the informal short-form text in the source documents. The vocabulary gap between "early-career professionals (2-5 years post-grad)" and how Blind or Reddit users actually write ("my first job", "when I started out") means the query vector lands far from the relevant chunk vectors in embedding space. The same question as "biggest mistake in one's first job" succeeded because its phrasing is closer to the source register.

**What you would change to fix it:** Add a query preprocessing step that strips formal qualifiers and shortens queries to 10 words or fewer before embedding. A reranker model after initial retrieval would also help recover relevant chunks that were missed due to phrasing mismatch.

---

## Spec Reflection

**One way the spec helped you during implementation:** Writing the chunking strategy in planning.md before touching any code forced an early decision about chunk size. Knowing chunks would be 300 characters shaped how the cleaning script was written — boilerplate removal needed to be aggressive enough that no chunk would be dominated by navigation text rather than content. Without the spec, cleaning would likely have been an afterthought.

**One way your implementation diverged from the spec, and why:** The spec assumed documents would be scraped programmatically using requests and BeautifulSoup. In practice, Reddit, Blind, and Hacker News all block automated scraping. The pipeline was changed to PDF export via browser print and pdfplumber text extraction. This introduced a new failure mode: chart data embedded as images in PDFs is invisible to pdfplumber, which caused the co-op stats retrieval failure documented above.

---

## AI Usage

**Instance 1**

- *What I gave the AI:* The Documents table from planning.md listing 10 sources, plus a description of each site's structure (Reddit thread, Blind post, HN thread)
- *What it produced:* A pdf_to_txt.py script using pdfplumber to extract text from PDFs and a regex-based cleaning function with a boilerplate phrase list
- *What I changed or overrode:* The initial boilerplate list was too short. After inspecting real output, I manually added 20+ additional phrases (sidebar rules, moderator names, ad text, Reddit UI labels) that the first version missed. I also added a regex to remove timestamp patterns like "6/9/26, 7:57 PM" that appeared on every PDF page.

**Instance 2**

- *What I gave the AI:* The Chunking Strategy section (300 chars, 50 overlap), the Retrieval Approach section (all-MiniLM-L6-v2, ChromaDB, top-k=5), and the architecture diagram from planning.md
- *What it produced:* embed_and_store.py that loaded chunks, embedded with sentence-transformers, and stored in ChromaDB with source metadata; and query.py that retrieved top-5 chunks and passed them to Groq's llama-3.3-70b-versatile with a grounding system prompt
- *What I changed or overrode:* The generated embed script crashed on re-runs because ChromaDB threw an error when trying to create a collection that already existed. I added a delete_collection call before create_collection to allow clean re-ingestion. I also verified the system prompt enforced grounding by testing an out-of-scope question ("best pizza near NEU") and confirming the model refused rather than hallucinating.
