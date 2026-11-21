# 2026-11-21 — Implementação do radar na VM

- **Agente:** Claude Code (máquina local com SSH para a VM Oracle)
- **Origem:** chat "Pessoal - Radar - Leilões" no claude.ai, 2026-11-21
- **Status/Resultado:** executado em 2026-11-21 pelo Claude Code, **com desvios**: leilaoimovel bloqueado por Cloudflare, fonte trocada para Mega Leilões; protótipo não estava disponível (página reconstruída); repo GitHub não criado (sem token). Detalhes: `infra/diario.md` (entradas "Pessoal - Radar - Leilões" de 2026-11-21) e `pessoal/radar/leiloes/README.md`.
- **Sucedido por:** [`2026-11-21-aprimoramento-v0.2.md`](2026-11-21-aprimoramento-v0.2.md)
- **Arquivos do protótipo citados:** `radar-leiloes-santos.html` e `santos.json` (84 imóveis de Santos/SP, coletados em 2026-11-21) — ainda não versionados; estão no chat do claude.ai.

Projeto: **Pessoal - Radar - Leilões**. Antes de qualquer coisa, rode `git pull` no `meu-contexto` e leia `AGENTS.md`, `autonomia.md`, `infra/vm-oracle.md`, `infra/cerebro-oracle.md`, `pessoal/radar/README.md` e `pessoal/radar/leiloes/README.md`. É infra pessoal que eu já pedi: execute com autonomia e pare só se houver risco crítico de segurança. Registre cada etapa em `infra/diario.md` (sem segredos) e faça commit+push.

## Objetivo
Um radar de leilões de imóveis que roda na VM Oracle. Ele coleta anúncios, guarda tudo com metadados em SQLite e publica uma página web estática com mapa. A fonte inicial é o leilaoimovel.com.br e o primeiro município é Santos/SP. Depois vou somar outras fontes (leiloeiros, Caixa, prefeituras), então cada fonte precisa ser um módulo plugável.

O protótipo aprovado está em `radar-leiloes-santos.html`, com os dados em `santos.json` (84 imóveis coletados em 21/11/2026). A página final deve manter esse visual e esse comportamento.

## Etapa 0 — teste de acesso (faça primeiro)
Na VM, teste `curl -sS -o /dev/null -w '%{http_code}'` com User-Agent de navegador em:
- `https://www.leilaoimovel.com.br/leilao-de-imovel/santos-sp`
- uma página `/imovel/...`

Em 21/11/2026 o site deu 403 para clientes fora do navegador num ambiente de nuvem, mas funcionou num Chrome real. O robots.txt permite tudo (`Allow: /`).
- **Se der 200:** siga com `requests` + `selectolax` ou BeautifulSoup.
- **Se der 403:** teste `curl_cffi` (impersonate chrome), que é leve. Se não resolver, pare essa parte e me apresente as opções com o custo de RAM de cada uma. A VM tem ~1 GB e Playwright/Chromium só entra se eu aprovar. As opções:
  - Playwright em horário espaçado;
  - outra fonte;
  - coletor rodando na minha máquina local e enviando o resultado para a VM.

## Repositório
- Crie um repo **privado** no GitHub `PedroAQCoutinho/pessoal-radar-leiloes`, com clone na VM pela chave SSH já configurada.
- Em `meu-contexto/pessoal/radar/leiloes/README.md`, registre o link do repo, as decisões e o status.

Estrutura sugerida:
```
radar/
  fontes/base.py           # interface: listar(municipio) -> urls; detalhar(url) -> dict
  fontes/leilaoimovel.py
  risco.py                 # regras versionadas (RISCO_VERSAO = "v1")
  geocode.py               # Nominatim com cache, 1 req/s, User-Agent identificado
  db.py                    # SQLite
  exportar.py              # gera site/data.json
  cli.py                   # radar coletar --fonte leilaoimovel --municipio santos-sp
site/index.html            # a partir do protótipo; lê data.json via fetch
data/radar.db              # fora do git
```

## Coleta no leilaoimovel.com.br (o que já sei)
- **Listagem por município:** `/leilao-de-imovel/<cidade>-<uf>`, com páginas `?pag=N` (Santos tem 6 páginas). Links de imóvel batem com `/imovel/<uf>/<cidade>/<slug>-imovel-<ID>`.
- **Na página do imóvel** (procure em `body`, porque às vezes o primeiro `main` é um modal):
  - **Título e endereço:** título em `h1`; endereço logo depois de `- <ID>` e antes de "Detalhes do Imóvel" ou "Área Total".
  - **Rótulos:** `Área Total:`/`Área Útil:`, `Situação:`, `Localização: UF /Cidade /Bairro /Rua`, `Precisão da geolocalização:`, `Tipo: <tipo> /<modalidade>`, `Leiloeiro:`, `Valor de Avaliação:`, `Data de Inclusão:` e `Descrição:` (vai até "Valor do Imóvel").
  - **Valores e praças:** `1ª/2ª Praça` ou `1º/2º Leilão`, com data e valor.
  - **Coordenadas:** no iframe `maps.google.com/maps?q=<lat>,<lon>`. Se faltar, use Nominatim e marque `precisao='aproximada'`.
  - **Links:** os textos exatos "Matricula" e "Edital" levam aos PDFs. Guarde só `origin+path`.
- **Falhas transitórias:** cerca de 20% das páginas voltaram sem dados na primeira tentativa. Faça retry com backoff e intervalo de 2–3 s entre requisições.
- **Sem valor:** 7 imóveis não divulgam valor. Guarde `NULL`.

## Banco (SQLite)
- `fontes`: id, nome, url_base, ativo.
- `coletas`: id, fonte, municipio, inicio, fim, n_ok, n_erro, versao_coletor.
- `imoveis`: chave única (fonte, id_externo), todos os campos acima, `primeira_vez_visto`, `ultima_vez_visto`, `hash_conteudo` (para detectar mudança) e `ativo`.
- `documentos`: imovel_id, tipo (matricula/edital), url, sha256, baixado_em, tem_texto, texto (extraído).
- `analises_risco`: imovel_id, versao_regra, score, nivel, fatores (JSON), ponto_critico, fonte_texto (anuncio/pdf), criado_em.

## Risco v1 (só matrícula e edital, sem processos)
Regras por pontos sobre o texto, implementadas em `risco.py`. No protótipo elas leram a descrição do anúncio. Aqui leia também o texto dos PDFs: baixe com `pdftotext` (poppler-utils). Se o PDF for só imagem, marque "PDF sem texto" e não rode OCR por enquanto. Guarde qual texto foi usado.

| Fator | Pontos |
|---|---|
| Direitos possessórios / matrícula "N/C" / sem matrícula | 3 |
| Vende direitos (e ações, aquisitivos, fiduciários), não a propriedade | 3 |
| Usufruto | 3 |
| Vende só fração/cota-parte do imóvel | 3 |
| Indisponibilidade averbada | 2 |
| Embargos / ação anulatória / agravo / suspensão | 2 |
| Ocupado | 2 (não informado = 1) |
| Dívidas de IPTU/condomínio passam ao arrematante (exceto se o edital diz "livre de ônus tributários") | 2 |
| Débito alto citado no edital | 1 |
| Hipoteca (exceto se o credor diz não ter mais crédito) | 1 |
| 2+ penhoras | 1 |
| Coproprietário com direito de preferência | 1 |
| Terreno de marinha / SPU / laudêmio | 1 |
| Matrícula não anexada | 1 |
| Edital não anexado | 1 |
| Modalidade judicial | 1 |

Níveis: 0–2 Baixo (verde), 3–4 Moderado (amarelo), 5–6 Alto (laranja), 7+ Muito alto (vermelho). O ponto crítico é o fator de maior peso, ignorando "judicial" e "não anexado".

Bug do protótipo a corrigir: o regex `/judicial/` também casava com "Extrajudicial". Use comparação exata da modalidade.

## Página
- `site/index.html` a partir do protótipo, lendo `data.json` gerado por `exportar.py`. Mantenha:
  - mapa Leaflet + OpenStreetMap;
  - pontos coloridos por risco;
  - filtro por município que recentraliza o mapa;
  - slider de valor;
  - chips de risco;
  - popup com links de anúncio, matrícula e edital, mais o ponto crítico.
- Servir pelo Caddy que já está na VM, num subdomínio sslip.io ou num caminho, com `basic_auth`. A senha vai para `~/.agentes-segredos/` (600) e nunca para o repo.
- Mostre na página a data da última coleta e a versão da regra de risco.

## Agendamento e agente
- `systemd` timer diário, fora de horário de pico. Logs limitados.
- Monitore o uso de RAM durante uma coleta completa e registre no diário.
- O CLI precisa ser chamável por um agente do agentes.chat (função "coletar município X da fonte Y" e "listar novidades desde a última coleta"). Documente isso no README do repo.
- Alerta no Telegram fica para depois. Deixe só o gancho `novidades()`.

## Critério de pronto
- Coleta completa de Santos/SP no banco.
- Página no ar com HTTPS + basic auth.
- Timer ativo e um reboot testado.
- README e diário atualizados no `meu-contexto`, com commit+push.
