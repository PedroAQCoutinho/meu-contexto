# Fase 1 — refazer a VM do zero, com calma

Executor: Claude Code na máquina local do Pedro (chave em `~/oracle/oracle-vm-1.key`).
Regra: cada etapa é **proposta com os comandos exatos → aprovada → executada →
registrada em `infra/diario.md` → commit+push**. Nada de pular etapa.

Acesso: `ssh -i ~/oracle/oracle-vm-1.key opc@147.15.67.178`

## ✅ Etapa 0 — Diagnóstico (feita)
Ver `vm-oracle.md` e `diario.md`.

## Etapa 0b — Inventário da instalação antiga (somente leitura)
Na VM (usar `sudo docker`, pois `opc` não está no grupo docker):
- `sudo docker ps -a` · `sudo docker volume ls` · `sudo docker network ls`
- Localizar compose/projeto do agentes.chat: `sudo docker inspect` (labels
  `com.docker.compose.project.working_dir`).
- Arquivos `.env`: **listar só os NOMES das variáveis**, nunca valores
  (ex.: `cut -d= -f1 .env`).
- Serviços/agendamentos criados: `systemctl list-units --type=service --state=running`,
  `crontab -l`, `sudo crontab -l`.
- Quem acessa a VM: `cat ~/.ssh/authorized_keys | awk '{print $1, $3}'` (tipo + comentário).
- O que responde em 80/443/3000 (ex.: `curl -sI http://127.0.0.1:3000`).

Na máquina local:
- O que são `~/agentes` e `~/.agentes-runner` (listar, sem abrir segredos).
- Há runner/processo rodando? `ps aux | grep -i agent`, `systemctl --user list-units`.

## Etapa 0c — Mapa de permissões
Tabela: credencial (nome, não valor) → serviço → o que permite → onde está gravada.
Inclui: chaves de modelo de IA, tokens GitHub, Telegram, Google, e qualquer acesso
que o runner local tenha à máquina do Pedro.

## Etapa D — Decisões do Pedro (antes de remover qualquer coisa)
1. **Shape da VM**: continuar na E2.1.Micro (1 GB) ou criar uma **A1 Flex**
   (2 OCPU / 12 GB, Always Free)? Recomendação: A1 — o agentes.chat não cabe em 1 GB.
2. Salvar algo da instalação antiga? (config de agentes, memória) → backup no Drive.
3. **Trocar todas as credenciais** do mapa 0c (feito pelo Pedro em cada serviço).
4. Chave GitHub da VM: manter pessoal (escrita em tudo) ou trocar por deploy key
   somente-leitura do `meu-contexto`?
5. Remover a instalação antiga (deleção → aprovação explícita).

## Etapa N — Instalação limpa (detalhar após Etapa D)
- Sistema atualizado; `opc` no grupo docker; rpcbind desativado.
- Firewall: só o necessário (22; 80/443 apenas se a interface web for exposta,
  com HTTPS e autenticação).
- agentes.chat pelo repositório oficial, **versão fixada**; ler a doc atual antes de propor.
- Permissão mínima por agente; aprovações ligadas para ações sensíveis.
- Configuração (sem segredos) versionada em `infra/agentes-chat/`.
- Bot Saci (Pedro cria no @BotFather; token gravado direto na VM, permissão 600).
- Piloto: Radar + alertas para o Pedro.

## Fora da fase 1
Modelo de IA (candidato DeepSeek para o Radar), coletores, MCP do cérebro, bancos.
