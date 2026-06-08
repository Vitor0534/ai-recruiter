# Skill: dispatch

## Objetivo
Construir e enviar envelopes de despacho para agentes especializados (ex.: Scout) e processar suas respostas.

## Entrada
- `agente` – nome do agente a ser chamado (ex.: "scout").
- `tarefa` – descrição da tarefa que o agente deve executar.
- `perfil_usuario` – conteúdo do arquivo `data/user-profile.md`.
- `contexto` – objeto contendo `area`, `localizacao`, `nivel_experiencia` e `habilidades`.

## Passos
1. **Montar Envelope de Despacho** conforme o modelo definido no plano:
   ```
   ## DESPACHO: {AGENTE}
   ### referencia_persona
   [conteúdo da persona do agente]
   ### tarefa
   {tarefa}
   ### perfil_usuario
   {perfil_usuario}
   ### contexto
   Area: {area}
   Localizacao: {localizacao}
   Nivel: {nivel_experiencia}
   Habilidades: {habilidades}
   ### saida_esperada
   Envelope de resposta com estado, resumo, dados e erros.
   ```
2. **Chamar `spawn_agent`** com o label descriptivo e o envelope como mensagem.
3. **Aguardar resposta** do agente.
4. **Interpretar resposta**:
   - Se `sucesso: true`, extrair a seção `dados`.
   - Se `sucesso: false`, coletar os erros e repassar ao orquestrador.
5. **Retornar** ao agente chamador (master‑recruiter) o conteúdo relevante.

## Saída
- Objeto JSON contendo `sucesso`, `erros` (lista) e `dados` (texto formatado).

## Tratamento de Erros
- Falha ao ler a persona: registrar erro e abortar.
- `spawn_agent` falha: incluir mensagem no campo `erros` e definir `sucesso: false`.
- Resposta fora do formato esperado: registrar erro e definir `sucesso: false`.
