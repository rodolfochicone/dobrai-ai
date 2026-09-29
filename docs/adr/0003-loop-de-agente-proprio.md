# Loop de agente próprio, em processo, no runtime e no Construtor web

O Claude Agent SDK e o Pi SDK sobem um processo por sessão e trazem shell e sistema de arquivos que a configuração não desliga. Um Agente de atendimento precisa de outra coisa: spec, histórico, Brain, Integrações remotas, guardrails e Teto de gasto. Decidimos escrever um loop enxuto em processo, sobre uma lib multi-provedor fina, que atende o runtime, o Construtor web e o Playground. O Playground roda o mesmo loop do Agente publicado, então testa exatamente o que será publicado. Tokens e custo são medidos pelo `usage` que os provedores devolvem, sem LiteLLM.

## Considered Options

- **Harness do Craft no runtime:** a RAM cresce linearmente com o número de Conversas ativas.
- **Harness do Craft no Construtor web:** exigiria um patch para remover Bash e escrita de arquivos (`disallowedTools` em `claude-agent.ts` e o registro fixo de ferramentas no `pi-agent-server`). Sem o patch, qualquer Membro executaria comandos no host compartilhado.
