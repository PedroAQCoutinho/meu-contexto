# agentes.chat na VM Oracle — configuração versionada (sem segredos)

Instalação limpa feita em 2026-11-19 (Claude Code). Detalhes da execução em `../diario.md`.

| Item | Valor |
|---|---|
| URL | https://147-15-67-178.sslip.io (Let's Encrypt via Caddy, renovação automática) |
| Versão | **v0.1.8**, fixada: `ghcr.io/agentes-chat/agentes:0.1.8@sha256:42bc349f…24bd5` |
| Compose oficial | `docker-compose.yml` = arquivo da tag `v0.1.8` (sha256 `62f9da59…8324e`), não editar |
| Ajustes locais | `docker-compose.override.yml` (limites de memória, Postgres enxuto) |
| Diretório na VM | `/home/opc/agentes-chat` (700); `.env` 600 só na VM |
| Serviço | `agentes-chat.service` (systemd): `compose up -d` no boot, para o app antes do Postgres no shutdown |
| Login | owner = conta do Pedro (e-mail pessoal); cadastro público fechado (`ALLOW_SIGNUP=false`) |
| Telemetria | `TELEMETRY=off`, `FEEDBACK_RELAY=false`, `PROBLEM_REPORT_URL=off` |

## Arquivos
- `docker-compose.yml` — oficial v0.1.8.
- `docker-compose.override.yml` — limites: app 512 MiB (+ swap até 1 GiB), Postgres 160 MiB, Caddy 64 MiB.
- `env.nomes` — nomes (e valores não secretos) do `.env`. Segredos gerados na VM com `openssl rand`.
- `agentes-chat.service` → `/etc/systemd/system/`.
- `daemon.json` → `/etc/docker/daemon.json` (logs json-file 10 MB × 3).
- `99-agentes-memoria.conf` → `/etc/sysctl.d/` (swappiness 20).

## Operação
```sh
ssh -i ~/oracle/oracle-vm-1.key opc@147.15.67.178
cd ~/agentes-chat
docker compose ps                      # opc está no grupo docker
docker compose logs -f agentes
curl -fsS https://147-15-67-178.sslip.io/api/health
```
Reboot planejado: `sudo systemctl reboot` (o serviço para o app antes do banco).

## Atualizar (só manual)
1. Ler o release notes da nova tag em github.com/agentes-chat/agentes-chat/releases.
2. `pg_dump` antes (`docker compose exec -T postgres pg_dump -U agentes -d agentes -Fc > ~/backups/…`).
3. Baixar o `docker-compose.yml` **da tag** (`raw.githubusercontent.com/agentes-chat/agentes-chat/vX.Y.Z/docker-compose.yml`),
   trocar `AGENTES_IMAGE` no `.env` pela tag+digest novos, `docker compose pull && docker compose up -d`.
4. Atualizar este README, `env.nomes` e o diário.

## Não instalado (de propósito)
Perfis `speech`, `embeddings`, `tailscale`, `backup` (S3); runner/"computador dos agentes" no servidor;
modelo de IA; bot do Telegram; coletores do Radar.
