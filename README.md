🧠 AI Research Agent
Autonomous Paper Curation System with Feedback Learning
> A fully autonomous, production-grade intelligent system that discovers, ranks, summarizes, and delivers cutting-edge AI research papers — personalized to your interests and improving over time.
<br>
📌 Table of Contents
Overview
Problem Statement
Key Features
System Architecture
AI Integration — Claude as the Brain
The Feedback Loop
Tech Stack
Data Model
Workflow Images
Implementation Challenges
Results & Validation
Future Work
Project Structure
Author
---
🚀 Overview
The AI Research Agent solves the problem of research overload by combining LLM reasoning, workflow automation, persistent memory, and real-time user feedback into a single autonomous pipeline.
With thousands of papers published weekly across arXiv, Semantic Scholar, HuggingFace, and Google Scholar — manually finding what's relevant is inefficient and non-personalized. This system changes that.
It fetches → ranks → summarizes → delivers → learns — fully on its own, on a configurable schedule, with no manual intervention required.
---
🎯 Problem Statement
2.1 The Information Overload Problem
As of 2025, arXiv alone receives over 15,000 new submissions per month in Computer Science. Semantic Scholar indexes over 200 million academic papers. Finding the 5–10 papers genuinely worth reading each week from this volume is a significant, time-consuming task.
2.2 Limitations of Existing Solutions
Static keyword alerts lack contextual ranking — they cannot distinguish a paper that merely mentions "LLM" from one that makes a significant contribution.
Manual curation is unsustainable at scale and is not personalized.
Existing recommendation systems do not learn from explicit user feedback in real time.
Most tools deliver raw paper links without AI-generated summaries or relevance explanations.
---
⚡ Key Features
Feature	Description
🤖 Fully Autonomous	Runs on a schedule with zero manual intervention
📡 Multi-Source Fetching	arXiv, Semantic Scholar, HuggingFace, Google Scholar — simultaneously
🧠 LLM-Powered Ranking	Claude API scores each paper 1–10 for relevance to your interests
✍️ TL;DR Summaries	2-line AI-generated summaries delivered with every paper
🔁 Feedback Learning	Like/Dislike buttons update preference scores in real time
📬 Telegram Delivery	Clean Markdown paper cards with inline interactive buttons
📊 Weekly Digest	AI-written narrative summary of the week's top papers
🗄️ Persistent Memory	Supabase database ensures zero duplicate deliveries
---
🏗️ System Architecture
The system is built as three distinct but interconnected n8n workflows, each handling a specific function within the overall agentic pipeline.
---
🔹 Workflow 1 — Main Agent (Scheduled Curation)
The core workflow runs on a configurable schedule (daily or every 2 days). It orchestrates the end-to-end paper discovery, ranking, and delivery pipeline.
Stage	Description
1. Trigger	Schedule node fires on configured interval
2. Topic Config	Defines user's research interests and Telegram channel ID
3. Parallel Fetch	4 HTTP Request nodes fetch papers from all 4 sources simultaneously
4. Merge	All source results merged into a unified dataset
5. Normalize	Code node standardizes paper schema across all sources
6. Dedup Check	Supabase query fetches all previously sent paper IDs for comparison
7. Preference Load	User preferences table fetched from Supabase for Claude context
8. AI Ranking	Claude API scores each paper 1–10, generates TL;DR and relevance reason
9. Build Final List	Top 8 papers selected, Claude response parsed and merged with paper data
10. Send Telegram	Each paper sent as formatted Markdown message with inline Like/Dislike buttons
11. Save Supabase	Paper metadata + Telegram message ID saved to `sent_papers` table
12. Log Sheets	Full paper record appended to Google Sheets for tracking
---
🔹 Workflow 2 — Feedback Loop (Webhook)
This always-active webhook workflow processes user reactions from Telegram in real time. When a user clicks 👍 or 👎 on any paper, Telegram fires a callback query to the n8n webhook endpoint.
Parse Reaction — extracts message ID, user ID, and reaction type from the callback payload
Lookup Paper — queries Supabase using the Telegram message ID to identify the exact paper reacted to
Build Feedback Data — extracts topic keywords from the paper title, computes score delta (+1 / -1)
Update Preferences — upserts the `user_preferences` table (updates score if topic exists, inserts new record otherwise)
Record Reaction — updates the `user_reaction` field on the `sent_papers` record for historical tracking
---
🔹 Workflow 3 — Weekly Digest
Every Sunday, this workflow:
Pulls the top-rated papers from the past 7 days, prioritizing papers with positive user reactions
Passes them to Claude to generate a cohesive weekly research narrative
Delivers a single formatted Telegram message summarizing the week in AI research
---
🧠 AI Integration — Claude as the Brain
Claude (claude-opus-4) serves three distinct roles within the pipeline:
📊 Paper Ranking & Scoring
Claude receives a batch of normalized papers (title + abstract) along with user topic preferences and historical feedback scores. It evaluates each paper's relevance on a scale of 1–10, ensuring papers directly advancing the user's research interests receive higher priority.
✍️ TL;DR Generation
For each paper, Claude generates a concise 2-sentence summary capturing the core contribution and practical significance — delivered alongside the paper in Telegram so users can immediately assess whether it warrants a full read.
🧾 Weekly Digest Authoring
Claude synthesizes the week's top papers into a cohesive narrative digest — identifying common themes, highlighting positively-reacted papers, and concluding with an observation about the dominant research trend of the week.
⚙️ Prompt Engineering Details
Ranking prompt instructs Claude to return structured JSON — an array of objects with `index`, `score`, `tldr`, and `why_relevant` fields
Temperature `0.3` for ranking (consistency), `0.7` for digest generation (creativity)
Response parsing strips markdown code fences with `.replace(/\```json|```/g, '').trim()`before`JSON.parse()`
---
🔁 The Feedback Loop
> This is what makes it an **agent** rather than a script.
A script executes a fixed pipeline. An agent perceives outcomes, updates its understanding, and modifies future behavior. Here's how the loop works:
```
User receives paper card on Telegram with Like/Dislike buttons
        ↓
User clicks reaction → Telegram fires callback_query to n8n webhook
        ↓
Workflow 2 identifies paper via telegram_message_id
        ↓
Topic keywords extracted from paper title
        ↓
Score incremented (+1 like) or decremented (-1 dislike) in user_preferences
        ↓
Next Workflow 1 run → Claude receives updated preference scores
        ↓
Rankings adjusted → more relevant papers surfaced
```
After 10–15 reactions, the system develops a measurable understanding of your research interests and begins surfacing increasingly relevant papers.
---
🛠️ Tech Stack
Component	Technology
Workflow Engine	n8n Cloud / Railway.app (self-hosted)
LLM / AI	Anthropic Claude (claude-opus-4)
Database	Supabase (PostgreSQL) — free tier sufficient
Delivery	Telegram Bot API
Logging	Google Sheets
Paper Sources	arXiv API, Semantic Scholar Graph API, HuggingFace Daily Papers API, Google Scholar via SerpAPI
---
🗄️ Data Model
`sent_papers` Table
Stores every paper delivered to the user. The `telegram_message_id` column is critical — it links each paper to its Telegram message, enabling the feedback loop.
Column	Type	Description
`paper_id`	TEXT UNIQUE	Unique identifier from source API
`title`	TEXT	Full paper title
`source`	TEXT	arxiv / semantic_scholar / huggingface / google_scholar
`url`	TEXT	Direct link to the paper
`score`	INTEGER	Claude's relevance score (1–10)
`tldr`	TEXT	Claude-generated 2-sentence summary
`sent_at`	TIMESTAMPTZ	Timestamp of delivery
`telegram_message_id`	BIGINT	Telegram message ID — used by feedback loop
`user_reaction`	TEXT	positive / negative / null
`user_preferences` Table
Stores cumulative topic preference scores derived from user feedback. Each like/dislike upserts a score delta into this table. Claude receives this as context during ranking, enabling personalized recommendations over time.
---
🖼️ Workflow Images
<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 16px; max-width: 900px; margin: 0 auto;">
  <!-- Row 1 -->
<div style="border-radius: 10px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.15);">
    <img src="Work Flow Images/Work Flow 1.png" alt="Work Flow 1 – Main Curation Agent" style="width: 100%; height: 220px; object-fit: cover; display: block;" />
    <p style="margin: 0; padding: 8px 12px; background: #f5f5f5; font-size: 13px; color: #444;">🔹 Workflow 1 — Main Curation Agent</p>
  </div>
  <div style="border-radius: 10px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.15);">
    <img src="Work Flow Images/Work Flow 2.png" alt="Work Flow 2 – Feedback Loop" style="width: 100%; height: 220px; object-fit: cover; display: block;" />
    <p style="margin: 0; padding: 8px 12px; background: #f5f5f5; font-size: 13px; color: #444;">🔹 Workflow 2 — Feedback Loop</p>
  </div>
  <!-- Row 2 -->
<div style="border-radius: 10px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.15);">
    <img src="Work Flow Images/Work Flow 3.png" alt="Work Flow 3 – Weekly Digest" style="width: 100%; height: 220px; object-fit: cover; display: block;" />
    <p style="margin: 0; padding: 8px 12px; background: #f5f5f5; font-size: 13px; color: #444;">🔹 Workflow 3 — Weekly Digest</p>
  </div>
  <div style="border-radius: 10px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.15);">
    <img src="Work Flow Images/Folder Structure.png" alt="Folder Structure" style="width: 100%; height: 220px; object-fit: cover; display: block;" />
    <p style="margin: 0; padding: 8px 12px; background: #f5f5f5; font-size: 13px; color: #444;">🗂️ Folder Structure</p>
  </div>
  <!-- Row 3: 1 image centered -->
<div style="grid-column: 1 / -1; display: flex; justify-content: center;">
    <div style="width: 50%; border-radius: 10px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.15);">
      <img src="Work Flow Images/Daily Research papers.png" alt="Daily Research Papers – Telegram Output" style="width: 100%; height: 220px; object-fit: cover; display: block;" />
      <p style="margin: 0; padding: 8px 12px; background: #f5f5f5; font-size: 13px; color: #444; text-align: center;">📬 Daily Research Papers — Telegram Output</p>
    </div>
  </div>
</div>
---
⚙️ Implementation Challenges
Challenge	Solution
Telegram `message_id` path mismatch	`$json.result.message_id` is the correct path when Save to Supabase is wired from Send to Telegram
Supabase upsert required a unique match condition	Added `topic_keyword` as the unique matching condition with a UNIQUE constraint on the column
Topic keywords with commas breaking Supabase filter	Changed separator from comma+space to underscore in Build Feedback Data node
Multi-source paper normalization	Code node handles different JSON schemas from arXiv (XML-converted), Semantic Scholar, HuggingFace, and SerpAPI
Claude returning non-JSON response	Added `.replace(/\```json|```/g, '').trim()`before`JSON.parse()` to strip markdown fences
Duplicate papers across sources	Two-stage dedup: (1) Supabase check against `sent_papers`, (2) title-based dedup within the current batch using a Set
Wire connection order affecting `$json` context	`Build Final Paper List` uses `$('Build Final Paper List').item.json` for paper data while `$json` refers to Telegram output for `message_id`
---
✅ Results & Validation
Successful end-to-end execution confirmed:
8 papers fetched, ranked, and delivered in a single workflow execution across all 4 sources
Zero duplicate deliveries — re-running the workflow on the same day produced no repeats
Telegram messages rendered with correct Markdown formatting and functional Like/Dislike inline buttons
`sent_papers` table correctly populated with `telegram_message_id` values
Feedback loop triggered successfully — positive reactions correctly stored in `user_preferences`
Weekly digest generated and delivered with Claude-authored narrative covering 5 papers
After 10–15 reactions, the system demonstrated measurable topic preference learning
---
🔮 Future Work
Railway.app deployment — permanent free self-hosted alternative to n8n Cloud trial
Citation count weighting — prioritize papers with high citation velocity
Semantic similarity scoring — embed paper abstracts and compare cosine similarity to liked paper embeddings
Multi-user support — serve multiple researchers with independent preference profiles
`/topics` Telegram command — allow users to update interest topics dynamically without editing n8n
Paper clustering — avoid sending multiple papers on the same sub-topic in the same batch
PDF ingestion — fetch and summarize full paper PDFs for higher-confidence TL;DRs
---
📂 Project Structure
```
AI-Research-Paper-Agent/
│
├── Work Flow Images/          # Screenshots of all n8n workflow diagrams
│   ├── workflow1.png
│   ├── workflow2.png
│   ├── workflow3.png
│   ├── supabase_schema.png
│   └── telegram_output.png
│
├── R1.json                    # n8n export — Workflow 1 (Main Agent)
├── R2.json                    # n8n export — Workflow 2 (Feedback Loop)
├── R3.json                    # n8n export — Workflow 3 (Weekly Digest)
├── AI_Research_Agent_Report.docx  # Full technical project report
└── README.md
```
> Import any `.json` file directly into your n8n instance via **Settings → Import Workflow**.
---
👤 Author
Shaik Asad Ahmed
B.Tech Computer Science (Artificial Intelligence), 4th Year
![GitHub](https://img.shields.io/badge/GitHub-shaikasadahmed2k23-181717?style=flat&logo=github)
![LinkedIn](https://img.shields.io/badge/LinkedIn-Shaik%20Asad%20Ahmed-0077B5?style=flat&logo=linkedin)
---
> *Built with n8n · Anthropic Claude · Supabase · Telegram · March 2026*
