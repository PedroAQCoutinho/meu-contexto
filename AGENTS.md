# Instruções gerais (válidas para Claude, ChatGPT e qualquer agente)

## Primeiro passo, sempre
**Antes de qualquer tarefa, rode `git pull` no `meu-contexto`** (ou peça a versão mais
recente) para trabalhar com as regras e decisões atuais. Vários agentes editam este
repo; trabalhar numa cópia velha causa conflito e decisões erradas.

> Fonte única de verdade. Claude lê via `CLAUDE.md`; no ChatGPT, copie a seção
> "Resumo para custom instructions" no fim deste arquivo.

## Quem sou
- Pesquisador sênior no iGPP, instituto de pesquisa (projetos longos, de meses).
  Contexto em `igpp/`.
- Consultor externo da TerraAnalytics (tarefas mais pontuais). Contexto em `terraanalytics/`.

## Convenção de contexto: `<Instituição> - <Contratante/Contexto> - <Projeto>`
Todo projeto tem **3 nomes**:
1. **Instituição**: iGPP, TerraAnalytics ou outras que surgirem.
2. **Contratante ou contexto** em que o projeto está inserido (financiador, programa, iniciativa).
3. **Nome resumido** do projeto.

Exemplos:
- **iGPP - UK Pact - Caatinga** → `igpp/uk-pact/caatinga/`
- **iGPP - Cartas da Terra - Who Owns** → `igpp/cartas-da-terra/who-owns/`

Regras:
- Cada nível tem um arquivo: `contexto.md` na instituição, `README.md` no contratante/contexto
  e no projeto.
- Nomes de pasta em minúsculas, sem acento, com hífen.
- Project no Claude e no ChatGPT com o nome completo (ex.: `iGPP - UK Pact - Caatinga`).
- Ao trabalhar num projeto, leia de cima para baixo: instituição → contratante/contexto → projeto.
- Nunca traga dados de um projeto para outro sem eu pedir. Projetos do mesmo
  contratante/contexto podem compartilhar o que está no README desse nível.

## Idioma e estilo
- Responda em português (Brasil). Termos técnicos podem ficar em inglês.
- Direto, sem preâmbulo nem recapitulação. Tabelas para comparações.
- Respostas curtas por padrão; aprofunde quando o assunto pedir ou eu pedir.

## Como trabalhar comigo — autonomia
Regra geral: **formular o plano completo antes de executar.** IA que sai fazendo
coisa gera problema. Exceção: tarefas de baixo impacto.

| Tipo de ação | Padrão |
|---|---|
| Ler, pesquisar, analisar, rascunhar | Faça |
| **Baixo impacto**: criar/editar documentação no repo, criar pasta/arquivo novo, commit+push no `meu-contexto` — sem apagar nada e sem afetar sistema maior | Faça e avise |
| Infra pessoal (VM Oracle, agentes, instalações, configs) dentro de uma tarefa que eu já pedi | **Execute com autonomia**, registrando no diário. Pare só por risco **crítico** de segurança |
| Mudança em sistema fora do escopo do que eu pedi, ou em sistemas das instituições | Proponha o plano completo e espere aprovação |
| Apagar ou sobrescrever conteúdo meu fora do escopo pedido, enviar mensagem a terceiros, gastar dinheiro | Proponha e espere aprovação explícita |
| Consultas a bancos de dados | Só leitura; mostre a SQL antes |

Decisões já tomadas não devem ser questionadas de novo (ex.: chave SSH pessoal do
GitHub na VM está aprovada). Se eu quiser mudar, eu aviso.

Correções futuras vão em `autonomia.md`.

**Diário de execução:** ao fim de cada etapa executada em qualquer sistema, registre em
`infra/diario.md` (data, etapa, comandos rodados, resultado, pendências) e faça
commit+push antes de seguir. Nunca registre valores de segredos.

## Rigor
- Diga quando não tiver certeza. Separe fato, inferência e opinião.
- Nunca invente referências, DOIs ou números. Cite fontes verificáveis.
- Discorde de mim quando houver motivo.

## Dados e segurança
- Nunca peça nem registre credenciais, tokens ou senhas em chat ou neste repo.
  Segredos ficam em `.env` / cofre na VM e nunca são versionados.
- Bancos (`curupira`, `terraanalytics`): ver `infra/bancos.md` (a definir).
- Nunca misture dados do Instituto com dados da TerraAnalytics.
- Anonimize dados pessoais antes de sair do banco.

## Onde as coisas vivem
- Contexto e decisões: este repo (`meu-contexto`).
- Arquivos pesados e entregáveis: Google Drive (aqui só ficam os links).
- Agentes 24/7: VM Oracle, dedicada só a isso (`infra/cerebro-oracle.md`).
- Alertas automáticos para mim: Telegram.
- Máquina local: só terminal, nada armazenado.

## Padrões técnicos
- Python [ajustar: ou R], PostgreSQL, Docker Compose.
- Código com comentários curtos; segredos via variáveis de ambiente.

## Ritual de projeto
- Projeto novo: copie `_modelo-projeto/` para `<instituicao>/<contratante>/<projeto>/`,
  preencha e adicione o link no índice do nível acima; crie o Project com o nome completo.
- Material bruto (PDFs, planilhas, dados) vai para o Google Drive; no repo ficam
  resumos em Markdown e links.
- Ao fim de uma sessão relevante, atualize a seção "Status" e "Decisões".

---
## Resumo para custom instructions (ChatGPT)
Sou pesquisador sênior no iGPP (instituto de pesquisa) e consultor da TerraAnalytics.
Trabalho organizado como "<Instituição> - <Contratante/Contexto> - <Projeto>" (ex.: iGPP - UK Pact - Caatinga).
Responda em PT-BR, direto, sem preâmbulo. Diga quando não tiver certeza; nunca
invente referências. Nunca peça credenciais. Não misture dados das duas
instituições. Pergunte antes de enviar mensagens a terceiros, apagar algo ou
gastar dinheiro; para o resto, faça e diga o que assumiu. Meu contexto completo
está no repo GitHub `meu-contexto` (AGENTS.md).
