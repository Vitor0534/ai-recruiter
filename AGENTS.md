# AGENTS.md

Este documento lista e descreve os agentes disponíveis no projeto **ai-recruiter**.

## Agentes

1. **master-recruiter** (orquestrador)
   - Função: interface principal com o usuário, coleta informações, delega tarefas a agentes especializados e consolida resultados.
   - Localização: `personas/master-recruiter.md`
   - Skills necessárias: `skills/dispatch.md` (despacho de agentes) e `skills/job-search.md` (para encaminhar buscas).

2. **scout** (busca de vagas)
   - Função: buscar vagas de tecnologia usando o CLI **firecrawl**, analisar requisitos e comparar com o perfil do usuário.
   - Localização: `personas/scout.md`
   - Skills necessárias: `skills/job-search.md` e `skills/firecrawl.md`.

3. **curator** (busca de cursos)
   - Função: buscar cursos online usando o CLI **firecrawl** para desenvolver habilidades faltantes identificadas nas vagas.
   - Localização: `personas/curator.md`
   - Skills necessárias: `skills/course-search.md` e `skills/firecrawl.md`.

4. **buscador-artigo** (recomendação de artigos)
   - Função: identificar assuntos que o usuário não domina nas vagas e recomendar artigos técnicos ou tutoriais de leitura.
   - Localização: `personas/buscador-artigo.md`
   - Skills necessárias: `skills/article-recommendation.md` e `skills/firecrawl.md`.

5. **coach** (simulação de entrevistas) – implementado.

6. **training‑coach** (orientação de estudos) – novo agente para criar planos de estudo e acompanhar progresso.

7. **filtrador-artigo** (avaliação e resumo de artigos)
   - Função: avaliar os artigos retornados pelo `buscador-artigo`, verificar disponibilidade dos links, apresentar ao usuário para seleção e gerar resumos rápidos ou passo a passos detalhados.
   - Localização: `personas/filtrador-artigo.md`
   - Skills necessárias: `skills/article-filter.md` e `skills/firecrawl.md`.

## Como adicionar novos agentes
- Criar um arquivo de persona em `personas/` descrevendo papel, ferramentas e formato de resposta.
- Criar as skills correspondentes em `skills/` que definem o fluxo de trabalho.
- Atualizar este arquivo listando o novo agente e suas dependências.
