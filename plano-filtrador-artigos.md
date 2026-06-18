# Plano — Filtrador de Artigos

## Visão Geral

Este plano descreve a implementação do agente **filtrador-artigo**, especializado em avaliar e aprofundar os artigos retornados pelo agente `buscador-artigo`. O filtrador visita cada link recomendado, verifica se o site está disponível, e gera um resumo do conteúdo — podendo ser um resumo rápido ou um passo a passo detalhado, conforme a escolha do usuário.

Antes de gerar qualquer resumo, o agente lista todos os artigos que serão investigados e pergunta ao usuário quais ele deseja investigar e em qual formato (resumo rápido ou passo a passo).

## Pré-requisitos

- Firecrawl instalado e configurado (`FIRECRAWL_API_KEY` definida)
- `data/artigos-recomendation.md` preenchido com resultados do agente `buscador-artigo`
- `data/user-profile.md` disponível para contexto do usuário
- O projeto já deve ter a persona `master-recruiter` e a skill `skills/dispatch.md`

## Diretrizes para Modelos MoE

- Sem instruções ambíguas. Cada passo deve indicar a ferramenta, o arquivo e o formato de saída.
- Não usar tabelas Markdown nas respostas. Use listas numeradas e pares chave-valor.
- Caminhos de arquivo devem ser relativos à raiz do projeto e incluir o prefixo `data/`.
- Se uma ferramenta falhar, reportar o erro no campo `erros` e não continuar silenciosamente.
- Não inventar dados. Se a busca web ou scrape falhar, informar o erro real.
- Persona e skill devem ser definidas como instruções, não como código executável.

## Arquitetura

```
Usuário
  └─> Master-Recruiter (Orquestrador)
          ├─> Scout (Busca de Vagas)
          ├─> Curator (Busca de Cursos)
          ├─> Buscador-Artigo (Busca de Artigos)
          └─> Filtrador-Artigo (Avaliação e Resumo de Artigos)
```

## Estrutura de Diretórios

```
ai-recruiter/
├── personas/
│   ├── master-recruiter.md
│   ├── buscador-artigo.md
│   └── filtrador-artigo.md  # NOVO: Agente de filtragem e resumo de artigos
├── skills/
│   ├── dispatch.md
│   ├── firecrawl.md
│   ├── article-recommendation.md
│   └── article-filter.md  # NOVO: Fluxo de filtragem e resumo de artigos
└── data/
    ├── artigos-recomendation.md
    └── artigos-filtrados.md  # NOVO: Resultados da filtragem e resumos
```

## Filtrador-Artigo — Agente de Avaliação e Resumo

### Responsabilidade
- Ler os artigos recomendados pelo `buscador-artigo` em `data/artigos-recomendation.md`.
- Visitar cada link para verificar se o site está disponível (acessível).
- Listar os artigos disponíveis ao usuário com status de disponibilidade.
- Perguntar ao usuário quais artigos ele deseja investigar.
- Perguntar ao usuário o formato desejado: **resumo rápido** ou **passo a passo**.
- Gerar o conteúdo no formato escolhido para cada artigo selecionado.
- Salvar os resultados em `data/artigos-filtrados.md`.

### Skills
- `skills/article-filter.md` — fluxo principal de filtragem, verificação e resumo.
- `skills/firecrawl.md` — referência para uso do CLI Firecrawl.

### Ferramentas
- `terminal` — executar comandos `firecrawl scrape`
- `read_file` — ler `data/artigos-recomendation.md` e `data/user-profile.md`
- `write_file` — gravar `data/artigos-filtrados.md`

### Entradas
- `data/artigos-recomendation.md` com artigos recomendados pelo buscador-artigo
- `data/user-profile.md` com área de interesse, nível e habilidades atuais
- Respostas do usuário (seleção de artigos e formato desejado)

### Saídas
- Arquivo `data/artigos-filtrados.md` com resumos ou passo a passos dos artigos selecionados
- Resposta ao Master-Recruiter com os dados estruturados e possíveis erros

## Fluxo Detalhado

### Passo 1 — Leitura dos artigos recomendados
- Usar `read_file` para ler `data/artigos-recomendation.md`.
- Extrair a lista de artigos com título, link e assuntos cobertos.

### Passo 2 — Verificação de disponibilidade
- Para cada artigo, executar `firecrawl scrape <url> --format markdown`.
- Se o scrape retornar conteúdo, marcar como **disponível**.
- Se o scrape falhar (timeout, 404, erro de rede), marcar como **indisponível**.
- Registrar o status de cada artigo.

### Passo 3 — Apresentação ao usuário
- Listar todos os artigos com o seguinte formato:

```
Artigos encontrados:

1. [título do artigo]
   link: [URL]
   assuntos: [assunto1, assunto2]
   status: disponível / indisponível

2. [título do artigo]
   link: [URL]
   assuntos: [assunto1, assunto2]
   status: disponível / indisponível

...
```

### Passo 4 — Interação com o usuário
- Perguntar: "Quais artigos você deseja investigar? (informe os números separados por vírgula, ou 'todos' para todos os disponíveis)"
- Perguntar: "Qual formato você prefere?"
  - **A) Resumo rápido** — um parágrafo curto explicando o que o artigo ensina, os pontos principais e a conclusão.
  - **B) Passo a passo** — um guia detalhado reproduzindo as instruções do artigo, com etapas numeradas e exemplos de código quando aplicável.

### Passo 5 — Geração de conteúdo
- Para cada artigo selecionado e disponível:
  - Usar o conteúdo já obtido no scrape do Passo 2.
  - Se o formato escolhido for **resumo rápido**:
    - Gerar um resumo de 3 a 5 frases cobrindo: tema principal, pontos-chave e conclusão.
  - Se o formato escolhido for **passo a passo**:
    - Extrair as etapas práticas do artigo.
    - Organizar em passos numerados.
    - Incluir trechos de código ou comandos quando presentes no artigo original.
    - Adicionar observações ou dicas relevantes.

### Passo 6 — Gravação dos resultados
- Gravar `data/artigos-filtrados.md` no formato definido abaixo.
- Retornar envelope de resposta ao Master-Recruiter.

## Skills

### article-filter.md

- Usar `read_file` para ler `data/artigos-recomendation.md`.
- Extrair lista de artigos (título, link, assuntos).
- Para cada artigo, executar `firecrawl scrape <url> --format markdown` para verificar disponibilidade e obter conteúdo.
- Apresentar lista de artigos com status ao usuário.
- Aguardar seleção do usuário (quais artigos investigar).
- Aguardar escolha de formato (resumo rápido ou passo a passo).
- Para cada artigo selecionado e disponível, gerar o conteúdo no formato escolhido.
- Gravar resultados em `data/artigos-filtrados.md`.
- Retornar envelope de resposta com estado, resumo, dados e erros.

## Protocolo de Handoff do Filtrador de Artigos

### Envelope de Despacho

```
## DESPACHO: FILTRADOR-ARTIGO
### referencia_persona
[Conteúdo completo de personas/filtrador-artigo.md]
### tarefa
Avaliar os artigos recomendados, verificar disponibilidade, apresentar ao usuário para seleção e gerar resumos ou passo a passos conforme escolha do usuário
### perfil_usuario
[Conteúdo de data/user-profile.md]
### artigos_recomendados
[Conteúdo de data/artigos-recomendation.md]
### saida_esperada
Envelope de resposta com estado, resumo, dados (lista de artigos filtrados com resumos) e erros.
```

### Formato de Resposta — Resumo Rápido

```
1. titulo: [título do artigo]
   link: [URL]
   status: disponível
   assuntos_cobertos: [assunto1, assunto2]
   resumo: [resumo de 3 a 5 frases sobre o conteúdo do artigo]

2. titulo: ...
   ...
```

### Formato de Resposta — Passo a Passo

```
1. titulo: [título do artigo]
   link: [URL]
   status: disponível
   assuntos_cobertos: [assunto1, assunto2]
   passo_a_passo:
     1. [Descrição do primeiro passo]
     2. [Descrição do segundo passo]
     3. [Descrição do terceiro passo]
     ...
   observacoes: [dicas ou notas relevantes]

2. titulo: ...
   ...
```

## Esquema de `data/artigos-filtrados.md`

```
Data da Filtragem: [AAAA-MM-DD HH:MM]
Fonte de Dados: `data/artigos-recomendation.md`
Formato Escolhido: [resumo rápido / passo a passo]
Artigos Selecionados: [número de artigos selecionados]

Resultados:

1. titulo: [título do artigo]
   link: [URL]
   status: disponível
   assuntos_cobertos: [assunto1, assunto2]
   formato: [resumo rápido / passo a passo]
   conteudo:
     [resumo ou passo a passo conforme formato escolhido]

2. titulo: ...
   ...

Artigos Indisponíveis:
- [título] — [URL] — [motivo do erro]
```

## Tarefas

1. Criar `skills/article-filter.md` com o fluxo completo de filtragem, verificação de disponibilidade e geração de resumos.
2. Criar `personas/filtrador-artigo.md` definindo papel, ferramentas e formato de saída.
3. Atualizar `AGENTS.md` para listar o novo agente e suas dependências.
4. Adicionar o novo menu ou opção ao `master-recruiter` para despachar o agente `filtrador-artigo`.
5. Testar o fluxo com artigos existentes em `data/artigos-recomendation.md` e verificar a gravação de `data/artigos-filtrados.md`.

## Regras de Erro

- Se `firecrawl scrape` falhar para um artigo, marcar como indisponível e informar o motivo ao usuário.
- Se todos os artigos estiverem indisponíveis, informar o usuário e sugerir executar novamente o `buscador-artigo`.
- Se o usuário não selecionar nenhum artigo, encerrar o fluxo e retornar ao menu.
- Se a leitura de `data/artigos-recomendation.md` falhar, interromper e relatar o erro.
- Se o arquivo `data/artigos-recomendation.md` estiver vazio ou sem artigos, orientar o usuário a executar primeiro o `buscador-artigo`.
