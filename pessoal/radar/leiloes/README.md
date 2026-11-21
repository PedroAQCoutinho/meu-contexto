# Pessoal - Radar - Leilões

Radar de leilões de imóveis.

- **Objetivo:** (o que quero encontrar e para quê: moradia, investimento…)
- **Fontes:** Mega Leilões (ativa). leilaoimovel.com.br e Caixa inativas (anti-bot). Ver tabela abaixo.
- **Filtros:** cidade/região, tipo de imóvel, valor máximo, desconto mínimo, ocupação, modalidade
- **Frequência de coleta:** diária, 04:30 BRT (systemd timer na VM)
- **Formato do alerta:** página com mapa (abaixo); Telegram depois (gancho `novidades()` pronto)

## Links
- **Site:** https://radar.147-15-67-178.sslip.io (HTTPS + basic auth; usuário `pedro`, senha só na VM em `~/.agentes-segredos/radar-site-senha`)
- **Repo:** https://github.com/PedroAQCoutinho/pessoal-radar-leiloes (privado) — **ainda precisa ser criado pelo Pedro** (ver Pendências). Código já está na VM em `~/pessoal-radar-leiloes`.
- Execução: `infra/diario.md`, entradas de 2026-11-21 "Pessoal - Radar - Leilões".

## Fontes testadas (2026-11-21, requisição comum com UA de navegador)
| Fonte | Resultado | Estado no radar |
|---|---|---|
| leilaoimovel.com.br | 403, desafio JS Cloudflare (VM e máquina local) | módulo inativo |
| Caixa (CSV por UF) | redireciona para CAPTCHA Radware/perfdrive (VM e local) | módulo inativo |
| **Mega Leilões** | 200; robots.txt `Allow: /` (só `/login` proibido) | **ativa** (Santos: 5 imóveis) |
| Zukerman, Frazão, Lance Judicial, Biasi, D1 Lance, Santander Imóveis, Emgea | 200 na home, sem anti-bot aparente | candidatas (robots/termos a conferir) |
| Superbid, Sold, Resale, Leilões Judiciais, Sodré Santoro | anti-bot / 403 | evitar |

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
- 2026-11-21 — (Claude Code, com autonomia dada pelo Pedro) **Não contornar anti-bot**: fontes com Cloudflare/CAPTCHA
  ficam como módulos inativos. O Claude Code bloqueou o teste com `curl_cffi` como contorno de proteção. Playwright
  não foi instalado (não aprovado e provavelmente também barrado).
- 2026-11-21 — **Fonte inicial trocada para Mega Leilões**: única grande fonte testada com acesso aberto e robots
  permitindo. Custo: Santos tem só 5 imóveis lá (o protótipo tinha 84 do leilaoimovel).
- 2026-11-21 — Código só com biblioteca padrão do Python 3.9 + `pdftotext` (sem pip, sem venv): pico de 39 MiB numa coleta.
- 2026-11-21 — Página em subdomínio `radar.147-15-67-178.sslip.io` servida pelo Caddy do agentes.chat via
  `docker-compose.override.yml` (o compose oficial não foi editado). Basic auth com bcrypt em env_file 600.
- 2026-11-21 — Risco v1 ignora menções genéricas de edital ("eventuais ônus, tais como hipotecas…",
  "foros, laudêmios", hipoteca que garante parcelamento), porque inflavam o score. Detalhes e limitações no README do repo.

## Status
- 2026-11-21 — Projeto criado; definição detalhada em chat próprio.
- 2026-11-21 — Protótipo feito no claude.ai (84 imóveis de Santos, coletados via navegador; o site
  bloqueou acesso fora de navegador real). Prompt de implementação em
  [`prompts/2026-11-21-implementacao-vm.md`](prompts/2026-11-21-implementacao-vm.md); execução pelo
  Claude Code ainda sem registro.
- 2026-11-21 — Etapa 0: leilaoimovel.com.br devolve 403 (desafio JS do Cloudflare) para a VM. `curl_cffi` não foi
  testado porque o Claude Code bloqueou a tentativa de contornar o anti-bot. **Bloqueado:** aguardando decisão
  do Pedro sobre a estratégia de coleta. Repo `pessoal-radar-leiloes` ainda não foi criado.

- 2026-11-21 — **No ar.** Coleta completa de Santos/SP (Mega Leilões): 5/5 ok, 0 erro. Riscos: 1 Muito alto, 1 Alto,
  1 Moderado, 2 Baixo. Matrículas da Mega costumam ser escaneadas ("PDF sem texto"; sem OCR). Timer diário ativo;
  reboot testado (tudo voltou sozinho). Página com HTTPS + basic auth (401 sem senha, 200 com senha).

## Pendências (Pedro)
1. **Criar o repo privado** `PedroAQCoutinho/pessoal-radar-leiloes` no GitHub, **vazio** (sem README/licença), com a
   descrição: `Radar de leilões de imóveis — site: https://radar.147-15-67-178.sslip.io`. Não precisa fazer mais
   nada: a próxima coleta diária faz o `git push` sozinha. Para enviar na hora:
   `ssh oracle 'cd ~/pessoal-radar-leiloes && git push -u origin main'`. O README do repo já tem o link do site.
2. Pegar a senha do site: `ssh oracle cat ~/.agentes-segredos/radar-site-senha`. Guardar no gerenciador de senhas.
3. Versionar o protótipo (`radar-leiloes-santos.html`, `santos.json`) no repo: a página atual foi
   **reconstruída pela especificação**, porque o protótipo não estava na máquina. Ajustar o visual depois.
4. Escolher as próximas fontes (sugestão: Zukerman e Frazão, fortes em SP) e mais municípios (`config.json` no repo).
5. Revisar os scores com os imóveis reais (`fatores` traz a evidência de cada ponto).

## Próximos passos
- Versionar o protótipo (`radar-leiloes-santos.html`, `santos.json`) junto com o repo do radar.
- Avaliar o risco v1 com os dados reais e ajustar pesos (ex.: "ocupação não informada" domina o ponto crítico).
