# Dobrai

SaaS em que Organizações criam Agentes de IA de atendimento com um Construtor e os publicam em Canais (widget, WhatsApp). Monólito modular em Bun + TypeScript estrito, Postgres, um VPS.

## Leia conforme a tarefa

- **Vocabulário:** `CONTEXT.md`. Nomes de tabela, tipo, função e rota usam esses termos. Conceito novo → proponha o termo e acrescente lá.
- **Decisões fechadas:** `docs/adr/`. Um fato que contradiga um ADR vira pergunta, não mudança silenciosa.
- **Arquitetura e fluxo de mensagem:** `docs/01-arquitetura.md`.
- **Tabelas, RLS e dono de cada dado:** `docs/02-modelo-de-dados.md`.
- **Spec do Agente e ferramentas do Construtor:** `docs/03-agent-spec.md`.
- **Canais, fila e idempotência:** `docs/04-canais.md`.
- **Slices, pastas, CI e deploy:** `docs/05-slices-e-deploy.md`.
- **O que vem a seguir:** `docs/06-roadmap.md`. **Riscos e perguntas abertas:** `docs/07-riscos.md`.

Ao trabalhar dentro de um slice, o `CLAUDE.md` dele é o ponto de partida; ele nasce na F1 junto com o slice.

## Regras

- **Fronteira de slice.** Outro slice só é alcançado pela API de módulo (`exports` do pacote) ou por eventos na fila. Cada slice usa o próprio papel de banco e só enxerga o próprio schema.
- **Isolamento.** Toda tabela de dados de Organização tem `org_id` e RLS; toda transação define `app.org_id`. Todo slice novo nasce com o teste de isolamento.
- **Conteúdo não confiável.** Mensagem de Contato, trecho do Brain e resposta de Integração entram no loop como dados, dentro do bloco saneado descrito em `docs/03-agent-spec.md`.
- **Publicar é humano.** O Construtor edita Rascunhos; Publicar existe só no console.
- **Segredos.** Chaves e credenciais vivem cifradas no banco e em claro só em memória, durante a chamada. Tokens de Assinatura do Membro nunca chegam ao Dobrai.
- **Preços com fonte.** Todo custo citado em documento ou tabela de preços traz URL e data.
- **Idioma.** Produto em pt-BR, com i18n preparado. Identificadores de domínio em português, como no glossário; os técnicos (db, http, crypto) em inglês.

## Comandos

Os scripts ficam no `package.json` da raiz e de cada slice, definidos na F1.1. Consulte-os lá.
