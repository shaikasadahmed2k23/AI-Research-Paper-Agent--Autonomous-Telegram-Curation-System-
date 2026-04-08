# 🧠 AI Research Agent – Autonomous Paper Curation System

## 🚀 Overview
AI Research Agent is a fully autonomous, production-grade intelligent system that discovers, ranks, summarizes, and delivers cutting-edge AI research papers with personalized recommendations.

It solves the problem of research overload by combining LLM reasoning, workflow automation, persistent memory, and real-time user feedback.

---

## 🎯 Problem Statement
With thousands of research papers published weekly across platforms like arXiv and Semantic Scholar, manually finding relevant papers is inefficient and non-personalized.

This system automates:
- 📄 Paper discovery
- 🧠 Relevance understanding
- 📊 Ranking & summarization
- 🔁 Continuous learning from user feedback

---

## ⚡ Key Features
- 🤖 Fully autonomous research agent (no manual intervention)
- 📡 Multi-source paper fetching (arXiv, Semantic Scholar, HuggingFace, Google Scholar)
- 🧠 LLM-powered ranking (Claude API)
- ✍️ AI-generated TL;DR summaries
- 🔁 Real-time feedback learning (Like/Dislike system)
- 📬 Telegram-based delivery with interactive buttons
- 📊 Weekly AI-generated research digest
- 🗄️ Persistent memory using Supabase (no duplicate papers)

---

## 🏗️ System Architecture

### 🔹 Workflow 1: Main Agent (Curation Pipeline)
- Fetch papers from 4 sources
- Normalize and merge data
- Deduplicate using database memory
- Rank papers using LLM
- Send top papers to Telegram
- Store metadata in database

### 🔹 Workflow 2: Feedback Loop
- Captures user reactions (👍 / 👎)
- Updates preference scores in real-time
- Improves future recommendations

### 🔹 Workflow 3: Weekly Digest
- Aggregates top papers of the week
- Generates AI-written summary report

---

## 🧠 AI Capabilities (Claude as Brain)
- 📊 Paper relevance scoring (1–10)
- ✍️ TL;DR generation (2-line summaries)
- 🧾 Weekly research digest generation
- 🎯 Personalized ranking using user preference memory

---

## 🛠️ Tech Stack
- **Workflow Automation:** n8n  
- **LLM / AI:** Anthropic Claude (claude-opus-4)  
- **Database:** Supabase (PostgreSQL)  
- **Delivery:** Telegram Bot API  
- **Logging:** Google Sheets  
- **APIs:** arXiv, Semantic Scholar, HuggingFace, SerpAPI  

---

## 📂 Project Structure