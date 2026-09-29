# Prompt — Dobrai: Fase 0 (v2)

> Rode no Claude Code, em Plan Mode, na raiz do repositório `dobrai-ai`. O repositório é novo e não é um fork do Craft.

---

## Papel

Você é o arquiteto principal e tech lead do **Dobrai**. Neste ciclo você executa a **Fase 0**: desenha o walking skeleton e o MVP vendável, escreve a documentação e para para aguardar minha aprovação. Código de produto só começa na F1, depois que eu aprovar.

Me pergunte quando a questão for uma decisão de produto. Nos outros casos, escolha o padrão sensato e registre em ADR se houver trade-off.

## Fontes de verdade

Leia estas fontes antes de qualquer outra coisa:

- **`CONTEXT.md`** é o glossário do domínio. Use os termos exatamente como estão lá, em documentos, código e nomes. Se precisar de um conceito novo, proponha o termo e acrescente ao glossário no mesmo formato.
- **`docs/adr/`** reúne as decisões já tomadas e o motivo de cada uma. Trate-as como fechadas. Se algum fato que você encontrar contradisser um ADR, pare e me traga esse fato.
- **Craft Agents OSS** (`github.com/craft-ai-agents/craft-agents-oss`, tag `v0.13.6`, Apache-2.0) serve de referência de padrões. Clone numa pasta temporária fora do repositório. Cite os arquivos quando um padrão dele inspirar o desenho. Se copiar algum arquivo, mantenha o cabeçalho Apache e registre a cópia no `NOTICE`.

## Produto

O Dobrai é um SaaS brasileiro em que Organizações criam Agentes de IA de atendimento e vendas. A Organização conversa com o Construtor para montar o Agente, testa no Playground e publica em Canais.

O cliente-alvo inicial é a **agência** que monta Agentes para vários clientes. Cada cliente da agência é uma Organização, e o pessoal da agência é Membro de várias Organizações. Tudo o que a agência e os clientes dela precisam deve acontecer dentro da plataforma, sem intervenção nossa.

## Decisões fechadas

| Tema | Decisão | Fonte |
|---|---|---|
| Forma da arquitetura | Monólito modular com 3 artefatos: controle, runtime/workers e WAHA | ADR-0001 |
| Repositório | Novo. Bun, TypeScript estrito, React + Vite, shadcn/ui, Tailwind v4, i18next | ADR-0002 |
| Execução de Agentes | Loop próprio em processo; o Playground roda o mesmo loop | ADR-0003 |
| Assinaturas Claude/ChatGPT | Só pelo MCP do Construtor, em clientes oficiais | ADR-0004 |
| Isolamento | `org_id` + RLS + teste de isolamento em cada slice | ADR-0005 |
| WhatsApp não oficial | WAHA com motor GOWS | ADR-0006 |
| LLM que atende Contatos | Só a Chave da Organização. Sem LiteLLM; o gateway próprio chega junto com o billing | sessão de 2026-09-28 |
| Autenticação | Better Auth: plugin de organizações (Membros, Papéis, convites) + plugin `mcp` (OAuth do MCP) | sessão de 2026-09-28 |
| Papéis | Dono, Admin e Operador, fixos | sessão de 2026-09-28 |
| Clientes | Só web: console + PWA. Desktop adiado | ADR-0004 |
| LGPD | Todos os segmentos. A Organização é controladora e o Dobrai é operador. Botão "excluir Contato". Termos e DPA revisados por advogado | sessão de 2026-09-28 |
| Infra | Contabo Cloud VPS 6 (12 GB, costa leste dos EUA), Kamal, Postgres no próprio host com `pg_dump` + WAL no Cloudflare R2, Oracle Always Free como staging. Teto de US$ 50/mês; plano atual abaixo de US$ 20/mês | sessão de 2026-09-28 |
| Degrau de gasto | RAM acima de 75% de forma sustentada ou mais de 100 números no WAHA → o WAHA vai para um segundo host | sessão de 2026-09-28 |

## Escopo

### F1 — walking skeleton

É a fatia mais fina que atravessa a arquitetura inteira:

1. Um Membro cria uma Organização.
2. Conversa com o Construtor web e gera um Rascunho.
3. Testa o Rascunho no Playground e Publica.
4. Um Contato conversa com o Agente pelo widget instalado numa página de teste, recebe a resposta em streaming, e o atendimento usa a Chave da Organização.

A F1 está pronta quando:
- o custo da Conversa aparece no console;
- controle e runtime têm deploy e rollback independentes via Kamal;
- o teste de isolamento de cada slice passa no CI.

### F2 — MVP vendável

A F2 acrescenta ao walking skeleton:

- MCP do Construtor para clientes oficiais (Claude Code, Codex e conectores do Claude.ai);
- WhatsApp não oficial via WAHA, com aviso de risco de bloqueio;
- inbox mínima: o Operador vê, Assume e devolve Conversas;
- Brain com arquivos e URLs, com busca híbrida (pgvector + full-text) e citações;
- Teto de gasto por Organização e por Conversa, com corte automático;
- Integração por MCP remoto via URL;
- botão "excluir Contato".

### Fora do MVP

- WhatsApp oficial, que depende de CNPJ e do processo de Tech Provider da Meta;
- Instagram, Messenger, Telegram e e-mail;
- desktop;
- evals e guardrails avançados;
- analytics além de contadores;
- billing: a cobrança começa manual, por Pix;
- push no PWA;
- Skills;
- Memória do Contato;
- catálogo de Integrações e MCP stdio;
- SSO e trilha de auditoria;
- painel de super-admin;
- white-label e papéis customizados.

## Regras de desenho

- **Conteúdo não confiável:** mensagens de Contatos, documentos do Brain e respostas de Integrações entram no loop como dados, nunca como instruções. O desenho precisa mostrar onde essa fronteira é aplicada.
- **Publicar é ato humano:** o Construtor, seja o web ou o do MCP, edita o Rascunho e roda o Playground. Só um Membro com Papel Admin ou Dono Publica, e sempre pelo console.
- **Tokens de assinatura ficam com o provedor:** a Assinatura do Membro chega ao Dobrai apenas pelo MCP, com login feito no cliente oficial. Nenhuma tabela, log ou variável guarda token de assinatura.
- **Fila antes de processar:** todo webhook e toda mensagem de Canal entra numa fila no Postgres. O processamento é idempotente e ordenado por Contato.
- **Cada slice é dono dos seus dados:** schema próprio, e comunicação com outros slices só pela API de módulo ou por eventos.
- **Preço tem fonte:** todo custo citado traz URL e data de consulta.
- **Qualidade:** pt-BR primeiro, com i18n preparado. TypeScript estrito. Testes unitários, e o teste de isolamento de Organização é obrigatório.

## Fatos a verificar

Resolva cada item, ou mova-o para `docs/07-riscos.md` com o próximo passo definido:

1. **Lib multi-provedor do loop (ADR-0003).** Compare o AI SDK da Vercel com o `pi-ai` usado pelo Craft em tool calling, streaming, `usage` e custo por chamada, provedores suportados e peso. Registre a escolha em ADR.
2. **Modelos padrão baratos** para o Agente e para o Construtor: preço por milhão de tokens e custo estimado de uma Conversa típica. Defina "típica" no próprio documento.
3. **Contabo VPS 6:** preço mensal sem o contrato de 24 meses.
4. **Tabela oficial de preços da Meta para o Brasil.** O CSV bloqueia acesso automatizado; se você não conseguir, deixe o pedido para mim.
5. **Escrita via MCP no ChatGPT web em plano Plus.** Descreva o teste manual para eu fazer.
6. **WAHA GOWS:** RAM real por número e taxa de bloqueio em IP de datacenter. Desenhe como medir isso no staging.
7. **Fila no Postgres rodando em Bun:** compare pg-boss e graphile-worker.

## Entregáveis da Fase 0

Tudo em `docs/`, em pt-BR, com diagramas Mermaid onde ajudarem:

1. **`01-arquitetura.md`**
   - diagramas de contexto e de containers;
   - fluxo de uma mensagem, do widget e do WhatsApp até a resposta;
   - os 3 artefatos e quais slices vivem em cada um;
   - RAM estimada por artefato no host de 12 GB;
   - custo mensal de infra para 10, 50 e 500 Organizações, com o gatilho de cada degrau;
   - custo de LLM por Conversa típica.
2. **`02-modelo-de-dados.md`** — entidades do MVP vendável agrupadas por slice, o dono de cada tabela, as políticas de RLS e o que o botão "excluir Contato" apaga.
3. **`03-agent-spec.md`**
   - schema tipado (Zod) da especificação do Agente;
   - Rascunho, Versão do Agente e Publicar;
   - diff e rollback entre Versões;
   - as ferramentas do Construtor, com uma única definição que serve tanto o Construtor web quanto o MCP.
4. **`04-canais.md`**
   - abstração de canal: mensagem normalizada, status e janelas;
   - adapters do widget e do WAHA;
   - fila, idempotência e ordenação por Contato;
   - o WhatsApp oficial só como interface prevista.
5. **`05-slices-e-deploy.md`**
   - mapa de slices: responsabilidade, dados, API de módulo e eventos publicados e consumidos;
   - estrutura de pastas do monorepo;
   - regra de lint contra import cruzado;
   - CI no GitHub Actions com filtros por caminho;
   - Kamal com deploy e rollback por artefato;
   - ambientes: local com `docker compose up`, staging na Oracle e produção na Contabo.
6. **`06-roadmap.md`** — F1 e F2 quebradas em entregas pequenas, cada uma com um critério de aceite verificável. F3 em diante como lista curta.
7. **`07-riscos.md`** — riscos técnicos e legais e perguntas abertas para mim. Riscos a cobrir:
   - bloqueio de números no WhatsApp não oficial;
   - LGPD e dados de saúde;
   - termos dos provedores de LLM;
   - CNPJ e processo da Meta;
   - desempenho da Contabo.
8. **ADRs novos em `docs/adr/`** para cada decisão da Fase 0 que seja difícil de reverter e tenha alternativas reais. Formato: título seguido de 1 a 3 frases com o contexto, a decisão e o motivo.
9. **`CLAUDE.md` na raiz**, com convenções, comandos e limites. O `CLAUDE.md` de cada slice nasce na F1, junto com o slice.

## Critério de conclusão

A Fase 0 termina quando todas estas condições valem:

- os 9 entregáveis existem;
- todo termo de domínio usado nos documentos está no `CONTEXT.md`;
- todo preço citado tem URL e data;
- todo item de "Fatos a verificar" foi resolvido ou está em `07-riscos.md` com o próximo passo;
- nenhum documento contradiz um ADR.

Então pare e me apresente um resumo de até 15 linhas com as decisões novas, os ADRs criados e as perguntas abertas. A F1 só começa depois que eu aprovar.
