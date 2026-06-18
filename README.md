# AI‑Recruiter

## Visão geral
Projeto que explora a criação de um **ambiente multi‑skill** usando **agentes (personas)** especializados. O objetivo é automatizar busca de vagas de emprego de acordo com as habilidades do usuário. O agente também é capaz de recomendar cursos online baseado nas especificações que o usuário não atente para as vagas, gerando em conjunto planos de estudo.

Cada agente tem um papel bem‑definido e interage por meio de *skills* reutilizáveis, permitindo que a IA:

1. **Coleta** informações do usuário (quiz de habilidades e preferências).
2. **Busca vagas** de tecnologia usando o CLI **Firecrawl**.
3. **Compara** requisitos das vagas com o perfil do usuário.
4. **Recomenda cursos** para suprir lacunas de competências.
5. (Futuro) **Simula entrevistas** e gera planos de estudo.

## Estrutura do repositório

```
ai-recruiter/
├─ AGENTS.md                 # Lista de agentes e suas dependências
├─ data/
│   ├─ personality-quiz.md   # Perguntas do quiz
│   ├─ user-profile.md       # Respostas do usuário (gerado após o quiz)
│   ├─ job-search-results.md # Últimos resultados do Scout
│   ├─ course-recommendations.md (gerado pelo Curator)
│   ├─ artigos-recomendation.md (gerado pelo Buscador de Artigos)
│   └─ artigos-filtrados.md (gerado pelo Filtrador de Artigos)
├─ personas/
│   ├─ master-recruiter.md   # Orquestrador – interface principal
│   ├─ scout.md              # Busca de vagas (firecrawl)
│   ├─ curator.md            # Busca de cursos (firecrawl)
│   ├─ buscador-artigo.md    # Busca de artigos para temas faltantes
│   ├─ filtrador-artigo.md   # Avaliação e resumo de artigos
│   ├─ coach.md              # Simulação de entrevistas e feedback
│   └─ training-coach.md     # Geração de plano de estudos personalizado
├─ skills/
│   ├─ dispatch.md           # Lógica de envelope e chamada a sub‑agentes
│   ├─ job-search.md         # Fluxo de busca de vagas
│   ├─ course-search.md      # Fluxo de busca de cursos
│   ├─ article-recommendation.md # Fluxo de busca de artigos
│   ├─ article-filter.md     # Fluxo de filtragem e resumo de artigos
│   ├─ training-coach.md     # Fluxo de geração de plano de estudos
│   └─ firecrawl.md          # Wrapper de comandos firecrawl
└─ plano‑*.md                # Documentos de planejamento e requisitos
```

## Agentes (personas)

| Agente | Papel | Arquivo | Skills principais |
|--------|-------|---------|-------------------|
| **master‑recruiter** | Orquestrador. Interage com o usuário, coleta o quiz, exibe menu e despacha sub‑agentes. | `personas/master-recruiter.md` | `skills/dispatch.md`, `skills/job-search.md` |
| **scout** | Busca vagas de emprego, analisa requisitos e gera resumo comparativo. | `personas/scout.md` | `skills/job-search.md`, `skills/firecrawl.md` |
| **curator** | Busca cursos online para suprir habilidades faltantes. | `personas/curator.md` | `skills/course-search.md`, `skills/firecrawl.md` |
| **buscador-artigo** | Busca artigos e conteúdos de leitura para habilidades e temas faltantes. | `personas/buscador-artigo.md` | `skills/article-recommendation.md`, `skills/firecrawl.md` |
| **coach** | Simula entrevistas e gera feedback personalizado. | `personas/coach.md` | `skills/interview-best-practices.md`, `skills/company-research.md`, `skills/interview-simulation.md`, `skills/firecrawl.md` |
| **filtrador-artigo** | Avalia artigos recomendados, verifica disponibilidade e gera resumos ou passo a passos. | `personas/filtrador-artigo.md` | `skills/article-filter.md`, `skills/firecrawl.md` |
| **training‑coach** | Cria planos de estudo personalizados e acompanha o progresso do usuário. | `personas/training-coach.md` | `skills/training-coach.md` |

## Como usar

1. **Clone o repositório**
   ```bash
   git clone <url-do-repo>
   cd ai-recruiter
   ```

2. **Instale e configure o Firecrawl**
    - Instale o CLI (`npm i -g @firecrawl/cli` ou conforme a documentação).
    - Defina a variável de ambiente: `export FIRECRAWL_API_KEY=seu_token`.

3. **Inicie a interação**
    - Abra o arquivo `personas/master-recruiter.md` e peça a uma IA (ChatGPT, Claude, etc.) para **executar** as instruções nele descritas.
    - A IA atuará como o agente *master‑recruiter*: cumprimentará o usuário, verificará/realizará o quiz e mostrará o menu:

      ```
      a) Responder o quiz
      b) Buscar vagas (opção A) → despacha Scout
      c) Buscar cursos (opção B) → despacha Curator
      d) Buscar artigos (opção D) → despacha Buscador-Artigo
      e) Filtrar artigos (opção F) → despacha Filtrador-Artigo
      ```

4. **Escolha a opção desejada**
    - **A – Buscar vagas**: o master‑recruiter cria um *envelope de despacho* e chama `spawn_agent` com a persona `scout`.
    - O Scout usa `skills/job-search.md` → executa `firecrawl search` e `firecrawl scrape`, compara as habilidades do usuário (arquivo `data/user-profile.md`) e devolve até 5 vagas formatadas.
    - O master‑recruiter grava o resultado em `data/job-search-results.md` e o apresenta ao usuário.

    - **B – Buscar cursos**: fluxo análogo com a persona `curator` e a skill `course‑search.md`.

5. **Itere**
    - Após cada operação o agente volta ao menu, permitindo novas consultas ou a atualização do quiz
    - Caso queira verificar como os dados são armazenados ou, apenas testar de primeira, o projeto tem alguns dados mockados como exemplo no ./data

## Dados gerados

- `data/user-profile.md` – Respostas do quiz (área de interesse, localização, nível, habilidades).
- `data/job-search-results.md` – Últimos 5 resultados de vagas, incluindo correspondência de habilidades.
- `data/course-recommendations.md` – Cursos sugeridos (gerado pelo Curator).
- `data/artigos-recomendation.md` – Artigos e materiais de leitura recomendados (gerado pelo Buscador de Artigos).
- `data/artigos-filtrados.md` – Artigos filtrados com resumos ou passo a passos (gerado pelo Filtrador de Artigos).
- `data/interview-simulation.md` – Resultados da simulação de entrevista gerados pelo Coach.
- `data/perfil-do-estudante.md` – Perfil de estudo gerado pelo Training-Coach.
- `data/planos-de-estudo.md` – Plano de estudos gerado pelo Training-Coach.

## Outros agentes disponíveis

- **coach** – Simula entrevistas e gera feedback com base em vagas e perfil do usuário.
- **filtrador-artigo** – Avalia artigos recomendados, verifica disponibilidade e gera resumos ou passo a passos.
- **training‑coach** – Cria planos de estudo e registra o perfil de aprendizado do usuário.

Esses agentes já possuem personas e skills no repositório e podem ser integrados ao fluxo do master‑recruiter conforme o uso do projeto evolui.

## Contribuindo

1. **Leia** `AGENTS.md` para entender a arquitetura de agentes e skills.
2. **Adicione** novas personas em `personas/` e suas skills correspondentes em `skills/`.
3. **Atualize** `AGENTS.md` com o novo agente e dependências.
4. **Teste** o fluxo completo usando a IA que executa o arquivo de persona principal.