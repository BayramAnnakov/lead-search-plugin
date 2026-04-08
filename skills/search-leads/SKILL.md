---
name: search-leads
description: Find and rank outbound leads matching your ICP using LinkedIn and web research. Supports direct ICP, inverse ICP (find leads FOR someone), product-based search, and "find similar" patterns. Returns scored results with personalized outreach angles.
license: MIT
compatibility: Works best with Anysite MCP (LinkedIn data) or Firecrawl MCP (website intelligence). Falls back to built-in WebSearch if no MCP tools available.
metadata:
  author: Onsa AI
  version: "1.1.0"
---

# Outbound Lead Search Agent

Find qualified leads matching your Ideal Customer Profile (ICP) using LinkedIn search, profile enrichment, and web research. Returns a ranked list with relevance reasoning and outreach angles.

## Instructions

### Step 0: Load ICP Context

**FIRST**, check for existing ICP definition:

1. Use Glob to search: `**/ICP_MODEL.md`, `**/ICP_TEMPLATE.md`, `**/ICP.md`
2. If found, read it and use as the base ICP
3. If not found, work with whatever the user provides in `$ARGUMENTS`

---

When given a lead search request, execute this workflow:

### Step 1: Parse the ICP

**Accept ANY format** - the ICP can come as:
- Natural language description ("Find VP of Sales at B2B SaaS companies in the US")
- Structured fields (Title, Industry, Size, Location)
- A person to find leads FOR ("Find leads for Alan M., patent attorney specializing in AI/ML")
- A product description ("We sell sales automation to mid-market B2B")
- A company URL ("Find companies similar to acme.com")
- An existing customer list ("Here are my 5 best customers, find more like them")

**Extract these search parameters** (infer what's missing):

| Parameter | Description | Example |
|-----------|-------------|---------|
| **Target Title(s)** | Job titles to search for | VP Sales, Head of Revenue, CRO |
| **Company Type** | Industry/vertical | B2B SaaS, FinTech, HealthTech |
| **Company Size** | Employee range | 50-200 employees |
| **Location** | Geographic focus | United States, Seattle, EU |
| **Keywords** | Additional search terms | AI, machine learning, Series A |
| **Behavioral Signals** | Trigger events to look for | Recently raised funding, hiring |

**If doing an "Inverse ICP"** (finding leads FOR someone):
1. Research the person's profile and expertise first
2. Define: "Who NEEDS this person's expertise?"
3. Translate that into searchable ICP parameters

If the request is too vague to search (e.g., "find me leads"), ask ONE clarifying question:
> "What do you sell and who typically buys it? One sentence is enough."

### Step 1b: Classify Query Complexity

Before searching, classify the query:

**Simple query** (1-2 constraints) - go straight to LinkedIn search:
> "Find VP of Sales at B2B SaaS in the US"
→ Title + Industry + Location → one LinkedIn search handles it

**Compound query** (3+ constraints, niche vertical, or non-searchable criteria) - decompose first:
> "Find female founders in Voice AI in Bay Area who raised funding recently"
→ 4 constraints: gender + role + niche vertical + geography + funding recency

**Decomposition strategy for compound queries:**

1. **Identify which constraints are API-searchable and which are not:**
   - Searchable: title/role, location, company size, industry, keywords
   - Partially searchable: funding stage (Crunchbase), YC batch (YC search)
   - NOT searchable via API: gender, ethnicity, personality traits

2. **Choose the search anchor** - the most specific searchable constraint:
   - Niche vertical (e.g., "Voice AI") → search companies first, then find people
   - Specific role at known companies → search people directly
   - Funding stage → search Crunchbase first, then find founders on LinkedIn

3. **Plan the search as sequential steps, not one query:**
   - Step A: Find COMPANIES matching the vertical (Crunchbase, DuckDuckGo, YC, LinkedIn company search)
   - Step B: Find PEOPLE at those companies (LinkedIn people search filtered by company)
   - Step C: Apply non-searchable filters manually (check names for gender, review profiles for other criteria)
   - Step D: Verify remaining criteria via enrichment (funding from Crunchbase/news, team size from LinkedIn)

4. **For gender-based queries:** No API supports gender filtering. After finding candidates, infer gender from first names. This is imperfect - acknowledge limitations for ambiguous or non-Western names. Cast a wider net and filter down rather than trying to pre-filter.

5. **For niche verticals** (Voice AI, quantum computing, synthetic biology, etc.): LinkedIn keyword search often fails because few people put niche terms in their title. Instead:
   - Search DuckDuckGo: "[vertical] startups [location] 2025 2026" to find company names
   - Search Crunchbase by keywords for that vertical
   - Search YC companies by industry if applicable
   - Then find founders/leaders at those specific companies on LinkedIn

### Step 2: Research Context (if needed)

**Load MCP tools first** using ToolSearch before making calls:
- `+anysite` - loads LinkedIn and DuckDuckGo tools
- `+firecrawl` - loads scrape/extract tools

**If user provided a company URL or person to research:**

Use MCP tools to gather context:

```
# Research a person (for Inverse ICP)
get_linkedin_profile: Background, expertise, specializations
get_linkedin_user_posts: Recent topics, thought leadership

# Research a company (for "find similar" requests)
get_linkedin_company: Size, industry, description
firecrawl_scrape: Website content, product positioning
```

**Extract from research:**
- Core expertise domains / product categories
- Target audience signals
- Geographic patterns
- Company size patterns
- Industry vertical patterns

### Step 3: Execute Search

**Run searches using AnySite MCP tools.** Make multiple parallel searches for better coverage.

**CRITICAL: Maximize parallel tool calls.** The biggest speed gain comes from batching independent calls in a single message. Do NOT call tools one at a time when they don't depend on each other.

**Parallelism rules:**
- **Batch together** (same message): calls that don't depend on each other's results
- **Wait for results** (next message): calls that need data from previous calls
- **Never sequentialize** independent searches - 3 parallel searches take the same time as 1

**Phase 1 - Discovery (all parallel in one message):**
```
# Simple query: batch 2-3 LinkedIn title variations
search_linkedin_users(title="VP Sales", location="US", count=15)       # ← parallel
search_linkedin_users(title="Head of Revenue", location="US", count=15) # ← parallel
search_linkedin_users(title="CRO", location="US", count=15)            # ← parallel

# Compound query: batch across sources
search_linkedin_users(keywords="voice AI", title="Founder", ...)  # ← parallel
duckduckgo_search("voice AI startups Bay Area funded 2025 2026")  # ← parallel
crunchbase db_search(keywords="voice AI", location="SF", ...)     # ← parallel
search_yc_companies(industries=["B2B"], count=10)                 # ← parallel
```

**Phase 2 - People at companies (depends on Phase 1, but internal calls parallel):**
```
# After Phase 1 returns company names, search for founders at each - all parallel
search_linkedin_users(company_keywords="Rime", title="Founder", count=5)   # ← parallel
search_linkedin_users(company_keywords="Deepgram", title="CEO", count=5)   # ← parallel
search_linkedin_users(company_keywords="ElevenLabs", title="CTO", count=5) # ← parallel
```

**Phase 3 - Enrichment (depends on Phase 2, but internal calls parallel):**
```
# Enrich top 10 candidates - all parallel
get_linkedin_profile(user="lilyjclifford")   # ← parallel
get_linkedin_profile(user="john-doe")         # ← parallel
get_linkedin_profile(user="jane-smith")       # ← parallel
get_linkedin_company(company="rime-ai")       # ← parallel
get_linkedin_company(company="deepgram")      # ← parallel
duckduckgo_search("Rime Labs funding 2025")   # ← parallel
duckduckgo_search("Deepgram Series B 2025")   # ← parallel
```

**In total: 3 sequential phases, but within each phase everything runs in parallel.**
A 10-lead search with enrichment should take ~3 rounds of tool calls, not 30+.

**Supplementary search patterns:**
```
# Find companies first, then people at those companies
search_linkedin_companies: keywords, size, industry
  -> Then search_linkedin_users at specific companies

# Web search for trigger events
duckduckgo_search: "[industry] Series A 2026 [location]"
duckduckgo_search: "[industry] hiring sales team"
```

### Step 4: Enrich Top Candidates

For the top 10-15 results from search. **This is the slowest phase - parallelism matters most here.**

**Batch ALL enrichment calls in one message** (they're all independent):

```
# All of these go in ONE message - they run in parallel:
get_linkedin_profile(user="candidate-1")          # ← parallel
get_linkedin_profile(user="candidate-2")          # ← parallel
get_linkedin_profile(user="candidate-3")          # ← parallel
get_linkedin_company(company="company-1")         # ← parallel
get_linkedin_company(company="company-2")         # ← parallel
duckduckgo_search("Company-1 funding news 2026")  # ← parallel
duckduckgo_search("Company-2 funding news 2026")  # ← parallel
```

If enriching 10+ leads, split into 2 batches of 5-7 calls each (some MCP servers throttle very large batches).

**What to extract per lead:**
- Current role and tenure
- Company size (exact or estimate)
- Industry vertical
- Recent company news / trigger events
- Relevant experience or expertise
- Content engagement (what they post/share about)
- Mutual connections or shared communities
- Tech stack signals (from company or job postings)

### Step 5: Score and Rank

Score each lead on a **1-10 relevance scale** based on ICP match:

| Score | Meaning | Criteria |
|-------|---------|----------|
| 9-10 | Perfect match | All ICP criteria match + strong trigger event |
| 7-8 | Strong match | Most criteria match, minor gaps |
| 5-6 | Moderate match | Core criteria match, some unknowns |
| 3-4 | Weak match | Partial match, significant gaps |
| 1-2 | Poor match | Minimal alignment |

**Scoring dimensions:**
1. **Title fit** (0-3): How closely does their role match the target?
2. **Company fit** (0-3): Size, industry, stage alignment?
3. **Timing signals** (0-2): Recent triggers (funding, hiring, role change)?
4. **Accessibility** (0-2): Active on LinkedIn? Open to outreach?

**Rank by score, break ties with:**
- Stronger trigger events first
- More accessible profiles first (active posters > lurkers)
- Better geographic match first

### Step 6: Generate Outreach Angles

For each top lead (score 7+), identify a **personalized outreach angle**:

1. **Shared connection**: Mutual contacts, same alma mater, same previous company
2. **Content hook**: Something they posted/shared that connects to your value prop
3. **Trigger hook**: Recent company event (funding, launch, hiring) that creates urgency
4. **Pain hypothesis**: Inferred pain point based on role + company stage + industry
5. **Compliment hook**: Genuine observation about their work, company, or content

**Draft a 2-3 sentence outreach opener** for the top 5 leads:
```
[Personalized observation about them/their company].
[Pain hypothesis or value connection].
[Soft question or CTA].
```

### Step 7: Present Output

Structure results as follows:

```markdown
## Lead Search Results

**ICP:** [One-line summary of what was searched]
**Searched:** [Date/time]
**Results:** [X] leads found, [Y] enriched, [Z] scored 7+

---

### Top Leads

#### 1. [Name] - [Title] at [Company] | Score: [X]/10

| Attribute | Detail |
|-----------|--------|
| **Company** | [Name] - [size] employees, [industry] |
| **Location** | [City, Country] |
| **LinkedIn** | [URL] |
| **Why they match** | [1-2 sentences on ICP alignment] |
| **Trigger event** | [Recent news/change or "None identified"] |
| **Outreach angle** | [Personalized opener suggestion] |

---

#### 2. [Name] - [Title] at [Company] | Score: [X]/10
...

---

### Search Summary

| Metric | Value |
|--------|-------|
| Searches executed | [X] |
| Profiles reviewed | [X] |
| Leads scoring 7+ | [X] |
| Leads scoring 5-6 | [X] |
| Average score | [X.X]/10 |

### Refinement Suggestions

Based on these results:
- **Narrow if too many:** [Suggestion to add a filter]
- **Broaden if too few:** [Suggestion to relax a constraint]
- **Adjacent search:** [Related ICP variation worth exploring]

### Outreach Priority

1. [Name] - [One-line reason to contact first]
2. [Name] - [One-line reason]
3. [Name] - [One-line reason]
```

---

## Tool Usage

The skill works with two Anysite MCP server variants. Check which is available and use accordingly:

### Variant A: Anysite Remote (meta-tools)

If `discover`/`execute` tools are available (`anysite-remote`), use the meta-tool workflow:

```
# Step 1: Discover endpoints (do this once per source)
discover(source="linkedin", category="search")
discover(source="linkedin", category="user")
discover(source="linkedin", category="company")
discover(source="crunchbase", category="db")
discover(source="yc", category="search")
discover(source="sec", category="search")

# Step 2: Execute searches
execute(source="linkedin", category="search", endpoint="sn_search_users",
  params={keywords: "AI", current_titles: ["CEO"], location: "Seattle",
          company_sizes: ["11-50","51-200"], count: 10})

execute(source="crunchbase", category="db", endpoint="db_search",
  params={keywords: "AI", location: "Seattle",
          employee_count_min: 11, employee_count_max: 250,
          last_funding_type: ["seed","series_a","series_b"], count: 10})

execute(source="yc", category="search", endpoint="search_companies",
  params={industries: ["B2B"], is_hiring: true, count: 10})

# Step 3: Enrich
execute(source="linkedin", category="user", endpoint="get_profile",
  params={user: "john-doe"})
execute(source="linkedin", category="company", endpoint="get_company",
  params={company: "acme"})

# Step 4: Paginate if needed
get_page(cache_key="<from execute result>", offset=10, limit=10)
```

**Crunchbase** is especially valuable for lead search because it provides:
- Funding history (round type, amount, date, investors)
- Employee count ranges
- Company categories and operating status
- Revenue ranges (when available)
- Trigger events: leadership hires, acquisitions, layoffs, news

**Crunchbase reliability note:** Complex multi-parameter queries (location + keywords + funding filters combined) may return 500 errors. If this happens:
- Simplify the query: use fewer filters, try keywords alone first
- Use `query_cache` to filter results after a simpler fetch succeeds
- Fall back to DuckDuckGo: `"[company] series A funding 2025 2026"` for funding verification

**Y Combinator** search is great for finding funded startups:
- Filter by batch (e.g., "Winter 2026"), industry, team size, hiring status
- Get founder details directly via `search_founders`

**SEC EDGAR** provides public company filings - useful for enterprise lead enrichment.

### Variant B: Anysite MCP (direct tools)

If individual named tools are available (`anysite-mcp`), use them directly:

```
# LinkedIn search
search_linkedin_users: keywords, title, location, count
linkedin_sn_search_users: keywords, current_titles, location, company_sizes, count
get_linkedin_profile: user alias or URL
get_linkedin_company: company alias or URL
get_linkedin_company_posts: company URN, count
get_linkedin_company_employee_stats: company URN

# Company search
search_linkedin_companies: keywords, count

# YC data
search_yc_companies: query, industries, batches, team_size_min/max, is_hiring, count
search_yc_founders: query, industries, batches, titles, count
get_yc_company: company slug

# SEC filings
search_sec_companies: entity_name, forms, date_from, date_to, count

# Web search
duckduckgo_search: query, count
```

### Firecrawl MCP (website intelligence, optional)

```
firecrawl_scrape url="https://company.com/about"
  -> Returns: page content (team size, product, market)

firecrawl_extract url="https://company.com" prompt="Extract company size, product, and target market"
  -> Returns: LLM-extracted structured data
```

### Built-in Web Search (always available fallback)

```
WebSearch query="[Company] news funding 2026"
```

### Recommended Search Workflow

**For simple queries (1-2 constraints):**
1. Parse ICP into search parameters
2. **LinkedIn search** (2-3 parallel searches with title variations)
3. **Enrich** top 10-15: profiles + company data
4. **DuckDuckGo** for trigger events
5. Score, rank, present

**For compound queries (3+ constraints or niche verticals):**
1. Parse ICP, identify searchable vs non-searchable constraints
2. **Find companies first** (parallel):
   - Crunchbase: by vertical keywords + location + funding stage
   - DuckDuckGo: "[vertical] startups [location] funded 2025 2026"
   - YC: by industry + batch (if targeting startups)
   - LinkedIn company search: by keywords
3. **Find people at those companies** (LinkedIn people search with company filter)
4. **Apply non-searchable filters** (gender from names, other criteria from profiles)
5. **Enrich** survivors: full profiles + company details + funding verification
6. Score, rank, present

### Fallback Chains

Data sources fail. Use these fallbacks:

| Need | Primary | Fallback 1 | Fallback 2 |
|------|---------|------------|------------|
| Company funding data | Crunchbase `db_search` | DuckDuckGo `"[company] funding raised"` | WebSearch |
| Company details | LinkedIn `get_company` | Crunchbase `company` | Firecrawl on website |
| Person profiles | LinkedIn `get_profile` | DuckDuckGo `"[name] [company] LinkedIn"` | WebSearch |
| Startup discovery | YC `search_companies` | Crunchbase `db_search` | DuckDuckGo |
| Trigger events | LinkedIn company posts | DuckDuckGo `"[company] news 2026"` | WebSearch |

When a source returns an error, do NOT retry the same query. Simplify params or switch to the fallback.

---

## Edge Cases

**Too few results (<5):**
- Broaden location (city -> country)
- Add title variations ("VP Sales" -> also "Director of Sales", "Head of Sales")
- Relax company size constraints
- Try searching companies first, then people

**Too many results (>50):**
- Add industry filters
- Narrow geography
- Add behavioral signals (recently posted, changed jobs)
- Increase minimum company size

**Inverse ICP (finding leads FOR someone):**
- Research the person's expertise first
- Ask: "Who has the problem this person solves?"
- Translate expertise -> pain points -> titles + industries that have those pains
- Example: Patent attorney for AI -> search for AI startup founders who need IP protection

**"Find more like X" requests:**
- Research the reference company/person
- Extract: size, industry, stage, geography, title
- Use as search template
- Note what made X a good customer and weight those factors

**No MCP tools available:**
- Use built-in WebSearch as fallback
- Search: "[title] [industry] [location] LinkedIn"
- Acknowledge reduced data quality in output

---

## Known Limitations

Be transparent about these with the user:

**Non-searchable criteria:**
- **Gender** - no API supports this. Infer from first names after search. Acknowledge this is imperfect for ambiguous or non-Western names. Cast a wider net and filter down.
- **Ethnicity, age, personality** - not available. Don't attempt to infer.
- **"Culture fit" or subjective traits** - can't be searched. Suggest the user review profiles manually.

**Niche verticals:**
- Keywords like "Voice AI", "synthetic biology", or "quantum computing" rarely appear in LinkedIn titles. Search for companies in the space first (via DuckDuckGo/Crunchbase), then find people at those companies.

**Funding recency:**
- Crunchbase has funding dates but complex queries may fail. Always have a DuckDuckGo fallback: `"[company name] funding raised 2025 2026"`.
- LinkedIn has no funding data at all.

**Search result quality:**
- LinkedIn keyword search is fuzzy - "voice AI founder" may return people who are AI founders with "voice" somewhere in their profile, not necessarily in voice AI.
- Title search is exact word match - "Founder" won't match "Co-Founder". Always search multiple title variations.
- Company-first search (find companies, then people) is more precise for niche verticals than people-first search.

**When results are thin (fewer than 5 quality matches):**
Tell the user honestly. Suggest:
1. Broadening one constraint (location, company size, or vertical)
2. Trying adjacent verticals (e.g., "conversational AI" instead of "voice AI")
3. Using DuckDuckGo to find curated lists or directories in the space

---

## Examples

### Example 1: Direct ICP
```
/search-leads
Find VP of Sales at B2B SaaS companies with 50-200 employees in the US
who recently raised Series A or B funding
```

### Example 2: Inverse ICP
```
/search-leads
Find leads for Alan M., partner at FisherBroyles specializing in
patent counsel for AI/ML, VR/AR, and NLP companies. He's based in Seattle.
```

### Example 3: Product-Based
```
/search-leads
We sell AI-powered sales automation tools. Our best customers are
mid-market B2B companies (50-200 employees) with growing sales teams.
Find 10 prospects in the US.
```

### Example 4: Similar to Existing Customers
```
/search-leads
Our best customers are:
- GrowthTech (85 employees, SaaS, VP Sales bought)
- DataInc (120 employees, Analytics, CRO bought)
- CloudFirst (200 employees, Infrastructure, Head of Revenue bought)
Find 10 more companies like these.
```

### Example 5: Minimal Input
```
/search-leads founders of AI startups in Seattle
```

### Example 6: Compound Query (multiple constraints)
```
/search-leads
Find female founders in Voice AI space in the Bay Area
who raised funding in the last 12 months
```
This is a compound query (niche vertical + gender + location + funding recency).
The agent should:
1. Search for Voice AI companies via DuckDuckGo + Crunchbase
2. Find founders at those companies on LinkedIn
3. Filter by gender using first names
4. Verify funding via Crunchbase or DuckDuckGo news search

---

## Search Request:
$ARGUMENTS
