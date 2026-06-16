# Plano — Recomendador de Artigos

## Visão Geral

Este plano descreve a implementação do agente **recomendador-artigos**, especializado em recomendar artigos e conteúdos de leitura para aprender os assuntos que o usuário ainda não domina e que aparecem como exigências nas vagas retornadas.

O agente usa os resultados da busca de vagas para identificar lacunas de conhecimento e busca artigos na internet. As recomendações são salvas em `data/artigos-recomendation.md`.

## Pré-requisitos

- Firecrawl instalado e configurado (`FIRECRAWL_API_KEY` definida)
- `data/job-search-results.md` preenchido com resultados de vagas do agente Scout
- `data/user-profile.md` disponível para contexto do usuário
- O projeto já deve ter a persona `master-recruiter` e a skill `skills/dispatch.md`

## Diretrizes para Modelos MoE

- Sem instruções ambíguas. Cada passo deve indicar a ferramenta, o arquivo e o formato de saída.
- Não usar tabelas Markdown nas respostas. Use listas numeradas e pares chave-valor.
- Caminhos de arquivo devem ser relativos à raiz do projeto e incluir o prefixo `data/`.
- Se uma ferramenta falhar, reportar o erro no campo `erros` e não continuar silenciosamente.
- Não inventar dados. Se a busca web falhar, informar o erro real.
- Persona e skill devem ser definidas como instruções, não como código executável.

## Arquitetura

```
Usuário
  └─> Master-Recruiter (Orquestrador)
          ├─> Scout (Busca de Vagas)
          ├─> Curator (Busca de Cursos)
          └─> Recomendador-Artigos (Busca de Artigos)
```

## Estrutura de Diretórios

```
ai-recruiter/
├── personas/
│   ├── master-recruiter.md
│   ├── scout.md
│   ├── curator.md
│   └── recomendador-artigos.md  # NOVO: Agente de recomendações de artigos
├── skills/
│   ├── dispatch.md
│   ├── firecrawl.md
│   └── article-recommendation.md  # NOVO: Fluxo de busca de artigos
└── data/
    ├── job-search-results.md
    ├── course-recommendations.md
    └── artigos-recomendation.md  # NOVO: Resultados de recomendação de artigos
```

## Recomendador-Artigos — Agente de Recomendação de Leitura

### Responsabilidade
- Identificar assuntos e habilidades faltantes nas vagas encontradas pelo Scout.
- Buscar artigos, posts técnicos e materiais de leitura relevantes na internet.
- Retornar recomendações com título, fonte, link, resumo e assuntos cobertos.
- Salvar a lista de recomendações em `data/artigos-recomendation.md`.

### Skills
- `skills/article-recommendation.md` — fluxo principal de busca e seleção de artigos.
- `skills/firecrawl.md` — referência para uso do CLI Firecrawl.

### Ferramentas
- `terminal` — executar comandos `firecrawl search` e `firecrawl scrape`
- `read_file` — ler `data/job-search-results.md` e `data/user-profile.md`
- `write_file` — gravar `data/artigos-recomendation.md`

### Entradas
- `data/job-search-results.md` com vagas e habilidades faltantes
- `data/user-profile.md` com área de interesse, nível e habilidades atuais

### Saídas
- Arquivo `data/artigos-recomendation.md` com até 5 artigos recomendados
- Resposta ao Master-Recruiter com os dados estruturados e possíveis erros

## Skills

### article-recommendation.md

- Usar `read_file` para ler `data/job-search-results.md`.
- Extrair as habilidades e assuntos faltantes únicos de todas as vagas.
- Agrupar os assuntos em uma lista priorizada, começando pelos mais recorrentes.
- Para cada assunto prioritário, executar busca:
  - `firecrawl search "artigo [assunto]" --json`
  - `firecrawl search "tutorial [assunto]" --json`
  - `firecrawl search "conteúdo sobre [assunto]" --json`
- Para os links mais relevantes, usar `firecrawl scrape <url> --format markdown` para obter resumo e confirmar relevância.
- Se o scrape falhar, usar título e descrição da busca como fallback.
- Selecionar até 5 artigos distintos que cubram os assuntos em falta.
- Para cada artigo recomendado, gerar:
  1. `titulo`
  2. `fonte`
  3. `link`
  4. `tipo`: artigo técnico, tutorial, blog, referência, post explicativo
  5. `assuntos_cobertos`
  6. `resumo`
- Retornar a lista de recomendações no formato esperado.

## Protocolo de Handoff do Recomendador de Artigos

### Envelope de Despacho

```
## DESPACHO: RECOMENDADOR-ARTIGOS
### referencia_persona
[Conteúdo completo de personas/recomendador-artigos.md]
### tarefa
Buscar artigos para os assuntos que o usuário não domina e que estão nas exigências das vagas encontradas
### perfil_usuario
[Conteúdo de data/user-profile.md]
### resultados_vagas
[Conteúdo de data/job-search-results.md]
### saida_esperada
Envelope de resposta com estado, resumo, dados (lista de artigos) e erros.
```

### Formato de Resposta Esperado

```
1. titulo: [título do artigo]
   fonte: [nome do site ou publicação]
   link: [URL]
   tipo: [artigo técnico, tutorial, blog, referência]
   assuntos_cobertos: [assunto1, assunto2]
   resumo: [resumo curto da recomendação]

2. titulo: ...
   ...
```

## Esquema de `data/artigos-recomendation.md`

```
Data da Recomendação: [AAAA-MM-DD HH:MM]
Fonte de Dados: `data/job-search-results.md`
Assuntos Prioritários:
  [assunto1, assunto2, ...]

Recomendações:
1. titulo: [título do artigo]
   fonte: [nome do site ou publicação]
   link: [URL]
   tipo: [artigo técnico, tutorial, blog, referência]
   assuntos_cobertos: [assunto1, assunto2]
   resumo: [resumo curto]

2. titulo: ...
   ...
```

## Tarefas

1. Criar `skills/article-recommendation.md` com o fluxo completo de busca de artigos e tratamento de erros.
2. Criar `personas/recomendador-artigos.md` definindo papel, ferramentas e formato de saída.
3. Atualizar `AGENTS.md` para listar o novo agente e suas dependências, se desejado.
4. Adicionar o novo menu ou opção ao `master-recruiter` para despachar o agente `recomendador-artigos`.
5. Testar o fluxo com resultados de vagas existentes e verificar a gravação de `data/artigos-recomendation.md`.

## Regras de Erro

- Se `firecrawl search` falhar, informar o erro exato no campo `erros`.
- Se `firecrawl scrape` falhar para um artigo, tentar outro link; se não houver alternativa, continuar com as demais recomendações.
- Se não for possível encontrar artigos para um assunto prioritário, informar o usuário e seguir com os demais assuntos.
- Se a leitura de `data/job-search-results.md` falhar, interromper e relatar o erro.
