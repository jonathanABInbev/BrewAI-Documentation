# Documentação da Arquitetura de Logs

## Sumário

1. [Visão Geral da Arquitetura](#visao-geral-da-arquitetura)
2. [Componentes da Arquitetura](#componentes-da-arquitetura)
    - [Modelos](#modelos)
    - [Triggers](#triggers)
    - [Informações de Log](#informações-de-log)
3. [Relações e Interações](#relações-e-interações)
4. [Diagramas e Fluxos](#diagramas-e-fluxos)
5. [Considerações e Próximos Passos](#considerações-e-próximos-passos)
6. [Consultas SQL e GraphQL na Arquitetura de Logs](#consultas-sql-e-graphql-na-arquitetura-de-logs)
    - [Query 1: Recuperar Logs de Auditoria de um Monitor Específico](#query-1-recuperar-logs-de-auditoria-de-um-monitor-específico)
    - [Query 2: Recuperar Informações de um Modelo Específico](#query-2-recuperar-informações-de-um-modelo-específico)
    - [Query 3: Listar Todos os Monitores de um Espaço Específico](#query-3-listar-todos-os-monitores-de-um-espaço-específico)
7. [Conclusão e Próximos Passos](#conclusão-e-próximos-passos)

## 1. Visão Geral da Arquitetura

A arquitetura de logs apresentada é composta por três principais entidades: **Modelos**, **Triggers** e **Informações de Log**. Essa estrutura é projetada para organizar e registrar eventos específicos dentro de um sistema, permitindo o monitoramento e auditoria detalhados das atividades.

## 2. Componentes da Arquitetura

### 2.1 Modelos

- **Descrição:** Entidade que agrupa diferentes categorias ou classes de dados no sistema.
- **Atributos:**
  - `Id` (Chave Primária): Identificador único do modelo.
  - `Nome Modelo`: Nome descritivo do modelo.
- **Relação:** Cada `Modelo` pode ter múltiplos `Triggers` associados.

### 2.2 Triggers

- **Descrição:** Entidade que representa eventos ou condições específicas associadas a um `Modelo`. Quando um evento ocorre, ele ativa o trigger correspondente.
- **Atributos:**
  - `Id Trigger` (Chave Primária): Identificador único do trigger.
  - `Nome Trigger`: Nome descritivo do trigger.
- **Relação:** Cada `Trigger` está ligado a um `Modelo` e pode estar associado a múltiplas `Informações de Log`.

### 2.3 Informações de Log

- **Descrição:** Entidade que armazena os detalhes específicos de um evento disparado por um `Trigger`. Essa informação é essencial para rastrear e entender as atividades no sistema.
- **Atributos:**
  - `Id Trigger` (Chave Estrangeira): Referência ao trigger que gerou a informação de log.
  - `Informações de Log`: Dados detalhados sobre o evento que foi capturado.
- **Relação:** Cada `Trigger` pode ter uma quantidade `n` de `Informações de Log` associadas.

## 3. Relações e Interações

- Um `Modelo` pode estar relacionado a vários `Triggers`.
- Cada `Trigger` está ligado a um `Modelo` específico e pode desencadear várias `Informações de Log`.
- As `Informações de Log` são específicas para cada `Trigger` e registram todos os detalhes relevantes de cada evento.

## 4. Diagramas e Fluxos

### 4.1 Diagrama Entidade-Relacionamento (ER)

(Incluir aqui o diagrama ER mostrando as relações entre `Modelos`, `Triggers`, e `Informações de Log`.)

### 4.2 Fluxos de Processo

(Incluir aqui fluxos de processo ou diagramas de sequência que ilustram o fluxo de dados e eventos entre os componentes da arquitetura.)

## 5. Considerações e Próximos Passos

- **Normalização dos Dados:** Considerar a normalização das tabelas para evitar redundância de dados.
- **Performance:** Avaliar a performance das consultas, especialmente em cenários com grande volume de logs.
- **Segurança:** Implementar políticas de segurança para proteger o acesso aos logs sensíveis. Isso pode incluir criptografia dos dados, autenticação robusta e controle de acesso baseado em funções.

## 6. Consultas SQL e GraphQL na Arquitetura de Logs

### 6.1 Query 1: Recuperar Logs de Auditoria de um Monitor Específico

**Objetivo:**
Essa query GraphQL é utilizada para recuperar os logs de auditoria associados a um monitor específico no sistema. Esses logs de auditoria contêm informações detalhadas sobre mudanças realizadas em configurações e eventos relacionados ao monitor.

**Query:**
```graphql
query getAuditLog($monitorId: ID!) {
  node(id: $monitorId) {
    ... on Monitor {
      id
      auditLog {
        edges {
          node {
            id
            user {
              id
              name
              email
            }
            timestamp
            changeSet {
              threshold
              stdDevMultiplier
              dynamicAutoThresholdEnabled
              operator
              downtimeStart
              downtimeDurationHrs
              downtimeFrequencyDays
              name
              notes
              contacts {
                monitorId
                emailAddress
                notificationChannelType
                integration {
                  name
                  providerName
                  alertSeverity
                  channelName
                }
              }
              metric
              isNormalized
              evaluationIntervalSeconds
              topKPercentileValue
              metricAtRankingKValue
              positiveClass
              customMetric {
                metric
                name
              }
              primaryWindowModelName
              primaryWindowModelEnvironmentName
              primaryWindowFilters {
                filterType
                operator
                dimension {
                  name
                }
                dimensionValues {
                  value
                }
              }
              comparisonWindowLengthMs
              comparisonWindowStartDelaySeconds
              comparisonWindowCustomBaseline {
                modelEnvironmentName
                filteredMovingWindowSeconds
                filteredMovingWindowDelaySeconds
                filteredStartDate
                filteredEndDate
              }
              comparisonWindowDimension {
                name
              }
              comparisonWindowModelName
            }
          }
        }
      }
    }
  }
}



