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
│   └─ course-recommendations.md (gerado pelo Curator)
├─ personas/
│   ├─ master-recruiter.md   # Orquestrador – interface principal
│   ├─ scout.md              # Busca de vagas (firecrawl)
│   └─ curator.md            # Busca de cursos (firecrawl)
├─ skills/
│   ├─ dispatch.md           # Lógica de envelope e chamada a sub‑agentes
│   ├─ job-search.md         # Fluxo de busca de vagas
│   ├─ course-search.md      # Fluxo de busca de cursos
│   └─ firecrawl.md          # Wrapper de comandos firecrawl
└─ plano‑*.md                # Documentos de planejamento e requisitos
```

## Agentes (personas)

| Agente | Papel | Arquivo | Skills principais |
|--------|-------|---------|-------------------|
| **master‑recruiter** | Orquestrador. Interage com o usuário, coleta o quiz, exibe menu e despacha sub‑agentes. | `personas/master-recruiter.md` | `skills/dispatch.md`, `skills/job-search.md` |
| **scout** | Busca vagas de emprego, analisa requisitos e gera resumo comparativo. | `personas/scout.md` | `skills/job-search.md`, `skills/firecrawl.md` |
| **curator** | Busca cursos online para suprir habilidades faltantes. | `personas/curator.md` | `skills/course-search.md`, `skills/firecrawl.md` |
| **coach** | Simulação de entrevistas (a ser implementado). | – | – |
| **training‑coach** | Cria planos de estudo personalizados e acompanha o progresso do usuário. | – | – |

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

## Extensões planejadas

- **coach** – Simulação de entrevistas técnicas.
- **training‑coach** – Plano de estudo contínuo, acompanhamento de progresso e métricas de aprendizado.

- **coach** – Simulação de entrevistas técnicas.
- **training‑coach** – Plano de estudo contínuo, acompanhamento de progresso e métricas de aprendizado.

## Contribuindo

1. **Leia** `AGENTS.md` para entender a arquitetura de agentes e skills.
2. **Adicione** novas personas em `personas/` e suas skills correspondentes em `skills/`.
3. **Atualize** `AGENTS.md` com o novo agente e dependências.
4. **Teste** o fluxo completo usando a IA que executa o arquivo de persona principal.