# Glossário

Fonte única dos termos. **Todo nome no código é em inglês** (arquivos, classes, funções, variáveis, testes, campos, eventos); **comentários, docs e mensagens podem ser em português** ([ADR 0003](adr/0003-language.md)). A coluna "Código" é o nome que se usa no código. Termo novo entra aqui antes de entrar no código.

## Termos do negócio

Os nomes em inglês ainda **não foram escolhidos**: escolha com o time e preencha a coluna "Código" antes de criar o primeiro módulo. As definições abaixo são só o que o desenho da arquitetura diz.

| Português (negócio) | Código | Definição |
|---|---|---|
| App de passivo | este app (`maud-glina-liability`) | Sabe de quem é cada pedaço. É com ele que a Glina fala. |
| App de gestão | `maud-glina-management` | Sabe o que existe e quanto vale. Este app só lê dele. |
| Porteiro | `maud-glina-api` | A única porta pública da Maud. Repassa as chamadas da Glina para este app. |
| Cliente | _a definir_ | Dono de pedaços. |
| Apólice | _a definir_ | Tem pedaços e um total conferido. Congela se o total conferido cair. |
| Pedaço | _a definir_ | Tem um dono e um estado. A soma dos pedaços tem de ser igual ao total conferido. |
| Reserva | _a definir_ | Pedaços separados para um cliente antes da confirmação. Expira. |
| Negócio | _a definir_ | _a definir com o time_ |
| Ajuste | _a definir_ | Aberto quando chega retificação de um fechamento já usado. Visível ao cliente. |
| Total conferido | _a definir_ | Total que a gestão conferiu com a custódia. Teto de vendidos + reservados. |
| Fechamento | _a definir_ | Publicado pela gestão por apólice, data e versão. Este app guarda uma cópia do que leu. |
| Versão usada / vigente | _a definir_ | Versão do fechamento que o negócio usou / que vale hoje. |
| Preço do dia | _a definir_ | _a definir com o time_ |
| Item de conferência | _a definir_ | Aberto quando a conferência diária não bate. Nada se corrige sozinho. |
| Item | `Item` | Registro do módulo de demonstração `example`. Apague quando o módulo sair. |

## Termos da arquitetura (vêm do boilerplate)

| Português | Código | Definição |
|---|---|---|
| Módulo | `src/maud/modules/<módulo>/` | Contexto de negócio. Só importa módulos abaixo dele, e só pelo `api.py`. |
| Fachada | `api.py` | Única parte de um módulo que outros módulos podem importar. Mudar é mudar contrato. |
| Slice / caso de uso | `features/<verbo_substantivo>/` | Um comando ou uma consulta, com handler, schemas, endpoint e teste juntos. |
| Comando / consulta | `<Acao>Command` / `<Acao>Query` | Entrada de uma slice que muda dados / que só lê. |
| Domínio | `domain/` | Regras puras do módulo: sem Django, sem banco, sem relógio. |
| Data-base | `base_date` | Dia a que um número se refere (não confundir com quando foi calculado). |
| Dinheiro | `Money` | Valor + moeda, em Decimal. Moedas diferentes não operam; arredondar é explícito. |
| Cota | `Shares` | Quantidade de cotas, em Decimal com até 8 casas. Excesso de casas é recusado. |
| Outbox | `OutboxEvent` | Fila de eventos para fora, gravada na mesma transação do fato; o Worker entrega. |
| Chave de idempotência | `Idempotency-Key` | Identificador do pedido; repetir não duplica o efeito. |
| Ledger | `append_only(...)` | Tabela de lançamentos só com INSERT. Correção é estorno, nunca UPDATE/DELETE. |
| Estorno | `reversal` | Lançamento que anula outro, referenciando-o. O original nunca é apagado. |
