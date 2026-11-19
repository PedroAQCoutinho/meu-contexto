# "Cérebro" 24/7 na VM Oracle

## Decisões
- 2026-11-19 — A VM Oracle (Always Free) é dedicada ao software de gerenciamento
  de agentes e seus coletores. Nada de bancos de trabalho nela (curupira e
  terraanalytics ficam em outros servidores).
- 2026-11-19 — Alertas para mim via **Telegram** (bot). WhatsApp fica para depois.
- 2026-11-19 — Coletores ainda NÃO serão implementados; primeiro fechar a configuração inicial.

## Recursos (conferir no console)
Always Free Ampere A1 foi reduzido em 2026 para ~2 OCPU / 12 GB no total.
Configurar alerta de orçamento em US$ 0,01 para evitar cobrança surpresa.

## Software de gerenciamento de agentes
- [ ] Escolher: agentes.chat / LibreChat / n8n / outro

## Arquitetura
```
                 ┌────────────── VM Oracle (Docker Compose) ──────────────┐
 agendador ──►   │ coletores (Python, 1 por fonte)                        │
 (cron/n8n)      │   ├─ concursos   ├─ leilões de imóveis   ├─ editais    │
                 │          │                                              │
                 │          ▼                                              │
                 │ Postgres "cerebro"  (itens brutos, dedup por hash)      │
                 │          │                                              │
                 │          ▼                                              │
                 │ triagem com LLM (filtra pelo meu perfil, resume, nota) │
                 │          │                                              │
                 │          ▼                                              │
                 │ notificador ──► Telegram (só para mim)             │
                 │ envio a colegas ──► SEMPRE com minha aprovação         │
                 │ servidor MCP ──► Claude / ChatGPT consultam o cérebro  │
                 └─────────────────────────────────────────────────────────┘
```

## Fontes (a definir)
| Agente | Fontes candidatas | Filtros |
|---|---|---|
| Concursos | | cargo, área, região, salário mínimo |
| Leilões de imóveis | | cidade, tipo, valor máx., desconto mín. |
| Oportunidades de projeto | agências de fomento, editais | área, valor, prazo |

Regras de coleta: preferir APIs/RSS/CSV oficiais; respeitar robots.txt e termos;
intervalo educado entre requisições.

## Canal de mensagens
- **Para mim (decidido):** bot do Telegram. Token do bot fica em `.env` na VM.
- **Para colegas (futuro):** WhatsApp Cloud API oficial (templates aprovados),
  sempre com aprovação manual.
- **Evitar** bibliotecas não oficiais (Baileys / Evolution modo WhatsApp Web) no meu
  número pessoal: violam os termos e o ban costuma ser permanente.

## Fases
1. Postgres + 1 coletor (o de maior valor) + notificação para mim.
2. Triagem com LLM + resumo diário.
3. Demais coletores.
4. Servidor MCP do cérebro (Claude/ChatGPT consultam o histórico).
5. Avaliar agentes.chat / n8n como camada de orquestração, se fizer falta.
