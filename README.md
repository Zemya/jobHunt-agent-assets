# N8N Resume Agent

Automated job search pipeline for a Canadian Computer Science graduate currently living in China.  
The system pulls the resume, searches free remote job boards + ATS sites via SerpApi, scores matches with an LLM, generates tailored cover letters, and logs high-potential roles to Google Sheets.

---

## Project Goal

Find **entry to early mid-level** roles in:
- Data Analyst / Junior Data Analyst / BI Analyst
- Junior Data Engineer / Analytics Engineer
- Junior Software Developer / Programmer (Python / Java)

**Constraints:**
- Canadian citizen → payment must work with a Canadian bank
- Currently based in China
- Willing to relocate to Canada for the right role
- Prefer free-to-apply positions
- Limited SerpApi quota (250 searches/month)

---

## Current Architecture
→ Schedule Trigger
→ Download resume from GitHub raw
→ Extract text from PDF
→ LLM Resume Analysis
→ Parallel job sources
├── Remotive API
├── Himalayas (multiple targeted searches)
└── SerpApi (Bing) – Canadian ATS queries
→ Merge
→ Basic salary / seniority filter
→ LLM Scoring + Cover Letter generation
→ Keep only match_score ≥ 80
→ Clean + detect email
→ Append to Google Sheets


---

## Evolution & Fine-Tuning Stages

### Stage 1 – Original (Failed)
- Adzuna API + Google Drive resume + Google Sheets + Gemini
- **Problem:** Adzuna geo-blocked from China (“Not available in your region”)
- **Result:** Abandoned Adzuna completely

### Stage 2 – Free Remote Boards
- Switched to Remotive + Himalayas + (temporarily) We Work Remotely & RemoteOK
- Moved resume to public GitHub raw URL (removed Google Drive dependency)
- Built normalization Code nodes to a common schema
- **Problem:** Many listings required paid applications or had hard location restrictions
- Removed paid-wall sources

### Stage 3 – ATS Scraping Attempts
- Tried direct HTTP requests to Greenhouse, Lever, Ashby career pages
- Tried “hidden” Greenhouse API endpoints
- **Result:** 404s, bot blocking, JavaScript-rendered pages → failed

### Stage 4 – Google Programmable Search
- Created Custom Search Engine with site: restrictions
- **Result:** Google Custom Search JSON API closed to new customers

### Stage 5 – Azure Bing Search
- Attempted classic Bing Search API and newer “Grounding with Bing”
- **Result:** Classic free tier no longer available; Grounding is $14 / 1k transactions (too expensive)

### Stage 6 – SerpApi + Bing (Current)
- Successfully integrated SerpApi as a proxy to Bing
- Works from China without VPN
- Free tier = 250 searches/month
- Queries focused on Canadian ATS domains (Greenhouse, Lever, Ashby)

### Stage 7 – Scoring & Filtering Refinement
- Iteratively tuned LLM scoring prompt for:
  - Entry-level realism
  - Canadian / remote preference
  - Avoiding over-penalizing lack of experience
- Lowered threshold from 85 → 80
- Fixed multiple pairing / field-loss bugs between Scoring and downstream nodes
- Fixed Google Sheets from “Append or Update” (was overwriting) → pure Append

---

## Key Lessons Learned

| Approach                      | Outcome                          | Reason |
|-------------------------------|----------------------------------|------|
| Adzuna                        | Failed                           | Geo-blocked in China |
| Direct ATS page scraping      | Failed                           | JS rendering + bot protection |
| Google Custom Search          | Failed                           | API closed to new users |
| Azure Bing (classic)          | Failed                           | Free tier retired |
| Azure Grounding with Bing     | Too expensive                    | $14 / 1k |
| SerpApi + Bing                | Working (with limits)            | Best free option that works from China |
| Complex multi-site queries    | Poor results                     | Bing ignores complex `site:A OR site:B` |
| Simple single-site queries    | Better results                   | Recommended |

---

## Current Open Issues

1. SerpApi still returns mixed quality – further query simplification needed.
2. Free boards (Himalayas + Remotive) produce very few true Canadian roles.
3. Occasional LLM insertion of test phrases from job descriptions into cover letters (mitigated in prompt).
4. Manual runs only (home lab + VPN). Not yet suitable for unattended server in China.

---

## Tech Stack

- **Orchestration:** n8n (self-hosted in Docker)
- **Resume storage:** GitHub raw
- **Job sources:** Remotive, Himalayas, SerpApi (Bing)
- **LLM:** Google Gemini (scoring + cover letters)
- **Storage:** Google Sheets
- **Future candidate:** DeepSeek (China-friendly alternative to Gemini)
---

## How to Run

1. Start the n8n Docker container on the home lab machine (VPN on).
2. Open the n8n UI and execute the workflow manually.
3. High-scoring jobs (≥ 80) are appended to the tracking Google Sheet with cover letters ready.

---

## License / Notes

Personal automation project.  
Not intended for commercial redistribution of job data.  
Respect robots.txt and terms of service of all job boards and APIs used.
