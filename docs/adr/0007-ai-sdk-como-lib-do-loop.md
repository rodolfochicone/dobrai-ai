# AI SDK 7 como camada multi-provedor do loop, com preços em tabela própria

O loop próprio ([ADR-0003](0003-loop-de-agente-proprio.md)) precisava de uma lib multi-provedor. Comparamos o AI SDK 7 (`ai`, Apache-2.0) com o `@earendil-works/pi-ai` que o Craft usa. O pi-ai traz tabela de preços embutida, mas ainda está na série 0.x, com remoções incompatíveis frequentes; carrega os SDKs de OpenAI, Anthropic, Google e Bedrock; e não tem cliente MCP. Escolhemos o AI SDK 7, por quatro razões: tem semver, um loop de ferramentas com vários passos (`stopWhen`), `usage` com tokens de cache, controle de cache por bloco na Anthropic e cliente MCP próprio (`@ai-sdk/mcp`, Streamable HTTP), que também serve às Integrações. O custo é calculado pela nossa tabela `llm.preco_modelo`, com fonte e data. Isso seria necessário de qualquer jeito para cobrar: o catálogo do pi mudaria a cada atualização do pacote.

## Consequences

- `packages/agent-loop` é a única parte do código que importa `ai`. Trocar de lib fica restrito a esse pacote e ao `llm.modelo(conexaoId)`.
- Rodar sob Bun não está documentado pelo AI SDK. A lib usa `fetch` e ESM, então deve funcionar, mas isso é confirmado na F1.5 com um teste de ponta a ponta contra um provedor real.
