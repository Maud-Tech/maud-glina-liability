# 0008: Este serviço é o app de passivo da arquitetura Glina

Status: aceito (26/11/2026)

## Contexto
A arquitetura Glina ([desenho](../glina-architecture.svg)) tem três peças, cada uma num repositório:

| Peça | Repositório | Onde roda |
|---|---|---|
| API (porteiro) | `maud-glina-api` | AWS, conta da Maud |
| App de passivo | `maud-glina-liability` (este) | Maud, dentro de casa |
| App de gestão | `maud-glina-management` | Maud, dentro de casa |

O controle de passivo é um app separado do app de gestão desde o começo, porque o app de gestão pode ser vendido sozinho.

## Decisão
- Este serviço **sabe de quem é cada pedaço**. É com ele que a Glina fala, passando pelo porteiro.
- Tem API, Backend, Worker e PostgreSQL próprios, no padrão do boilerplate (ADRs 0001 a 0007).
- **Bancos separados, nenhuma tabela em comum** com o app de gestão.
- **Só o passivo chama a gestão, e só para ler.** A gestão não sabe que o passivo existe.
- Nenhum banco tem rota de fora. A VPN libera só o backend do passivo.
- O passivo só reserva ou vende enquanto vendidos + reservados ≤ total conferido pela gestão, e grava a versão usada.
- Conferência diária: soma dos pedaços = total conferido, e versão em uso = versão vigente. Se o total cai, a apólice congela e vira item de conferência. Nada se corrige sozinho.
- O passivo negocia a preço oficial e, se chega retificação, abre ajuste visível ao cliente.

## Consequências
- Tudo que este app precisa da gestão ele lê pela API de leitura dela e guarda uma cópia (com a versão). Nunca pelo banco dela.
- A leitura da gestão é um adapter em `integration/adapters/`, como qualquer sistema de fora.
- Dois apps para rodar, cada um com o seu banco e o seu backup.
