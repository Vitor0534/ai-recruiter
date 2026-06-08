# Plano – Coach de Treinamento

## Visão Geral

Este plano descreve a implementação do agente **training‑coach**, responsável por orientar o usuário na execução dos cursos recomendados pelo **curator**. O agente coleta informações de disponibilidade e foco do estudante, gera um plano de estudos estruturado e acompanha o progresso, respondendo dúvidas e oferecendo recomendações.

## Pré‑requisitos

- O projeto já deve conter os agentes **curator** e **coach** (ou **training‑coach**) implementados.
- O agente **training‑coach** deve ter acesso ao arquivo de perfil do estudante (`data/perfil-do-estudante.md`) e ao arquivo de planos de estudo (`data/planos-de-estudo.md`).
- O agente utiliza apenas as ferramentas de arquivo (`read_file`, `write_file`, `edit_file`) e o sub‑agente (`spawn_agent`) para delegar tarefas quando necessário.

## Diretrizes para Modelos MoE

- O agente deve responder em linguagem natural, mas os dados estruturados (horários, horas por dia, foco, etc.) devem ser salvos em arquivos Markdown seguindo o esquema definido.
- Não deve gerar tabelas Markdown em respostas ao usuário; use listas numeradas com pares chave‑valor.
- Todos os caminhos de arquivo devem ser relativos à raiz do projeto com prefixo explícito `data/`.
- Em caso de erro, informe o usuário e inclua a mensagem de erro no campo `erros`.

## Arquitetura

```
┌─────────────────────────────────────────────────┐
│                  Usuário                          │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│              Master‑Recruiter (Orquestrador)              │
│  - Interface principal com o usuário             │
│  - Coordena os agentes especializados            │
│  - Consolida resultados e apresenta ao usuário   │
└──┬──────────────┬──────────────┬────────────────┘
   │              │              │
   ▼              ▼              ▼
┌─────────┐  ┌──────────┐  ┌──────────────┐
│ SCOUT   │  │ CURATOR  │  │ TRAINING‑COACH │
│ (Busca  │  │ (Busca   │  │ (Orientação de│
│  de     │  │  Cursos) │  │  Estudos)   │
│  Vagas) │  └──────────┘  └──────────────┘
└─────────┘
```

## Estrutura de Diretórios

```
ai-recruiter/
├── AGENTS.md
├── personas/
│   ├── master-recruiter.md
│   ├── scout.md
│   ├── curator.md
│   └── training-coach.md
├── skills/
│   ├── firecrawl.md
│   ├── dispatch.md
│   ├── job-search.md
│   ├── course-search.md
│   └── training-coach.md
└── data/
    ├── personality-quiz.md
    ├── user-profile.md
    ├── job-search-results.md
    ├── course-recommendations.md
    ├── perfil-do-estudante.md
    └── planos-de-estudo.md
```

## Responsabilidade do Training‑Coach

- **Coleta de dados**: Perguntar ao usuário sobre disponibilidade diária, duração total do plano, foco de estudo (ex.: teoria, prática, projetos) e tecnologia escolhida.
- **Persistência**: Salvar os dados coletados em `data/perfil-do-estudante.md`.
- **Geração de plano**: Com base na tecnologia e no perfil, criar um plano de estudos estruturado (dias, horas, tópicos, links de curso, prazos) e salvar em `data/planos-de-estudo.md`.
- **Acompanhamento**: Responder dúvidas do usuário, sugerir ajustes de cronograma e fornecer feedback sobre o progresso.

## Skills

- `skills/training-coach.md` – Fluxo completo: perguntas, validação, escrita de arquivos, geração de plano e respostas.

## Protocolo de Handoff

O agente **training‑coach** recebe um envelope de despacho semelhante aos demais:

```
## DESPACHO: TRAINING‑COACH
### referencia_persona
[Conteúdo completo de personas/training-coach.md]

### tarefa
Orientar o usuário na criação de um plano de estudos para a tecnologia X

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
Tecnologia: [nome]
Disponibilidade: [horas por dia]
Duração: [número de semanas]
Foco: [teoria/prática/projetos]

### saida_esperada
Envelope de resposta com estado, resumo, dados (planos) e erros se houver
```

## Esquemas dos Arquivos de Dados

### data/perfil-do-estudante.md
```
Tecnologia: [nome]
Horas por dia: [número]
Duração do plano: [número de semanas]
Foco: [teoria/prática/projetos]
```

### data/planos-de-estudo.md
```
Data de Geração: [AAAA-MM-DD HH:MM]
Plano:
1. Dia 1 – 2h: Tópico A (link)
2. Dia 2 – 2h: Tópico B (link)
...
```

## Tasks

1. **Criar `skills/training-coach.md`** – Implementar fluxo de coleta, validação, escrita e geração.
2. **Criar `personas/training-coach.md`** – Descrever papel, ferramentas e formato de resposta.
3. **Atualizar `AGENTS.md`** – Listar o novo agente e suas dependências.
4. **Adicionar `data/perfil-do-estudante.md` e `data/planos-de-estudo.md`** – Arquivos de dados de perfil e plano.
5. **Testar** – Simular um usuário que deseja estudar React por 4 semanas, 3h/dia, foco em prática.

## Fluxo

```
Usuário solicita plano de estudos → Master‑Recruiter cria envelope de despacho → spawn_agent dispara Training‑Coach → Training‑Coach coleta dados → grava em perfil-do-estudante.md → gera plano → grava em planos-de-estudo.md → retorna ao usuário
```

## Regras de Erro

- Se falhar ao escrever em `data/perfil-do-estudante.md` ou `data/planos-de-estudo.md`, informe o erro e interrompa.
- Se o usuário fornecer dados inválidos (ex.: horas negativas), solicite correção.
- Se o plano não puder ser gerado (ex.: tecnologia desconhecida), notifique o usuário.

## Entregável

Plano de estudos completo, arquivos de perfil e plano gerados, e agente **training‑coach** funcional dentro do fluxo multi‑agente.
