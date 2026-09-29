# Monólito modular com três artefatos de deploy

O prompt v1 exigia deploy independente por slice: cerca de 13 serviços, cada um com imagem, pipeline, versão e contratos próprios. Para uma pessoa operando com ~US$ 50/mês, isso é custo de microsserviços antes do primeiro cliente. Decidimos manter os slices como módulos, cada um com schema próprio, API de módulo e eventos pela fila no Postgres, empacotados em três artefatos separados por perfil de execução: plano de controle (API + console), runtime/workers e WhatsApp não oficial (WAHA). Cada artefato tem deploy e rollback independentes. A fronteira de contexto para sessões de LLM vem de pastas, `CLAUDE.md` por slice e lint contra import cruzado, não do deploy.

## Consequences

- Um slice vira artefato próprio quando uma métrica justificar: consumo de RAM, falha que precise ser isolada ou ritmo de deploy.
- Slices conversam só pela API de módulo e por eventos. Ler tabela de outro slice continua proibido, mesmo com os dois no mesmo processo.
