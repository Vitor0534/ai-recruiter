# Skill: course-search

## Objetivo
Buscar cursos online usando o CLI **firecrawl** que ajudem o usuário a desenvolver as habilidades faltantes identificadas nas vagas.

## Entrada
- `habilidades_faltantes` – lista de habilidades que o usuário precisa desenvolver (extraídas de `data/job-search-results.md`).

## Passos
1. **Ler resultados de vagas**
   - Usar `read_file` para acessar `data/job-search-results.md`
   - Extrair todas as habilidades faltantes únicas de todas as vagas listadas
   - Se não houver habilidades faltantes, retornar erro "Nenhuma habilidade faltante identificada"

2. **Para cada habilidade faltante, buscar cursos**
   - Construir query: `query="curso ${habilidade} online"`
   - Executar: `firecrawl search "$query" --json`
   - Se o comando falhar, registrar o erro e tentar a próxima habilidade

3. **Selecionar até 2 cursos por habilidade**
   - Priorizar cursos de plataformas conhecidas (Alura, Udemy, Coursera, Rocketseat, DIO, etc.)
   - Se não houver resultados, tentar query alternativa: `query="aprender ${habilidade}"`

4. **Obter detalhes do curso**
   - Para cada URL selecionada, executar:
     ```sh
     firecrawl scrape <url> --format markdown
     ```
   - Extrair: título, duração, conteúdo programático, nível (iniciante/intermediário/avançado)
   - Se o scrape falhar, usar título e descrição da busca e registrar a falha

5. **Consolidar resultados**
   - Limitar a 5 cursos no total
   - Priorizar cursos que abordam múltiplas habilidades faltantes
   - Remover duplicatas (mesmo curso em URLs diferentes)

6. **Montar resposta**
   - Formatar cada curso no formato:
     ```text
     1. titulo: <título do curso>
        plataforma: <nome da plataforma>
        duracao: <duração estimada>
        link: <URL>
        habilidades_abordadas: [h1, h2]
        nivel: <iniciante, intermediário ou avançado>
     ```
   - Incluir campo `erros` caso alguma extração tenha falhado

## Saída Esperada
Um **envelope de resposta** JSON com os campos:
```json
{
  "sucesso": true,
  "erros": [],
  "dados": "<texto formatado conforme acima>"
}
```
Se houver falha crítica (ex.: nenhuma habilidade faltante), retornar:
```json
{
  "sucesso": false,
  "erros": ["<mensagem de erro>"]
}
```

## Tratamento de Erros
- Falha ao ler `data/job-search-results.md`: registrar erro e abortar.
- Nenhuma habilidade faltante encontrada: retornar `sucesso: false` com erro indicando "Nenhuma habilidade faltante identificada".
- Falha no `firecrawl search` para uma habilidade: registrar erro, tentar query alternativa, se falhar novamente, pular essa habilidade e continuar com as restantes.
- Falha no `firecrawl scrape` de uma URL específica: usar título/descrição da busca, registrar erro na lista `erros` mas continuar com os demais cursos.
- Se nenhum curso for encontrado após todas as buscas: retornar `sucesso: false` com erro "Nenhum curso encontrado para as habilidades identificadas".