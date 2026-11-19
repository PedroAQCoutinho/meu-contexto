# Acesso aos bancos sem expor credenciais

Ambos são PostgreSQL. Padrão:

1. **Usuário somente-leitura** em cada banco (pedir ao DBA, ou criar se tiver permissão):
   ```sql
   CREATE ROLE ia_leitura LOGIN PASSWORD '<definir fora do repo>';
   GRANT CONNECT ON DATABASE curupira TO ia_leitura;
   GRANT USAGE ON SCHEMA public TO ia_leitura;
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO ia_leitura;  -- ou só views
   ALTER ROLE ia_leitura SET statement_timeout = '60s';
   ```
2. **Gateway MCP na VM Oracle**: um servidor MCP por banco (ou um com dois
   "perfis" isolados), expondo apenas: `listar_tabelas`, `descrever_tabela`,
   `consultar (SELECT)`. Credenciais em `.env` na VM, nunca no repo.
3. **Exposição**: HTTPS (Caddy/Let's Encrypt) + token/OAuth. Conectar como
   conector personalizado no Claude e no ChatGPT (Developer mode).
4. **Rede**: se o banco só aceita IPs liberados, pedir liberação do IP fixo da VM
   — ou túnel/VPN, se o instituto oferecer.

## Pendências
- [ ] Política do Instituto permite? (curupira)
- [ ] Contrato da TerraAnalytics permite? (terraanalytics)
- [ ] Usuários somente-leitura criados
- [ ] Gateway no ar
