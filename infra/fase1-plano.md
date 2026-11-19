# Fase 1 — VM Oracle + Hermes (sem modelo de IA)

Executor: Claude Code na máquina local do Pedro (chave em `~/oracle/oracle-vm-1.key`).
Regra: **cada etapa é proposta com os comandos exatos, aprovada, executada e
registrada** (resultado em `infra/vm-oracle.md` ou aqui). Nada de pular etapa.

Acesso: `ssh -i ~/oracle/oracle-vm-1.key opc@147.15.67.178`

## Etapa 0 — Diagnóstico (somente leitura, nada é alterado)
- `cat /etc/os-release` · `uname -m` · `nproc` · `free -h` · `df -h`
- `docker --version` / `podman --version`
- Firewall da VM: `sudo firewall-cmd --list-all` (Oracle Linux) ou `sudo iptables -S`
- Portas escutando: `sudo ss -tlnp`
- Saída: preencher a tabela de `infra/vm-oracle.md` (trocar *suposto* por **confirmado**).

## Etapa 1 — Sistema e Docker
- Atualizar pacotes do sistema.
- Instalar Docker Engine + plugin Compose pelo repositório oficial adequado ao SO
  confirmado na etapa 0; habilitar serviço; adicionar `opc` ao grupo docker.
- Teste: `docker run --rm hello-world`.

## Etapa 2 — Segurança básica
- Confirmar entrada só na porta 22 (firewall da VM e Security List da Oracle).
- SSH apenas por chave (sem senha). Atualizações automáticas de segurança.
- Alerta de orçamento na Oracle (US$ 0,01) — feito pelo Pedro no console.

## Etapa 3 — Bot Saci no Telegram
- Pedro cria o bot no @BotFather (nome "Saci", usuário `saci_bot` ou reserva).
- Token NUNCA no repo nem no chat: Pedro grava direto na VM em
  `~/hermes/.env` com permissão 600.

## Etapa 4 — Hermes Agent (Docker, versão fixada)
- Ler a documentação oficial atual do Hermes e propor o `docker-compose.yml`
  com versão fixa (sem `latest`).
- Perfis: `pessoal` (Telegram) e `radar` (coletas; vazio por enquanto).
- Sem modelo de IA configurado ainda (decisão pendente). Validar só que o
  gateway sobe e reinicia sozinho após reboot.

## Etapa 5 — Hermes lê o `meu-contexto`
- Gerar chave SSH na VM; Pedro adiciona como **Deploy key somente-leitura**
  em GitHub → meu-contexto → Settings → Deploy keys.
- Clonar na VM e agendar `git pull` periódico.

## Etapa 6 — Backup
- Backup de memória/perfis/configs (sem segredos) para o Google Drive
  `AI-AGENTS/HERMES`. Ferramenta a propor (ex.: rclone). Pode ficar para a fase 2.

## Fora da fase 1
Modelo de IA, coletores do Radar, MCP do cérebro, bancos de dados.
