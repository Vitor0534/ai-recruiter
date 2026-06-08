# Coach — Agente de Simulação de Entrevista

## Visão Geral

Sistema multi‑agente que auxilia o usuário a se preparar para entrevistas de emprego. O **coach** realiza simulações de entrevista baseadas nas vagas de interesse do usuário, incorporando as melhores práticas de mercado, dicas específicas da empresa e perguntas genéricas relevantes.

## Objetivo

- Investigar, a partir da internet, boas práticas atuais para entrevistas de tecnologia.
- Pesquisar a empresa da vaga selecionada (site institucional, avaliações de funcionários, discussões em comunidades) para gerar perguntas e feedback contextualizado.
- Oferecer ao usuário duas opções de simulação:
  1. **Entrevista especializada** – perguntas focadas na empresa e na vaga em questão.
  2. **Entrevista genérica** – perguntas padrão de mercado que cobrem habilidades técnicas, comportamentais e de cultura.

## Pré‑requisitos

- **Firecrawl** instalado e configurado (`FIRECRAWL_API_KEY` definida).
- Dados da vaga selecionada disponíveis em `data/job-search-results.md` (gerados pelo agente **Scout**).
- Perfil do usuário em `data/user-profile.md`.

## Diretrizes para Modelos MoE

- Cada passo deve ser explícito quanto à ferramenta, ao prompt e ao formato de saída.
- Saídas estruturadas devem usar **listas numeradas** com pares chave‑valor (sem tabelas markdown).
- Todos os caminhos de arquivo são relativos à raiz do projeto e devem iniciar com `data/`.
- Em caso de falha de ferramenta, registrar o erro no campo `erros` e interromper o fluxo.
- Não gerar código executável; o agente age apenas por meio de respostas conversacionais.

## Arquitetura

```
┌─────────────────────┐
│ Usuário              │
└───────┬─────────────┘
        ▼
┌─────────────────────┐
│ Master‑Recruiter    │ (orquestrador)
│ - Constrói envelope │
│ - Despacha Coach    │
└───────┬─────────────┘
        ▼
┌─────────────────────┐
│ COACH               │ (simulação)
│ - Busca boas‑práticas
│ - Pesquisa empresa
│ - Gera perguntas
└─────────────────────┘
```

## Estrutura de Diretórios

```
ai-recruiter/
├── AGENTS.md
├── personas/
│   ├── Master-Recruiter.md
│   └── coach.md                # NOVO: agente de simulação de entrevista
├── skills/
│   ├── firecrawl.md
│   ├── interview‑best‑practices.md   # NOVO: boas‑práticas de entrevista
│   ├── company‑research.md            # NOVO: pesquisa de empresa via Firecrawl
│   └── interview‑simulation.md        # NOVO: fluxo de simulação
└── data/
    ├── user-profile.md
    ├── job-search-results.md
    └── interview‑simulation.md       # NOVO: histórico da última simulação
```

## Coach – Agente de Simulação de Entrevista

**Responsabilidade**: conduzir a simulação de entrevista, adaptando‑a ao contexto da vaga ou a um cenário genérico.

**Skills**:
- `skills/interview-best-practices.md` – coleta de boas‑práticas atuais (fontes: blogs, artigos, guias de empresas de tecnologia).
- `skills/company-research.md` – pesquisa da empresa alvo (site institucional, Glassdoor, Reddit, StackOverflow) usando `firecrawl search` e `firecrawl scrape`.
- `skills/interview-simulation.md` – lógica de geração de perguntas, avaliação das respostas do usuário e feedback.
- `skills/firecrawl.md` – comandos e regras do CLI Firecrawl.

**Ferramentas do Zed**:
- `terminal` – executar `firecrawl search` e `firecrawl scrape`.
- `read_file` / `write_file` – acessar `data/job-search-results.md`, `data/user-profile.md` e gravar `data/interview-simulation.md`.

**Fluxo de Trabalho**
1. **Carregar contexto** – ler a vaga selecionada (`data/job-search-results.md`) e o perfil do usuário.
2. **Perguntar ao usuário** – oferecer escolha entre entrevista especializada ou genérica.
3. **Buscar boas‑práticas** – executar `firecrawl search "interview best practices tech" --json` e resumir os resultados.
4. **Pesquisar empresa** (se especializado) – buscar nome da empresa + "culture" / "interview experience" e extrair pontos relevantes.
5. **Gerar perguntas** – combinar:
   - Perguntas técnicas relacionadas às habilidades da vaga.
   - Perguntas comportamentais baseadas nas boas‑práticas.
   - Perguntas específicas da empresa (missão, produtos, cultura).
6. **Conduzir simulação** – apresentar cada pergunta, receber a resposta do usuário, avaliar (comparar com respostas modelo) e fornecer feedback imediato.
7. **Persistir resultado** – gravar resumo da simulação em `data/interview-simulation.md` (data, tipo de entrevista, principais pontos de melhoria).
8. **Encerrar** – devolver ao Master‑Recruiter um envelope de resposta contendo:
   - `estado`: "concluído"
   - `resumo`: breve feedback geral
   - `dados`: link para o arquivo de histórico
   - `erros` (se houver).

## Plano de Implementação (plano‑choach‑entrevistador)

1. **Criar skill `interview-best-practices.md`** – descrição dos passos de busca, análise e estrutura de saída.
2. **Criar skill `company-research.md`** – fluxo de pesquisa da empresa, tratamento de falhas e extração de tópicos relevantes.
3. **Criar skill `interview-simulation.md`** – algoritmo de geração de perguntas, lógica de feedback e formato de armazenamento.
4. **Criar persona `personas/coach.md`** – papel, ferramentas, referências às skills acima e formato de Response Envelope.
5. **Atualizar `AGENTS.md`** – incluir novo agente **coach** na lista, apontando para `personas/coach.md` e listando as skills necessárias.
6. **Atualizar Master‑Recruiter** – adicionar opção de menu (por exemplo, "C") que despacha o agente **coach** via `spawn_agent` com o envelope adequado.
7. **Criar arquivo de histórico** `data/interview-simulation.md` (inicialmente vazio) para armazenar resultados.
8. **Testes manuais**:
   - Selecionar a opção de entrevista especializada e validar que o agente pesquisa a empresa e gera perguntas.
   - Selecionar a opção genérica e validar que o agente usa apenas boas‑práticas.
   - Simular falha de `firecrawl search` (desconectar internet) e confirmar que o erro é reportado e o fluxo retorna ao menu.

## Esquema do Envelope de Despacho (Master‑Recruiter → Coach)

```
## DESPACHO: COACH
### referencia_persona
[conteúdo completo de personas/coach.md]

### tarefa
Conduzir simulação de entrevista para a vaga selecionada ou genérica.

### perfil_usuario
[conteúdo de data/user-profile.md]

### vaga_selecionada
[entrada de data/job-search-results.md – vaga escolhida pelo usuário]

### contexto
Tipo de entrevista desejado (especializada ou genérica) – a ser confirmado com o usuário.

### saida_esperada
Envelope de resposta contendo estado, resumo, caminho do histórico e erros, se houver.
```

## Formato da Resposta do Coach

```
estado: concluído
resumo: "Entrevista especializada concluída. Pontos fortes: comunicação clara, experiência com React. Pontos a melhorar: aprofundar conhecimento em arquitetura de micro‑serviços."
dados: data/interview-simulation.md
erros: []
```

---

Este plano segue o mesmo padrão de documentação usado em `plano.md` e está pronto para ser implementado pelos desenvolvedores.
