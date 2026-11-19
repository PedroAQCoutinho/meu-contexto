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
