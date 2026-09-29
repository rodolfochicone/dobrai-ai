# pg-boss com FIFO por Contato como fila

Mensagens do mesmo Contato precisam ser processadas em ordem, e Contatos diferentes em paralelo, numa fila dentro do Postgres. Escolhemos o pg-boss (MIT), que roda em Bun e tem uma política de fila, `key_strict_fifo`, que dá FIFO por `singletonKey` sem travar as outras chaves. Ele também tem fila de falhas (`deadLetter`), `redrive()` e agendamento. Descartamos o graphile-worker: tem latência menor, mas não documenta suporte a Bun, não tem fila de falhas e desaconselha uma `queue_name` por chave em alta cardinalidade, que seria exatamente um Contato por fila.

## Consequences

- Um job que esgota as tentativas trava o Contato até alguém reprocessar ou apagar o job. O turno converte erros de modelo e de ferramenta em `mensagens.aoFalhar` e só devolve à fila os erros transitórios. Um alerta observa `getBlockedKeys()`.
- A busca de jobs fica mais lenta conforme cresce o número de chaves distintas com jobs na fila (cerca de 1 s com 200 mil chaves). Esse número é métrica de capacidade.
- Sem `useListenNotify`, o pg-boss consulta a fila a cada 2 s. As filas de atendimento usam notificação.
