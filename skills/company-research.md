# Company Research Skill

## Purpose
Gather contextual information about a target company for interview simulations using Firecrawl.

## Steps
1. **Identify Company Name** – read the selected job entry from `data/job-search-results.md` to obtain the company name.
2. **Search** – run `firecrawl search "<company name> interview experience" --json` and also `firecrawl search "<company name> culture" --json`.
3. **Scrape** – for the top 2 results of each search, execute `firecrawl scrape <url> --format markdown` to obtain readable content.
4. **Extract Topics** – from each scraped page pull up to three key points about:
   - Company culture & values
   - Typical interview process & stages
   - Recent tech projects or products relevant to the role.
5. **Aggregate** – produce a numbered list of the extracted points.

## Expected Output Format
```
1. <topic> – <brief description>
2. ...
```

## Error Handling
If any `firecrawl` command fails, record the error message in an `erros` array and abort the skill.
