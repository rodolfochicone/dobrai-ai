# 07 — Riscos e perguntas abertas

## Riscos técnicos

| # | Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|---|
| T1 | Número de cliente bloqueado no WhatsApp não oficial | média | alto | aviso de risco aceito na conexão; Agente só responde; lido e "digitando"; medição no staging (abaixo); WAHA num host no Brasil se o IP dos EUA pesar; oferecer o WhatsApp oficial (F2.10) a quem não pode arriscar o número |
| T2 | Mudança de protocolo do WhatsApp quebra o WAHA (passkey e LID em 2026) | alta | médio | versão do WAHA fixa, testada no staging antes da produção; alerta de Canal desconectado; atualização mensal |
| T3 | Contato travado na fila (`key_strict_fifo`) | média | médio | turno converte erro não transitório em `aoFalhar`; alerta sobre `getBlockedKeys()` ([ADR-0008](adr/0008-pg-boss-fila-por-contato.md)) |
| T4 | AI SDK ou transformers.js com problema sob Bun | baixa | médio | teste de ponta a ponta na F1.5 e na F2.4; alternativa: o processo afetado roda com Node na mesma imagem |
| T5 | Busca vetorial perde recall com RLS e muitas Organizações | média | médio | pgvector ≥ 0.8 com `hnsw.iterative_scan`; recall medido no teste do slice `brain` |
| T6 | Um único servidor: queda = parada total | média | alto | aceito na fase bootstrap. RPO ≤ 5 min (WAL no R2), RTO ≤ 2 h (restaurar num VPS novo com Kamal); teste de restauração mensal |
| T7 | CPU compartilhada da Contabo com desempenho variável | média | médio | monitorar steal time e latência do turno; o Kamal permite mudar de provedor sem mudar código |
| T8 | Prompt injection de Contato, documento ou Integração leva o Agente a usar ferramenta indevida | média | alto | só ferramentas liberadas por Agente; conteúdo externo em bloco `<dados>` saneado; limite de chamadas de ferramenta por turno; Teto de gasto. Aprovação humana de ação fica para depois (risco aceito) |
| T9 | Abuso no widget queima a chave do cliente | média | médio | limite por visitante e por IP; Teto por Conversa e por Organização |
| T10 | Modelo padrão descontinuado (Haiku 4.5 tem data mínima em 15/10/2026) | alta | baixo | nenhum padrão depende do Haiku 4.5; `llm.preco_modelo` com data de vigência |
| T11 | Volume de sessões do WAHA perdido | baixa | médio | backup diário do volume no R2; no pior caso, cada Organização lê o QR code de novo; Conversas ficam no Postgres |
| T12 | Timeout do kamal-proxy corta o SSE do widget | média | baixo | `response_timeout` maior na rota do SSE, keep-alive a cada 15 s, reconexão automática no widget; testar na F1.7 |

## Riscos legais e de negócio

| # | Risco | Mitigação |
|---|---|---|
| L1 | Dados de saúde e outros dados sensíveis (LGPD, art. 11) passando por LLM fora do país | Organização como controladora e Dobrai como operador, no DPA; botão excluir Contato; o console mostra onde cada provedor processa os dados. Termos e DPA revisados por advogado antes do primeiro cliente pagante (F2.9) |
| L2 | Provedores com transferência para países sem adequação (ex.: DeepSeek, China) | o console não sugere DeepSeek como padrão; a Organização que escolher vê o aviso |
| L3 | Termos dos provedores de LLM | Assinaturas só pelo MCP em clientes oficiais ([ADR-0004](adr/0004-assinaturas-so-via-mcp-em-clientes-oficiais.md)); a Chave da Organização roda sob o contrato da própria Organização com o provedor |
| L4 | WhatsApp não oficial viola os termos do WhatsApp | aviso explícito na conexão e nos termos do Dobrai; o cliente assume o risco do número |
| L5 | Aprovação da Meta como Tech Provider demora ou é negada | CNPJ já existe: iniciar Business Verification e App Review já, em paralelo à F1; o WhatsApp não oficial segue como canal vendável até lá |
| L6 | Marca "Dobrai" sem registro | o domínio `dobrai.ai` já está registrado, mas domínio não protege a marca; busca e depósito no INPI |
| L7 | WAHA muda a licença de novo (recursos pagos viraram gratuitos em 2026.6) | versão fixa; o adaptador isola o WAHA do resto do código, e Baileys direto é o plano B |

## Fatos a verificar (situação ao fim da Fase 0)

| # | Fato | Situação |
|---|---|---|
| 1 | Lib multi-provedor do loop | resolvido: AI SDK 7 ([ADR-0007](adr/0007-ai-sdk-como-lib-do-loop.md)) |
| 2 | Modelos padrão e custo por Conversa típica | resolvido em [01](01-arquitetura.md) |
| 3 | Preço mensal da Contabo sem contrato de 24 meses | resolvido: VPS 6 na costa leste por US$ 12,00/mês ([01](01-arquitetura.md)); imposto no checkout não confirmado |
| 4 | Tabela oficial de preços da Meta para o Brasil | resolvido ([04](04-canais.md)); atenção à mudança de 01/10/2026 nas mensagens de atendimento |
| 5 | Escrita via MCP no ChatGPT web (Plus) | teste manual abaixo, para você fazer |
| 6 | RAM real do WAHA por número e taxa de bloqueio | plano de medição abaixo |
| 7 | Fila no Postgres sob Bun | resolvido: pg-boss ([ADR-0008](adr/0008-pg-boss-fila-por-contato.md)) |
| 8 | Preço de transcrição de áudio por provedor (OpenAI, Google) | pendente: pesquisar no início da F2.3 e registrar em `llm.preco_modelo` |

### Teste manual: MCP do Construtor no ChatGPT Plus (F2.1)

1. Com o MCP do Construtor no staging, numa conta ChatGPT Plus: Configurações → Apps e conectores → Avançado → ligar o modo desenvolvedor.
2. Criar um conector com a URL do MCP do staging e fazer o login OAuth do Dobrai.
3. Pedir "liste meus agentes" e depois "mude o tom de voz do Agente X para mais formal".
4. Passa se aparecer uma linha nova em `agentes.alteracao_rascunho` com `origem = construtor_mcp`. Se o plano Plus não permitir escrita, o site do Dobrai lista o ChatGPT como "somente leitura" até isso mudar.

### Medição do WAHA (staging, antes do primeiro cliente)

- **Montagem:** 3 a 5 chips pré-pagos, metade conectada a um WAHA no staging (IP brasileiro, Oracle São Paulo) e metade a um WAHA na Contabo (IP dos EUA).
- **Carga:** conversas reais de entrada vindas de outros aparelhos, durante 2 a 4 semanas, em ritmo de atendimento (poucas mensagens por hora, picos no horário comercial).
- **Métricas:** RSS por sessão em repouso e com carga (`docker stats`), reconexões, restrições e bloqueios por grupo de IP.
- **Decisão:** se o grupo dos EUA tiver mais restrições que o do Brasil, o contêiner do WAHA vai para um host brasileiro (Oracle São Paulo ou Magalu) e o resto fica onde está.

## Perguntas abertas para você

1. **Preço do Dobrai para agências.** Ainda indefinido. Não é decisão técnica, mas bloqueia a primeira venda, mesmo com cobrança manual por Pix.

## Respondidas depois da Fase 0 (28/09/2026)

| Pergunta | Resposta | Onde entrou |
|---|---|---|
| CNPJ existe? | sim | processo de Tech Provider começa já; WhatsApp oficial na F2.10, depois da aprovação ([06](06-roadmap.md)) |
| Agente chama Operador? | sim, Transfere sozinho | `transferir_para_operador` e `transferencia` na spec ([03](03-agent-spec.md), [04](04-canais.md)); F2.3 |
| Transcrição de áudio? | por clique | botão Transcrever na inbox; o Agente responde `mensagens.aoReceberAudio` ([04](04-canais.md)); F2.3 |
| Domínio | `dobrai.ai` já registrado | `app.dobrai.ai` e `chat.dobrai.ai` ([05](05-slices-e-deploy.md)); registro da marca no INPI continua pendente (L6) |
