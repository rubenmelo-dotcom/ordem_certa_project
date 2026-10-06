# OrdemCerta

Sistema web de gestão de ordens de serviço para assistências técnicas de eletrônicos (celulares, notebooks, consoles), com área de acompanhamento para o cliente.

## Por que existe

Pequenas assistências controlam OS em papel, planilhas ou WhatsApp. Por isso, equipamentos se perdem no fluxo, orçamentos são aprovados sem registro, o estoque não bate, o cliente liga toda hora perguntando "está pronto?" e o dono não tem números do negócio.

O OrdemCerta centraliza tudo isso em um só lugar.

## Funcionalidades

- **Clientes e equipamentos:** cadastro e histórico por cliente.
- **Ordens de serviço:** numeração sequencial, defeito relatado e fotos de entrada.
- **Fluxo controlado:** máquina de estados da OS, com permissões por perfil em cada transição.
- **Orçamento:** montado com peças do estoque e serviços do catálogo; o cliente aprova ou recusa pelo sistema.
- **Estoque integrado:** baixa automática ao iniciar o reparo e estorno se a OS for cancelada.
- **Rastreabilidade:** histórico de status, comentários internos e públicos, fotos de entrada e saída.
- **Notificações:** e-mail ao cliente nos marcos importantes da OS.
- **Painéis e relatórios:** por perfil (faturamento, produtividade por técnico, peças mais usadas).

## Perfis de usuário

| Perfil | Responsabilidades |
|---|---|
| Gerente/dono | Visão do negócio, configurações e equipe |
| Atendente | Recebe e entrega equipamentos, abre OS |
| Técnico | Diagnóstico, orçamento e reparo |
| Cliente | Acompanha a OS e aprova ou recusa o orçamento |

## Stack

Python 3.12 · Django 5.2 LTS · SQLite · Django Templates · Tailwind CSS v4 (CLI standalone)

## Como executar

> _A definir._

## Licença

> _A definir._
