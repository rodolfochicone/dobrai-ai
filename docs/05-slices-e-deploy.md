# 05 — Slices e deploy

Monólito modular com três artefatos ([ADR-0001](adr/0001-monolito-modular-com-tres-artefatos.md)). Todo o código de domínio mora em slices; os artefatos só montam rotas e workers.

## Mapa de slices

| Slice | Responsabilidade | Schema | API de módulo (principal) | Eventos e jobs |
|---|---|---|---|---|
| `identidade` | Usuários, Organizações, Membros, Papéis, convites, OAuth do MCP | `identidade` (Better Auth) | `sessaoAtual(req)`, `papelNaOrganizacao(orgId, usuarioId)` | — |
| `agentes` | Agente, Rascunho, Versão, Publicar, rollback, diff | `agentes` | `lerRascunho`, `alterarRascunho`, `publicar`, `versaoPublicada(agenteId)` | publica `agente.publicado` |
| `construtor` | Sessão do Construtor web, ferramentas do Construtor, servidor MCP | `construtor` | `ferramentasDoConstrutor` | — |
| `llm` | Chave da Organização, preços, `evento_uso`, Teto de gasto, transcrição | `llm` | `modelo(conexaoId)`, `registrarUso`, `verificarTeto`, `transcrever(conexaoId, audio)` | — |
| `runtime` | Executa o turno do Agente: compila a spec, busca no Brain, chama ferramentas, respeita tetos | `runtime` | `executarTurno` | consome job `responder` |
| `conversas` | Contato, Conversa, mensagens, inbox (Transferir, Assumir, devolver, transcrever áudio), excluir Contato | `conversas` | `registrarMensagemRecebida`, `registrarResposta`, `transferir`, `assumir`, `devolver`, `transcreverAudio`, `excluirContato` | publica `contato.excluido`, `conversa.transferida`, `conversa.assumida`, `conversa.devolvida`; agenda `responder` e `encerrar_por_inatividade` |
| `canais` | Canal, roteamento, adaptadores (widget, WAHA), ingress | `canais` | `enviar`, `indicarDigitando`, `marcarComoLida` | consome job `enviar` e `contato.excluido` |
| `brain` | Fontes de conhecimento, extração, trechos, embeddings, busca híbrida | `brain` | `buscar(orgId, agenteId, consulta)` | consome job `indexar_fonte` |
| `integracoes` | Integrações por MCP remoto, lista de ferramentas | `integracoes` | `ferramentasDoAgente(spec.integracoes)` | consome job `verificar_integracao` |

## Pastas

```text
dobrai-ai/
├─ apps/
│  ├─ controle/        # Hono: rotas de todos os slices para Membros, MCP, console estático
│  ├─ execucao/        # Hono: widget.js, ingress de Canais (widget, webhook do WAHA) + workers da fila
│  └─ console/         # SPA React + Vite, instalável como PWA; compõe as telas exportadas pelos slices
├─ slices/
│  └─ <slice>/
│     ├─ package.json  # "exports": ./modulo, ./eventos, ./rotas, ./workers, ./ui
│     ├─ CLAUDE.md
│     ├─ migrations/   # SQL (dbmate), inclusive papel, grants e políticas de RLS
│     ├─ src/
│     └─ test/         # inclui isolamento.test.ts
├─ packages/
│  ├─ agent-loop/      # loop sobre o AI SDK; sem banco, sem estado
│  ├─ contratos/       # schemas Zod dos eventos e jobs, versionados
│  ├─ db/              # pool por papel, transação com app.org_id, helper do teste de isolamento
│  ├─ cripto/          # envelope encryption
│  ├─ ui/              # componentes shadcn/ui compartilhados
│  └─ i18n/            # i18next, pt-BR
├─ infra/
│  ├─ compose.yml      # dev local: Postgres (pgvector), WAHA
│  └─ kamal/           # deploy.controle.yml, deploy.execucao.yml
└─ docs/
```

## Fronteiras

- **O pacote é a fronteira.** Cada slice é um pacote do workspace; o campo `exports` publica só `modulo`, `eventos`, `rotas`, `workers` e `ui`. Importar um arquivo interno de outro slice quebra a resolução do módulo (Bun e TypeScript com `moduleResolution: "bundler"` respeitam `exports`).
- **Slice só importa `modulo` e `eventos` de outro slice.** `rotas`, `workers` e `ui` são para os apps. Um script de CI (`scripts/checar-fronteiras.ts`) lê os imports de `slices/**` e falha se a regra quebrar.
- **O banco confirma a fronteira.** Um papel por slice, com acesso só ao próprio schema ([02](02-modelo-de-dados.md)). Cada processo abre um pool pequeno por slice (máximo 3 conexões): 9 slices × 2 artefatos × 3 = 54 conexões, abaixo do padrão de 100 do Postgres.
- **Pacotes compartilhados não conhecem domínio.** `agent-loop`, `db`, `cripto`, `ui` e `i18n` não importam nada de `slices/`. `contratos` só tem schemas de eventos e jobs.

## Contratos

- **API de módulo** é TypeScript: mudança incompatível quebra o typecheck do slice que consome. É o teste de contrato de graça que o monólito dá.
- **Eventos e jobs** vivem na fila e sobrevivem a um deploy. Cada payload tem schema Zod em `packages/contratos` com sufixo de versão (`responder.v1`). Campo novo opcional mantém a versão; mudança incompatível cria `v2`, e o consumidor aceita as duas até a fila da antiga esvaziar. Cada consumidor tem teste com um exemplo de cada versão que aceita.
- **Evento na mesma transação.** A fila (pg-boss) está no mesmo Postgres, então o evento é criado na transação que grava o dado. Não há outbox separado.

## Fila

pg-boss com a política `key_strict_fifo` ([ADR-0008](adr/0008-pg-boss-fila-por-contato.md)):

| Fila | Chave (`singletonKey`) | Para quê |
|---|---|---|
| `responder` | `contato_id` | turno do Agente; FIFO por Contato, Contatos em paralelo |
| `enviar` | `contato_id` | entrega pelo adaptador, na ordem certa |
| `indexar_fonte` | `fonte_id` | extração e embeddings |
| `encerrar_por_inatividade` | `conversa_id` | agendado para `ultima_mensagem_em + 24 h` |
| `verificar_integracao` | `integracao_id` | atualiza a lista de ferramentas |

- **Agrupamento** ([04](04-canais.md)): o job `responder` nasce com `startAfter` igual à janela do Canal. O primeiro job da janela responde a todas as mensagens pendentes; os seguintes encontram tudo respondido e terminam sem fazer nada. Validar na F1.7 que `startAfter` convive com `key_strict_fifo`.
- **Chave bloqueada.** Na `key_strict_fifo`, um job que esgota as tentativas trava o Contato até alguém reprocessar ou apagar o job. O turno trata erro de modelo e de ferramenta enviando `mensagens.aoFalhar` e concluindo; só erro transitório (rede, 429, 5xx) volta para a fila. Um alerta observa `getBlockedKeys()`.
- **Latência.** Ligar `useListenNotify` e `notify: true` nas filas `responder` e `enviar`; sem isso, o pg-boss consulta a cada 2 s.

## Artefatos

| Artefato | Endereço | Imagem | Monta |
|---|---|---|---|
| `controle` | `app.dobrai.ai` | `apps/controle` | rotas de Membros de todos os slices, `/mcp`, Playground (chama `runtime.executarTurno` no próprio processo), console estático |
| `execucao` | `chat.dobrai.ai` | `apps/execucao` | `widget.js`, ingress público do widget (`/v1/widget/*`) e SSE; webhook do WAHA só na rede interna; workers de todas as filas |
| `waha` | `devlikeapro/waha:gows-<versão>` (`gows-arm` no staging) | sessões de WhatsApp; fala só com `execucao` |
| Postgres | `pgvector/pgvector` (Postgres 17) | acessório do Kamal, volume persistente |

Contatos só tocam `execucao`; Membros só tocam `controle`. Um deploy ou uma queda do controle não interrompe atendimento.

**Sessões do WAHA ficam num volume local**, não no nosso Postgres. O armazenamento Postgres do WAHA cria um banco por sessão, e a documentação dele sugere `max_connections=3000`, o que disputaria conexões com os slices. O volume entra no backup diário para o R2; perdê-lo obriga cada Organização a ler o QR code de novo, mas não perde Conversas. Configuração: `WHATSAPP_DEFAULT_ENGINE=GOWS`, `WAHA_API_KEY` (restrita ao `execucao`), `WAHA_MEDIA_STORAGE=S3` apontando para o R2, reinício automático das sessões ligado.

## CI e deploy

- **`ci.yml`** (todo push e PR): `bun install`, typecheck, checagem de fronteiras, testes unitários, testes de integração com Postgres em service container (todos os testes de isolamento) e build do console.
- **Deploy por artefato**: `deploy-controle.yml` e `deploy-execucao.yml`, disparados em push na `main` com filtro de caminho. O `controle` reage a `apps/controle`, `apps/console`, `slices/**`, `packages/**`; o `execucao`, a `apps/execucao`, `slices/**`, `packages/**`. Mudança só no console não reinicia a execução.
- **Imagens** marcadas com o SHA do commit, build multi-arquitetura (`builder.arch` com amd64 para a Contabo e arm64 para o staging na Oracle), cache do GitHub Actions (`builder.cache.type: gha`). Registro: GHCR. Se as imagens privadas passarem da franquia gratuita de armazenamento, apagar tags antigas no workflow ou trocar de registro; verificar na F1.8.
- **Kamal 2** ([configuração](https://kamal-deploy.org/docs/configuration/)): um arquivo por artefato. No `execucao`, o papel `web` atende o ingress e o papel `job` roda os workers (`cmd` próprio). O kamal-proxy dá TLS automático, troca sem downtime com checagem em `/up` e não bufferiza `text/event-stream`. O `response_timeout` padrão de 30 s é aumentado na rota do SSE, e o servidor manda um comentário de keep-alive a cada 15 s; o widget reconecta sozinho.
- **Segredos** em `.kamal/secrets`, lidos de secrets do GitHub no workflow; `secrets.staging` para o staging.
- **Migrações** rodam no hook de pré-deploy do Kamal, com dbmate, uma pasta e uma tabela de controle por slice. Como os dois artefatos rodam versões diferentes durante um deploy, toda migração segue expandir/contrair: adicionar antes, remover só depois que nenhum código usa.
- **Rollback**: `kamal rollback <versão>` com o arquivo de configuração do artefato; o Kamal mantém os últimos 5 contêineres (`retain_containers`). Rollback de código não desfaz migração; por isso a regra de expandir/contrair.
- **Postgres e WAHA** são acessórios do Kamal: `kamal deploy` não mexe neles, e trocar de versão (`kamal accessory reboot`) tem downtime. O WAHA muda de versão por workflow manual, depois de testado no staging, fora do horário comercial.

## Ambientes

| Ambiente | Onde | Para quê |
|---|---|---|
| Local | `docker compose up` (Postgres e WAHA) + `bun --watch` nos apps | desenvolvimento |
| Staging | Oracle Always Free, São Paulo (ARM): `app.staging.dobrai.ai`, `chat.staging.dobrai.ai` | testar deploy, migração e WAHA com IP brasileiro |
| Produção | Contabo Cloud VPS 6, costa leste dos EUA: `app.dobrai.ai`, `chat.dobrai.ai` | clientes |

O DNS de `dobrai.ai` fica no Cloudflare (plano gratuito), com proxy ligado para o CDN do `widget.js`.

## Backup e observabilidade

- **Backup:** `pg_dump` diário e arquivamento contínuo de WAL, ambos no Cloudflare R2. Teste de restauração mensal no staging.
- **Logs:** JSON no stdout com `org_id`, `conversa_id` e id de requisição; lidos com `kamal app logs`.
- **Erros:** Sentry no plano gratuito.
- **Rastreamento por Conversa:** `runtime.execucao_turno` guarda cada passo do turno com duração e custo ([02](02-modelo-de-dados.md)).
- **Alertas:** disco, RAM acima de 75%, falha de backup, chaves bloqueadas na fila, Canal desconectado.
