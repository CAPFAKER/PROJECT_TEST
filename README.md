# Bolão SaaS

Projeto de migração do app **Comissões dos Bolões** (HTML único com dados no celular) para um SaaS em .NET, com foco em controle de bolões, registro de vendas de cotas e cálculo de comissões das funcionárias e da gerente de lotéricas.

## Objetivo

Transformar o app atual, que roda num único arquivo HTML e guarda os dados no navegador de um aparelho, em uma aplicação web SaaS. Cada lotérica passa a ter conta própria, dados na nuvem, acesso de vários aparelhos e fechamento de comissões confiável.

Nesta primeira fase, cada conta representa **uma lotérica**, operada pelo dono ou pela gerente. Login individual para cada funcionária fica planejado como evolução futura.

## Status do Projeto

Status atual: **fase de documentação e planejamento técnico**, com a solution .NET criada e o domínio de cálculo de comissões implementado e coberto por testes.

## Documentação

### Visão Geral

- [Documento Técnico de Migração](./DOCUMENTO_TECNICO_MIGRACAO.md)

### Documentos Principais

- [Entidades do Banco de Dados](./docs/01-entidades-banco.md)
- [Arquitetura do Projeto](./docs/02-arquitetura-projeto.md)
- [Funcionalidades do Produto](./docs/03-funcionalidades.md)
- [Modelagem Completa do Banco de Dados](./docs/04-modelagem-completa-banco.md)
- [Arquitetura em Ambiente Real Simulado](./docs/05-arquitetura-ambiente-real-simulado.md)
- [Esboço Inicial das Telas](./docs/06-esboco-inicial-telas.md)
- [Board de Próximos Passos](./docs/07-board-proximos-passos.md)
- [Padrão de Branches](./docs/08-padrao-branches.md)
- [Wireframe SVG para Figma](./docs/wireframes/bolao-wireframes.svg)

## Ordem Recomendada de Leitura

1. [Documento Técnico de Migração](./DOCUMENTO_TECNICO_MIGRACAO.md)
2. [Funcionalidades do Produto](./docs/03-funcionalidades.md)
3. [Entidades do Banco de Dados](./docs/01-entidades-banco.md)
4. [Arquitetura do Projeto](./docs/02-arquitetura-projeto.md)
5. [Modelagem Completa do Banco de Dados](./docs/04-modelagem-completa-banco.md)
6. [Arquitetura em Ambiente Real Simulado](./docs/05-arquitetura-ambiente-real-simulado.md)
7. [Esboço Inicial das Telas](./docs/06-esboco-inicial-telas.md)
8. [Board de Próximos Passos](./docs/07-board-proximos-passos.md)
9. [Padrão de Branches](./docs/08-padrao-branches.md)

## Escopo Inicial do MVP

- Cadastro e login da lotérica (dono ou gerente).
- Cadastro de funcionárias com percentual, meta e bônus.
- Configuração das faixas de comissão por meta.
- Cadastro de bolões (valor da cota, tarifa, quantidade de cotas).
- Lançamento de vendas de cotas com data e histórico rastreável.
- Fechamento mensal de comissões, incluindo a gerente.
- Envio do extrato pelo WhatsApp (link pronto).
- Relatórios simples por período.

## Stack Planejada

- .NET 10 (ou .NET 8 LTS).
- ASP.NET Core.
- Blazor Web App (Interactive Server).
- ASP.NET Core Identity.
- Entity Framework Core.
- PostgreSQL.
- Hangfire ou Quartz.NET para jobs futuros (fechamento automático, lembretes).
- Serilog para logs estruturados.

## Estrutura Atual

```text
BolaoSaaS/
  DOCUMENTO_TECNICO_MIGRACAO.md
  README.md
  index.html              (app atual, mantido na raiz para o link do GitHub Pages continuar funcionando)
  Bolao.slnx
  docker-compose.yml
  docs/
    README.md
    01-entidades-banco.md ... 08-padrao-branches.md
    wireframes/
      bolao-wireframes.svg
  legado/
    index.html            (cópia de referência do app atual)
    planilhas/            (versões em Numbers e Planilhas Google)
  src/
    Bolao.Web/
    Bolao.Application/
    Bolao.Domain/
    Bolao.Infrastructure/
  tests/
    Bolao.Tests/
```

## Como Rodar Localmente

```bash
docker compose up -d          # sobe PostgreSQL e pgAdmin
dotnet build Bolao.slnx
dotnet test Bolao.slnx
dotnet run --project src/Bolao.Web
```

## Próximos Passos

1. Revisar e ajustar os wireframes no Figma.
2. Validar a modelagem do banco.
3. Configurar EF Core, PostgreSQL e Identity.
4. Implementar cadastros (funcionárias, bolões, faixas).
5. Implementar lançamento de vendas e fechamento usando o domínio já pronto.
6. Migrar dados do app legado via importação do backup JSON.

## Origem do Projeto

Este SaaS nasce do app **Comissões dos Bolões**, um HTML único que já calcula comissões por faixa de meta sobre a tarifa, considera a gerente que também vende e envia extratos pelo WhatsApp, mas guarda tudo apenas no aparelho de quem usa. O app legado continua disponível na pasta `legado/`.
