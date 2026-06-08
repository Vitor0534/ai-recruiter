# Interview Simulation Skill

## Purpose
Generate interview questions, conduct a simulated interview session, evaluate user answers, and produce feedback.

## Inputs
- `type`: "especializada" or "genérica" (provided by Coach after user choice).
- `job`: job description object extracted from `data/job-search-results.md`.
- `profile`: user profile from `data/user-profile.md`.
- `best_practices`: list from `interview-best-practices.md`.
- `company_info` (optional): list from `company-research.md` when type is "especializada".

## Steps
1. **Select Technical Questions** – based on required skills in `job`, pick 3‑5 technical questions (hard‑coded examples for demo).
2. **Add Behavioral Questions** – take first 2 items from `best_practices` and turn each into a behavioral question.
3. **Add Company‑Specific Questions** – if `company_info` present, create 1‑2 questions referencing culture or recent projects.
4. **Run Simulation** – for each question:
   - Present the question (output to user).
   - Receive user's answer (assume Coach captures it).
   - Compare answer length and presence of key terms; generate brief feedback.
5. **Summarize** – compile overall strengths and improvement points.
6. **Persist** – write a markdown summary to `data/interview-simulation.md` with date, type, and feedback.

## Output
Return a JSON‑compatible envelope (markdown block) with:
```
estado: concluído
resumo: "..."
dados: data/interview-simulation.md
erros: []
```

## Error Handling
If any required input is missing, add a message to `erros` and set `estado` to `erro`.
