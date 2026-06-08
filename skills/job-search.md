# Skill: job-search

## Objetivo
Buscar vagas de emprego usando o CLI **firecrawl**, analisar os resultados e comparar com o perfil do usuário.

## Entrada
- `area` – área de interesse (ex.: "desenvolvimento frontend").
- `localizacao` – cidade ou "remoto".
- `nivel_experiencia` – "Júnior", "Pleno" ou "Sênior".
- `habilidades_usuario` – lista de habilidades extraídas de `data/user-profile.md`.

## Passos
1. **Construir query**
   ```sh
   query="vagas ${area} ${localizacao}"
   ```
2. **Buscar vagas** usando firecrawl:
   ```sh
   firecrawl search "$query" --json
   ```
   - O comando devolve um JSON contendo, para cada resultado, `url`, `title`, `description` e, quando disponível, `metadata`.
   - Se o comando falhar, registrar o erro em `erros` e abortar.
3. **Selecionar até 5 resultados**
   - Ordenar pelos que contêm a palavra‑chave de nível de experiência que corresponde ao usuário.
   - Caso não existam vagas do nível exato, incluir vagas do nível adjacente e anotar a discrepância.
4. **Obter detalhes da vaga**
   - Para cada URL selecionada, executar:
     ```sh
     firecrawl scrape <url> --format markdown
     ```
   - Se o scrape falhar, usar apenas o `title` e `description` retornados pela busca e registrar a falha.
5. **Extrair requisitos de habilidades**
   - Analisar o markdown retornado (ou a descrição) procurando por listas de requisitos ou palavras‑chave de habilidades.
   - Normalizar todas as habilidades para minúsculas e remover pontuação.
6. **Comparar com habilidades do usuário**
   - `habilidades_correspondentes` = interseção entre `habilidades_usuario` e `habilidades_vaga`.
   - `habilidades_faltantes` = diferença `habilidades_vaga` – `habilidades_usuario`.
   - `contagem_correspondencia` = "X de Y habilidades correspondem" onde X = |correspondentes| e Y = |habilidades_vaga|.
7. **Montar resposta**
   - Formatar até 5 vagas no formato listado no plano:
     ```text
     1. titulo: <título>
        empresa: <empresa>
        localizacao: <cidade ou Remoto>
        link: <URL>
        habilidades_correspondentes: [h1, h2]
        habilidades_faltantes: [h3, h4]
        contagem_correspondencia: X de Y habilidades correspondem
     ```
   - Incluir campo `erros` caso alguma extração tenha falhado.

## Saída Esperada
Um **envelope de resposta** JSON com os campos:
```json
{
  "sucesso": true,
  "erros": [],
  "dados": "<texto formatado conforme acima>"
}
```
Se houver falha crítica (ex.: busca falhou), retornar:
```json
{
  "sucesso": false,
  "erros": ["<mensagem de erro>"]
}
```

## Tratamento de Erros
- Falha no `firecrawl search`: registrar erro e abortar.
- Falha no `firecrawl scrape` de uma URL específica: usar título/descrição da busca, registrar erro na lista `erros` mas continuar com as demais vagas.
- Se nenhuma vaga for encontrada, retornar `sucesso: false` com erro indicando "Nenhum resultado encontrado".
