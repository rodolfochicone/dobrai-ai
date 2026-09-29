# Repositório novo, com o Craft Agents só como referência

O plano era partir de um fork do craft-agents-oss. O mapeamento da v0.13.6 mostrou que o MVP não aproveitaria quase nada:

- o harness sobe um processo por sessão;
- o server é single-user;
- a persistência é em arquivos;
- o messaging-gateway serve ao dono do agente, não a clientes finais;
- a UI depende de `window.electronAPI` em 598 pontos.

O fork traria 2.025 arquivos mortos, rebranding obrigatório de 254 arquivos pela política de marca e um fluxo de login do Claude que viola os termos da Anthropic (ver ADR-0004). Decidimos por um repositório novo, na mesma stack: Bun workspaces, TypeScript estrito, React + Vite, shadcn/ui, Tailwind v4 e i18next.

## Consequences

- Todo arquivo copiado do Craft mantém o cabeçalho Apache-2.0 e ganha atribuição no `NOTICE`.
- O desktop, quando vier, nasce como fork do Craft num repositório separado e se conecta ao Dobrai pelo MCP do Construtor. O rebranding e a sincronização com o upstream ficam lá.
- O upstream publica releases como despejos de um bot (~55 mil linhas em 90 dias), sem histórico granular. Acompanhar é por diff entre tags.
