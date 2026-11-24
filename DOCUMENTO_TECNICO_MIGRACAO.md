# Bolão SaaS - Documento Técnico de Migração

## 1. Objetivo

Transformar o app `Comissões dos Bolões` em um SaaS web em .NET para controle de bolões, registro de vendas de cotas e cálculo de comissões das funcionárias e da gerente de lotéricas.

Nesta primeira fase, cada conta representa uma lotérica. Acesso individual para funcionárias, várias lotéricas por dono e cobrança por plano ficam planejados como evolução futura.

## 2. Contexto do Legado

O projeto legado é um único arquivo HTML com CSS e JavaScript embutidos, publicado no GitHub Pages e usado no celular como atalho na tela inicial. Também existem versões em planilha para Numbers e Planilhas Google.

Funcionalidades atuais:

- Cadastro de funcionárias com % sem meta, % da gerente sobre ela, meta (em R$ ou em cotas) e bônus.
- Marcação de uma funcionária como gerente (gerente que também vende).
- Faixas de comissão por meta, aplicadas sobre a tarifa (padrão: 40% da meta paga 15%, 60% paga 23%, 100% paga 28%; abaixo da primeira faixa paga 0%).
- Escolha da base da comissão: tarifa, cota inteira ou valor do bolão.
- Cadastro de bolões com valor total da cota, tarifa por cota, valor do bolão por cota calculado (cota menos tarifa), quantidade de cotas e valor total.
- Lançamento de vendas com data da venda e horário de registro (histórico rastreável).
- Fechamento por período com barra de meta, faixa atingida e quanto falta para a próxima.
- Comissão da gerente: percentual definido por funcionária sobre a base das vendas dela, mais a comissão das próprias vendas.
- Meta da lotérica.
- Envio do extrato pelo WhatsApp com mensagem pronta.
- Marcação de bolões como pagos.
- Botão Salvar, backup e restauração em JSON.

Limitações atuais:

- Dados ficam presos ao navegador de um aparelho (`localStorage`).
- No iPhone, o Safari pode apagar os dados de sites pouco usados.
- Não há login, controle de acesso nem histórico de quem lançou cada venda.
- Não há acesso simultâneo de vários aparelhos ou pessoas.
- Backup depende de ação manual (copiar e colar JSON).
- Não há relatórios por mês anterior além do que está no aparelho.
- Arquivo HTML único dificulta manutenção e testes.

## 3. Decisão de Produto para o MVP

O MVP deve resolver bem um problema principal:

> Permitir que a lotérica registre as vendas de cotas de bolões com confiança e feche as comissões do mês, de cada funcionária e da gerente, sem erro e sem perder dados.

Cada conta representa uma lotérica. Quem acessa no MVP é o dono e a gerente.

Fora do MVP inicial:

- Login individual para cada funcionária.
- Várias lotéricas por dono (rede de lotéricas).
- Integração com sistemas da Caixa.
- Conferência automática de resultados.
- Pagamento das comissões dentro do sistema.
- App nativo (o MVP é web responsivo, instalável como PWA).

## 4. Stack Recomendada

### Backend

- ASP.NET Core.
- .NET 10 (ou .NET 8 LTS).
- Entity Framework Core.
- PostgreSQL.

### Frontend

- Blazor Web App com renderização Interactive Server.
- Layout mobile first, porque o uso principal é no celular do balcão.
- PWA para instalar na tela inicial.

### Autenticação

- ASP.NET Core Identity com e-mail e senha.
- Papéis: `Dono` e `Gerente` no MVP; `Funcionaria` no futuro.

### Jobs em Segundo Plano

- Hangfire ou Quartz.NET.

Uso futuro:

- Fechamento automático no último dia do mês.
- Lembrete de bolões sem vendas lançadas.
- Envio automático do extrato.

### Pagamentos (assinatura do SaaS)

- Mercado Pago, já que o público é brasileiro.
- Modelar planos desde o início e integrar cobrança só depois do primeiro fluxo validado.

## 5. Arquitetura Inicial

Arquitetura em camadas simples:

- `Bolao.Web`: páginas Blazor, autenticação, endpoints.
- `Bolao.Application`: casos de uso, DTOs, validações, interfaces de repositório.
- `Bolao.Domain`: entidades e regras de negócio, incluindo o cálculo de comissões.
- `Bolao.Infrastructure`: EF Core, PostgreSQL, repositórios, jobs.
- `Bolao.Tests`: testes do domínio e dos casos de uso.

A regra mais sensível do produto (faixas de meta, base da comissão e parte da gerente) fica isolada no domínio, sem depender de banco ou tela, e coberta por testes.

## 6. Regras de Negócio Migradas

### 6.1 Valores do bolão

- `ValorBolaoPorCota = ValorTotalCota - TarifaPorCota`.
- `ValorTotalBolao = ValorTotalCota x QuantidadeCotas`.
- A tarifa não pode ser maior que o valor total da cota.
- A soma das cotas vendidas não pode passar da quantidade de cotas do bolão.

### 6.2 Base da comissão

Configurável na lotérica, com padrão `Tarifa`:

- `Tarifa`: cotas vendidas x tarifa por cota.
- `CotaInteira`: cotas vendidas x valor total da cota.
- `ValorBolao`: cotas vendidas x valor do bolão por cota.

### 6.3 Comissão da funcionária

- Sem meta cadastrada: `base x % sem meta`.
- Com meta: calcula o atingimento no período (`vendido em R$ / meta` ou `cotas / meta`) e aplica o percentual da maior faixa atingida sobre toda a base do período.
- Abaixo da primeira faixa: aplica o percentual configurado para abaixo da faixa (padrão 0%).
- Bônus fixo quando o atingimento chega a 100%.

### 6.4 Comissão da gerente

- Sobre cada funcionária que não seja a gerente: `base da funcionária x % da gerente definido nela`.
- Se a gerente também vende, recebe ainda a própria comissão como funcionária.
- A gerente não recebe percentual de gerente sobre as próprias vendas.

### 6.5 Período

- O fechamento considera a data da venda.
- Bolões marcados como pagos podem ser excluídos do fechamento.

## 7. Migração de Dados

- O app legado exporta um backup em JSON (`config`, `funcs`, `boloes`, `lanc`).
- O SaaS terá uma tela de importação que lê esse JSON e cria funcionárias, bolões e vendas, marcando a origem como `Legado`.
- Vendas migradas sem horário de registro recebem a data do bolão.

## 8. Riscos

- Diferença de arredondamento entre o legado (JavaScript) e o .NET: usar `decimal` e arredondar só na exibição e no fechamento.
- Uso em celular com internet instável: telas leves e confirmação clara de salvamento.
- Dados sensíveis de remuneração: acesso restrito por papel e por lotérica.
