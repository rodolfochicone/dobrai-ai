# Um papel de banco por slice

A regra "nenhum slice lê tabela de outro" poderia depender só de revisão de código. Decidimos que o Postgres a garante: cada slice conecta com o papel `app_<slice>`, que tem `USAGE` só no próprio schema e não tem `BYPASSRLS`. Cada processo abre um pool pequeno por slice (máximo 3 conexões), o que dá 54 conexões para 9 slices em 2 artefatos. O custo é mais pools e mais credenciais para gerenciar. Em troca, uma query atravessando a fronteira falha no primeiro teste, e extrair um slice para serviço próprio no futuro não exige desembaraçar acessos escondidos.
