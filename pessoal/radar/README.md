# Pessoal - Radar

Agentes 24/7 na VM Oracle (agentes.chat) que buscam oportunidades e me alertam.
Arquitetura geral e decisões de infra: `../../infra/cerebro-oracle.md`.

## Regras comuns a todos os radares
- Preferir APIs/RSS/CSV oficiais; respeitar robots.txt e termos; intervalo educado entre requisições.
- Coleta em código agendado (barato e estável); IA só para triagem, resumo e conversa.
- Deduplicar itens; alertar só o que é novo e passa nos filtros.
- Alertas para mim via Telegram (bot Saci), quando existir.
- RAM da VM é ~1 GB: coletores leves, sem navegador headless sem necessidade.

## Radares
| Radar | Status |
|---|---|
| [Leilões de imóveis](leiloes/README.md) | em definição |
| Concursos públicos | a criar |
| Oportunidades de projeto | a criar |
