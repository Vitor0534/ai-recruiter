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

4. **coach** (simulação de entrevistas) – implementado.

5. **training‑coach** (orientação de estudos) – novo agente para criar planos de estudo e acompanhar progresso.

## Como adicionar novos agentes
- Criar um arquivo de persona em `personas/` descrevendo papel, ferramentas e formato de resposta.
- Criar as skills correspondentes em `skills/` que definem o fluxo de trabalho.
- Atualizar este arquivo listando o novo agente e suas dependências.
