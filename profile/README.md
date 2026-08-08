# Pombo Correio

**Automação de relacionamento pós-venda, integrada ao ERP.**

O Pombo Correio conversa com os clientes de um negócio no momento certo, sem que ninguém precise
lembrar de fazer isso.

---

## O problema

Um salão de beleza atende centenas de pessoas por mês. Cada uma tem um ritmo próprio: a unha pede
retorno em três semanas, o cabelo em dois meses, e algumas simplesmente somem.

O dono sabe disso. Mas não tem como acompanhar pessoa por pessoa, todo dia, e mandar a mensagem
certa na hora certa. O resultado é receita que ficou na mesa — cliente que voltaria se tivesse
sido lembrado, e cliente que se foi sem ninguém perceber.

## O que fazemos

Lemos os dados que **já existem no ERP** — clientes, serviços do catálogo, atendimentos
realizados, agendamentos — e transformamos isso em conversa.

```
   ERP                Pombo Correio                    Cliente
    │                       │                             │
    │  atendimento          │                             │
    ├──────────────────────▶│                             │
    │                       │  "faz 2 dias que você        │
    │                       │   fez as unhas, ficou        │
    │                       ├──── tudo bem?" ────────────▶│
    │                       │                             │
```

Quem define as regras é o dono do negócio: **qual evento**, **quanto tempo depois**, **qual
mensagem**.

### O que o produto não é

**Não é um CRM.** Não tem funil de vendas, não tem gestão de equipe, não substitui o ERP. O ERP
continua sendo o dono dos dados; nós só os usamos para conversar.

---

## Os princípios

**Nunca virar spam.** É a restrição que molda o produto inteiro. Se um cliente faz quatro serviços
numa visita, ele recebe **uma** mensagem — não quatro. Há intervalo mínimo entre contatos, teto de
campanhas simultâneas e orçamento diário de envio. O dono controla todos esses números.

**Consentimento em primeiro lugar.** É o primeiro critério verificado antes de qualquer envio,
antes até de checar se o telefone é válido. Quem pede para sair, sai — e nenhuma sincronização do
ERP desfaz isso.

**Simples de operar.** Dois containers: a aplicação e o banco. Sem fila externa, sem orquestrador
de workflow, sem cluster analítico. A infraestrutura precisa caber no orçamento de um salão.

---

## Arquitetura

Quatro repositórios, com fronteiras deliberadas.

```
┌──────────────────────────┐        ┌──────────────────────────┐
│  Pombo.Correio.          │  HTTP  │  Pombo.Correio.          │
│  WebApplication          │───────▶│  Backend                 │
│                          │        │                          │
│  React · umi 4 · antd 6  │        │  .NET 10 · PostgreSQL    │
│  A interface             │        │  API + workers + domínio │
└──────────────────────────┘        └──────────────────────────┘
             ▲                                   ▲
             │        descrevem e orientam       │
      ┌──────┴───────────────────┬───────────────┴──────┐
      │                          │                      │
┌─────┴────────────────┐  ┌──────┴───────────────┐
│  Pombo.Correio.Docs  │  │ Pombo.Correio.Claude │
│                      │  │                      │
│  O que o produto faz │  │ Como se constrói     │
│  e por quê           │  │ e se revisa          │
└──────────────────────┘  └──────────────────────┘
```

### Por que essa separação

| Repositório | Responde | Muda quando |
|---|---|---|
| **Backend** | Como o produto funciona por dentro | Nasce uma funcionalidade |
| **WebApplication** | Como o produto é usado | Nasce uma tela |
| **Docs** | *O que* o produto faz, e **por quê** | Uma regra de negócio é decidida |
| **Claude** | *Como* se escreve, testa e revisa | Um padrão de trabalho amadurece |

**Documentação em repositório próprio** porque ela sobrevive ao código. Uma regra decidida hoje
vale independentemente de qual serviço a implementa amanhã — e separá-la impede que a explicação
morra junto com uma refatoração.

**Backend e front separados** porque têm ciclos diferentes. A interface muda muito mais que o
domínio, e amarrar os dois num repositório só faz cada ajuste visual arrastar o núcleo do produto.

**`Claude` existe** porque o desenvolvimento aqui é assistido por IA desde o primeiro commit. Ele
guarda a arquitetura, as convenções e as *skills* — procedimentos executáveis para criar uma
entidade, uma tela, um componente, e para revisar segurança e qualidade depois. A revisão de
segurança é obrigatória em toda entrega.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Backend | .NET 10 · EF Core · MediatR · PostgreSQL |
| Frontend | React · TypeScript · umi 4 · Ant Design 6 |
| Base | NextFoundry — scaffolding próprio, multi-tenant desde a primeira migration |
| Mensageria | Atrás de uma interface — o provedor é decisão de custo, não de arquitetura |

**Multi-tenant no dado desde o início.** Um cliente hoje, muitos amanhã, sem migration de risco no
meio do caminho.

---

<sub>Pombo Correio · desenvolvido com assistência de IA, revisado por gente.</sub>
