# Curator — Agente de Busca de Cursos

## Visão Geral

Sistema multi-agente que auxilia usuários em sua jornada de desenvolvimento de carreira, combinando busca de empregos, identificação de lacunas de habilidades, recomendações de cursos e simulação de entrevistas.

**Objetivo**: Opção B do menu funciona. Agente busca cursos via Firecrawl, mostra resultados que complementam as habilidades faltantes identificadas nas vagas.

## Pré-requisitos

- Firecrawl instalado e configurado (`FIRECRAWL_API_KEY` definida)
- Scout já implementado e funcional
- `data/job-search-results.md` preenchido com resultados da busca de vagas

## Diretrizes para Modelos MoE

Todas as personas e skills são projetadas para modelos Mixture of Experts (MoE). Siga estas regras:

- Sem instruções ambíguas. Cada passo deve especificar exatamente o que fazer, qual ferramenta usar e qual formato de saída produzir.
- Sem tabelas markdown em nenhuma saída. Use listas numeradas com pares chave-valor para dados estruturados.
- Todos os caminhos de arquivo devem ser relativos à raiz do projeto com prefixo explícito `data/`. A raiz do projeto é o diretório `ai-recruiter/`.
- Se uma ferramenta falhar, o agente deve relatar a falha no campo `erros` e não continuar silenciosamente.
- Nunca invente dados. Se um `firecrawl search` ou `firecrawl scrape` falhar, reporte o erro exato e pare. **Exceção**: Em `skills/course-analysis.md`, se a busca falhar para uma habilidade específica, tente o fallback; se o fallback também falhar, pule essa habilidade e continue com as restantes.
- O agente NÃO deve escrever scripts Python, scripts de shell ou qualquer código para implementar personas. O agente personifica cada papel diretamente através do seu comportamento e respostas conversacionais. Personas são instruções comportamentais, não código a ser gerado.

## Arquitetura

```
┌─────────────────────────────────────────────────┐
│                  Usuário                          │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│              Master-Recruiter (Orquestrador)              │
│  - Interface principal com o usuário             │
│  - Coordena os agentes especializados            │
│  - Consolida resultados e apresenta ao usuário   │
└──┬──────────────┬──────────────┬────────────────┘
   │              │              │
   ▼              ▼              ▼
┌─────────┐  ┌──────────┐  ┌──────────────┐
│ SCOUT   │  │ CURATOR  │  │ COACH        │
│ (Busca  │  │ (Busca   │  │ (Simulação   │
│  de     │  │  de      │  │  de          │
│  Vagas) │  │  Cursos) │  │  Entrevistas)│
└─────────┘  └──────────┘  └──────────────┘
```

**Escopo**: Implementar o bloco Curator e conectá-lo ao Master-Recruiter.

## Estrutura de Diretórios

```
ai-recruiter/
├── AGENTS.md                   # (existente)
├── personas/
│   ├── master-recruiter.md          # (existente)
│   ├── scout.md            # (existente)
│   └── curator.md          # NOVO: Agente de busca de cursos
├── skills/
│   ├── firecrawl.md        # (existente)
│   ├── dispatch.md         # (existente)
│   ├── job-search.md       # (existente)
│   └── course-search.md    # NOVO: Capacidades de busca de cursos
└── data/
    ├── personality-quiz.md       # (existente)
    ├── user-profile.md           # (existente)
    ├── job-search-results.md       # (existente)
    └── course-recommendations.md   # NOVO: Últimos resultados do Curator
```

## Curator - Agente de Busca de Cursos

**Responsabilidade**: Buscar cursos online via Firecrawl que ajudem o usuário a desenvolver as habilidades faltantes identificadas nas vagas.

**Skills**:
- `skills/course-search.md` — Fluxo completo: comandos firecrawl, leitura de resultados de vagas, extração de habilidades faltantes, busca de cursos, formato de resposta e tratamento de erros. **OBRIGATÓRIO CARREGAR.**
- `skills/firecrawl.md` — Comandos e regras do CLI Firecrawl.

**Ferramentas do Zed**:
- `terminal` — executar comandos `firecrawl search` e `firecrawl scrape`
- `read_file` — ler `data/job-search-results.md` para extrair habilidades faltantes

**Ferramentas de acesso web**:
- Sempre use `firecrawl search` via `terminal` como seu método primário de acesso à web
- Se o firecrawl falhar consistentemente, você PODE usar `curl` ou `wget` como fallback — note as limitações (sem renderização JS, possíveis bloqueios anti-bot, HTML bruto em vez de markdown limpo)
- Evite `Fetch`, `webfetch` ou outras ferramentas HTTP — prefira firecrawl primeiro, depois curl/wget apenas como recuperação

**Fontes de Dados**:
- Busca Firecrawl (plataformas como Alura, Udemy, Coursera, Rocketseat, DIO, etc.)

**Entradas**:
- Habilidades faltantes extraídas de `data/job-search-results.md`

**Saídas**:
- Retornar resultados ao Master-Recruiter via Response Envelope (Master-Recruiter salva em `data/course-recommendations.md`)
- Exibir lista de até 5 cursos com título, plataforma, duração, link, habilidades abordadas e nível do curso

## Skills

### course-search.md

- **Ferramenta Zed**: `terminal` — executar comandos CLI do `firecrawl`
  - **Descoberta de cursos**: `firecrawl search "curso [habilidade_faltante]" --json`
    - Retorna JSON com: url, title, description, metadata para cada resultado
  - **Detalhes completos do curso**: `firecrawl scrape <url> --format markdown`
    - Usar em URLs individuais de cursos dos resultados da busca para obter descrição, conteúdo programático e duração
    - Se a extração falhar ou expirar, usar o título e descrição do resultado da busca
- **Ferramenta de leitura**: `read_file` — ler `data/job-search-results.md`
- Extrair todas as habilidades faltantes únicas de todas as vagas
- Para cada habilidade faltante, buscar até 2 cursos recomendados
- Para cada curso encontrado, extrair: título, plataforma (da URL ou título), duração, link, habilidades abordadas
- Retornar até 5 cursos no total (priorizar cursos que abordam múltiplas habilidades faltantes)

## Protocolo de Handoff do Curator

**Contexto passado:** habilidades faltantes das vagas de `data/job-search-results.md`

**Envelope de Despacho** (construído pelo Master-Recruiter):

```
## DESPACHO: CURATOR
### referencia_persona
[Conteúdo completo de personas/curator.md]

### tarefa
Buscar cursos para desenvolver as habilidades faltantes identificadas nas vagas

### contexto_busca
Habilidades faltantes: [lista_de_habilidades_faltantes]

### resultados_vagas
[Conteúdo de data/job-search-results.md]

### saida_esperada
Envelope de resposta com estado, resumo, dados (lista de cursos) e erros se houver
```

**Formato de dados da resposta:**
```
1. titulo: [título do curso]
   plataforma: [nome da plataforma]
   duracao: [duração estimada]
   link: [URL]
   habilidades_abordadas: [habilidade1, habilidade2]
   nivel: [iniciante, intermediário, avançado]

2. [próximo curso no mesmo formato]
```

## Esquemas dos Arquivos de Dados

### data/course-recommendations.md

Armazena os resultados mais recentes do Curator para uso pelo usuário:

```
Data da Busca: [AAAA-MM-DD HH:MM]
Habilidades Buscadas:
  [habilidade1, habilidade2, ...]

Resultados:
1. titulo: [título do curso]
   plataforma: [nome da plataforma]
   duracao: [duração estimada]
   link: [URL]
   habilidades_abordadas: [habilidade1, habilidade2]
   nivel: [iniciante, intermediário, avançado]

2. titulo: [próximo título de curso]
   ...
```

## Tasks

### 1. Escrever `skills/course-search.md`

Contendo:
- Comandos firecrawl para busca de cursos
- Como ler e extrair habilidades faltantes de `data/job-search-results.md`
- Como analisar resultados JSON
- Como extrair duração e conteúdo programático
- Formato de resposta esperado
- Tratamento de erros

### 2. Escrever `personas/curator.md`

Contendo:
- Papel e responsabilidade do Curator
- Ferramentas disponíveis (terminal com firecrawl, read_file)
- Referência obrigatória à skill `skills/course-search.md`
- Referência à skill `skills/firecrawl.md` (existente)
- Formato do Response Envelope
- Regras de tratamento de erros

### 3. Conectar `spawn_agent` no Master-Recruiter para despachar o Curator

- Quando o usuário digitar "B", o Master-Recruiter deve construir o envelope de despacho
- Chamar `spawn_agent` com o prompt de despacho
- Analisar a resposta do Curator
- Salvar resultados em `data/course-recommendations.md`
- Exibir cursos ao usuário
- Retornar ao menu

### 4. Testar: busca de cursos funcional

- Usuário seleciona B
- Curator lê `data/job-search-results.md`
- Curator busca cursos via Firecrawl
- Resultados exibidos com habilidades abordadas

### 5. Testar: tratamento de erro

- Se a busca falhar, o erro é reportado ao usuário
- Agente retorna ao menu

## Fluxo

```
Usuário seleciona B no menu
        │
        ▼
Master-Recruiter constrói envelope de despacho para Curator
        │
        ▼
spawn_agent dispara Curator com persona + contexto
        │
        ▼
Curator lê data/job-search-results.md → extrai habilidades faltantes
        │
        ▼
Curator executa firecrawl search → firecrawl scrape nas URLs
        │
        ▼
Curator formata cursos recomendados
        │
        ▼
Curator retorna envelope de resposta com até 5 cursos
        │
        ▼
Master-Recruiter salva em data/course-recommendations.md → exibe ao usuário → retorna ao menu
```

## Regras de Erro

- Se `firecrawl search` falhar, reportar o erro ao usuário e retornar ao menu.
- Se `firecrawl scrape` falhar em uma URL específica, usar o título/descrição do resultado da busca e anotar que a extração falhou.
- Se `spawn_agent` falhar, dizer ao usuário o que deu errado e retornar ao menu.
- Se não houver habilidades faltantes em `data/job-search-results.md`, informar ao usuário que não há lacunas de habilidades a serem preenchidas.
- Se a busca retornar resultados mas algumas extrações falharem, mostrar resultados parciais com detalhes completos onde disponíveis.

## Entregável

Curator totalmente funcional, busca de cursos end-to-end a partir do menu.