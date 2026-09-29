# WAHA (motor GOWS) para o WhatsApp não oficial

Comparamos quatro alternativas:

- **Worker Baileys do Craft:** foi feito para o dono falar com o próprio agente e não cria sessão por contato.
- **Evolution API:** a licença exige exibir a marca Evolution no frontend ou comprar licença comercial, e a manutenção desacelerou.
- **Baileys direto:** armazenamento de sessão e reconexão ficariam por nossa conta.
- **whatsapp-web.js:** consome ~350 MB por número.

Decidimos pelo WAHA com o motor GOWS. É Apache-2.0, com todos os recursos gratuitos desde a versão 2026.6.1, usa ~50 MB por número, guarda sessões no Postgres e publica releases mensais. Ele roda como artefato próprio (ADR-0001). O slice de WhatsApp não oficial fala com ele por HTTP e recebe os webhooks dele na fila.

## Consequences

- O Agente só responde; nunca inicia conversa com número frio. Ele marca as mensagens como lidas e mostra "digitando".
- O aviso de risco de bloqueio aparece na conexão do número e nos termos de uso.
- Risco não verificado: números brasileiros conectados a partir de IP de datacenter nos EUA podem ser mais bloqueados. É preciso medir e, se for o caso, mover o WAHA para um host no Brasil.
