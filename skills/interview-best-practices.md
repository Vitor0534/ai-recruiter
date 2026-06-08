# Interview Best Practices Skill

## Purpose
Collect up‑to‑date best practices for technical interview preparation using Firecrawl.

## Steps
1. **Search** – run `firecrawl search "interview best practices tech" --json`.
2. **Extract** – from each result keep the title, source URL and a short summary (max 2 sentences).
3. **Aggregate** – create a numbered list of the top 5 most relevant items.
4. **Output** – return a markdown block with the list.

## Expected Output Format
```
1. <title> – <summary> (source: <url>)
2. ...
```

## Error Handling
If the `firecrawl` command fails, capture the error message in an `erros` array and abort the skill.
