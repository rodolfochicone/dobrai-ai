# Dobrai

Plataforma SaaS em que Organizações criam, testam, publicam e operam Agentes de IA de atendimento e vendas, conversando com um Construtor, e os disponibilizam em Canais como widget web e WhatsApp.

## Language

### Organizações e pessoas

**Organização**:
A unidade de isolamento de dados, permissões e cobrança da plataforma.
_Avoid_: tenant, empresa, conta, workspace, cliente

**Usuário**:
Pessoa com login no Dobrai.
_Avoid_: user, cliente

**Membro**:
Usuário vinculado a uma Organização com um papel; um Usuário pode ser Membro de várias Organizações.
_Avoid_: usuário (quando o papel na Organização importa), colaborador

**Papel**:
Nível de acesso de um Membro numa Organização: Dono, Admin (constrói, publica e configura) ou Operador.
_Avoid_: role, perfil, cargo

**Operador**:
Membro que atende Conversas pela inbox, assumindo e devolvendo ao Agente.
_Avoid_: atendente, agente humano

### Agentes

**Agente**:
Assistente de IA configurado por uma Organização para atender Contatos em Canais.
_Avoid_: bot, chatbot, assistente

**Construtor**:
Meta-agente com que um Membro conversa, no console ou a partir de um cliente oficial como Claude Code ou Codex, para criar e editar Rascunhos.
_Avoid_: builder, Agent Builder, meta-agente

**Sessão do Construtor**:
Conversa entre um Membro e o Construtor sobre o Rascunho de um Agente.
_Avoid_: conversa (reservado a Contatos), chat

**Publicar**:
Ato humano, feito no console, que transforma o Rascunho numa nova Versão do Agente; o Construtor nunca publica.
_Avoid_: deploy, lançar, subir

**Versão do Agente**:
Especificação imutável e publicável de um Agente.
_Avoid_: release, snapshot

**Rascunho**:
Especificação editável de um Agente, ainda não publicada.
_Avoid_: draft

**Playground**:
Ambiente de teste em que o Rascunho se comporta exatamente como se comportará publicado.
_Avoid_: sandbox, simulador

**Skill**:
Pacote de instruções que um Agente pode usar.
_Avoid_: habilidade

**Integração**:
Conexão que dá ferramentas a um Agente: um MCP remoto ou uma API REST.
_Avoid_: source, fonte, conector

### Modelos de linguagem

**Conexão de LLM**:
Acesso a modelos de linguagem que o Dobrai usa em nome de uma Organização ou de um Membro.
_Avoid_: provider, credencial, chave (sozinho)

**Chave da Organização**:
Conexão de LLM com chave de API da própria Organização; é a única que atende Contatos, no Agente publicado e no Playground.
_Avoid_: BYOK, API key

**Assinatura do Membro**:
Plano de consumo pessoal de um Membro, como Claude Pro/Max ou ChatGPT, com que ele opera o Construtor a partir do cliente oficial do provedor; o Dobrai nunca a recebe nem a armazena, por isso não é uma Conexão de LLM.
_Avoid_: OAuth, plano, login do Claude

### Canais e conversas

**Tipo de canal**:
Modalidade de atendimento: widget web, WhatsApp oficial, WhatsApp não oficial e afins.
_Avoid_: plataforma, provedor

**Canal**:
Instância conectada de um Tipo de canal, como um número de WhatsApp ou um widget em um domínio, ligada a exatamente um Agente.
_Avoid_: conexão, instância, número

**Contato**:
Pessoa que conversa com um Agente, identificada dentro da Organização pelo identificador do canal; não há fusão automática entre identificadores diferentes.
_Avoid_: usuário, cliente, cliente final, lead

**Conversa**:
Sequência de mensagens entre um Contato e a Organização que termina por resolução ou por inatividade; uma nova mensagem depois disso abre outra Conversa.
_Avoid_: atendimento, sessão, ticket, chat

**Transferir**:
Ato do Agente de passar a Conversa aos Operadores; ele para de responder e a Conversa aguarda alguém Assumir.
_Avoid_: handoff, transbordo, escalar

**Assumir**:
Ato de um Operador tomar uma Conversa do Agente, transferida ou não, que para de responder até a Conversa ser devolvida.
_Avoid_: handoff, transbordo

**Teto de gasto**:
Limite de custo de LLM de uma Organização ou de uma Conversa que, ao ser atingido, interrompe o Agente.
_Avoid_: budget, cota, limite (sozinho)

### Conhecimento

**Brain**:
Conhecimento da empresa compartilhado pelos Agentes de uma Organização, com escopo de Organização ou de Agente.
_Avoid_: base de conhecimento, memória compartilhada, KB

**Fonte de conhecimento**:
Origem de conteúdo que alimenta o Brain, como arquivo, URL ou pasta do Drive.
_Avoid_: source, brain source

**Memória do Contato**:
Fatos sobre um Contato lembrados entre Conversas; é dado pessoal do Contato, não conhecimento da empresa.
_Avoid_: memória (sozinho), perfil
