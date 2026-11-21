# Diário de execução

Uma entrada por etapa executada. Nunca registrar valores de segredos.

## 2026-11-19 — Etapa 0, tentativa 1 (Claude Code)
- `ssh` → timeout na porta 22. Nada rodou na VM. Corrigido no console da OCI.

## 2026-11-19 — Etapa 0, tentativa 2 (Claude Code)
- Diagnóstico somente leitura concluído. Detalhes em `vm-oracle.md`.
- Achados: shape E2.1.Micro (1 OCPU / ~1 GB), Docker 29.10 com containers ativos
  em 80/443/3000, firewalld liberando ssh/http/https, rpcbind exposto (bloqueado pelo firewalld).

## 2026-11-19 — Acesso da VM ao GitHub (Claude Code)
- Gerada chave ed25519 na VM; pública cadastrada como chave SSH pessoal do Pedro.
- Pendente: `~/.ssh/config` na VM, teste `ssh -T`, clone.

## Etapa 1 (antiga) — substituída pela reinstalação de 2026-11-19 (abaixo)
- Suspensa até inventário da instalação antiga e decisão sobre o shape da VM.

## 2026-11-19 — Reinstalação do agentes.chat, Etapa 1: inventário (Claude Code)
Somente leitura (`docker ps/volume/network/images/stats/inspect`, `systemctl`, `crontab`, `ss`, `find`).
- Projeto Compose `agentes-chat` em `/home/opc/agentes-chat` (`docker-compose.yml` do commit
  `58e6263` + `docker-compose.override.yml`), instalado em 2026-11-16 via `prepare-vm.sh`
  (e antes uma tentativa `curl https://agentes.chat/install | sh`).
- Containers (todos `unless-stopped`):

  | Container | Imagem | Portas | RAM (uso / limite) |
  |---|---|---|---|
  | agentes-chat-agentes-1 | ghcr.io/agentes-chat/agentes:latest (digest fixado no .env) | 127.0.0.1:3000 | 170 MiB / 576 MiB |
  | agentes-chat-caddy-1 | caddy:2.11.7-alpine | 80, 443 tcp+udp | 15 MiB / 96 MiB |
  | agentes-chat-postgres-1 | pgvector/pgvector:0.8.7-pg18 | 5432 interna | 94 MiB / 192 MiB |
- Volumes: `agentes-chat_pgdata`, `agentes-chat_files`, `agentes-chat_caddy-data` (77 MB no total).
  Redes: `agentes-chat_default`, `_embeddings`, `_speech`. Imagens: 4,5 GB (app 3,7 GB).
- Domínio usado: `agentes.147-15-67-178.sslip.io`, perfil `https` (Caddy + Let's Encrypt), signup fechado,
  owner via link de claim (`owner-link.txt`).
- Restos fora do projeto: `/opt/agentes/.runtime/bun` (do instalador), `/root/.bun` (cache do
  `@agenteschat/setup` 0.1.1), `/tmp/agentes-install.*`, swapfile extra `/swap-agentes` (2 GB, no fstab).
  Sem cron, sem timers ou serviços systemd próprios (`agentes-network-check*` citados no script não existem mais).
- `authorized_keys`: 1 chave (`ssh-rsa`, comentário `ssh-key-2026-11-16`).
- Nomes das variáveis do `.env` antigo: COMPOSE_PROJECT_NAME, NODE_ENV, PUBLIC_URL, DOMAIN,
  COMPOSE_PROFILES, **POSTGRES_PASSWORD**, **AGENTES_SECRET**, **OWNER_CLAIM_SHA256**, ALLOW_SIGNUP,
  AGENTES_MEMORY, DATABASE_POOL_MAX, OUTBOX_MAX_IN_FLIGHT, OUTBOX_QUICK_SLOTS,
  OUTBOX_MAX_IN_FLIGHT_CEILING, TELEMETRY, LOG_LEVEL, AGENTES_IMAGE.
- RAM antes: 629 MiB usados / 945, swap 477 MiB usados (dois swapfiles: 945 MiB + 2 GB), swappiness 60.
  Disco `/`: 14 GB usados de 30.
- Máquina local (não alterada): `~/agentes/personal/shared/bin` (vazio de dados, pasta do app desktop) e
  `~/.agentes-runner/orgs/<id>/` = **runner do agentes.chat** rodando (`agentes-runner app`, aberto pelo
  `agentes-desktop`), conectado a `https://agentes.147-15-67-178.sslip.io`, com executores `claude` e `codex`
  e conta `codex` salva. Ou seja: a instância antiga podia rodar comandos no notebook do Pedro.

## 2026-11-19 — Etapas 2 e 3: backup e remoção da instalação antiga (Claude Code)
- Backup: `pg_dump -Fc` com o banco no ar; depois `docker compose stop` e `tar` dos 3 volumes e de
  `~/agentes-chat` (inclui `.env` e `owner-link.txt`). Arquivo único na VM:
  `~/backups/agentes-chat-antigo-20261119T175843Z.tar.gz` (10 MB, `chmod 600`, pasta 700).
  **Contém segredos: nunca vai para o repo nem para o Drive.**
- Remoção: `docker compose down -v --rmi all --remove-orphans`; `rm -rf ~/agentes-chat /opt/agentes
  /root/.bun /tmp/agentes-install.*`; `docker system prune -af --volumes`. Docker ficou sem containers,
  volumes, imagens nem redes extras. Disco `/`: 14 GB → 9,1 GB usados.
- `/swap-agentes` tratado na Etapa 5 (swap).
- **Credenciais da instalação antiga, para trocar/revogar** (nomes, nunca valores):
  - `AGENTES_SECRET`, `POSTGRES_PASSWORD`, `OWNER_CLAIM_SHA256` (+ token do `owner-link.txt`): morreram
    com a instalação; a nova usa valores novos. Nada a fazer além de não reutilizar.
  - Senha da conta owner criada na instância antiga (se reutilizada em outro lugar, trocar lá).
  - Chaves de modelo de IA / conectores que tenham sido cadastrados no app antigo (ficavam cifradas no
    banco): revogar no provedor (Anthropic/OpenAI/etc.), se houver.
  - **Token do runner local** (`~/.agentes-runner/orgs/<id>/config.json`, campo `token`) e a sessão `codex`
    em `accounts/codex`: o servidor que os aceitava não existe mais; desconectar/remover o runner no app
    desktop e, se quiser, deslogar o Codex dali.

## 2026-11-19 — Etapas 4 e 5: preparo da VM e economia de recursos (Claude Code)
- `dnf -y upgrade` (308 pacotes, kernel UEK 6.12.0-208; reboot feito na Etapa 8).
- `usermod -aG docker opc`. firewalld `public`: removido `dhcpv6-client` → só `ssh http https`.
- Desligados (`systemctl disable --now`): rpcbind(+socket, mascarados), pmcd, pmie, pmie_farm, pmlogger,
  pmlogger_farm e timers do PCP, libstoragemgmt, gssproxy, dtprobed, atd. Mantidos: iscsid, tuned, oracle-cloud-agent.
- Swap: removidos `/swap-agentes` (2 GB) e `/.swapfile` antigo (945 MB); criado `/.swapfile` único de 2 GB.
  `vm.swappiness=20` em `/etc/sysctl.d/99-agentes-memoria.conf`.
- `/etc/docker/daemon.json`: logs json-file 10 MB × 3 para todos os containers.
- Limites no `docker-compose.override.yml`: app 512 MiB (`AGENTES_MEMORY`, swap até 1 GiB), Postgres 160 MiB
  (shared_buffers 32 MB, max_connections 15, sem paralelismo, jit off), Caddy 64 MiB. App: `DATABASE_POOL_MAX=5`,
  4 jobs simultâneos. Sem speech/embeddings/tailscale/backup.

## 2026-11-19 — Etapas 6 a 8: instalação v0.1.8, HTTPS e testes (Claude Code)
- Lidos README, SELF_HOSTING.md e install.sh da tag v0.1.8. Instalação feita **à mão** (não pelo
  `install.sh`/setup): o setup tenta transformar o servidor em "computador dos agentes" e baixa o compose do
  `main`; à mão fica com compose da tag e imagem `0.1.8@sha256:42bc349f…`.
- `.env` gerado na VM (segredos com `openssl rand`, 600). Subidos só postgres+app em localhost; owner criado
  via `POST /api/auth/owner-claim` com token de uso único (descartado) e senha aleatória de 32 caracteres
  (guardada só em `~/.agentes-segredos/owner-senha`, 600). Depois `compose up -d` com Caddy.
- Certificado Let's Encrypt (emissor YE2) obtido por tls-alpn-01, válido até 2027-02-17.
- Testes de fora (máquina local): `/api/health` 200; login 200; senha errada 401; sign-up → `SIGNUP_CLOSED`;
  http → 308 https; porta 3000 fechada.
- Reboot 1: app não voltou — esperado, eu tinha dado `compose stop agentes` antes (doc: container parado à mão
  fica parado). Criado `agentes-chat.service` (systemd) que sobe no boot e para o app antes do banco.
  Reboot 2 sem intervenção: os 3 containers voltaram healthy, HTTPS 200 de fora.
- Sessões criadas nos testes revogadas: `DELETE FROM sessions;` (2 linhas).
- **Consumo antes → depois** (com o agentes.chat rodando nos dois casos):

  | | Antes (instalação antiga) | Depois |
  |---|---|---|
  | RAM usada | 629 / 945 MiB | 663 / 945 MiB (logo após boot) |
  | Swap usado | 477 MiB de 2,9 GiB | 435 MiB de 2 GiB |
  | App / Postgres / Caddy | 170 / 94 / 15 MiB | 216 / 68 / 11 MiB |
  | Disco `/` | 14 GB | 14 GB (imagens 4,55 GB) |
  | Serviços systemd rodando | 34 | 25 |

  Leitura: os serviços desligados liberaram pouco (já estavam quase todos em swap); o ganho real é
  previsibilidade (limites por container, logs limitados, um swap só). Maior consumidor restante fora do
  agentes.chat: oracle-cloud-agent (~85 MiB RSS + ~145 MiB swap).

## Pendências
- Pedro: entrar, **trocar a senha** e apagar `~/.agentes-segredos/owner-senha` na VM.
- Pedro: desconectar/remover o runner local (`agentes-runner` do app desktop) que apontava para a instância antiga.
- Revogar chaves de IA/conectores cadastradas no app antigo, se houve.
- Decidir se desliga o oracle-cloud-agent (≈230 MiB entre RAM e swap).
- Backups do banco novo (perfil `backup` exige S3; alternativa: `pg_dump` em cron + cópia sem segredos).
- ~~Security List da OCI: conferir 22/80/443~~ — confirmado pelo Pedro em 2026-11-21.
- Monitor externo de `/api/health` (alerta no Telegram, quando o Saci existir).
- Fora desta tarefa: modelo de IA, bot do Telegram, coletores do Radar.

## 2026-11-21 — Status das pendências (Pedro)
- Feito: Security List só 22/80/443; custom instructions do ChatGPT atualizadas.
- Depois (Pedro): trocar senha owner + apagar `~/.agentes-segredos/owner-senha`; remover runner antigo;
  revogar chaves antigas; criar Projects no claude.ai; preencher UK Pact/Caatinga/TerraAnalytics; bancos.
- Radar de leilões: tratado em chat próprio e repo próprio (`pessoal/radar/leiloes/`).

## 2026-11-21 — Pessoal - Radar - Leilões: Etapa 0 (teste de acesso)
- Na VM: `curl` com User-Agent de Chrome em `https://www.leilaoimovel.com.br/leilao-de-imovel/santos-sp` → **403**,
  página "Just a moment..." (desafio JS do Cloudflare). Sem links de imóvel na resposta, então a página `/imovel/...` não foi testada.
- Teste com `curl_cffi` (impersonate) **não rodou**: o classificador de permissões do Claude Code bloqueou a ação
  por ser contorno de proteção anti-bot. Nada foi instalado na VM.
- Estado da VM no teste: RAM 668/945 MiB usada (277 MiB disponível), swap 541 MiB; Python 3.9.25; sem `pdftotext`;
  Caddy só em container (`agentes-chat-caddy-1`, caddy:2.11.7-alpine).
- Protótipo (`radar-leiloes-santos.html`, `santos.json`) não encontrado na máquina local.
- **Pendência:** Pedro escolher a estratégia de coleta (ver `pessoal/radar/leiloes/README.md`). Repo e coletor não foram criados.

## 2026-11-21 — Pessoal - Radar - Leilões: escolha de fonte e incidente de RAM
**Decisão (autonomia dada pelo Pedro ao sair):** fonte inicial = **Mega Leilões** (megaleiloes.com.br).
- Testes de acesso (UA de navegador, requisição comum, sem contorno):
  leilaoimovel 403 Cloudflare (VM e local); Caixa CSV → CAPTCHA Radware/perfdrive (VM e local);
  Mega Leilões 200 na VM, robots.txt `Allow: /` (só `/login` proibido). Santos/SP tem **5** imóveis lá hoje.
  Outros com 200 sem anti-bot aparente (candidatos a próximas fontes): Zukerman, Frazão, Lance Judicial,
  Biasi, D1 Lance, Santander Imóveis, Emgea. Com anti-bot: Superbid, Sold, Resale, Leilões Judiciais, Sodré Santoro (403).
- VM: GitHub por SSH configurado (`~/.ssh/config`, known_hosts com fingerprint ED25519
  `SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU`, igual ao publicado); `ssh -T` autenticou como PedroAQCoutinho.
- Código escrito e testado na máquina local (scratchpad, Python 3.13): coleta de 2 imóveis reais ok,
  geocode Nominatim ok, edital com texto, **matrículas da Mega são escaneadas (PDF sem texto)**. 9 testes de risco ok.
- **Incidente:** `sudo dnf install -y git-core poppler-utils` na VM levou a VM a swap pesado; ~10:20 BRT
  o SSH parou de responder (porta 22 aceita TCP, banner não vem) e o agentes.chat (`/api/health`) ficou fora.
  Sem OCI CLI local, não consegui reiniciar. Aguardando recuperação; se não voltar, **reboot pelo console OCI**.
  Lição: na E2.1.Micro, `dnf` precisa de `--setopt=install_weak_deps=False`, cache já pronto, e de preferência
  parar o agentes.chat antes (ou trocar para a A1 Flex).

## 2026-11-21 — Pessoal - Radar - Leilões: implantação na VM (concluída)
- **Recuperação:** a VM voltou sozinha ~11:44 BRT (sem reboot); o `dnf` terminou às 11:46 BRT
  (`git-core-2.52.0`, `poppler-utils-21.01.0`). O agentes.chat ficou inacessível por ~1h20 durante o swap.
- **Git na VM:** identidade global `Pedro Coutinho (VM Oracle)` / `PedroAQCoutinho@users.noreply.github.com`.
  Clone do `meu-contexto` em `~/meu-contexto` (a máquina local não tem credencial de push; os pushes desta sessão
  foram pela VM via `git bundle`). Houve um conflito com o commit 356d058 (registro de prompts), resolvido mantendo as duas versões.
- **Código:** `~/pessoal-radar-leiloes` (git local, remote `git@github.com:PedroAQCoutinho/pessoal-radar-leiloes.git`).
  `git ls-remote` → *Repository not found*: **repo ainda não existe** (criar exige token/gh, que não estão disponíveis).
  9 testes de risco ok no Python 3.9.
- **Coleta completa Santos/SP (Mega Leilões):** 5/5 ok, 0 erro, 53 s. **Pico de RSS do coletor: 39 MiB.**
  RAM do sistema antes/depois: 688/687 MiB usados, swap 539/538 MiB (sem impacto mensurável).
  Pela unit systemd (2ª coleta): 5 "igual", deduplicação ok.
- **Página:** `https://radar.147-15-67-178.sslip.io`, Caddy do agentes.chat.
  - Backup antes da mudança: `~/backups/docker-compose.override.yml.20261121T152844Z`.
  - `~/agentes-chat/docker-compose.override.yml`: o serviço `caddy` ganhou `env_file` (`~/.agentes-segredos/radar-caddy.env`)
    e 2 bind mounts `:ro,z` (SELinux enforcing): `site/` → `/srv/radar` e `deploy/radar.caddy` → `/etc/caddy/radar.caddy`.
    `configs.caddyfile` = cópia do Caddyfile oficial v0.1.8 + `import /etc/caddy/radar.caddy`.
  - `docker compose up -d caddy` (só o Caddy foi recriado). Certificado Let's Encrypt YE2 (http-01), válido até 2027-02-19.
  - Segredos novos (600): `~/.agentes-segredos/radar-site-senha` (usuário/senha, `openssl rand`) e `radar-caddy.env` (hash bcrypt).
  - Testes de fora: `/` e `/data.json` sem senha → 401; senha errada → 401; com senha → 200; http → 308;
    `/api/health` do agentes.chat → 200. SRI do Leaflet 1.9.4 conferido.
- **Agendamento:** `/etc/systemd/system/radar-leiloes.{service,timer}` (07:30 UTC ± 15 min, `Persistent=true`,
  `MemoryMax=200M`, `Nice=10`, `ExecStartPost=-git push`). `journald`: `/etc/systemd/journald.conf.d/50-limite.conf`
  (`SystemMaxUse=150M`, 30 dias). Journal atual: 8,7 MB.
- **Reboot de teste** (`systemctl reboot`, 15:33 UTC): em ~2 min os 3 containers voltaram healthy, timer ativo,
  site 401/200, health 200. RAM depois: 632 MiB, swap 374 MiB.
- **Pendências:** Pedro criar o repo vazio (o push sai sozinho na próxima coleta); pegar a senha do site; versionar o protótipo.
  Detalhes em `pessoal/radar/leiloes/README.md`.
