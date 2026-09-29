# Assinaturas de consumo só via MCP em clientes oficiais

A ideia era o Membro construir Agentes com a própria assinatura do Claude ou do ChatGPT. Três fatos pesaram contra oferecer isso dentro do Dobrai:

- A Anthropic proíbe terceiros de oferecer login Claude.ai e de "collect, store, or intermediate Claude.ai credentials or session tokens".
- A OpenAI proíbe disponibilizar a conta a terceiros.
- Os dois planos têm limites por janela de 5 horas.

Decidimos expor o Construtor como MCP remoto (Streamable HTTP, com OAuth pelo plugin `mcp` do Better Auth). O Membro conecta esse MCP no Claude Code, no Codex ou nos conectores do Claude.ai, logado pelo fluxo oficial do provedor. O Dobrai nunca recebe nem armazena um token de assinatura, e o Agente publicado usa apenas a Chave da Organização.

## Consequences

- Pelo MCP, o Construtor edita o Rascunho e roda o Playground. Publicar é sempre um clique humano no console.
- O desktop próprio foi adiado. Além da zona cinzenta nos termos, assinar os builds custaria ~US$ 20–33/mês, porque empresas brasileiras não são elegíveis ao Azure Artifact Signing. Fontes consultadas em 28/09/2026: [Apple Developer Program](https://developer.apple.com/support/enrollment/) (US$ 99/ano), [Azure Artifact Signing](https://azure.microsoft.com/en-us/products/artifact-signing) (elegibilidade), [opções de assinatura no Windows](https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/code-signing-options) (certificado OV de US$ 150–300/ano).
- A escrita via MCP no ChatGPT web em plano Plus ainda precisa de verificação manual.
