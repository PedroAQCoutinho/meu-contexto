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

## Etapa 1 — NÃO executada
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
