# Contexto do projeto

> O documento que humanos e agentes de IA leem para entender o negócio. Curto de propósito: decisão vira ADR, termo vira glossário.
> Desenho completo: [`glina-architecture.svg`](glina-architecture.svg).

## 1. Em uma frase
O **app de passivo** sabe de quem é cada pedaço. É com ele que a Glina fala, passando pelo porteiro.

## 2. Quem é dono do quê
| Assunto | Dono | Observação |
|---|---|---|
| Clientes, apólices, pedaços e estado, reservas, negócios, ajustes | **Este app** | No Postgres dele |
| Cópia dos fechamentos lidos | **Este app** | Lidos da gestão; o original é dela |
| Fechamentos, total conferido, pendências | App de gestão (`maud-glina-management`) | Este app só lê |
| Portal do cliente: cadastro, telas, pagamento | Sistema da Glina | Fala só com o porteiro |
| Porta pública: chave, limite de chamadas, log | Porteiro (`maud-glina-api`) | Sem banco, sem regra |

## 3. Atores e parceiros
```
Sistema da Glina ──https + chave──► porteiro (AWS) ──VPN, aberta de dentro──► API deste app
                                                                                   │
                                                              este app ──lê (sentido único)──► API de leitura do app de gestão
```
A resposta à Glina volta pelo mesmo caminho. A gestão não sabe que este app existe.

## 4. As peças
| Peça | O que faz |
|---|---|
| API | abrir conta, reservar, confirmar, vender, saldo, posição, preço do dia |
| Backend | um dono por pedaço; só vende até o total conferido pela gestão; grava a versão usada |
| Worker | expira reservas, lê a gestão, conferência diária, congela a apólice se o total conferido cair |
| PostgreSQL | clientes, apólices, pedaços e estado, reservas, negócios, ajustes, cópia dos fechamentos lidos |

## 5. Regras que não podem quebrar
Como não vende o que não existe:
1. A gestão confere a posição de cada apólice com a custódia e publica o total conferido. Só o que bateu conta.
2. O passivo só reserva ou vende enquanto vendidos + reservados ≤ total conferido.
3. Conferência diária: soma dos pedaços = total conferido, e versão em uso = versão vigente. Se o total cai, a apólice congela e vira item de conferência. Nada se corrige sozinho.

E também:
- Um dono por pedaço.
- O passivo negocia a preço oficial e, se chega retificação, abre ajuste visível ao cliente.
- Banco separado do app de gestão, nenhuma tabela em comum.
- Só o passivo chama a gestão, e só para ler.
- Nenhum banco tem rota de fora. A VPN libera só o backend do passivo.

Elas viram testes em `tests/acceptance/`, `tests/properties/` e `tests/concurrency/`.

## 6. Fora de escopo
- Cadastro do cliente, telas e pagamento: sistema da Glina.
- Fechamento, PL, conferência com a custódia: app de gestão.
- Porta pública: porteiro.

## 7. Fontes
- Desenho "Arquitetura Glina" (porteiro na AWS + app de passivo + app de gestão), fechado com o time em 26/11/2026.
