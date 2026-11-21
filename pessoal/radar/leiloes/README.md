# Pessoal - Radar - Leilões

Radar de leilões de imóveis.

- **Objetivo:** (o que quero encontrar e para quê: moradia, investimento…)
- **Fontes:** (sites/leiloeiros/bancos)
- **Filtros:** cidade/região, tipo de imóvel, valor máximo, desconto mínimo, ocupação, modalidade
- **Frequência de coleta:**
- **Formato do alerta:**

## Decisões
<!-- AAAA-MM-DD — decisão — por quê -->
- 2026-11-21 — Fonte inicial leilaoimovel.com.br (robots.txt `Allow: /`), primeiro município Santos/SP.
- 2026-11-21 — Saída: página web simples na VM com mapa (Leaflet/OSM), pontos coloridos por risco,
  filtro por município e slider de valor, popup com links de anúncio/matrícula/edital e ponto crítico.
  Dados em SQLite com metadados de coleta.
- 2026-11-21 — Risco v1 avalia só matrícula e edital (regras por pontos; processos ficam para depois).
  Regras no prompt de implementação; Pedro vai avaliar e ajustar.
- 2026-11-21 — Implementação na VM delegada ao Claude Code. MCP não é necessário agora; CLI primeiro,
  MCP opcional depois.

## Status
- 2026-11-21 — Projeto criado; definição detalhada em chat próprio.
- 2026-11-21 — Protótipo feito no claude.ai (84 imóveis de Santos, coletados via navegador; o site
  bloqueou acesso fora de navegador real). Prompt de implementação em
  [`prompts/2026-11-21-implementacao-vm.md`](prompts/2026-11-21-implementacao-vm.md); execução pelo
  Claude Code ainda sem registro.

## Próximos passos
- Retomar no Claude Code: checar status da execução / Etapa 0 (acesso da VM ao site).
- Versionar o protótipo (`radar-leiloes-santos.html`, `santos.json`) junto com o repo do radar.
- Avaliar o risco v1 com os dados reais e ajustar pesos (ex.: "ocupação não informada" domina o ponto crítico).
