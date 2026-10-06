# MVP Scope & Delivery Plan v1.0

## 1. Objetivo

Este documento define o escopo oficial do primeiro MVP da plataforma Industrial IoT.

Ele estabelece:

- funcionalidades incluídas;
- funcionalidades excluídas;
- prioridades;
- épicos;
- dependências;
- ordem de implementação;
- milestones;
- critérios de aceite;
- Definition of Done;
- critérios de qualidade;
- estratégia de testes;
- critérios para entrada em homologação;
- critérios para entrada em produção.

O objetivo principal é impedir expansão descontrolada de escopo durante a primeira fase de desenvolvimento.

---

# 2. Objetivo do MVP

O MVP deve provar que a plataforma é capaz de executar, de ponta a ponta, o seguinte fluxo:

```text
Industrial Device
      ↓
Edge
      ↓
Protocol Adapter
      ↓
Device Adapter
      ↓
Normalized Telemetry
      ↓
MQTT / HTTPS
      ↓
IoT Core
      ↓
TimescaleDB
      ↓
Alarm Processing
      ↓
API
      ↓
Realtime Dashboard
```

O MVP deve permitir que um cliente real tenha:

```text
Organization
↓
Site
↓
Area
↓
System
↓
Asset
↓
Device
↓
Telemetry
↓
Dashboard
↓
Alarm
```

funcionando de forma isolada e segura.

---

# 3. Critério principal de sucesso

O MVP será considerado tecnicamente válido quando for possível:

1. criar uma organização;
2. cadastrar usuários;
3. cadastrar um site;
4. cadastrar ativos;
5. cadastrar gateway e devices;
6. provisionar um Edge;
7. conectar pelo menos um equipamento industrial real ou simulador;
8. coletar dados via protocolo industrial;
9. normalizar métricas;
10. armazenar telemetria;
11. visualizar estado atual;
12. consultar histórico;
13. detectar perda de comunicação;
14. gerar alarme;
15. reconhecer alarme;
16. operar Store & Forward;
17. garantir isolamento entre tenants;
18. auditar operações sensíveis.

---

# 4. Escopo funcional do MVP

O MVP será composto pelos seguintes domínios:

```text
Identity
Tenancy
Asset Management
Device Management
Edge
Telemetry
Events
Alarms
Dashboard
Audit
Observability
```

---

# 5. Fora do MVP

Os seguintes recursos ficam explicitamente fora da primeira versão:

```text
OTA completo

Digital Twin completo

Mobile App nativo

Machine Learning

IA generativa

Manutenção preditiva

Kafka

Kubernetes

Microservices

Billing

Marketplace

CMMS completo

ERP

Workflow Engine

Advanced Reporting

Custom Dashboard Builder

Multi-region

High Availability complexa

White-label avançado
```

Esses recursos poderão ser planejados após estabilização do MVP.

---

# 6. Comandos remotos

A infraestrutura para comandos deve existir no MVP.

Porém comandos operacionais reais poderão ser ativados em uma segunda milestone.

Isso significa implementar inicialmente:

```text
Command entity

Command contract

MQTT topic

Command queue

Command status

Authorization structure

Audit structure
```

Mas não é obrigatório liberar:

```text
START
STOP
RESET
SET_SPEED
```

para equipamentos reais no primeiro release.

---

# 7. Prioridades

Serão utilizadas três prioridades:

```text
P0
Obrigatório para MVP

P1
Importante após núcleo funcional

P2
Preparado arquiteturalmente, mas não obrigatório
```

---

# 8. Matriz de prioridade

| Componente | Prioridade |
|---|---|
| Authentication | P0 |
| Organizations | P0 |
| Memberships | P0 |
| Sites | P0 |
| Areas | P0 |
| Systems | P0 |
| Assets | P0 |
| Gateways | P0 |
| Devices | P0 |
| Device Models | P0 |
| Metric Definitions | P0 |
| Device Metric Mapping | P0 |
| Edge Provisioning | P0 |
| Modbus Adapter | P0 |
| Device Adapter Interface | P0 |
| MQTT | P0 |
| HTTP Ingestion | P0 |
| Telemetry | P0 |
| Historical Telemetry | P0 |
| Device Connectivity | P0 |
| Store & Forward | P0 |
| Events | P0 |
| Basic Alarm Engine | P0 |
| Basic Dashboard | P0 |
| Audit | P0 |
| RLS | P0 |
| Observability | P0 |
| WebSocket | P1 |
| Notifications | P1 |
| Commands | P1 |
| Dashboard Templates | P1 |
| Reports | P1 |
| Device Twin | P2 |
| Firmware / OTA | P2 |
| Advanced Analytics | P2 |
| External Webhooks | P2 |

---

# 9. Primeiro cenário de referência

O desenvolvimento deve possuir um cenário padrão para testes.

Exemplo recomendado:

```text
Organization
Industrial Demo

Site
Plant 01

Area
Utilities

System
Compressed Air

Asset
Compressor 01

Gateway
Edge 01

Devices
Energy Meter
Pressure Sensor
```

Métricas:

```text
electrical.voltage.line_l1_l2

electrical.current.l1

electrical.active_power.total

electrical.energy.imported

pressure.discharge

machine.running
```

Esse cenário deve ser reutilizado em:

- testes;
- homologação;
- documentação;
- demonstrações;
- desenvolvimento frontend.

---

# 10. Equipamento inicial

O primeiro equipamento real ou simulado deverá possuir protocolo simples e previsível.

Recomendação:

```text
Modbus TCP
```

ou:

```text
Modbus RTU via gateway
```

Evitar começar simultaneamente com:

```text
Modbus
OPC UA
BACnet
OCPP
SNMP
```

---

# 11. Epic 01 — Repository Foundation

Objetivo:

criar fundação técnica do projeto.

Entregas:

```text
Monorepo

pnpm

Turborepo

TypeScript

lint

format

testing

environment configuration

Docker Compose

CI baseline
```

Estrutura:

```text
apps/
packages/
adapters/
infrastructure/
docs/
tests/
```

---

# 12. Critério de aceite Epic 01

Deve ser possível executar:

```text
pnpm install
pnpm lint
pnpm test
pnpm build
```

sem erros.

Docker Compose deve iniciar infraestrutura mínima.

---

# 13. Epic 02 — Core Infrastructure

Serviços:

```text
PostgreSQL + TimescaleDB

Redis

EMQX

Object Storage opcional

Backend API

Frontend
```

Entregas:

```text
Docker Compose local

health checks

environment separation

migration system

logging

OpenTelemetry baseline
```

---

# 14. Critério de aceite Epic 02

Ambiente novo deve poder ser iniciado a partir do repositório documentado.

Nenhuma configuração manual oculta será aceita.

---

# 15. Epic 03 — Identity

Implementar:

```text
User

Login

Logout

Sessions

Password hashing

Membership

Current User
```

Segurança:

```text
Argon2id

secure cookies/tokens

rate limiting

audit
```

---

# 16. Critério de aceite Identity

Usuário deve conseguir:

```text
login
logout
retrieve own identity
```

Usuário inválido:

```text
cannot authenticate
```

Sessão revogada:

```text
cannot continue
```

---

# 17. Epic 04 — Multi-Tenancy

Implementar:

```text
Organization

Membership

Organization context

RBAC

RLS
```

---

# 18. Teste crítico de Multi-Tenancy

Criar:

```text
Organization A

Organization B
```

Usuário de A não pode acessar:

```text
site
asset
device
telemetry
alarm
```

de B.

Mesmo conhecendo UUID.

---

# 19. Critério obrigatório

O teste cross-tenant deve existir automaticamente no CI.

Falha nesse teste:

```text
blocks merge
```

---

# 20. Epic 05 — Operational Hierarchy

Implementar:

```text
Site
Area
System
AssetType
Asset
```

Relacionamentos:

```text
Organization
↓
Site
↓
Area
↓
System
↓
Asset
```

---

# 21. Asset Tree

Implementar consulta:

```text
GET /sites/{id}/asset-tree
```

Frontend deve conseguir navegar na hierarquia.

---

# 22. Epic 06 — Device Registry

Implementar:

```text
Gateway

Device Model

Device

Metric Definition

Device Metric Mapping
```

---

# 23. Device Registration

Device deve poder ser associado a:

```text
Organization

Site

Gateway

Asset

Device Model
```

---

# 24. Epic 07 — Edge Provisioning

Implementar:

```text
Gateway registration

Activation token

Provisioning API

Gateway credential

Configuration download

Heartbeat
```

---

# 25. Fluxo obrigatório

```text
Admin creates Gateway
↓
Activation Token
↓
Edge starts
↓
Provisioning API
↓
Credential obtained
↓
MQTT connection
↓
Configuration sync
↓
Heartbeat
```

---

# 26. MVP de certificados

Preferência:

```text
X.509
```

Caso PKI atrase o desenvolvimento inicial, poderá ser utilizado temporariamente:

```text
gateway_id
+
individual secret
```

desde que:

- TLS permaneça obrigatório;
- segredo seja individual;
- exista rotação;
- arquitetura não impeça migração para X.509.

---

# 27. Epic 08 — Edge Runtime

Implementar módulos:

```text
Configuration

Polling

Protocol Adapter

Device Adapter

Normalization

Outbox

MQTT

Heartbeat

Diagnostics
```

---

# 28. Edge Startup

Obrigatório:

```text
load config
↓
initialize SQLite
↓
load adapters
↓
connect cloud
↓
sync configuration
↓
start polling
```

---

# 29. Edge Offline

Se Cloud estiver indisponível:

```text
Edge continues reading devices
```

e armazena:

```text
local queue
```

---

# 30. Epic 09 — Protocol Adapter

Primeiro adapter:

```text
Modbus
```

Selecionar inicialmente:

```text
Modbus TCP
```

como prioridade.

Modbus RTU pode entrar ainda no MVP se implementação não aumentar significativamente o risco.

---

# 31. Protocol Adapter Interface

Deve implementar:

```text
connect()

disconnect()

read()

write()

health()
```

---

# 32. Epic 10 — Device Adapter

Criar interface independente do protocolo.

Responsabilidades:

```text
raw data
↓
scaling
↓
unit conversion
↓
semantic metric
```

---

# 33. Primeiro Device Adapter

Criar pelo menos:

```text
1 Device Adapter real
```

e:

```text
1 Simulator Adapter
```

Simulator deve ser utilizado no CI e homologação.

---

# 34. Epic 11 — Metric Catalog

Implementar:

```text
MetricDefinition
```

com:

```text
key
quantity
canonical_unit
datatype
aggregation_type
```

---

# 35. Catálogo inicial

Incluir um conjunto reduzido.

Elétricas:

```text
electrical.voltage.line_l1_l2
electrical.voltage.line_l2_l3
electrical.voltage.line_l3_l1

electrical.current.l1
electrical.current.l2
electrical.current.l3

electrical.active_power.total
electrical.energy.imported
electrical.frequency
```

Processo:

```text
pressure.discharge

temperature.motor

machine.running
```

---

# 36. Epic 12 — Telemetry Ingestion

Implementar:

```text
MQTT ingestion

HTTP ingestion

schema validation

tenant validation

device validation

deduplication

raw persistence

normalized persistence
```

---

# 37. Pipeline obrigatório

```text
Receive
↓
Authenticate
↓
Authorize
↓
Validate Schema
↓
Validate Device
↓
Validate Tenant
↓
Deduplicate
↓
Store RAW
↓
Store Telemetry
↓
Publish Internal Event
```

---

# 38. Epic 13 — TimescaleDB

Criar:

```text
Telemetry hypertable
```

Índices:

```text
device_id
metric_id
observed_at
organization_id
```

Estratégia exata deve ser validada por benchmark.

---

# 39. Continuous Aggregates

Podem ser implementados no MVP para:

```text
1 minute

5 minutes

1 hour
```

desde que não atrasem ingestão básica.

---

# 40. Epic 14 — Historical API

Implementar consulta por:

```text
device
metric
time range
resolution
aggregation
```

---

# 41. Critério de histórico

Histórico deve permanecer correto após:

```text
internet outage

late telemetry

retransmission
```

usando:

```text
observed_at
```

e não apenas `received_at`.

---

# 42. Epic 15 — Current State

Criar armazenamento ou consulta eficiente do último valor conhecido.

API:

```text
GET /devices/{id}/telemetry/current
```

---

# 43. Current State deve incluir

```text
value

unit

quality

observed_at
```

---

# 44. Epic 16 — Connectivity

Implementar:

```text
heartbeat

last_seen

expected communication interval

derived connectivity state
```

Estados:

```text
ONLINE
DEGRADED
OFFLINE
UNKNOWN
DISABLED
```

---

# 45. Epic 17 — Store & Forward

Edge SQLite deve suportar:

```text
persistent outbox

retry

restart recovery

deduplication

dead letter
```

---

# 46. Teste obrigatório Store & Forward

Cenário:

```text
Cloud online
↓
disconnect internet
↓
generate telemetry
↓
restart Edge
↓
generate more telemetry
↓
restore internet
↓
send backlog
```

Resultado:

```text
no data loss

no duplicate telemetry

correct observed_at
```

---

# 47. Epic 18 — Events

Implementar:

```text
Event entity

Event API

Device events

Connectivity events
```

Tipos iniciais:

```text
DEVICE_ONLINE

DEVICE_OFFLINE

DEVICE_DEGRADED

GATEWAY_ONLINE

GATEWAY_OFFLINE

DEVICE_FAULT
```

---

# 48. Epic 19 — Basic Alarm Engine

Implementar regras simples:

```text
>

>=

<

<=

==

!=
```

Com:

```text
threshold

duration

severity
```

---

# 49. Alarm lifecycle

```text
ACTIVE
↓
ACKNOWLEDGED
↓
CLEARED
```

ACK não elimina a condição.

---

# 50. Alarm MVP

Exemplo:

```text
pressure.discharge > 10 bar

duration = 5 seconds

severity = WARNING
```

---

# 51. Epic 20 — Basic Dashboard

Dashboard inicial deve ser deliberadamente simples.

Mostrar:

```text
Asset identity

Device status

Current values

Trend chart

Active alarms

Recent events
```

---

# 52. Não criar Dashboard Builder

No MVP:

```text
fixed templates
```

ou templates configurados no código.

Editor drag-and-drop:

```text
out of scope
```

---

# 53. Dashboard Realtime

Preferência:

```text
WebSocket
```

Se isso atrasar significativamente o MVP, polling curto poderá ser usado temporariamente.

Porém WebSocket permanece P1 prioritário.

---

# 54. Epic 21 — Audit

Implementar:

```text
login

membership changes

device creation

device changes

gateway provisioning

alarm acknowledgement

command activity

security events
```

---

# 55. Audit Interface

Usuário autorizado deve conseguir consultar:

```text
who
what
when
resource
```

---

# 56. Epic 22 — Observability

Implementar desde o MVP:

```text
structured logs

metrics

correlation IDs

health endpoints
```

---

# 57. Métricas mínimas

```text
API latency

HTTP errors

MQTT ingestion rate

telemetry processed

telemetry rejected

duplicate messages

queue depth

device online count

device offline count

database health
```

---

# 58. Epic 23 — Command Infrastructure

Implementar P1:

```text
CommandDefinition

Command entity

Command status

Authorization

MQTT publishing

Command result

Audit
```

---

# 59. Primeiro comando seguro

Antes de permitir controle de máquina, testar com:

```text
READ_CONFIGURATION
```

ou:

```text
PING
```

Depois:

```text
RESET
```

ou outro comando não crítico.

---

# 60. Não iniciar com START/STOP

Comandos que podem movimentar máquinas devem ser liberados somente depois da validação integral de:

```text
RBAC

expiration

replay protection

local safety

audit

operator confirmation
```

---

# 61. Milestone M0 — Foundation

Entregas:

```text
repository

monorepo

Docker

CI

PostgreSQL

TimescaleDB

Redis

EMQX

API skeleton

Frontend skeleton

Edge skeleton
```

Resultado:

```text
entire development environment starts
```

---

# 62. Milestone M1 — Core Domain

Entregas:

```text
Identity

Organizations

Memberships

RLS

Sites

Areas

Systems

Assets

Gateways

Devices

Metrics
```

Resultado:

a plataforma consegue representar o ambiente industrial.

---

# 63. Milestone M2 — Edge Connectivity

Entregas:

```text
Provisioning

Edge configuration

Heartbeat

MQTT authentication

Protocol adapter

Device adapter

Simulator
```

Resultado:

Edge aparece online na plataforma.

---

# 64. Milestone M3 — Telemetry

Entregas:

```text
Polling

Normalization

MQTT ingestion

HTTP ingestion

RAW data

TimescaleDB

Current state

Historical API
```

Resultado:

dados de campo aparecem no backend.

---

# 65. Milestone M4 — Resilience

Entregas:

```text
Store & Forward

Retry

Deduplication

Late data

Dead letter

Offline restart
```

Resultado:

perda temporária de internet não causa perda de dados.

---

# 66. Milestone M5 — Operations

Entregas:

```text
Device connectivity

Events

Alarm Engine

Alarm API

Basic Dashboard

Realtime
```

Resultado:

usuário consegue monitorar um ativo real.

---

# 67. Milestone M6 — Security Validation

Entregas:

```text
tenant isolation tests

RLS tests

MQTT ACL tests

rate limiting

audit review

secret scan

security baseline
```

Resultado:

MVP pode entrar em homologação externa.

---

# 68. Milestone M7 — Command Foundation

Entregas:

```text
CommandDefinition

Command API

MQTT command

Command Result

Expiration

Deduplication

Audit
```

Resultado:

infraestrutura bidirecional validada.

---

# 69. Milestone M8 — Release Candidate

Entregas:

```text
bug fixing

documentation

deployment

backup

restore test

monitoring

smoke tests

performance tests

release notes
```

Resultado:

```text
MVP v1.0 Release Candidate
```

---

# 70. Sequência recomendada

```text
M0 Foundation
      ↓
M1 Core Domain
      ↓
M2 Edge Connectivity
      ↓
M3 Telemetry
      ↓
M4 Resilience
      ↓
M5 Operations
      ↓
M6 Security
      ↓
M7 Commands
      ↓
M8 Release
```

M6 não significa que segurança só começa nessa etapa.

Segurança deve existir desde M0.

M6 representa validação consolidada.

---

# 71. Workstreams

O trabalho poderá ser dividido em:

```text
Platform Backend

Frontend

Edge

Infrastructure

Quality / Testing

Security
```

---

# 72. Backend Workstream

Responsável por:

```text
Domain

API

Database

RLS

Telemetry

Events

Alarms

Commands

Audit
```

---

# 73. Frontend Workstream

Responsável por:

```text
Authentication

Navigation

Asset hierarchy

Device management

Telemetry visualization

Alarm interface

Operational dashboard
```

---

# 74. Edge Workstream

Responsável por:

```text
Provisioning

Adapters

Polling

Normalization

SQLite

Outbox

MQTT

Configuration

Commands

Diagnostics
```

---

# 75. Infrastructure Workstream

Responsável por:

```text
Docker

CI/CD

database

broker

cache

monitoring

backups

environments
```

---

# 76. Quality Workstream

Responsável por:

```text
unit tests

integration tests

contract tests

e2e

performance tests

security tests
```

Não significa que QA seja o único responsável por testes.

Cada desenvolvedor é responsável pelos testes de sua implementação.

---

# 77. Dependency Map

```text
Repository Foundation
        ↓
Infrastructure
        ↓
Domain
        ↓
Device Registry
        ↓
Edge Provisioning
        ↓
Adapters
        ↓
Telemetry
        ↓
History
        ↓
Connectivity
        ↓
Events
        ↓
Alarms
        ↓
Dashboard
```

---

# 78. Desenvolvimento paralelo

Após contratos definidos:

Backend pode desenvolver:

```text
API
```

enquanto Edge desenvolve:

```text
TelemetryEnvelope
```

e Frontend desenvolve:

```text
UI usando mocked contracts
```

Isso só é seguro porque:

```text
packages/contracts
```

será compartilhado.

---

# 79. Branch Strategy

Recomendação:

```text
main

develop

feature/*
fix/*
```

---

# 80. Main

Representa:

```text
production-ready
```

Não permitir push direto.

---

# 81. Develop

Representa:

```text
integration
```

Features entram através de pull request.

---

# 82. Feature Branch

Exemplo:

```text
feature/telemetry-ingestion

feature/edge-provisioning

feature/alarm-engine
```

Evitar branches gigantes correspondentes ao MVP inteiro.

---

# 83. Pull Request

Todo PR deve conter:

```text
objective

changes

tests

migration impact

security impact

screenshots when UI

contract impact
```

---

# 84. PR size

Preferir PRs pequenos e revisáveis.

Um PR que altera simultaneamente:

```text
database

API

frontend

Edge

infrastructure
```

sem necessidade deve ser dividido.

---

# 85. Code Review

Obrigatório para:

```text
develop

main
```

Revisão deve verificar:

```text
architecture

security

tests

naming

tenant handling

error handling
```

---

# 86. Architecture Decision Record

Criar ADR quando decisão afetar:

```text
architecture

database model

protocol

security

contract

major dependency

deployment
```

---

# 87. Estrutura ADR

```text
ADR-0001-title.md

Status
Context
Decision
Consequences
Alternatives
```

---

# 88. Definition of Ready

Uma tarefa está pronta para desenvolvimento quando possui:

```text
objective

acceptance criteria

dependencies

contract

security implications

test scenario
```

quando aplicável.

---

# 89. Definition of Done

Uma tarefa somente é considerada concluída quando:

1. código implementado;
2. lint aprovado;
3. build aprovado;
4. testes unitários aprovados;
5. testes de integração quando necessários;
6. contratos atualizados;
7. migration criada quando necessária;
8. documentação atualizada;
9. logs adequados;
10. erros tratados;
11. segurança analisada;
12. tenant isolation preservado;
13. PR revisado;
14. CI aprovado.

---

# 90. DoD adicional para API

Endpoint deve possuir:

```text
authentication

authorization

validation

error handling

OpenAPI documentation

tests

correlation ID
```

---

# 91. DoD adicional para entidade tenant

Deve possuir:

```text
organization_id

RLS policy

cross-tenant test
```

---

# 92. DoD adicional para Edge

Implementação deve testar:

```text
restart

offline mode

invalid config

network failure

retry
```

---

# 93. DoD adicional para Adapter

Adapter deve possuir:

```text
fixtures

unit tests

conversion tests

invalid response tests

health handling
```

---

# 94. Test Pyramid

Prioridade:

```text
many unit tests

sufficient integration tests

targeted E2E tests
```

Evitar depender apenas de testes end-to-end.

---

# 95. Unit Tests

Obrigatórios principalmente para:

```text
domain rules

normalization

unit conversion

alarm evaluation

command validation

authorization helpers
```

---

# 96. Integration Tests

Obrigatórios para:

```text
PostgreSQL

RLS

TimescaleDB

Redis

EMQX

API repositories
```

---

# 97. Contract Tests

Obrigatórios entre:

```text
Cloud ↔ Edge

Backend ↔ Frontend

MQTT producer ↔ consumer
```

---

# 98. E2E Flow

Teste principal:

```text
create organization
↓
create site
↓
create asset
↓
create gateway
↓
provision Edge
↓
register device
↓
generate telemetry
↓
ingest
↓
query current data
↓
query history
↓
trigger alarm
↓
acknowledge alarm
```

---

# 99. Failure E2E

Outro teste obrigatório:

```text
disconnect internet
↓
generate telemetry
↓
reconnect
↓
verify backlog
↓
verify no duplicates
```

---

# 100. Security Test

Fluxo:

```text
Organization A user
↓
request Organization B resource
↓
deny
```

Testar em:

```text
API
repository
RLS
```

---

# 101. Performance Baseline

O MVP não precisa provar escala global.

Mas deve possuir baseline mensurável.

Testar pelo menos:

```text
100 devices

1 metric/sec per device
```

ou carga equivalente.

---

# 102. Carga de referência inicial

Exemplo:

```text
100 devices

10 metrics/device

10-second publishing
```

Resultado:

```text
100 metrics/second
```

Esse número é ponto inicial de teste, não limite arquitetural.

---

# 103. Performance Metrics

Medir:

```text
ingestion latency

API latency

DB write latency

historical query latency

broker latency

memory

CPU
```

---

# 104. Performance Criteria

Metas finais devem ser estabelecidas após benchmark inicial.

Não inventar SLAs sem dados.

Registrar baseline de cada release.

---

# 105. Data Integrity Criteria

Nenhuma perda de telemetria deve ocorrer em:

```text
normal temporary network outage
```

dentro da capacidade do buffer configurado.

---

# 106. Duplicate Criteria

Retransmissão deve resultar em:

```text
exactly one logical telemetry record
```

para o mesmo:

```text
message_id
```

---

# 107. Clock Criteria

Medições devem preservar:

```text
observed_at
```

mesmo quando recebidas posteriormente.

---

# 108. Quality Gate CI

Pipeline mínimo:

```text
install
↓
lint
↓
type check
↓
unit tests
↓
integration tests
↓
contract tests
↓
build
↓
security scans
```

---

# 109. Security Gate

Bloquear merge em caso de:

```text
known critical vulnerability

secret detected

tenant isolation failure

RLS failure
```

---

# 110. Database Migration Rules

Migration:

```text
immutable after merge
```

Nunca editar migration já executada em ambiente compartilhado.

Criar nova migration.

---

# 111. Migration Testing

CI deve:

```text
create empty DB
↓
apply all migrations
↓
run tests
```

---

# 112. Rollback

Nem toda migration precisa possuir rollback automático.

Porém mudanças destrutivas exigem plano explícito.

---

# 113. Seed Data

Manter seeds para:

```text
development

testing
```

Nunca misturar dados de teste com produção.

---

# 114. Demo Seed

Criar cenário padrão:

```text
Demo Organization

Demo Site

Demo Asset

Demo Devices
```

---

# 115. Environment Strategy

Ambientes:

```text
local

development

staging

production
```

---

# 116. Local

Objetivo:

```text
developer productivity
```

Executado via:

```text
Docker Compose
```

---

# 117. Development

Objetivo:

```text
continuous integration testing
```

Pode receber atualizações frequentes.

---

# 118. Staging

Deve reproduzir produção o máximo possível.

Objetivo:

```text
release validation
```

---

# 119. Production

Somente releases aprovados.

Deploy rastreável por:

```text
Git commit

version

container image
```

---

# 120. Versioning

Plataforma:

```text
Semantic Versioning
```

Exemplo:

```text
1.0.0
```

Durante desenvolvimento:

```text
0.x
```

---

# 121. Edge Versioning

Edge possui versão independente.

Exemplo:

```text
edge 0.3.0
```

Cloud deve registrar versão conectada.

---

# 122. Adapter Versioning

Adapter também possui versão.

Exemplo:

```text
weg.mmw04@1.1.0
```

---

# 123. Release Candidate

Exemplo:

```text
1.0.0-rc.1
```

---

# 124. Deployment

Produção inicial poderá utilizar:

```text
Docker
```

sem Kubernetes.

Infraestrutura mínima:

```text
reverse proxy

API

frontend

worker

PostgreSQL / TimescaleDB

Redis

EMQX

monitoring
```

---

# 125. Backup Gate

Antes de produção:

```text
backup executed
+
restore successfully tested
```

obrigatório.

---

# 126. Monitoring Gate

Antes da produção devem existir:

```text
application health

database health

broker health

ingestion metrics

error metrics

disk usage
```

---

# 127. Logging Gate

Deve ser possível investigar:

```text
specific message_id

specific device

specific API request

specific command
```

---

# 128. Security Gate Production

Obrigatório:

```text
HTTPS

MQTTS

RBAC

RLS

individual Edge credential

audit

rate limits

secret separation

no default password
```

---

# 129. Documentation Gate

Antes da primeira produção devem existir:

```text
Architecture Blueprint

Domain Model

Telemetry Specification

Security Architecture

API & Edge Contracts

MVP Scope

Deployment Guide

Developer Manual

Operations Runbook
```

---

# 130. Acceptance Scenario A — Normal Operation

```text
Device online
↓
Edge reads
↓
Telemetry reaches Cloud
↓
Dashboard updates
```

Aceite:

```text
correct metric
correct unit
correct timestamp
```

---

# 131. Acceptance Scenario B — Internet Failure

```text
Cloud unreachable
```

Aceite:

```text
Edge continues acquisition

queue grows

Cloud connection restored

queue drains

timestamps preserved
```

---

# 132. Acceptance Scenario C — Device Offline

```text
Device stops answering
```

Aceite:

```text
device status changes

event generated

dashboard reflects condition
```

---

# 133. Acceptance Scenario D — Alarm

```text
metric crosses threshold
```

Aceite:

```text
duration respected

alarm becomes ACTIVE

user acknowledges

condition normalizes

alarm becomes CLEARED
```

---

# 134. Acceptance Scenario E — Tenant Isolation

```text
User A
attempts access to B
```

Aceite:

```text
no data leakage
```

---

# 135. Acceptance Scenario F — Edge Restart

```text
Edge restarted
```

Aceite:

```text
configuration loaded

SQLite preserved

pending queue preserved

operation resumes
```

---

# 136. Acceptance Scenario G — Duplicate Message

Mesma:

```text
message_id
```

enviada duas vezes.

Aceite:

```text
one logical record
```

---

# 137. Acceptance Scenario H — Invalid Telemetry

Métrica inexistente.

Aceite:

```text
rejected/quarantined

logged

no automatic metric creation
```

---

# 138. Acceptance Scenario I — Security

Credential revoked.

Aceite:

```text
new connection rejected
```

---

# 139. Acceptance Scenario J — Audit

Usuário reconhece alarme.

Aceite:

audit contém:

```text
actor

action

alarm

timestamp

organization
```

---

# 140. MVP Release Criteria

Release somente poderá ocorrer se:

```text
all P0 epics completed

all critical tests pass

no critical security issue

tenant isolation validated

Store & Forward validated

deployment validated

backup restore validated

documentation updated
```

---

# 141. Bug Severity

## Critical

```text
security breach
data loss
cross-tenant leakage
platform unavailable
dangerous command behavior
```

Bloqueia release.

---

# 142. High

```text
major functionality unavailable
incorrect telemetry
alarm failure
provisioning failure
```

Normalmente bloqueia release.

---

# 143. Medium

```text
workaround exists
limited functional issue
```

Pode ser avaliado.

---

# 144. Low

```text
cosmetic
minor UX
non-critical improvement
```

Não bloqueia necessariamente.

---

# 145. Technical Debt

Toda dívida técnica aceita deve possuir:

```text
issue

reason

impact

priority
```

Não utilizar:

```text
TODO
```

como único registro.

---

# 146. Scope Change

Mudança de escopo durante MVP deve responder:

```text
Is it necessary for P0?

Does it block acceptance?

Does it fix architecture/security?
```

Caso contrário:

```text
post-MVP backlog
```

---

# 147. Regra contra scope creep

Durante MVP:

> Nova funcionalidade não entra apenas porque é útil.

Ela entra apenas se for necessária para provar a arquitetura, operar o cenário de referência ou cumprir segurança mínima.

---

# 148. Backlog pós-MVP

Primeiros candidatos:

```text
Commands operational release

Notifications

Dashboard Templates

Reporting

Device Twin

Firmware management

OTA

Webhooks

OPC UA adapter

BACnet adapter

OCPP adapter

Advanced permissions

SSO

Advanced analytics
```

---

# 149. Release 1.1 sugerida

Após MVP:

```text
Commands

Notifications

Dashboard templates

basic reporting
```

---

# 150. Release 1.2 sugerida

```text
Device Twin

remote configuration

diagnostics

firmware management foundation
```

---

# 151. Release 1.3 sugerida

```text
OTA

additional protocols

external integrations
```

---

# 152. Release 2.x

Possíveis áreas:

```text
Analytics

anomaly detection

predictive maintenance

fleet intelligence

advanced reporting

AI
```

---

# 153. Ordem recomendada das primeiras issues

```text
001 initialize monorepo

002 configure lint/typecheck

003 docker compose infrastructure

004 database connection

005 migrations foundation

006 organization entity

007 user/session

008 membership

009 RLS

010 site

011 area

012 system

013 asset type

014 asset

015 gateway

016 device model

017 device

018 metric definition

019 metric mapping

020 edge skeleton

021 gateway provisioning

022 MQTT authentication

023 heartbeat

024 Modbus adapter

025 device adapter interface

026 simulator adapter

027 polling engine

028 SQLite outbox

029 telemetry contract

030 MQTT ingestion

031 HTTP ingestion

032 raw messages

033 Timescale telemetry

034 current state

035 historical API

036 connectivity engine

037 Store & Forward tests

038 events

039 alarm rules

040 alarms

041 dashboard

042 audit interface

043 security validation

044 commands foundation

045 release hardening
```

---

# 154. Parallelization suggestion

Backend team:

```text
001–019
029–040
042–044
```

Edge team:

```text
020–028
037
044
```

Frontend team:

```text
auth UI
asset navigation
devices
dashboard
alarms
```

Infrastructure:

```text
003
CI/CD
observability
deploy
backup
```

---

# 155. Team Rule

Nenhuma equipe deve criar contratos paralelos fora de:

```text
packages/contracts
```

Caso seja necessário alterar contrato:

```text
update shared contract first
```

---

# 156. Completion Definition

O MVP estará funcionalmente concluído quando:

```text
one real industrial system
```

estiver conectado de ponta a ponta e os critérios de aceite forem atendidos.

Não será considerado concluído apenas porque:

```text
screens exist
```

ou:

```text
API endpoints exist
```

---

# 157. Product Validation

Antes da produção ampla, executar piloto controlado com:

```text
1 Organization

1 Site

1 Gateway

1–5 Devices
```

---

# 158. Pilot Objective

Validar:

```text
connectivity

stability

data quality

usability

alarm behavior

network outages

Edge recovery

operational support
```

---

# 159. Pilot Duration

Não definir duração arbitrária neste documento.

A saída do piloto dependerá de:

```text
stability criteria
```

e não apenas de calendário.

---

# 160. Exit Criteria do piloto

```text
no critical data loss

no tenant issue

no critical security issue

stable Edge

stable ingestion

correct alarms

acceptable dashboard operation
```

---

# 161. MVP Architecture Freeze

Após aprovação deste documento:

Mudanças em:

```text
domain model

telemetry envelope

tenant model

security baseline

Edge contract
```

devem passar por ADR.

---

# 162. Não significa congelamento absoluto

Correções arquiteturais continuam possíveis.

Porém devem ser deliberadas e documentadas.

---

# 163. Artefatos oficiais

Antes do desenvolvimento pleno, a equipe deve considerar como fontes oficiais:

```text
Architecture Blueprint v1.0

Domain Model v1.0

Telemetry & Messaging Specification v1.0

Security Architecture v1.0

API & Edge Contracts v1.0

MVP Scope & Delivery Plan v1.0
```

---

# 164. Próximo documento

Com o escopo do MVP definido, o próximo documento deve ser:

```text
Development Manual v1.0
```

Esse manual consolidará as decisões anteriores em instruções práticas para o time.

---

# 165. Conteúdo do Development Manual

O manual deverá incluir:

```text
Project purpose

Architecture overview

Technology stack

Repository setup

Local environment

Repository structure

Coding standards

Domain rules

Database rules

API rules

Edge rules

Telemetry rules

Security rules

Git workflow

Pull requests

Testing

CI/CD

Logging

Observability

Migrations

Documentation

Definition of Ready

Definition of Done

Release process

Prohibited practices
```

---

# 166. Decisões consolidadas

O MVP Scope & Delivery Plan v1.0 estabelece:

1. O MVP comprovará fluxo industrial completo de ponta a ponta.
2. Multi-tenancy é P0.
3. RLS é P0.
4. Edge é parte do MVP, não componente posterior.
5. Modbus será o protocolo industrial inicial.
6. MQTT será o canal principal Edge → Cloud.
7. HTTP ingestion será suportado.
8. Store & Forward é P0.
9. Telemetria RAW + normalizada é P0.
10. catálogo semântico de métricas é P0.
11. device connectivity é P0.
12. eventos são P0.
13. alarmes básicos são P0.
14. dashboard operacional básico é P0.
15. auditoria é P0.
16. commands entram inicialmente como infraestrutura P1.
17. OTA fica fora.
18. Digital Twin completo fica fora.
19. analytics avançado fica fora.
20. microservices ficam fora.
21. Kubernetes fica fora.
22. CI e testes fazem parte do desenvolvimento desde o início.
23. isolamento de tenant deve ser testado no CI.
24. ambiente Edge offline deve ser testado.
25. contract tests são obrigatórios.
26. releases devem ser rastreáveis.
27. backup e restore devem ser testados antes de produção.
28. mudanças centrais pós-freeze exigem ADR.
29. piloto controlado antecede expansão.
30. scope creep deve ser explicitamente rejeitado.

---

# 167. Critério final

O primeiro release não deve tentar ser uma plataforma IoT completa.

Ele deve ser:

> uma fundação industrial confiável, segura e extensível, capaz de conectar um ativo real, transportar seus dados de forma resiliente, armazená-los corretamente, transformá-los em informação operacional e manter isolamento completo entre clientes.

Tudo que não contribua diretamente para esse objetivo deve permanecer fora do MVP.

---

**Documento:** MVP Scope & Delivery Plan  
**Versão:** 1.0  
**Status:** Baseline inicial / Scope Freeze Candidate  
**Dependências:** Architecture Blueprint v1.0, Domain Model v1.0, Telemetry & Messaging Specification v1.0, Security Architecture v1.0, API & Edge Contracts v1.0  
**Próximo documento:** Development Manual v1.0