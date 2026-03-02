# hiring-manager-finder 🔍 — Deep research via distributed workers — yes, one runs on a laptop in France.

<p align="center">
  <img src="assets/hero.png" alt="hiring-manager-finder hero" width="1100">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="Node.js">
  <img src="https://img.shields.io/badge/Puppeteer-40B5A4?style=for-the-badge&logo=puppeteer&logoColor=white" alt="Puppeteer">
  <img src="https://img.shields.io/badge/Claude-191919?style=for-the-badge&logo=anthropic&logoColor=white" alt="Claude">
</p>

A distributed research system that identifies hiring managers for target companies. Workers run on multiple machines (including a laptop in France via SSH) to parallelize LinkedIn and company page research. Uses LLM to match titles, departments, and reporting structures.

> **This is a closed-source project.** The README documents the architecture and learnings.

## What it does

- Identifies likely hiring managers for PM roles at target companies
- Distributes research across multiple workers (California server + France laptop)
- Searches LinkedIn, company about pages, and press releases
- Uses LLM to score relevance: title match, department alignment, seniority level
- Deduplicates across sources and merges into CRM
- SSH tunnel to France worker for geo-distributed rate limit avoidance

## How it works

```
                    Coordinator
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
   ┌─────────────────┐    ┌─────────────────┐
   │  US Worker       │    │  France Worker   │
   │  (Lightsail)     │    │  (macOS laptop)  │
   │  Chrome/Puppeteer│    │  Safari via SSH  │
   └────────┬────────┘    └────────┬────────┘
            │                       │
            ▼                       ▼
     Browse Company           Browse LinkedIn
     Pages + LinkedIn         + Press Releases
            │                       │
            └───────────┬───────────┘
                        ▼
                Extract People Data
                        │
                        ▼
               ┌─────────────────┐
               │  Claude Scoring  │
               │  - Title match   │
               │  - Department    │
               │  - Seniority     │
               └────────┬────────┘
                        │
                        ▼
                   PostgreSQL
                 (deduplicate)
                        │
                        ▼
                   Sync to CRM
```

## Tech stack

- **Runtime:** Node.js workers on Lightsail + macOS (France)
- **AI:** Claude for relevance scoring and title matching
- **Browser:** Puppeteer with Safari (France) and Chrome (US)
- **Database:** PostgreSQL (RDS)
- **Networking:** SSH tunnels, distributed task queue

## What I learned

- **Geo-distributing workers isn't just about scale — it's about appearing as different users from different locations.** Rate limits and bot detection are IP and region-aware.
- **Safari on macOS via SSH in France was surprisingly stable** as a headless-ish browser for research. Lower bot detection rate than headless Chrome.
- **LLM relevance scoring eliminated 80% of false positives** that keyword matching alone would have included. "VP of Product Marketing" is not a hiring manager for a PM role — the LLM knows that.
