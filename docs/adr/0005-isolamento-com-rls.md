# Isolamento entre Organizações com `org_id` e Row-Level Security

Cada slice já tem schema próprio no mesmo Postgres. Um schema por Organização multiplicaria os schemas e as migrações (N Organizações × M slices). Decidimos usar tabelas compartilhadas, com `org_id` obrigatório e RLS. O `org_id` é definido por transação, com `SET LOCAL`, e a aplicação conecta com um papel sem `BYPASSRLS`. Cada slice tem um teste de integração obrigatório que tenta ler dados de outra Organização e precisa falhar.
