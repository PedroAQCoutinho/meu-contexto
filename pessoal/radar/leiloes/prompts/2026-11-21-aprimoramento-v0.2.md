# 2026-11-21 — Aprimoramento v0.2: painel lateral, mais fontes, filtros de leiloeiro e motivo

- **Agente:** Claude Code (máquina local com SSH para a VM Oracle)
- **Origem:** chat "Pessoal - Radar - Leilões" no claude.ai, 2026-11-21
- **Anterior:** [`2026-11-21-implementacao-vm.md`](2026-11-21-implementacao-vm.md) (v0.1 no ar, só Mega Leilões)
- **Status/Resultado:** a executar. Atualizar com link para `infra/diario.md` ao terminar.

---

Projeto: **Pessoal - Radar - Leilões**. Rode `git pull` no `meu-contexto` e no `~/pessoal-radar-leiloes` (na VM), depois leia `AGENTS.md`, `autonomia.md`, `pessoal/radar/leiloes/README.md` e as entradas do Radar em `infra/diario.md`.

Você tem autonomia para executar: é infra pessoal que eu pedi. Pare só se houver risco crítico de segurança. Registre cada etapa no diário e faça commit+push nos dois repos.

## Limites que continuam valendo
- **Sem contornar anti-bot.** Isso inclui:
  - simular navegador (`curl_cffi`/impersonate);
  - navegador "stealth";
  - resolver CAPTCHA;
  - proxies rotativos;
  - forjar cookies do Cloudflare.

  Fonte que exige isso fica **inativa** e vai listada no relatório.
- **Antes de ativar cada fonte:** respeite o robots.txt e confira os termos de uso. Use User-Agent identificado (`radar-leiloes/0.2 (+contato no README do repo)`), 1 requisição por vez por host, de 2 a 3 s entre requisições e backoff em 429/5xx.
- **VM de ~1 GB.** Só biblioteca padrão do Python + `pdftotext`.
  - Nada de `dnf`/pip sem necessidade. Se for inevitável, use `--setopt=install_weak_deps=False` e pare o agentes.chat antes (lição do incidente de RAM de 2026-11-21).
  - Coleta grande sempre com `MemoryMax`, `Nice` e em lotes.

## 1. Painel lateral no lugar do popup
- Clicar num ponto (ou num item da lista) abre um **painel fixo à direita** com todos os metadados. O mapa continua usável.
- O ponto selecionado fica destacado (anel/borda maior). O painel tem botão de fechar e a tecla Esc também fecha.
- **Conteúdo do painel:**
  - nível de risco com pontuação e **ponto crítico** em destaque;
  - lista de fatores com a evidência de cada um (o trecho do texto que gerou o ponto);
  - valores, praças, ocupação e área;
  - **leiloeiro** e **motivo do leilão** (itens 3 e 4);
  - links do anúncio, da matrícula, do edital e da página do leiloeiro;
  - fonte, data da coleta e versão da regra.
- No celular (largura < 760 px), o painel vira uma gaveta que sobe de baixo.
- O hash `#imovel-<id>` abre o painel direto, para eu mandar link de um imóvel.

## 2. Máximo de pontos: coletar direto dos leiloeiros
O leilaoimovel.com.br continua bloqueado (Cloudflare) e fica inativo. Ele só agrega os sites dos leiloeiros, então a meta é cobrir os leiloeiros. Numa amostra de Santos/SP (84 imóveis, 33 leiloeiros), seis deles concentram ~60%:

| Prioridade | Leiloeiro | Imóveis na amostra |
|---|---|---|
| 1 | AlienaJud | 15 |
| 2 | D1Lance Leilões | 11 |
| 3 | Cleber Cardoso Leilões | 9 |
| 4 | Positivo Leilões | 7 |
| 5 | Grupo Lance | 5 |
| — | Mega Leilões (já ativo) | 4 |

Depois desses: Sublime, Destak, Sato, JM Leilões, Up Leilão, AGS, Leilão Brasil, Je Leilões, Portal Zuk, Frazão, Lance Judicial, Biasi, Santander Imóveis e Emgea.

**Ataque por plataforma, não por site.** Muitos leiloeiros usam o mesmo software. Os links de documentos da amostra mostram pelo menos estes padrões:
- `/preview/<uuid>.pdf`: AlienaJud, Positivo, Sublime, AGS;
- `/previewlote/<n>/<n>/<uuid>.pdf`: Up Leilão, Destak;
- `static.suporteleiloes.com.br/<site>/bens/...`: Leilão Brasil, Je Leilões, Valero.

Para cada plataforma, escreva um adaptador e uma lista de sites que a usam. Um adaptador novo pode ativar vários leiloeiros de uma vez. Prefira JSON/API pública que o próprio site use, sitemap, RSS ou CSV oficial a raspar HTML.

**Escopo geográfico:** colete **tudo** que cada fonte oferece no Brasil, não só Santos. O filtro de município fica na página.
- Faça a primeira carga em lotes ao longo de alguns dias, para não pesar na VM nem nos sites.
- Depois do primeiro dia, a coleta diária só busca o que é novo ou mudou (`hash_conteudo`).

**Deduplicação entre fontes:** o mesmo imóvel aparece em vários sites.
- Chave primária: número de matrícula + cartório. Na falta, endereço normalizado + coordenada a menos de 30 m + valor parecido.
- Guarde todas as origens. O painel lista "também anunciado em…".

**Geocodificação:**
- Prefira a coordenada da fonte.
- Use o Nominatim só quando faltar: 1 req/s, cache em SQLite e `precisao` marcada.
- Com milhares de imóveis, priorize os de praça mais próxima.

## 3. Leiloeiro como filtro e metadado
- **Banco:** tabela `leiloeiros` com nome normalizado, site, plataforma e, se houver no site, matrícula na Junta Comercial. O imóvel aponta para ela.
- **Página:** filtro de leiloeiro com seleção múltipla e busca. O nome e o link do site aparecem no painel.

## 4. Motivo do leilão como filtro e metadado
Classifique cada imóvel num `motivo`, guardando a **evidência** (o trecho do texto) e a **confiança** (alta/média/baixa). Use estas categorias:
- Alienação fiduciária (Lei 9.514/97, leilão extrajudicial de banco ou credor)
- Venda direta / pós-leilão de banco
- Execução hipotecária
- Execução fiscal federal (União/PGFN/INSS, Comprei PGFN)
- Execução fiscal estadual ou municipal (IPTU, ICMS)
- Trabalhista (Vara do Trabalho / TRT)
- Cobrança de condomínio
- Execução cível / título extrajudicial (dívida comum)
- Falência / recuperação judicial
- Alienação judicial (extinção de condomínio, divórcio, inventário)
- Não identificado

**Sinais a usar:**
- modalidade;
- comitente/exequente (banco, condomínio, União, Fazenda);
- vara ou juízo ("Vara do Trabalho", "Vara Federal", "Vara de Execuções Fiscais", "Vara de Falências");
- número CNJ do processo: o segmento `J` em `NNNNNNN-DD.AAAA.J.TR.OOOO` indica a justiça (5 = Trabalho, 4 = Federal, 8 = Estadual);
- palavras-chave do edital.

Escreva testes com exemplos reais para cada categoria. O motivo vira filtro com seleção múltipla e aparece no painel.

## Página: filtros finais
- **Filtros:** município (centraliza o mapa), slider de valor, nível de risco, leiloeiro, motivo e modalidade.
- **Contadores e URL:** mostre a contagem por filtro e mantenha os filtros na URL, para eu poder salvar uma busca.
- **Desempenho:** com milhares de pontos, use agrupamento de marcadores (Leaflet.markercluster via cdnjs) e carregue o `data.json` compactado.
- **Mapa:** não mexa na `Referrer-Policy` (`strict-origin-when-cross-origin` foi o que destravou os tiles do OSM).

## Risco
Mantenha a regra v1. Corrija só o peso de "ocupação não informada" para 0,5 e não deixe esse fator virar ponto crítico. Ele dominava o ponto crítico na amostra. Isso vira a versão `v1.1`, registrada nas análises.

## Relatório final (no diário e no README do projeto)
1. Uma tabela de fontes: ativa ou inativa, motivo, plataforma e nº de imóveis.
2. Totais:
   - imóveis únicos;
   - imóveis com coordenada;
   - distribuição por risco, por motivo e por leiloeiro (top 15).
3. Memória e tempo da coleta.
4. O que ficou de fora e por quê.

## Critério de pronto
- Pelo menos as 5 fontes prioritárias testadas; as acessíveis, ativas.
- Página com painel lateral e os novos filtros.
- Testes de motivo e de risco passando.
- Timer seguindo.
- Diário e README atualizados, com commit+push.
