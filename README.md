# Lead Search Plugin for Claude Code

AI-powered outbound lead search agent. Describe your Ideal Customer Profile in natural language, get ranked leads from LinkedIn with personalized outreach angles.

## What It Does

The `/search-leads` skill turns a natural language ICP description into a ranked list of qualified leads:

1. **Parses your ICP** - accepts any format (natural language, structured fields, product description, or even "find leads FOR someone")
2. **Runs parallel LinkedIn searches** - multiple title variations for better coverage
3. **Enriches top candidates** - full profiles, company data, trigger events
4. **Scores and ranks** - 1-10 relevance scale based on title fit, company fit, timing signals, and accessibility
5. **Generates outreach angles** - personalized opener for each top lead

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
```

## MCP Tools

Works best with these MCP servers (optional - falls back to built-in WebSearch):

| MCP Server | What It Provides |
|------------|-----------------|
| **Anysite MCP** | LinkedIn search, profile enrichment, company data, DuckDuckGo |
| **Firecrawl MCP** | Website scraping, structured extraction, web search |

## Output

Returns a structured report with:
- Top 10 leads scored 1-10 with reasoning
- Company details, trigger events, location
- Personalized outreach angle for each lead
- Search summary and refinement suggestions
- Outreach priority ranking

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

## License

MIT - [Onsa AI](https://onsa.ai)
