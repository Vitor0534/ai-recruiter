# Skill: article-recommendation

## Objetivo
Buscar artigos e conteúdos de leitura relevantes para os assuntos e habilidades que o usuário ainda não domina, conforme exigido nas vagas encontradas pelo Scout.

## Entrada
- `resultados_vagas` – texto ou conteúdo de `data/job-search-results.md`.
- `perfil_usuario` – texto de `data/user-profile.md` (opcional, usado para contexto de área, nível e habilidades atuais).

## Passos
1. **Ler resultados de vagas**
   - Usar `read_file` para ler `data/job-search-results.md` caso o conteúdo não venha completo no envelope.
   - Extrair todas as habilidades faltantes únicas de todas as vagas listadas.
   - Se não houver habilidades faltantes, retornar erro "Nenhuma habilidade faltante identificada".

2. **Priorizar assuntos**
   - Agrupar os assuntos por frequência de ocorrência nas habilidades faltantes.
   - Ordenar do mais recorrente ao menos recorrente.
   - Limitar a lista a um conjunto de assuntos prioritários.

3. **Buscar artigos para cada assunto prioritário**
   - Construir queries de busca:
     - `artigo [assunto]`
     - `tutorial [assunto]`
     - `conteúdo sobre [assunto]`
   - Executar:
     ```sh
     firecrawl search "artigo [assunto]" --json
     ```
   - Se falhar, tentar a query alternativa: `firecrawl search "tutorial [assunto]" --json`.
   - Registrar qualquer erro em `erros` e continuar com os demais assuntos.

4. **Selecionar resultados relevantes**
   - Para cada resultado de busca, extrair `url`, `title`, `description` e `metadata` quando disponível.
   - Priorizar artigos técnicos, blogs e posts que mencionem claramente o assunto.
   - Para cada link selecionado, executar:
     ```sh
     firecrawl scrape <url> --format markdown
     ```
   - Se o scrape falhar, usar o `title` e `description` da busca como fallback e registrar o erro.

5. **Extrair informações do artigo**
   - Para cada artigo recomendado, gerar:
     1. `titulo`
     2. `fonte` – nome do site ou publicação extraído da URL ou do título.
     3. `link`
     4. `tipo` – artigo técnico, tutorial, blog, referência ou post explicativo.
     5. `assuntos_cobertos` – lista de assuntos relacionados ao artigo.
     6. `resumo` – resumo curto baseado no conteúdo extraído ou na descrição da busca.

6. **Selecionar até 5 artigos**
   - Evitar duplicatas de conteúdo ou título.
   - Priorizar artigos que cubram múltiplos assuntos prioritários.
   - Manter no máximo 5 recomendações no total.

7. **Montar resposta**
   - Formatar cada artigo recomendado da seguinte forma:
     ```text
     1. titulo: <título do artigo>
        fonte: <nome do site ou publicação>
        link: <URL>
        tipo: <artigo técnico, tutorial, blog, referência>
        assuntos_cobertos: [assunto1, assunto2]
        resumo: <resumo curto>
     ```
   - Incluir o campo `erros` com mensagens claras se houver falhas nas buscas ou extrações.

## Saída Esperada
- Um envelope de resposta com os campos:
  - `sucesso: true | false`
  - `erros: []` ou lista de mensagens de erro
  - `dados: <texto formatado conforme acima>`

- Em caso de falha crítica, retornar:
  ```json
  {
    "sucesso": false,
    "erros": ["<mensagem de erro>"]
  }
  ```

## Tratamento de Erros
- Falha ao ler `data/job-search-results.md`: registrar erro e abortar.
- Falha no `firecrawl search` para um assunto: registrar erro, tentar query alternativa e continuar com os demais assuntos.
- Falha no `firecrawl scrape` de um URL específica: usar título/descrição como fallback e registrar erro.
- Se não houver artigos suficientes após todas as buscas: retornar os artigos coletados e incluir mensagem no `erros` explicando a limitação.
- Se não for possível extrair nenhum assunto prioritário: retornar `sucesso: false` com erro "Nenhum assunto prioritário identificado".
