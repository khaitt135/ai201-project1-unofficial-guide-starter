# Project 1 Planning: The Unofficial Guide

> Write this document before you write any pipeline code.
> Your spec and architecture diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Update the Retrieval Approach and Chunking Strategy sections if you change your approach during implementation.
> Update this file before starting any stretch features.

---

## Domain

Co-op experience review. Students provide feedback and stories to draw out nuances in early career job search and planning. Topics range from securing an offer to ending one with high satisfaction.


---

## Documents

<!-- List your specific sources: URLs, subreddit names, forum threads, or file descriptions.
     Aim for at least 10 sources that together cover different subtopics or perspectives within your domain. -->

| # | Source | Description | URL or location |
|---|--------|-------------|-----------------|
| 1 | Reddit r/NEU | Student discusses how co-op differs/is better than traditional internships | https://www.reddit.com/r/NEU/comments/1sdx6ne/how_is_this_coop_differentbetter/ |
| 2 | Reddit r/NEU | Student shares co-op search statistics and outcomes | https://www.reddit.com/r/NEU/comments/1pl27a1/coop_search_stats/ |
| 3 | Reddit r/NEU | Students share how they landed their co-op | https://www.reddit.com/r/NEU/comments/1i057k8/how_did_you_get_a_coop/ |
| 4 | Reddit r/NEU | Student posts co-op search results and experience | https://www.reddit.com/r/NEU/comments/1sf3qem/coop_search_results/ |
| 5 | Reddit r/NEU | Student reflects on how unique Northeastern's co-op program is | https://www.reddit.com/r/NEU/comments/1thsny7/sometimes_i_forget_how_insane_northeasterns_coop/ |
| 6 | Hacker News | Discussion thread on post-grad opportunities | https://news.ycombinator.com/item?id=30619899 |
| 7 | Hacker News | Discussion thread on career shifts | https://news.ycombinator.com/item?id=47860627 |
| 8 | Hacker News | Discussion thread on career developments for SWE in the age of AI | https://news.ycombinator.com/item?id=47832297 |
| 9 | Blind | Early career professionals debate better pay vs. better companies | https://www.teamblind.com/post/early-career-better-pay-or-better-companies-mac2eu5l |
| 10 | Blind | Early career professional discusses prep culture and its pressures | https://www.teamblind.com/post/early-years-of-my-career-and-prep-culture-is-a-nightmare-uth1spzx |

---

## Chunking Strategy

**Chunk size:**

Fixed-size: 300 characters

**Overlap:**
50 characters

**Reasoning:**
Reviews are generally short. If an answer is longer than a typical chunk, its paragraphs are also structured by different key ideas so this chunking strategy works regardless of edge cases.
---

## Retrieval Approach

<!-- Which embedding model are you using (e.g., all-MiniLM-L6-v2 via sentence-transformers)?
     How many chunks will you retrieve per query (top-k)?
     If you were deploying this for real users and cost wasn't a constraint, what tradeoffs
     would you weigh in choosing a different embedding model — context length, multilingual
     support, accuracy on domain-specific text, latency? -->

**Embedding model:**
all-MiniLM-L6-v2 via sentence-transformers

**Top-k:**
k = 5. 

**Production tradeoff reflection:**

When deployed for real user, take into consideration the conversation context and token limit. It can take longer processing time to search for the right keyword and consolidate them into a proper answers so there must be a template or documentation map.
---

## Evaluation Plan

| # | Question | Expected Answer |
|---|----------|-----------------|
| 1 | How many applications did NEU students typically send before landing a co-op in recent cycles? | Based on r/NEU posts, students report sending 100–300+ applications, with many citing 150–200 as typical before getting an offer |
| 2 | What specific strategies did students who landed competitive co-ops (FAANG/top startups) say made the difference? | Students cite leetcode grinding, networking through LinkedIn/cold email, applying early in the cycle, and having prior project experience |
| 3 | What do early-career professionals (2–5 years post-grad) say was the biggest mistake they made in their first co-op or job? | Common answers include not negotiating offers, staying too long at a low-growth company, and not building a visible portfolio |
| 4 | What technical skills do Blind and HN users most commonly recommend for new CS grads entering the job market in 2024–2025? | System design, DSA for interviews, a strong GitHub, and at least one cloud platform (AWS/GCP) come up most frequently |
| 5 | Is it better to take a high-paying co-op at an unknown company or a lower-paying one at a well-known company, according to early-career professionals? | Blind discussions lean toward brand name early on for exit opportunities, but several counter that comp + real ownership at startups beats prestige |

---

## Anticipated Challenges

<!-- What could go wrong? Name at least two specific risks with reasoning.
     Consider: noisy or inconsistent documents, missing source attribution, off-topic
     retrieval, chunks that split key information across boundaries. -->

1. Source inconsistencies that challenge output processing time

2. Off-topic retrieval because too often reviews are negative experiences that generate skewed narratives

---

## Architecture

flowchart LR
    A["Document Ingestion\n──────────────\nReddit r/NEU\nHacker News\nTeam Blind\n──────────────\ntool: requests\n+ BeautifulSoup"] --> B["Chunking\n──────────────\nChunk size: 300 chars\nOverlap: 50 chars\n──────────────\ntool: custom\nPython splitter"]

    B --> C["Embedding\n──────────────\nall-MiniLM-L6-v2\n──────────────\ntool: sentence-\ntransformers"]

    C --> D["Vector Store\n──────────────\nStores chunks\n+ metadata\n──────────────\ntool: ChromaDB"]

    D --> E["Retrieval\n──────────────\nSemantic search\ntop-k = 5\n──────────────\ntool: ChromaDB\nquery()"]

    E --> F["Generation\n──────────────\nGrounded answer\n+ source citation\n──────────────\ntool: Groq\nllama-3.3-70b"]

## AI Tool Plan

| Pipeline Stage | AI Tool | Input I'll Provide | Expected Output | How I'll Verify |
|---------------|---------|-------------------|-----------------|-----------------|
| Document Ingestion | Claude | Documents table (source URLs, types) + description of each site's structure | A script that fetches and saves raw text from each URL to .txt files | Open 2–3 saved files and confirm they contain actual post/comment text, not HTML tags or nav menus |
| Chunking | Claude | Chunking Strategy section (300 chars, 50 overlap) + one sample .txt file | A `chunk_text()` function that splits text and returns chunks with source metadata | Print 5 chunks and check each is ~300 chars, has overlap with neighbors, and includes source filename |
| Embedding + Vector Store | Claude | Architecture diagram + Retrieval Approach section (all-MiniLM-L6-v2, ChromaDB) | Code that embeds all chunks and stores them in ChromaDB with source metadata | Query ChromaDB directly for a test string and confirm it returns 5 chunks with correct source fields |
| Retrieval | Claude | Evaluation Plan (5 test questions) + retrieval spec (top-k=5) | A `retrieve(query)` function that returns top-5 chunks with distance scores | Run all 5 test questions and check that returned chunks visibly relate to each question; flag any score above 0.6 |
| Generation | Claude | Grounding requirement + output format (answer + source list) + Gradio skeleton | A `generate(query)` function and Gradio UI wired end-to-end | Ask a question not in my documents and confirm the system says it doesn't have enough information rather than guessing |
```

**Milestone 3 — Ingestion and chunking:**

**Milestone 4 — Embedding and retrieval:**

**Milestone 5 — Generation and interface:**
