# Lead Search Plugin for Claude Code

AI-powered outbound lead search agent. Describe your Ideal Customer Profile in natural language, get ranked leads from LinkedIn, Crunchbase, and Y Combinator with personalized outreach angles.

## What It Does

The `/search-leads` skill turns a natural language ICP description into a ranked list of qualified leads:

1. **Parses your ICP** - accepts any format (natural language, structured fields, product description, inverse ICP, or "find more like these customers")
2. **Classifies query complexity** - simple queries go straight to LinkedIn; compound queries (niche vertical + funding stage + gender + location) get decomposed into sequential search-then-filter steps
3. **Searches multiple sources in parallel** - LinkedIn profiles, Crunchbase funding data, YC startup directory, DuckDuckGo for news
4. **Enriches top candidates** - full profiles, company data, funding history, trigger events
5. **Scores and ranks** - 1-10 relevance scale based on title fit, company fit, timing signals, and accessibility
6. **Generates outreach angles** - personalized opener for each top lead

## Installation

```bash
# Add the marketplace
/plugin marketplace add BayramAnnakov/lead-search-plugin

# Install the plugin
/plugin install lead-search@lead-search-marketplace
```

## Usage

```bash
# Direct ICP
/search-leads Find VP of Sales at B2B SaaS companies with 50-200 employees in the US

# Inverse ICP (find leads FOR someone)
/search-leads Find leads for Alan M., patent attorney specializing in AI/ML and VR/AR

# Product-based
/search-leads We sell sales automation tools to mid-market B2B. Find 10 prospects.

# Similar to existing customers
/search-leads Our best customers are GrowthTech (85 emp, SaaS) and DataInc (120 emp, Analytics). Find more like them.

# Compound query (niche vertical + multiple constraints)
/search-leads Find female founders in Voice AI space in the Bay Area who raised funding in the last 12 months
```

## Data Sources

Works with two Anysite MCP server variants, plus optional Firecrawl. Falls back to built-in WebSearch when no MCP tools are available.

| Source | What It Provides | Used For |
|--------|-----------------|----------|
| **LinkedIn** (Anysite) | Profile search, company data, employee stats, posts | Finding people, enriching profiles, company details |
| **Crunchbase** (Anysite) | Funding rounds, investors, employee counts, categories | Funding verification, company discovery by stage |
| **Y Combinator** (Anysite) | Startup directory, founder search, batch/industry filters | Finding funded startups, filtering by hiring status |
| **SEC EDGAR** (Anysite) | Public company filings (10-K, 10-Q) | Enterprise lead enrichment |
| **DuckDuckGo** (Anysite) | Web search | Trigger events, funding news, fallback for everything |
| **Firecrawl** (optional) | Website scraping, structured extraction | Company website intelligence |
| **WebSearch** (built-in) | Basic web search | Always-available fallback |

## Search Patterns

| Pattern | Example | Strategy |
|---------|---------|----------|
| **Direct ICP** | "VP Sales at B2B SaaS, 50-200 emp, US" | LinkedIn people search with title variations |
| **Inverse ICP** | "Find leads FOR a patent attorney" | Research the person, flip to "who needs them?" |
| **Product-based** | "We sell X to Y. Find prospects." | Infer ICP from product description |
| **Find similar** | "More companies like these 3 customers" | Extract pattern from examples, search by template |
| **Compound query** | "Female founders in Voice AI, Bay Area, funded" | Decompose: companies first -> people -> filter -> verify |
| **Niche vertical** | "Quantum computing startups" | Company-first search (Crunchbase/DuckDuckGo), then founders |

## Output

Returns a structured report with:
- Top 10 leads scored 1-10 with reasoning
- Company details, trigger events, location
- Personalized outreach angle for each lead
- Search summary and refinement suggestions
- Outreach priority ranking

## Known Limitations

- **Gender/demographics**: No API supports filtering by gender. The skill infers from first names after search - imperfect for ambiguous or non-Western names
- **Niche verticals**: Keywords like "Voice AI" rarely appear in LinkedIn titles. The skill searches for companies first, then finds people at those companies
- **Crunchbase reliability**: Complex multi-parameter queries may fail. The skill falls back to DuckDuckGo for funding verification
- **Funding recency**: Requires Crunchbase data or news search. LinkedIn has no funding information

## ICP Template

Save as `ICP_MODEL.md` in your project root for the skill to auto-load:

```markdown
# ICP

## Target
[TITLE] at [COMPANY TYPE] with [SIZE] employees in [LOCATION]

## Signals
- Recently raised funding
- Hiring in sales/marketing
- Using competitor tools

## Disqualifiers
- Too small (<10 employees)
- Wrong geography
- Already a customer
```

## Version History

- **v1.1.0** - Compound query decomposition, Crunchbase/YC/SEC data sources, fallback chains, known limitations
- **v1.0.0** - Initial release: LinkedIn search, scoring, outreach angles

## License

MIT - [Onsa AI](https://onsa.ai)
