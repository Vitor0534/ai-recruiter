# Skill: firecrawl

## Objetivo
Definir os comandos e regras básicas para uso do CLI **firecrawl** nas buscas de vagas e cursos.

## Comandos Principais

### Busca (Search)
```sh
firecrawl search "termo de busca" --json
```
- Retorna resultados em formato JSON
- Campos retornados: `url`, `title`, `description`, `metadata`
- Use aspas duplas para preservar espaços na query

### Scrap (Scrape)
```sh
firecrawl scrape <url> --format markdown
```
- Extrai conteúdo da página em formato markdown limpo
- Ideal para obter descrições completas de vagas ou cursos
- Se falhar, o conteúdo da busca (`title`, `description`) pode ser usado como fallback

## Regras de Uso

1. **Sempre verificar a API key**
   - Certifique-se de que `FIRECRAWL_API_KEY` está definida no ambiente
   - Comando para verificar: `echo $FIRECRAWL_API_KEY`

2. **Timeout e limites**
   - Firecrawl pode ter timeout em páginas muito grandes
   - Limitar busca a 10 resultados por query para evitar timeout

3. **Tratamento de erros**
   - Se o comando retornar erro de conexão, verificar conexão com internet
   - Se retornar erro de autenticação, verificar a API key
   - Se retornar erro de rate limit, aguardar alguns segundos e tentar novamente

4. **Formato de saída**
   - Usar `--json` para buscas que precisam de processamento programático
   - Usar `--format markdown` para extração de conteúdo legível

5. **Plataformas suportadas**
   - Firecrawl agrega resultados de múltiplas fontes
   - Para vagas: Indeed, Catho, LinkedIn, Glassdoor, Infojobs
   - Para cursos: Alura, Udemy, Coursera, Rocketseat, DIO, e outros sites educacionais