# VM Oracle (Always Free)

Uso exclusivo: gerenciador de agentes (Hermes) e seus coletores.

| Item | Valor |
|---|---|
| IP público | 147.15.67.178 |
| Usuário SSH | `opc` (imagem Oracle Linux → usar `dnf`) |
| Chave | `~/oracle-vm-1.key` na máquina local (a chave NUNCA entra no repo) |
| Shape | _a confirmar_ (A1 Flex até 2 OCPU / 12 GB no Always Free) |
| Portas de entrada | só 22 (SSH). Telegram usa conexão de saída. |

Acesso: `ssh -i ~/oracle-vm-1.key opc@147.15.67.178`

Atalho opcional em `~/.ssh/config` da máquina local:
```
Host oracle
    HostName 147.15.67.178
    User opc
    IdentityFile ~/oracle-vm-1.key
```
