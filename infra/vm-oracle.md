# VM Oracle (Always Free)

Uso exclusivo: gerenciador de agentes (Hermes) e seus coletores.

> Legenda: **confirmado** = informado por você ou verificado na VM;
> *suposto* = inferido, ainda não verificado.

| Item | Valor |
|---|---|
| IP público | 147.15.67.178 (**confirmado**) |
| Usuário SSH | `opc` (**confirmado**) |
| Chave | `~/oracle/oracle-vm-1.key` na máquina local (**confirmado**; a chave NUNCA entra no repo) |
| SO | Oracle Linux Server 9.8, kernel 6.12.0 UEK (**confirmado**) |
| Arquitetura / CPU / RAM | x86_64, 2 vCPU, 945 MiB RAM + 2,9 GiB swap (**confirmado**) |
| Shape | *suposto: VM.Standard.E2.1.Micro* (AMD Always Free, 1 OCPU / 1 GB). **Não** é A1 Flex |
| Disco | `/` 30 GB (46% usado, 17 GB livres); `/var/oled` 15 GB (**confirmado**) |
| Docker | 29.10.0 já instalado e **com containers rodando** (**confirmado**); Podman ausente |
| Firewall da VM | firewalld ativo, zona `public` em `ens3`: serviços `ssh`, `http`, `https`, `dhcpv6-client` (**confirmado**) |
| Portas escutando | 22 (sshd), 80 e 443 (docker-proxy), 111 (rpcbind) em 0.0.0.0; 3000 (docker) e PCP 4330/44321 só em localhost (**confirmado**) |
| Security List OCI | entrada TCP 22 existe (Pedro, console). Demais regras: *a verificar* |

Acesso: `ssh -i ~/oracle/oracle-vm-1.key opc@147.15.67.178`

Atalho opcional em `~/.ssh/config` da máquina local:
```
Host oracle
    HostName 147.15.67.178
    User opc
    IdentityFile ~/oracle/oracle-vm-1.key
```

## Etapa 0 — tentativa 1 (2026-11-19): sem conexão
- `ssh ... opc@147.15.67.178` → `Connection timed out` na porta 22. Nenhum comando rodou na VM.
- Chave local OK (permissão 600).
- Testes locais: ping 100% de perda (não conclusivo, ICMP bloqueado por padrão na OCI);
  `nc` TCP 22 na VM → timeout; `nc` TCP 22 no github.com → sucesso
  (a rede local **não** bloqueia saída na 22).
- Console OCI (Pedro): instância **Running**, IP público **147.15.67.178** (**confirmado**).
- Hipóteses restantes (*a verificar no console*): Security List/NSG sem entrada TCP 22;
  subnet sem rota `0.0.0.0/0` → Internet Gateway (subnet privada).

## Etapa 0 — tentativa 2 (2026-11-19): diagnóstico concluído
Conexão normalizou depois de ajuste no console da OCI. Resultados na tabela acima.

Achados que mudam o plano da fase 1:
- **RAM de ~1 GB**, não 12 GB: 678 MiB já em uso e 340 MiB em swap antes do Hermes.
- **A VM não está vazia**: Docker já instalado, com containers publicando 80/443
  (443 também em UDP, típico de proxy reverso com HTTP/3) e um serviço em 127.0.0.1:3000.
  Origem desconhecida; contradiz o "uso exclusivo" acima.
- Há uma segunda sessão de usuário (`/run/user/988`), *suposto: conta de serviço*.
- rpcbind (111) escuta em 0.0.0.0; o firewalld bloqueia, mas é um serviço desnecessário.
- `iptables -S` mostra só as regras da Oracle e do Docker; o firewalld usa nftables.

## Verificar no primeiro acesso
- [x] Sistema operacional e versão (`cat /etc/os-release`)
- [x] CPU / memória / disco (`nproc`, `free -h`, `df -h`)
- [x] Docker instalado? Sim, 29.10.0, com containers ativos
- [x] Portas abertas no firewall da VM
- [ ] Security List da Oracle (regras além da TCP 22)
- [ ] O que são os containers em 80/443/3000?
- [ ] Confirmar o shape (metadados da instância ou console)
