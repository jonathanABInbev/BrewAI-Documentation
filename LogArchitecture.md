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
![Diagrama ER de banco de dados](https://github.com/user-attachments/assets/ccb6ba19-0f36-447e-bb54-1f3023cbbf22)

## 5. Consultas SQL e GraphQL na Arquitetura de Logs

### 5.1 Query 1: Recuperar Logs de Auditoria de um Monitor Específico

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
```
**Parâmetro:**

- $monitorId: ID! - Identificador único do monitor para o qual se deseja obter os logs de auditoria.

**Descrição:**

- A query começa identificando o nó específico (node) com base no ID fornecido ($monitorId).
- Se o nó identificado for um Monitor, a query continua a obter os auditLog relacionados a este monitor.
- Os logs de auditoria incluem:
  - **ID do Log** (id)
  - **Usuário** (user): Inclui id, name, e email.
  - **Timestamp** (timestamp): Momento em que a mudança ocorreu.
  - **ChangeSet** (changeSet): Detalha todas as mudanças aplicadas, como alterações em limiares, métricas, notas, contatos, e configurações específicas do monitor.

**Estrutura de Resposta:**

- A resposta é uma lista (edges) de logs de auditoria, onde cada log contém um nó (node) com os detalhes mencionados.

**Uso no Sistema:**

- Essa query é essencial para monitorar mudanças realizadas em monitores específicos, possibilitando auditorias detalhadas e análises retroativas de alterações críticas.
- Pode ser utilizada por administradores do sistema ou outros serviços que precisem rastrear a evolução das configurações de um monitor e os responsáveis por essas mudanças.

### **5.2 Exemplo de Resultado da Query 1**
**Objetivo:** O exemplo a seguir demonstra como interpretar a resposta da query getAuditLog, que retorna os logs de auditoria associados a um monitor específico.

**Query:**
```graphql
query getModelDetails($id: ID!) {
  node(id: $id) {
    ... on Model {
      id
      name
      uri
      createdAt
      space {
        id
        name
        createdAt
      }
      modelType
      modelPredictionVolume(startTime: "2024-08-21T13:00:00.000Z" endTime: "2024-08-28T13:00:00.000Z") {
        totalVolume
      }
      performanceMetrics(startDate: "2024-08-21T13:00:00.000Z" endDate: "2024-08-28T13:00:00.000Z") {
        accuracy
        mape
      }
      monitors(first: 50) {
        totalCount
        edges {
          node {
            id
            name
            creator {
              name
              email
              status
            }
            createdDate
            uri
            threshold
            latestComputedValue
            isManaged
            isTriggered
            driftMetric
            status
          }
        }
      }
      driftStatus {
        status
      }
      dataQualityStatus {
        status
      }
    }
  }
}
```
**Parâmetro:**

- `$id: String!` - Identificador único do modelo que você deseja consultar.

**Descrição:**

- **id**: Identificador único do modelo.
- **name**: Nome do modelo.
- **uri**: URI (Uniform Resource Identifier) que referencia o modelo.
- **createdAt**: Data e hora em que o modelo foi criado.
- **space**: Detalhes do espaço ao qual o modelo pertence.
- **modelType**: Tipo do modelo (ex.: categórico, regressão).
- **modelPredictionVolume**: Volume de predições feitas pelo modelo dentro de um intervalo de tempo especificado.
- **performanceMetrics**: Métricas de performance do modelo, como precisão e MAPE (Erro Percentual Absoluto Médio).
- **monitors**: Lista de monitores associados ao modelo.
- **driftStatus**: Status de drift do modelo.
- **dataQualityStatus**: Status de qualidade dos dados associados ao modelo.

**Interpretação do Resultado:**

A resposta desta query fornecerá uma visão completa sobre o modelo, incluindo sua performance e os monitores que estão acompanhando o modelo. Isso é útil para auditar a qualidade do modelo, entender sua utilização e gerenciar o desempenho ao longo do tempo.


### 5.3 Query 3: Listar Todos os Monitores de um Espaço Específico

**Objetivo:** Esta query GraphQL é usada para listar todos os monitores associados a um espaço específico dentro do sistema, trazendo informações detalhadas sobre cada monitor, como ID, nome, categoria, métricas de drift e qualidade de dados.

**Query:**
```graphql
query getMonitorsBySpace($id: ID!) {
  node(id: $id) {
    ... on Space {
      name
      monitors(first: 50) {
        totalCount
        edges {
          node {
            id
            name
            monitorCategory
            threshold
            latestComputedValue
            driftMetric
            dataQualityMetric
            customMetric {
              id
              name
            }
          }
        }
      }
    }
  }
}
```
**Parâmetro:**

- `$id: String!` - Identificador único do espaço que você deseja consultar.

**Descrição:**

- **name**: Nome do espaço.
- **monitors**: Lista de monitores associados ao espaço.
- **totalCount**: Número total de monitores no espaço.
- **edges**: Detalhes dos monitores, incluindo ID, nome, categoria, limiar, métricas de drift, e métricas de qualidade de dados.

**Interpretação do Resultado:**

Essa query é fundamental para administrar e auditar os monitores dentro de um espaço específico. Com ela, é possível verificar rapidamente o status dos monitores, identificar problemas de qualidade de dados ou detectar drift em modelos de machine learning.




