# VM Oracle (Always Free)

Uso exclusivo: gerenciador de agentes (Hermes) e seus coletores.

> Legenda: **confirmado** = informado por você ou verificado na VM;
> *suposto* = inferido, ainda não verificado.

| Item | Valor |
|---|---|
| IP público | 147.15.67.178 (**confirmado**) |
| Usuário SSH | `opc` (**confirmado**) — *suposto: imagem Oracle Linux, pois `opc` é o usuário padrão dela* |
| Chave | `~/oracle/oracle-vm-1.key` na máquina local (**confirmado**; a chave NUNCA entra no repo) |
| Shape | _a confirmar_ (A1 Flex até 2 OCPU / 12 GB no Always Free) |
| Portas de entrada | *planejado: só 22 (SSH)*; estado atual não verificado. Telegram usa conexão de saída. |

Acesso: `ssh -i ~/oracle/oracle-vm-1.key opc@147.15.67.178`

Atalho opcional em `~/.ssh/config` da máquina local:
```
Host oracle
    HostName 147.15.67.178
    User opc
    IdentityFile ~/oracle/oracle-vm-1.key
```

## Verificar no primeiro acesso
- [ ] Sistema operacional e versão (`cat /etc/os-release`)
- [ ] CPU / memória / disco (`nproc`, `free -h`, `df -h`)
- [ ] Docker instalado?
- [ ] Portas abertas (firewall da VM + Security List da Oracle)
