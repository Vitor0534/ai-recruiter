# Skill: article-filter

## Objetivo
Avaliar os artigos recomendados pelo `buscador-artigo`, verificar a disponibilidade de cada link, apresentar a lista ao usuário para seleção, e gerar resumos rápidos ou passo a passos detalhados conforme a escolha do usuário.

## Entrada
- `artigos_recomendados` – texto ou conteúdo de `data/artigos-recomendation.md`.
- `perfil_usuario` – texto de `data/user-profile.md` (opcional, usado para contexto de área, nível e habilidades atuais).

## Passos
1. **Ler artigos recomendados**
   - Usar `read_file` para ler `data/artigos-recomendation.md` caso o conteúdo não venha completo no envelope.
   - Extrair a lista de artigos com título, link e assuntos cobertos.
   - Se o arquivo estiver vazio ou sem artigos, retornar erro "Nenhum artigo recomendado encontrado. Execute primeiro o buscador-artigo."

2. **Verificar disponibilidade de cada artigo**
   - Para cada artigo da lista, executar:
     ```sh
     firecrawl scrape <url> --format markdown
     ```
   - Se o scrape retornar conteúdo, marcar o artigo como **disponível** e armazenar o conteúdo extraído para uso posterior.
   - Se o scrape falhar (timeout, 404, erro de rede), marcar como **indisponível** e registrar o motivo do erro.
   - Se todos os artigos estiverem indisponíveis, retornar erro "Todos os artigos estão indisponíveis. Sugerimos executar novamente o buscador-artigo."

3. **Apresentar lista ao usuário**
   - Listar todos os artigos no seguinte formato:
     ```text
     Artigos encontrados:

     1. [título do artigo]
        link: [URL]
        assuntos: [assunto1, assunto2]
        status: disponível / indisponível

     2. [título do artigo]
        link: [URL]
        assuntos: [assunto1, assunto2]
        status: disponível / indisponível
     ```

4. **Coletar seleção do usuário**
   - Perguntar: "Quais artigos você deseja investigar? (informe os números separados por vírgula, ou 'todos' para todos os disponíveis)"
   - Se o usuário não selecionar nenhum artigo, encerrar o fluxo e retornar ao menu.

5. **Coletar formato desejado**
   - Perguntar: "Qual formato você prefere?"
     - **A) Resumo rápido** — um parágrafo curto explicando o que o artigo ensina, os pontos principais e a conclusão.
     - **B) Passo a passo** — um guia detalhado reproduzindo as instruções do artigo, com etapas numeradas e exemplos de código quando aplicável.

6. **Gerar conteúdo para cada artigo selecionado**
   - Usar o conteúdo já obtido no scrape do Passo 2.
   - Se o formato escolhido for **resumo rápido**:
     - Gerar um resumo de 3 a 5 frases cobrindo: tema principal, pontos-chave e conclusão.
   - Se o formato escolhido for **passo a passo**:
     - Extrair as etapas práticas do artigo.
     - Organizar em passos numerados.
     - Incluir trechos de código ou comandos quando presentes no artigo original.
     - Adicionar observações ou dicas relevantes.

7. **Gravar resultados**
   - Usar `write_file` para gravar `data/artigos-filtrados.md` no seguinte formato:
     ```text
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

8. **Montar resposta**
   - Retornar envelope de resposta com os campos `sucesso`, `resumo`, `dados` e `erros`.
   - Incluir o campo `erros` com mensagens claras se houver falhas nos scrapes.

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
- Falha ao ler `data/artigos-recomendation.md`: registrar erro e abortar.
- Arquivo vazio ou sem artigos: retornar erro orientando executar primeiro o `buscador-artigo`.
- Falha no `firecrawl scrape` de um artigo: marcar como indisponível, registrar motivo e continuar com os demais.
- Todos os artigos indisponíveis: retornar `sucesso: false` com erro sugerindo executar novamente o `buscador-artigo`.
- Usuário não seleciona nenhum artigo: encerrar fluxo e retornar ao menu.
- Qualquer exceção inesperada: capturar, registrar em `erros` e definir `sucesso: false`.
