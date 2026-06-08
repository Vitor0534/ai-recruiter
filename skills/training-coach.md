# Skill: training-coach

## Objetivo
Implementar o fluxo completo do agente **training‑coach**: coletar dados do estudante, validar, persistir em `data/perfil-do-estudante.md`, gerar plano de estudos estruturado e salvar em `data/planos-de-estudo.md`.

## Entrada
O envelope de despacho enviado pelo **master‑recruiter** deve conter:

- `tecnologia`: nome da tecnologia a ser estudada.
- `horas_por_dia`: número de horas que o estudante pode dedicar diariamente.
- `duracao_semanais`: número de semanas para o plano.
- `foco`: foco de estudo (ex.: teoria, prática, projetos).

## Passos
1. **Validação**
   - Verificar que `horas_por_dia` e `duracao_semanais` são números positivos.
   - Se inválido, responder com envelope de erro solicitando correção.

2. **Persistir perfil**
   - Criar/overwrite `data/perfil-do-estudante.md` com o conteúdo estruturado:
     ```markdown
     Tecnologia: <tecnologia>
     Horas por dia: <horas_por_dia>
     Duração do plano: <duracao_semanais> semanas
     Foco: <foco>
     ```

3. **Gerar plano**
   - Calcular total de horas de estudo: `horas_por_dia * duracao_semanais * 7`.
   - Dividir em blocos de estudo diários, criando uma lista de tópicos.
   - Para simplificação, usar um placeholder de tópicos baseado na tecnologia (ex.: "React – Conceitos Básicos", "React – Componentes", etc.).
   - Criar arquivo `data/planos-de-estudo.md` com estrutura:
     ```markdown
     Data de Geração: <YYYY-MM-DD HH:MM>
     Plano:
     1. Dia 1 – <horas_por_dia>h: <Tópico 1> (<link>)
     2. Dia 2 – <horas_por_dia>h: <Tópico 2> (<link>)
     ...
     ```

4. **Responder**
   - Retornar envelope de resposta:
     ```text
     ## RESPOSTA: TRAINING‑COACH
     ### estado
     sucesso: true
     ### resumo
     Plano de estudos gerado e salvo em data/planos-de-estudo.md.
     ### dados
     <conteúdo do arquivo planos-de-estudo.md>
     ### erros
     []
     ```

## Tratamento de Erros
- Falha ao escrever arquivo: incluir mensagem de erro no campo `erros` e definir `sucesso: false`.
- Dados inválidos: solicitar correção e não gerar plano.
