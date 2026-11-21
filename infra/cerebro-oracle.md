# "Cérebro" 24/7 na VM Oracle

## Decisões
- 2026-11-19 — A VM Oracle (Always Free) é dedicada ao software de gerenciamento
  de agentes e seus coletores. Nada de bancos de trabalho nela (curupira e
  terraanalytics ficam em outros servidores).
- 2026-11-19 — Alertas para mim via **Telegram** (bot). WhatsApp fica para depois.
- 2026-11-19 — Coletores ainda NÃO serão implementados; primeiro fechar a configuração inicial.

- 2026-11-19 — ~~Agente: Hermes Agent com perfis `pessoal` e `radar`.~~ Revisto:
- 2026-11-19 — Agente: **agentes.chat em piloto** (motivo: memória compartilhada entre
  agentes, aprendizados "democratizados"). Piloto só com Radar + alertas para mim,
  nada de iGPP/TerraAnalytics. Instalação antiga: remover e reinstalar limpo (aprovado). Critério de saída: travamentos/perda de dados
  frequentes → migrar para Hermes (plano B). Instalação antiga (Codex) será
  inventariada e refeita do zero.
- 2026-11-19 — **Reinstalação feita**: agentes.chat **v0.1.8** (imagem fixada por digest) na E2.1.Micro,
  público em https://147-15-67-178.sslip.io (Caddy + Let's Encrypt), cadastro fechado, owner = Pedro,
  telemetria desligada, sem runner no servidor. Config em `infra/agentes-chat/`.
- A confirmar na doc do agentes.chat: integração com Telegram, separação de memória
  por contexto, consumo de RAM.
- 2026-11-19 — Instalação via **Docker**; versão **fixada**, atualização só manual.
- 2026-11-19 — Agentes leem o `meu-contexto` por clone na VM + `git pull` agendado.
  Planejado: deploy key somente-leitura. **Realizado:** chave SSH *pessoal* (escrita em
  todos os repos) — **aprovado pelo Pedro**; não requestionar.
- 2026-11-19 — Backup (memória, perfis, configs sem segredos) no Google Drive:
  `AI-AGENTS/HERMES` (nome mantido) (https://drive.google.com/drive/folders/1yvTfiZDsq8Zi-7xr_S0LEUvn9KcKz7R-).
- 2026-11-19 — Bot do Telegram: nome **Saci**, usuário desejado `@saci_bot` (conferir disponibilidade; reservas: `saci_pedro_bot`, `saci_radar_bot`).
- 2026-11-21 — Radar de Leilões usa **SQLite** próprio (não o Postgres "cerebro" do diagrama) por economia de RAM
  e simplicidade; migrar se/quando o cérebro central existir.
- Pendente (sem pressa): provedor/modelo de IA e teto de gasto. Candidato: DeepSeek.

## Recursos (conferir no console)
**VM atual: VM.Standard.E2.1.Micro — 1 OCPU / ~1 GB RAM** (confirmado na Etapa 0).
O agentes.chat **já rodou nela** (instalação antiga) → manter esta VM e otimizar memória
(swap, limites por container, serviços desnecessários desligados). Se um dia faltar,
Always Free oferece Ampere A1 (até 2 OCPU / 12 GB).
Configurar alerta de orçamento em US$ 0,01 para evitar cobrança surpresa.


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
5. Reavaliar orquestração (agentes.chat × Hermes × n8n) ao fim do piloto.
