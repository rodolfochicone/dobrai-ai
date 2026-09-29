# Embeddings do Brain gerados localmente com multilingual-e5-small

O Brain precisa de embeddings, e cada Organização paga seus modelos com a própria chave. Só que a Anthropic não tem API de embeddings, então uma Organização que só usa Claude ficaria sem Brain. Usar uma chave nossa poria custo de LLM no nosso bolso e o Dobrai como intermediário do dado perante outro provedor. Decidimos gerar os embeddings no nosso próprio host com o `intfloat/multilingual-e5-small` (MIT, 118M parâmetros, 384 dimensões, ~0,3 GB de RAM, versão int8 pronta para transformers.js). Custa zero, funciona igual para todas as Organizações e o conteúdo do Brain não sai do nosso servidor.

## Considered Options

- `text-embedding-3-small` com a chave da Organização: US$ 0,02 por milhão de tokens, mas exige chave OpenAI.
- `voyage-4-lite`: mesmo preço, com 200 milhões de tokens grátis, mas é mais um provedor e mais um contrato de dados.
- `bge-m3`: mais qualidade e contexto de 8k tokens, mas ~1–1,5 GB de RAM e 5 a 10 vezes mais lento em CPU.

Preços consultados em 28/09/2026: [OpenAI embeddings](https://developers.openai.com/api/docs/guides/embeddings), [Voyage](https://docs.voyageai.com/docs/pricing). Modelo: [intfloat/multilingual-e5-small](https://huggingface.co/intfloat/multilingual-e5-small).

## Consequences

- A dimensão (384) fica gravada na coluna `brain.trecho.embedding`. Trocar de modelo exige recalcular todos os trechos, feito como job de reindexação por Fonte.
- O e5 exige os prefixos `query:` e `passage:`, e aceita no máximo 512 tokens por trecho, o que limita o tamanho dos trechos.
- Rodar transformers.js sob Bun não está verificado. Se falhar, o worker de embeddings roda com Node dentro da mesma imagem da execução. Confirmar na F2.4.
- A qualidade da busca em pt-BR é medida na F2.4 com perguntas reais. Se o recall não servir, o próximo degrau é o `multilingual-e5-base` (768 dimensões, ~0,5 GB).
