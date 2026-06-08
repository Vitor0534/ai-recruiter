# Skill: parse-job-markdown

## Objetivo
Receber o conteúdo Markdown de uma vaga e extrair:
- empresa
- localizacao
- descricao (primeiros 200 caracteres)
- skills_requeridas (lista de palavras‑chave, normalizadas)
- link original (passado como parâmetro)

## Entrada
- `markdown` – string contendo o conteúdo Markdown.
- `link` – URL da vaga.

## Saída
JSON conforme protocolo de `job-data-scraper`.

## Implementação
- Use expressões regulares para localizar:
  - **Empresa**: linha que contém `**Habilidades:**` precedida por texto que não seja URL, ou logo alt text.
  - **Localização**: linha contendo "Trabalho remoto" ou "Remoto".
  - **Descrição**: primeiro parágrafo após a linha do título da vaga.
  - **Habilidades**: lista após `**Habilidades:**`.
- Normalizar habilidades: lower case, remove pontuação.
- Se algum campo não for encontrado, usar valores padrão (`N/A` ou `[]`).

## Tratamento de Erros
- Em caso de exceção, retornar `sucesso: false` e mensagem de erro.
