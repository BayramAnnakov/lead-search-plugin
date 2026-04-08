---
name: search-leads
description: Find and rank outbound leads matching your ICP using LinkedIn and web research. Supports direct ICP, inverse ICP (find leads FOR someone), product-based search, and "find similar" patterns. Returns scored results with personalized outreach angles.
license: MIT
compatibility: Works best with Anysite MCP (LinkedIn data) or Firecrawl MCP (website intelligence). Falls back to built-in WebSearch if no MCP tools available.
metadata:
  author: Onsa AI
  version: "1.0.0"
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

**Primary search tool:**
```
search_linkedin_users:
  keywords: [derived from ICP]
  title: [target titles]
  location: [target geography]
  count: 10-25 per search
```

**Search strategy:**
1. **Start specific** - use exact title + industry + location
2. **Expand if needed** - broaden title variations or location
3. **Run parallel searches** for different title variations:
   - Search 1: Primary title (e.g., "VP Sales")
   - Search 2: Alternative title (e.g., "Head of Revenue")
   - Search 3: Adjacent title (e.g., "CRO", "Director of Sales")

**Supplementary searches:**
```
# Find companies first, then people at those companies
search_linkedin_companies: keywords, size, industry
  -> Then search_linkedin_users at specific companies

# Web search for trigger events
duckduckgo_search: "[industry] Series A 2026 [location]"
duckduckgo_search: "[industry] hiring sales team"
```

### Step 4: Enrich Top Candidates

For the top 10-15 results from search:

```
# Get full profile details
get_linkedin_profile: Full background, experience, skills, education

# Get company details
get_linkedin_company: Company size, industry, description, specialties

# Check for trigger events
get_linkedin_company_posts: Recent announcements, product launches
duckduckgo_search: "[Company] funding hiring news 2026"
```

**Run enrichment calls in parallel** where independent (e.g., profile + company for the same lead can be parallel).

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

### MCP Tools Priority

**1. Anysite MCP** (primary - structured LinkedIn data):
```
search_linkedin_users keywords="AI startup founder" title="CEO" location="Seattle" count=20
  -> Returns: name, title, company, location, profile URL

get_linkedin_profile profile_url="linkedin.com/in/john-doe"
  -> Returns: full background, experience, education, skills, followers

get_linkedin_company company_url="linkedin.com/company/acme"
  -> Returns: size, industry, HQ, description, specialties

get_linkedin_company_posts company_url="..." count=5
  -> Returns: recent posts, announcements

get_linkedin_company_employee_stats company_url="..."
  -> Returns: employee count, growth rate, department breakdown

search_linkedin_companies keywords="B2B SaaS" count=10
  -> Returns: company name, size, industry, URL

duckduckgo_search query="Acme Corp Series A funding 2026"
  -> Returns: search results with URLs and snippets
```

**2. Firecrawl MCP** (website intelligence):
```
firecrawl_scrape url="https://company.com/about"
  -> Returns: page content (team size, product, market)

firecrawl_extract url="https://company.com" prompt="Extract company size, product, and target market"
  -> Returns: LLM-extracted structured data

firecrawl_search query="[Company] funding announcement"
  -> Returns: web search results with content
```

**3. Built-in Web Search** (always available fallback):
```
WebSearch query="[Company] news funding 2026"
```

### Search Workflow

1. Parse ICP into search parameters
2. **Anysite `search_linkedin_users`** (2-3 parallel searches with title variations)
3. Deduplicate results across searches
4. **Anysite `get_linkedin_profile`** + `get_linkedin_company` (parallel enrichment for top 10-15)
5. **Anysite `duckduckgo_search`** for trigger events on top companies
6. Score, rank, and generate outreach angles
7. Present structured results

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

---

## Search Request:
$ARGUMENTS
