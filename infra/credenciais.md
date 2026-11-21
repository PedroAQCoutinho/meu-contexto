# Inventário de credenciais

**Só nomes, nunca valores.** Os valores vivem no gerenciador de senhas (fonte da verdade)
e, quando um sistema precisa deles, num `.env` com permissão 600 na máquina que os usa.

Regras:
- Um token por uso (nunca reaproveitar o mesmo token em dois sistemas).
- Permissão mínima e, quando o serviço permitir, data de expiração.
- Ao criar, trocar ou revogar: atualizar esta tabela (commit) e o gerenciador de senhas.
- Vazou ou passou por lugar suspeito → revogar no provedor primeiro, depois trocar.

| Credencial | Serviço | Escopo / o que permite | Onde está gravada | Criada | Expira / rotação | Como revogar |
|---|---|---|---|---|---|---|
| `oracle-vm-1.key` | SSH VM Oracle | login `opc` (sudo) | local `~/oracle/` | — | sem expiração | trocar chave em `~/.ssh/authorized_keys` da VM |
| chave ed25519 da VM | GitHub (chave SSH pessoal) | leitura+escrita em todos os repos (aprovado) | VM `~/.ssh/` | 2026-11-19 | sem expiração | GitHub → Settings → SSH keys |
| senha owner agentes.chat | agentes.chat | admin da instância | VM `~/.agentes-segredos/owner-senha` ⚠️ (trocar e apagar) | 2026-11-19 | trocar já | app → conta |
| `POSTGRES_PASSWORD` | Postgres do agentes.chat | banco da instância | VM `agentes-chat/.env` | 2026-11-19 | — | regenerar + reiniciar compose |
| `AGENTES_SECRET` | agentes.chat | cifra sessões/segredos do app | VM `agentes-chat/.env` | 2026-11-19 | — | regenerar (invalida sessões) |
| Certificado TLS | Let's Encrypt (Caddy) | HTTPS | VM (volume Caddy) | 2026-11-19 | renovação automática | — |
| token Telegram (Saci) | Telegram | controlar o bot | *a criar* | | | @BotFather → /revoke |
| chave de modelo de IA | *a definir (DeepSeek?)* | uso pago da API | *a criar* | | definir teto de gasto | painel do provedor |
| token MCP agentes.chat | agentes.chat | controlar agentes via Claude/ChatGPT | *a criar* | | | app → tokens |
| usuário somente-leitura `curupira` | iGPP | SELECT | *aguardando DBA* | | | DBA |
| usuário somente-leitura `terraanalytics` | TerraAnalytics | SELECT | *aguardando DBA* | | | DBA |

## Pendentes de revogação (instalação antiga)
- Chaves de IA/conectores cadastrados no app antigo, se houve.
- Token do runner local antigo (`~/.agentes-runner`).
