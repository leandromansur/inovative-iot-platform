# Development Manual v1.0

## Plataforma Industrial IoT

**Versão:** 1.0  
**Status:** Baseline oficial para desenvolvimento  
**Objetivo:** manual operacional da equipe de desenvolvimento  
**Escopo:** MVP v1  
**Arquitetura:** Edge + Cloud, multitenant, multimarcas e multiprotocolo

---

# 1. Finalidade deste documento

Este manual é a principal referência operacional para o desenvolvimento da plataforma Industrial IoT.

A equipe não precisa consultar individualmente os documentos de arquitetura para iniciar tarefas rotineiras.

Este manual consolida as decisões definidas em:

- Architecture Blueprint v1.0;
- Domain Model v1.0;
- Telemetry & Messaging Specification v1.0;
- Security Architecture v1.0;
- API & Edge Contracts v1.0;
- MVP Scope & Delivery Plan v1.0.

Os documentos especializados continuam sendo referências arquiteturais detalhadas.

Quando existir divergência entre implementação e arquitetura aprovada, a implementação deve ser revisada.

Alterações estruturais devem ser documentadas através de ADR.

---

# 2. Objetivo da plataforma

Construir uma plataforma Industrial IoT capaz de conectar ativos e equipamentos de diferentes fabricantes, coletar e normalizar seus dados, armazenar históricos, gerar eventos e alarmes e apresentar essas informações em aplicações operacionais.

A plataforma deverá suportar:

- múltiplos clientes;
- múltiplas instalações;
- múltiplos fabricantes;
- gateways Edge;
- dispositivos conectados diretamente;
- MQTT;
- HTTP/REST;
- protocolos industriais;
- operação offline no Edge;
- telemetria;
- histórico;
- alarmes;
- eventos;
- dashboards;
- comandos futuros;
- integrações externas;
- analytics futuros.

---

# 3. Objetivo do MVP

O MVP deverá comprovar o seguinte fluxo completo:

```text
EQUIPAMENTO INDUSTRIAL
        │
        ▼
PROTOCOL ADAPTER
        │
        ▼
DEVICE ADAPTER
        │
        ▼
EDGE
        │
        ▼
STORE & FORWARD
        │
        ▼
MQTT / HTTPS
        │
        ▼
IoT CORE
        │
        ▼
TELEMETRY
        │
        ▼
TIMESCALEDB
        │
        ├──► ALARM ENGINE
        │
        ├──► EVENTS
        │
        └──► API
               │
               ▼
           DASHBOARD
```

Ao final do MVP deverá ser possível conectar pelo menos um sistema industrial real de ponta a ponta.

---

# 4. Princípios arquiteturais

Todas as implementações devem respeitar os seguintes princípios.

## 4.1 Vendor-neutral

O Core não pode depender de fabricantes específicos.

Correto:

```text
WEG
Schneider
Siemens
ABB
Sensor genérico
        │
        ▼
Adapter
        │
        ▼
Modelo semântico comum
```

Incorreto:

```text
Core
└── lógica específica WEG
```

---

# 5. Asset não é Device

Essa separação é obrigatória.

**Asset** representa aquilo que possui significado operacional para o cliente.

Exemplo:

```text
Compressor 01
```

**Device** representa algo conectado à infraestrutura IoT.

Exemplo:

```text
Compressor 01
├── Inversor
├── Medidor
├── Sensor de pressão
└── Sensor de vibração
```

Portanto:

```text
Asset != Device
```

Nunca tratar essas entidades como sinônimos.

---

# 6. Edge-first

Falha da Internet não pode significar perda imediata de dados.

O Edge deve continuar:

- coletando;
- normalizando;
- registrando timestamps;
- armazenando dados localmente;
- executando funções locais permitidas.

Quando a comunicação retornar:

```text
Store
   ↓
Forward
```

---

# 7. API-first

A interface web é cliente da plataforma.

Ela não é o núcleo da plataforma.

Fluxo:

```text
Frontend
Mobile
Integration
External System
       │
       ▼
      API
       │
       ▼
     Core
```

Nenhuma regra crítica deve existir exclusivamente no frontend.

---

# 8. Security by design

Segurança faz parte da arquitetura.

Não será aceita a abordagem:

```text
primeiro funciona
depois protegemos
```

Toda funcionalidade deve considerar:

```text
Identity
Authorization
Tenant
Validation
Audit
```

---

# 9. Arquitetura de referência

```text
┌─────────────────────────────────────────────────────┐
│                 APPLICATION LAYER                   │
│                                                     │
│ Dashboard │ Reports │ Alarms │ Analytics │ API     │
└────────────────────────▲────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────┐
│                    IoT CORE                        │
│                                                     │
│ Identity │ Tenancy │ Assets │ Devices             │
│ Telemetry │ Events │ Alarms │ Commands │ Audit    │
└────────────────────────▲────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────┐
│                 INGESTION LAYER                    │
│                                                     │
│ MQTT │ HTTPS │ Validation │ Event Processing       │
└────────────────────────▲────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────┐
│                    EDGE                            │
│                                                     │
│ Protocol Adapters                                  │
│ Device Adapters                                    │
│ Polling                                            │
│ Normalization                                      │
│ Store & Forward                                    │
│ Commands                                           │
└────────────────────────▲────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────┐
│               CONNECTED PRODUCTS                   │
│                                                     │
│ Sensors │ Meters │ Drives │ PLC │ Machines         │
└─────────────────────────────────────────────────────┘
```

---

# 10. Estratégia arquitetural do backend

O backend será inicialmente um:

```text
MODULAR MONOLITH
```

Não criar microserviços sem ADR específico justificando a necessidade.

Módulos:

```text
identity
tenancy
sites
assets
devices
telemetry
events
alarms
commands
notifications
audit
integrations
```

Cada módulo deve possuir limites claros.

---

# 11. Arquitetura interna dos módulos

Estrutura recomendada:

```text
module/
├── domain/
├── application/
├── infrastructure/
└── presentation/
```

Exemplo:

```text
devices/
├── domain/
│   ├── device.entity.ts
│   └── device.repository.ts
│
├── application/
│   └── register-device.usecase.ts
│
├── infrastructure/
│   └── postgres-device.repository.ts
│
└── presentation/
    └── device.controller.ts
```

---

# 12. Stack oficial

## Backend

```text
TypeScript
Node.js
NestJS
```

## Frontend

```text
React
TypeScript
Vite
```

Principais bibliotecas:

```text
TanStack Query
React Router
Zod
ECharts
Tailwind CSS
```

## Banco

```text
PostgreSQL
TimescaleDB
```

## Broker

```text
EMQX
```

## Cache / processamento assíncrono

```text
Redis
```

## Edge

```text
TypeScript
Node.js
SQLite
```

A linguagem Edge pode futuramente mudar caso requisitos de hardware justifiquem.

## Observabilidade

```text
OpenTelemetry
Prometheus
Grafana
Loki
Tempo
```

## Infraestrutura

```text
Docker
Docker Compose
```

Não usar Kubernetes no MVP.

---

# 13. Estrutura inicial do repositório

```text
iot-platform/
│
├── apps/
│   ├── api/
│   ├── web/
│   ├── worker/
│   └── edge/
│
├── packages/
│   ├── domain/
│   ├── contracts/
│   ├── telemetry/
│   ├── device-sdk/
│   ├── protocol-sdk/
│   ├── security/
│   ├── config/
│   └── observability/
│
├── adapters/
│   ├── protocols/
│   │   ├── modbus/
│   │   ├── mqtt/
│   │   └── opcua/
│   │
│   └── devices/
│       ├── weg/
│       ├── schneider/
│       └── siemens/
│
├── infrastructure/
│   ├── docker/
│   ├── migrations/
│   ├── monitoring/
│   └── scripts/
│
├── docs/
│   ├── architecture/
│   ├── adr/
│   ├── api/
│   ├── protocols/
│   └── security/
│
├── tests/
│   ├── integration/
│   └── e2e/
│
├── docker-compose.yml
├── package.json
├── pnpm-workspace.yaml
└── README.md
```

---

# 14. Monorepo

Utilizar:

```text
pnpm
+
Turborepo
```

Objetivo:

compartilhar:

- schemas;
- DTOs;
- contratos;
- tipos;
- validações;
- SDKs.

---

# 15. Regra dos contratos

Nenhuma equipe deve criar versões próprias de um mesmo contrato.

Fonte oficial:

```text
packages/contracts
```

É proibido manter separadamente:

```text
Backend DTO
Frontend Interface
Edge Interface
```

representando a mesma mensagem.

---

# 16. Domínio principal

Hierarquia operacional:

```text
Organization
   │
   └── Site
        │
        └── Area
             │
             └── System
                  │
                  └── Asset
```

Conectividade:

```text
Site
├── Gateway
│    └── Device
│
└── Asset
     └── Device
```

---

# 17. Organization

`Organization` representa o tenant.

Todo dado pertencente a cliente deve possuir direta ou indiretamente:

```text
organization_id
```

Sempre que possível, incluir explicitamente esse campo.

Isso facilita:

- RLS;
- auditoria;
- consultas;
- troubleshooting.

---

# 18. User e Membership

Usuário é global.

```text
User
```

não pertence diretamente a uma única Organization.

A relação acontece por:

```text
Membership
```

Exemplo:

```text
User
├── Organization A → OWNER
└── Organization B → VIEWER
```

---

# 19. Roles iniciais

```text
PLATFORM_ADMIN

ORGANIZATION_OWNER
ORGANIZATION_ADMIN

ENGINEER
TECHNICIAN
OPERATOR
VIEWER
```

No código, não basear regras diretamente apenas nos nomes dos roles.

Utilizar permissions.

Exemplo:

```text
asset.read
asset.write

device.read
device.configure

telemetry.read

alarm.acknowledge

command.execute
command.execute.high
```

---

# 20. Site

Representa instalação.

Exemplos:

```text
Fábrica Rio
Hospital Centro
Posto GNV Barra
```

Campo obrigatório:

```text
timezone
```

Persistência:

```text
UTC
```

Apresentação:

```text
timezone do Site
```

---

# 21. Area

Subdivisão física ou funcional.

Exemplo:

```text
Site
└── Utilidades
    └── Casa de Máquinas
```

Pode possuir:

```text
parent_area_id
```

---

# 22. System

Sistema funcional.

Exemplos:

```text
Sistema de Ar Comprimido

HVAC

Sistema Elétrico

Sistema Fotovoltaico

Carregamento EV
```

---

# 23. Asset

Representa ativo operacional.

Exemplos:

```text
Compressor 01
Bomba 03
Chiller 02
Painel QGBT
Transformador 01
```

Assets podem possuir outros Assets:

```text
Compressor
├── Motor
├── Sistema de óleo
└── Resfriador
```

---

# 24. Asset Type

Chave funcional.

Exemplos:

```text
compressor
pump
motor
transformer
generator
ups
electrical_panel
ev_charger
hvac_unit
```

O Asset Type será usado futuramente para associar:

```text
dashboard template
alarm template
analytics
maintenance
```

---

# 25. Gateway

Gateway conecta dispositivos de campo à plataforma.

Pode ser:

- Industrial PC;
- PLC;
- embedded Linux;
- gateway dedicado.

Campos principais:

```text
organization_id
site_id
serial_number
software_version
status
last_seen_at
```

---

# 26. Device

Device representa uma entidade conectada.

Exemplos:

```text
Sensor
Meter
Drive
PLC
Controller
UPS
Protection Relay
```

Pode estar associado a:

```text
Asset
Gateway
DeviceModel
```

---

# 27. Device Model

Representa definição técnica conhecida.

Exemplo:

```text
manufacturer = WEG

family = MMW

model = MMW04

device_type = ENERGY_METER

adapter_key = weg.mmw04
```

---

# 28. Semantic Metric Model

Este é um dos elementos mais importantes da plataforma.

Equipamentos específicos não devem determinar nomes internos das grandezas.

Exemplo:

```text
WEG register
Schneider register
Siemens register
        │
        ▼
electrical.active_power.total
```

---

# 29. MetricDefinition

Campos:

```text
id

key

name

quantity

canonical_unit

datatype

aggregation_type

category

description
```

---

# 30. Metric Key

Padrão hierárquico:

```text
electrical.voltage.line_l1_l2

electrical.current.l1

electrical.active_power.total

electrical.energy.imported

pressure.oil

pressure.discharge

temperature.motor

vibration.rms

machine.running
```

Não usar:

```text
voltage1

V12

reg40001

tensao_mm04
```

como chave semântica global.

---

# 31. Unidade canônica

Cada Metric Definition possui unidade oficial.

Exemplo:

```text
Voltage → V

Current → A

Pressure → bar

Temperature → °C

Power → kW

Energy → kWh
```

Adapters devem converter unidades antes da entrada no modelo normalizado.

---

# 32. Aggregation Type

Toda métrica deve ser classificada como:

```text
GAUGE
COUNTER
STATE
```

Exemplo:

```text
voltage → GAUGE

pressure → GAUGE

energy → COUNTER

machine.running → STATE
```

---

# 33. Protocol Adapter

Responsabilidade:

```text
industrial protocol
        ↓
raw values
```

Exemplo:

```text
Modbus register
→ integer
```

Não deve conhecer:

- tenant;
- dashboard;
- alarmes;
- usuários.

---

# 34. Device Adapter

Responsabilidade:

```text
raw device data
        ↓
semantic metrics
```

Exemplo:

```text
register 40101 = 2217
scale 0.1
        ↓
electrical.voltage.line_l1_l2 = 221.7 V
```

---

# 35. Protocol Adapter Interface

Contrato conceitual:

```typescript
interface ProtocolAdapter {
  connect(config): Promise<void>;

  disconnect(): Promise<void>;

  read(request): Promise<RawReadResult>;

  write(request): Promise<RawWriteResult>;

  health(): Promise<AdapterHealth>;
}
```

---

# 36. Device Adapter Interface

Contrato conceitual:

```typescript
interface DeviceAdapter {
  normalize(
    raw: RawReadResult,
    context: DeviceContext
  ): NormalizedMetric[];

  encodeCommand(
    command: DeviceCommand,
    context: DeviceContext
  ): ProtocolWriteRequest[];

  getCapabilities(): DeviceCapabilities;
}
```

---

# 37. Regra crítica dos Adapters

Adapter não pode:

```text
persistir telemetria

publicar MQTT diretamente

enviar notificação

avaliar alarme

verificar usuário

acessar tenant
```

Ele apenas traduz.

---

# 38. Telemetry Envelope

Contrato canônico:

```json
{
  "schema_version": "1.0",
  "message_id": "uuid",
  "message_type": "telemetry",
  "organization_id": "uuid",
  "gateway_id": "uuid",
  "device_id": "uuid",
  "observed_at": "2026-10-06T13:30:00.000Z",
  "sequence": 12345,
  "payload": {
    "metrics": {
      "pressure.discharge": 7.3
    }
  }
}
```

---

# 39. message_id

Toda mensagem deve possuir:

```text
UUID
```

Utilizado para:

- deduplicação;
- rastreamento;
- idempotência;
- troubleshooting.

Retransmissão da mesma mensagem:

```text
MESMO message_id
```

---

# 40. observed_at e received_at

`observed_at`

momento real da medição.

`received_at`

momento em que Cloud recebeu.

Exemplo:

```text
observed_at = 10:00
received_at = 10:15
```

Histórico deve priorizar:

```text
observed_at
```

---

# 41. Raw Message

Payload original deve poder ser armazenado separadamente da telemetria normalizada.

Objetivos:

- auditoria;
- troubleshooting;
- reprocessamento;
- análise de adapters.

---

# 42. Telemetry Pipeline

Fluxo obrigatório:

```text
Receive
↓
Authenticate
↓
Authorize
↓
Schema Validation
↓
Device Validation
↓
Tenant Validation
↓
Deduplication
↓
RAW Storage
↓
Semantic Validation
↓
Telemetry Storage
↓
Internal Event
```

---

# 43. Semantic Validation

JSON válido não significa dado válido.

Exemplo:

```text
frequency = 50000 Hz
```

deve poder ser rejeitado ou marcado como inválido.

Validar:

- datatype;
- minimum;
- maximum;
- enum;
- unit;
- metric existence.

---

# 44. Unknown Metric

Produção não deve criar automaticamente métricas desconhecidas.

Uma métrica desconhecida deve ser:

```text
rejected
```

ou:

```text
quarantined
```

---

# 45. Unknown Device

Device desconhecido:

```text
reject
```

Autocadastro silencioso é proibido.

---

# 46. Metric Quality

Estados:

```text
GOOD
UNCERTAIN
BAD
STALE
UNKNOWN
```

---

# 47. Eventos

Event não é Telemetry.

Exemplo:

```text
DEVICE_FAULT
DEVICE_OFFLINE
HIGH_PRESSURE
```

não deve ser reduzido indiscriminadamente a:

```text
fault = 1
```

---

# 48. Event Severity

```text
INFO
NOTICE
WARNING
CRITICAL
```

---

# 49. Conectividade

Não utilizar:

```text
is_online: boolean
```

como modelo central.

Estados:

```text
ONLINE
DEGRADED
OFFLINE
UNKNOWN
DISABLED
```

Derivados de:

```text
heartbeat
telemetry
expected interval
last_seen
```

---

# 50. Estado operacional

Estado operacional é diferente da conectividade.

Exemplo:

```text
connectivity = ONLINE

operational_status = FAULT
```

Manter conceitos separados.

---

# 51. Heartbeat

Gateway deve enviar heartbeat.

Intervalo inicial recomendado:

```text
30 segundos
```

mas configurável.

Payload deve informar pelo menos:

```text
uptime
software version
queue depth
```

Opcional:

```text
CPU
memory
disk
```

---

# 52. MQTT

Broker oficial:

```text
EMQX
```

MQTT é o principal canal Edge ↔ Cloud.

---

# 53. MQTT Topics

Device:

```text
v1/{organization_id}/{device_id}/telemetry

v1/{organization_id}/{device_id}/event

v1/{organization_id}/{device_id}/status

v1/{organization_id}/{device_id}/command

v1/{organization_id}/{device_id}/command-result
```

Gateway:

```text
v1/{organization_id}/gateway/{gateway_id}/heartbeat

v1/{organization_id}/gateway/{gateway_id}/status

v1/{organization_id}/gateway/{gateway_id}/configuration
```

---

# 54. MQTT QoS

Padrão:

```text
Telemetry → QoS 1

Events → QoS 1

Commands → QoS 1

Command Result → QoS 1
```

Heartbeat poderá usar QoS 0 ou 1.

Não utilizar QoS 2 inicialmente.

---

# 55. MQTT Retain

```text
telemetry → false

event → false

command → false
```

Status poderá utilizar retain quando apropriado.

---

# 56. MQTT Last Will

Gateways devem possuir LWT.

Objetivo:

identificar desconexão inesperada.

---

# 57. HTTPS Ingestion

Endpoint:

```text
POST /api/v1/ingestion/telemetry
```

Deve utilizar o mesmo envelope conceitual do MQTT.

Não criar dois modelos diferentes de telemetria.

---

# 58. Store & Forward

Obrigatório no Edge.

Fluxo:

```text
Measurement
↓
SQLite Outbox
↓
MQTT Publish
↓
ACK
↓
Remove from Queue
```

---

# 59. Edge Outbox

Estrutura mínima:

```text
id

message_id

topic

payload

created_at

attempt_count

last_attempt_at

status
```

Status:

```text
PENDING
SENT
ACKNOWLEDGED
FAILED
```

---

# 60. Retry

Utilizar backoff.

Exemplo:

```text
1s
2s
5s
10s
30s
60s
```

Depois intervalo configurável.

---

# 61. Dead Letter

Mensagem permanentemente inválida não deve bloquear a fila.

Criar mecanismo:

```text
dead_letter
```

---

# 62. Edge Startup

Fluxo:

```text
Start
↓
Load Local Config
↓
Open SQLite
↓
Load Adapters
↓
Connect MQTT
↓
Sync Cloud Configuration
↓
Start Polling
↓
Start Command Consumer
↓
Start Heartbeat
```

---

# 63. Edge offline

Se Cloud estiver indisponível durante inicialização:

```text
usar última configuração válida
```

e continuar aquisição local.

---

# 64. Configuração Edge

Toda configuração deve possuir:

```text
version
```

Edge mantém:

```text
current_version
reported_version
```

Nova configuração:

```text
download
↓
validate
↓
prepare
↓
apply
↓
health check
```

Em falha:

```text
rollback
```

---

# 65. Provisionamento Edge

Fluxo:

```text
Admin creates Gateway
↓
Activation Token
↓
Edge starts
↓
Provisioning API
↓
Credential issued
↓
Token invalidated
↓
MQTT connect
```

---

# 66. Activation Token

Obrigatoriamente:

```text
single-use

short-lived

cryptographically random
```

---

# 67. Identidade de Device/Gateway

Preferência:

```text
X.509 + mTLS
```

Fallback temporário para MVP:

```text
client_id
+
individual secret
```

Desde que:

- TLS seja obrigatório;
- segredo seja individual;
- exista rotação;
- arquitetura permita migração para certificado.

Nunca utilizar senha compartilhada global.

---

# 68. Segurança humana

Password hashing:

```text
Argon2id
```

Não armazenar senha reversível.

MFA previsto desde o início.

Obrigatório para:

```text
PLATFORM_ADMIN
ORGANIZATION_OWNER
```

---

# 69. Sessões

Sessões devem ser:

- revogáveis;
- rastreáveis;
- expiradas;
- protegidas.

Preferência web:

```text
short-lived access token
+
rotating refresh token
```

ou sessão server-side equivalente.

---

# 70. Multi-tenancy

Tenant:

```text
Organization
```

Proteção obrigatória:

```text
Authentication
↓
Application Authorization
↓
Tenant Context
↓
PostgreSQL RLS
```

Nunca confiar apenas em:

```sql
WHERE organization_id = ?
```

---

# 71. PostgreSQL RLS

Tabelas de tenant devem possuir policies adequadas.

Conta normal da aplicação não pode possuir:

```text
SUPERUSER
BYPASSRLS
```

---

# 72. Teste cross-tenant

Obrigatório no CI.

Cenário:

```text
User Organization A
↓
request UUID Organization B
```

Resultado:

```text
NO DATA LEAK
```

Uma falha desse teste bloqueia merge.

---

# 73. Secrets

Nunca armazenar segredo em:

```text
Git

source code

Dockerfile

frontend

logs

documentation
```

Desenvolvimento:

```text
.env
```

permitido localmente.

Obrigatório:

```text
.env.example
.gitignore
```

---

# 74. API

Base:

```text
/api/v1
```

JSON como formato inicial.

IDs:

```text
UUID
```

Timestamps:

```text
ISO 8601 UTC
```

---

# 75. Response Envelope

Objeto:

```json
{
  "data": {}
}
```

Coleção:

```json
{
  "data": [],
  "meta": {}
}
```

Erro:

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Resource not found",
    "details": {},
    "correlation_id": "uuid"
  }
}
```

---

# 76. Correlation ID

Obrigatório.

Header:

```text
X-Correlation-ID
```

Deve acompanhar fluxos críticos entre:

```text
API
Core
Broker
Edge
Audit
```

---

# 77. Paginação

Preferência:

```text
cursor-based
```

Exemplo:

```text
?limit=50&cursor=...
```

Default:

```text
50
```

Máximo inicial:

```text
200
```

---

# 78. API principal do MVP

## Auth

```text
POST /api/v1/auth/login
POST /api/v1/auth/logout
GET  /api/v1/auth/me
GET  /api/v1/auth/sessions
```

## Organizations

```text
GET
POST
PATCH
```

## Memberships

```text
GET
POST
PATCH
DELETE
```

## Sites

```text
GET
POST
PATCH
```

## Areas

```text
GET
POST
PATCH
```

## Systems

```text
GET
POST
PATCH
```

## Assets

```text
GET
POST
PATCH
```

## Gateways

```text
GET
POST
PATCH
```

## Device Models

```text
GET
POST
PATCH
```

## Devices

```text
GET
POST
PATCH
```

## Metrics

```text
GET
POST
PATCH
```

---

# 79. Telemetry APIs

Ingestão:

```text
POST /api/v1/ingestion/telemetry
```

Batch:

```text
POST /api/v1/ingestion/telemetry/batch
```

Atual:

```text
GET /api/v1/devices/{id}/telemetry/current
```

Histórico:

```text
GET /api/v1/devices/{id}/telemetry
```

---

# 80. Histórico

Filtros:

```text
metrics
from
to
resolution
aggregation
```

Resoluções:

```text
raw
1m
5m
15m
1h
1d
```

Agregações dependem de `aggregation_type`.

---

# 81. Commands

Frontend nunca publica diretamente em MQTT.

Fluxo obrigatório:

```text
Frontend
↓
API
↓
Authentication
↓
Authorization
↓
Validation
↓
Audit
↓
MQTT
↓
Edge
↓
Device
```

---

# 82. Command Model

Estados:

```text
CREATED
QUEUED
SENT
RECEIVED
EXECUTING
SUCCESS
FAILED
TIMEOUT
CANCELLED
EXPIRED
```

Todo comando deve possuir:

```text
command_id

expires_at

correlation_id
```

---

# 83. Command Safety

Nunca iniciar operação de máquinas utilizando comandos arbitrários.

Command Types devem ser previamente definidos.

Exemplo:

```text
SET_SPEED
```

deve conter:

- datatype;
- unidade;
- min;
- max;
- role exigida;
- criticality.

---

# 84. Segurança industrial

A plataforma não substitui sistemas certificados de segurança funcional.

Continuam locais:

```text
Emergency Stop

Safety Relay

Safety PLC

Interlocks

Drive Safety Functions
```

Cloud não deve ser o único mecanismo de segurança de máquina.

---

# 85. Dashboard

Dashboard deve consumir:

```text
Semantic Metrics
```

Nunca:

```text
Modbus registers
```

ou payload específico de fabricante.

Fluxo:

```text
Device
↓
Adapter
↓
Metric
↓
Telemetry API
↓
Dashboard
```

---

# 86. Dashboard MVP

Mostrar:

- identificação do Asset;
- conectividade;
- estado operacional;
- valores atuais;
- gráfico histórico;
- alarmes ativos;
- eventos recentes.

Não criar drag-and-drop builder no MVP.

---

# 87. Alarm Engine

Operadores iniciais:

```text
>
>=
<
<=
==
!=
```

Regra:

```text
metric
operator
threshold
duration
severity
```

---

# 88. Alarm Lifecycle

```text
NORMAL
↓
ACTIVE
↓
ACKNOWLEDGED
↓
CLEARED
```

ACK não significa normalização da condição.

---

# 89. Audit

Audit Log é diferente de application log.

Audit deve responder:

```text
Quem?

Fez o quê?

Quando?

Em qual recurso?

Em qual tenant?
```

---

# 90. Eventos obrigatórios de auditoria

```text
LOGIN_SUCCESS
LOGIN_FAILURE

USER_CREATED
ROLE_CHANGED

DEVICE_CREATED
DEVICE_CREDENTIAL_REVOKED

GATEWAY_PROVISIONED

ALARM_ACKNOWLEDGED

COMMAND_CREATED
COMMAND_EXECUTED
COMMAND_FAILED
```

---

# 91. Logging

Logs devem ser estruturados.

Nunca registrar:

```text
password
access token
refresh token
private key
secret
Authorization header
```

---

# 92. Observabilidade

Implementar desde o início.

Métricas mínimas:

```text
API latency

HTTP errors

messages received

messages rejected

duplicate messages

ingestion latency

queue depth

device online count

device offline count

DB health
```

---

# 93. Health endpoints

```text
GET /health/live
GET /health/ready
```

`live`:

processo está executando.

`ready`:

dependências essenciais disponíveis.

---

# 94. Banco de dados

Banco principal:

```text
PostgreSQL
```

Time series:

```text
TimescaleDB
```

Edge:

```text
SQLite
```

---

# 95. Regras de banco

Banco:

```text
snake_case
```

Código:

```text
camelCase
```

Classes:

```text
PascalCase
```

Todos os timestamps:

```text
TIMESTAMPTZ
```

---

# 96. JSONB

Pode ser usado para:

- metadata;
- propriedades auxiliares;
- configuração extensível.

Não armazenar exclusivamente em JSONB informações necessárias para:

- segurança;
- relacionamento;
- filtros críticos;
- regras centrais.

---

# 97. Migrations

Migration executada em ambiente compartilhado:

```text
IMMUTABLE
```

Nunca editar migration antiga.

Criar uma nova.

---

# 98. Ordem inicial de migrations

```text
001 extensions

002 organizations

003 users

004 memberships

005 sites

006 areas

007 systems

008 asset_types

009 assets

010 gateways

011 device_models

012 devices

013 metric_definitions

014 device_metric_mappings

015 device_credentials

016 raw_messages

017 telemetry

018 events

019 alarm_rules

020 alarms

021 audit_logs
```

Depois:

```text
022 command_definitions

023 commands
```

---

# 99. Git Workflow

Branches:

```text
main
develop

feature/*
fix/*
```

`main`:

production-ready.

`develop`:

integração.

Não permitir push direto para branches protegidas.

---

# 100. Feature Branches

Exemplos:

```text
feature/telemetry-ingestion

feature/edge-provisioning

feature/alarm-engine
```

Evitar branch:

```text
feature/build-whole-platform
```

---

# 101. Pull Requests

Todo PR deve informar:

```text
Objective

Changes

Tests

Database impact

Contract impact

Security impact

Screenshots
```

quando aplicável.

---

# 102. Regra de PR

Preferir PR pequeno, focado e revisável.

Separar mudanças independentes.

---

# 103. Code Review

Revisão deve verificar:

- arquitetura;
- segurança;
- domínio;
- testes;
- nomenclatura;
- tenant handling;
- contratos;
- erro;
- logs.

---

# 104. Architecture Decision Record

Criar ADR quando houver alteração relevante em:

```text
architecture

database

protocol

security

public contract

major dependency

deployment
```

Estrutura:

```text
Status

Context

Decision

Consequences

Alternatives
```

---

# 105. Coding Standards

Todo código deve:

- ser tipado;
- evitar `any`;
- possuir nomes claros;
- possuir tratamento explícito de erro;
- evitar funções excessivamente grandes;
- separar domínio de infraestrutura;
- utilizar contratos compartilhados;
- evitar dependências circulares.

---

# 106. Regras de domínio

Controller não implementa regra de negócio.

Fluxo:

```text
Controller
↓
Use Case / Application Service
↓
Domain
↓
Repository
```

---

# 107. Regra de abstração

Não criar abstração antecipadamente apenas por possibilidade futura.

Primeiro:

```text
use case real
```

Depois:

```text
abstração
```

Mas não violar os boundaries já definidos.

---

# 108. Dependências

Toda nova dependência deve justificar:

- necessidade;
- manutenção;
- licença;
- impacto de segurança;
- tamanho;
- alternativas.

Biblioteca abandonada não deve entrar no core.

---

# 109. Test Strategy

Pirâmide:

```text
MUITOS UNIT TESTS

INTEGRATION TESTS SUFICIENTES

E2E DIRECIONADOS
```

---

# 110. Unit Tests

Obrigatórios principalmente em:

```text
domain rules

normalization

unit conversion

alarm evaluation

command validation

permissions
```

---

# 111. Integration Tests

Obrigatórios para:

```text
PostgreSQL

RLS

TimescaleDB

Redis

EMQX

repositories

ingestion
```

---

# 112. Contract Tests

Obrigatórios entre:

```text
Backend ↔ Frontend

Cloud ↔ Edge

Producer ↔ MQTT Consumer

Edge ↔ Adapter
```

---

# 113. E2E principal

Fluxo:

```text
Create Organization
↓
Create Site
↓
Create Asset
↓
Create Gateway
↓
Provision Edge
↓
Create Device
↓
Generate Telemetry
↓
Ingest
↓
Query Current
↓
Query History
↓
Generate Alarm
↓
Acknowledge Alarm
```

---

# 114. E2E de falha de Internet

```text
Connect Edge
↓
Disable Internet
↓
Generate Telemetry
↓
Restart Edge
↓
Generate More Telemetry
↓
Restore Internet
↓
Drain Queue
```

Validar:

```text
no loss

no duplicates

correct observed_at
```

---

# 115. Security Tests

Obrigatórios:

```text
cross-tenant access

RLS

invalid credentials

revoked credentials

MQTT ACL

unauthorized commands

expired commands
```

---

# 116. Performance baseline

Primeiro baseline sugerido:

```text
100 devices

10 metrics/device

publish every 10 seconds
```

Resultado:

```text
~100 metric values/second
```

Isso é teste inicial, não limite arquitetural.

---

# 117. CI

Pipeline mínimo:

```text
install
↓
lint
↓
typecheck
↓
unit tests
↓
integration tests
↓
contract tests
↓
build
↓
security scan
```

---

# 118. Security Gate

Bloquear merge em caso de:

```text
critical known vulnerability

secret detected

tenant isolation failure

RLS failure
```

---

# 119. Environment Strategy

Ambientes:

```text
local

development

staging

production
```

Não compartilhar:

- banco;
- credentials;
- secrets;
- broker credentials;

entre ambientes.

---

# 120. Local Development

Todo desenvolvedor deve conseguir iniciar o projeto com documentação reproduzível.

Objetivo:

```text
git clone

configuration

pnpm install

docker compose up

pnpm dev
```

Nenhuma configuração manual não documentada será considerada aceitável.

---

# 121. Production

Produção inicial utilizará:

```text
Docker
```

Estrutura aproximada:

```text
Reverse Proxy

Frontend

API

Worker

PostgreSQL / TimescaleDB

Redis

EMQX

Monitoring
```

---

# 122. Deploy

Todo deploy deve ser rastreável por:

```text
version

Git commit

container image
```

---

# 123. Backup

Antes da primeira produção:

```text
backup
+
restore test
```

é obrigatório.

Backup sem teste de restauração não é considerado estratégia validada.

---

# 124. Definition of Ready

Tarefa só deve entrar em desenvolvimento quando possuir, quando aplicável:

```text
Objective

Acceptance Criteria

Dependencies

Contract

Security implications

Test scenario
```

---

# 125. Definition of Done

Uma tarefa está concluída somente quando:

1. código implementado;
2. lint aprovado;
3. typecheck aprovado;
4. build aprovado;
5. testes unitários aprovados;
6. integrações testadas quando aplicável;
7. contratos atualizados;
8. migration criada quando necessária;
9. documentação atualizada;
10. tratamento de erros implementado;
11. logs adequados;
12. segurança revisada;
13. tenant isolation preservado;
14. PR revisado;
15. CI aprovado.

---

# 126. DoD adicional de API

Endpoint precisa possuir:

```text
Authentication

Authorization

Validation

Errors

OpenAPI

Tests

Correlation ID
```

---

# 127. DoD de entidades tenant

Obrigatório:

```text
organization_id

RLS policy

cross-tenant test
```

---

# 128. DoD de Edge

Testar:

```text
restart

offline startup

invalid config

network failure

retry

queue persistence
```

---

# 129. DoD de Adapter

Obrigatório:

```text
fixtures

normalization tests

unit conversion tests

invalid response tests

health behavior
```

---

# 130. Definition of Release

Release só pode ocorrer quando:

```text
P0 completed

critical tests pass

no critical security issues

tenant isolation validated

Store & Forward validated

deployment validated

backup restore validated

documentation updated
```

---

# 131. Severidade de bugs

## Critical

```text
security breach

cross-tenant leakage

data loss

dangerous command behavior

platform unavailable
```

Bloqueia release.

## High

```text
incorrect telemetry

alarm failure

provisioning failure

major feature unavailable
```

Normalmente bloqueia release.

## Medium

Problema com workaround.

## Low

Cosmético ou impacto limitado.

---

# 132. Proibições arquiteturais

É proibido:

1. frontend escrever diretamente no broker para comandos;
2. frontend acessar banco;
3. Edge acessar banco Cloud;
4. adapter persistir telemetria;
5. adapter avaliar alarmes;
6. criar métrica automaticamente em produção;
7. usar credencial global em Devices;
8. usar HTTP sem TLS em produção;
9. armazenar secret no Git;
10. desabilitar RLS para solucionar problema;
11. backend usar DB superuser;
12. usar `organization_id` recebido como verdade sem validação;
13. misturar lógica específica de fabricante no Core;
14. editar migration antiga executada;
15. criar contrato paralelo fora de `packages/contracts`;
16. usar dashboard-specific payload na ingestão;
17. executar comando crítico sem autorização;
18. usar Cloud como sistema de segurança funcional;
19. ignorar erros silenciosamente;
20. fazer merge com teste cross-tenant falhando.

---

# 133. Scope do MVP

P0:

```text
Identity

Tenancy

Sites

Areas

Systems

Assets

Gateways

Devices

Device Models

Metrics

Edge Provisioning

Modbus

Device Adapter

MQTT

HTTP Ingestion

Telemetry

History

Connectivity

Store & Forward

Events

Basic Alarms

Basic Dashboard

Audit

RLS

Observability
```

---

# 134. Fora do MVP

Não implementar inicialmente:

```text
Kubernetes

Microservices

Kafka

AI

Machine Learning

Predictive Maintenance

Digital Twin completo

OTA completo

Dashboard Builder

Native Mobile App

Billing

Marketplace

Multi-region
```

---

# 135. Commands no MVP

A infraestrutura de commands deve existir.

Porém controle físico real pode permanecer desabilitado inicialmente.

Primeiros comandos seguros para testes:

```text
PING

READ_CONFIGURATION
```

Somente depois:

```text
RESET
```

Comandos como:

```text
START
STOP
SET_SPEED
```

exigem validação operacional adicional.

---

# 136. Cenário de referência

O desenvolvimento deverá manter cenário padrão.

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
├── Energy Meter
└── Pressure Sensor
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

---

# 137. Primeiro protocolo industrial

Prioridade:

```text
Modbus TCP
```

Modbus RTU poderá ser adicionado ainda no MVP se não comprometer a entrega.

Não implementar diversos protocolos simultaneamente no início.

---

# 138. Simulator

Deve existir um simulador oficial.

Uso:

- desenvolvimento;
- CI;
- homologação;
- demonstração;
- testes de falha.

O simulador deve gerar dados plausíveis.

---

# 139. Ordem de implementação

## Milestone M0 — Foundation

```text
Monorepo
Docker
CI
PostgreSQL
TimescaleDB
Redis
EMQX
API skeleton
Web skeleton
Edge skeleton
```

---

# 140. Milestone M1 — Core Domain

```text
Identity
Organizations
Membership
RLS
Sites
Areas
Systems
Assets
Gateways
Devices
Metrics
```

---

# 141. Milestone M2 — Edge Connectivity

```text
Provisioning

Configuration

Heartbeat

MQTT auth

Protocol Adapter

Device Adapter

Simulator
```

---

# 142. Milestone M3 — Telemetry

```text
Polling

Normalization

MQTT ingestion

HTTP ingestion

RAW

TimescaleDB

Current State

Historical API
```

---

# 143. Milestone M4 — Resilience

```text
Store & Forward

Retry

Deduplication

Late Data

Dead Letter

Offline Restart
```

---

# 144. Milestone M5 — Operations

```text
Connectivity

Events

Alarm Engine

Alarm API

Dashboard

Realtime
```

---

# 145. Milestone M6 — Security Validation

```text
Tenant tests

RLS tests

MQTT ACL tests

Rate limits

Secret scan

Audit validation
```

---

# 146. Milestone M7 — Command Foundation

```text
CommandDefinition

Command API

MQTT Command

Command Result

Expiration

Deduplication

Audit
```

---

# 147. Milestone M8 — Release Candidate

```text
Bug fixing

Deployment

Backups

Restore test

Monitoring

Performance tests

Documentation

Release notes
```

---

# 148. Primeiras issues recomendadas

```text
001 Initialize monorepo

002 Configure lint/typecheck

003 Create Docker Compose

004 Configure database

005 Create migration system

006 Organization

007 User/session

008 Membership

009 RLS

010 Site

011 Area

012 System

013 AssetType

014 Asset

015 Gateway

016 DeviceModel

017 Device

018 MetricDefinition

019 DeviceMetricMapping

020 Edge skeleton

021 Gateway provisioning

022 MQTT authentication

023 Heartbeat

024 Modbus TCP adapter

025 Device Adapter interface

026 Simulator

027 Polling engine

028 SQLite outbox

029 Telemetry contracts

030 MQTT ingestion

031 HTTP ingestion

032 RawMessage

033 Timescale telemetry

034 Current state

035 Historical API

036 Connectivity engine

037 Store & Forward tests

038 Events

039 AlarmRule

040 Alarm

041 Dashboard

042 Audit UI/API

043 Security validation

044 Command foundation

045 Release hardening
```

---

# 149. Distribuição sugerida por equipe

## Backend

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

## Edge

```text
Provisioning
Protocols
Adapters
Polling
SQLite
MQTT
Store & Forward
Commands
Diagnostics
```

## Frontend

```text
Authentication

Hierarchy navigation

Assets

Devices

Dashboard

Telemetry visualization

Alarms
```

## Infrastructure

```text
Docker

CI/CD

Databases

EMQX

Redis

Observability

Deployment

Backup
```

---

# 150. Trabalho paralelo

Backend, Edge e frontend podem trabalhar simultaneamente desde que os contratos sejam definidos primeiro.

Exemplo:

```text
packages/contracts
        │
        ├── Backend implementation
        ├── Edge implementation
        └── Frontend mocks
```

Contrato deve preceder implementações paralelas.

---

# 151. Regras de segurança não negociáveis

1. TLS obrigatório em produção.
2. Password hashing Argon2id.
3. Credencial individual para Gateway/Device.
4. MQTT autenticado.
5. MQTT ACL.
6. RLS em dados de tenant.
7. Nenhuma credencial no Git.
8. Nenhum comando direto frontend → MQTT.
9. Auditoria de ações sensíveis.
10. Separação dev/staging/prod.
11. Teste cross-tenant obrigatório.
12. Comandos possuem expiração.
13. Replay de comando deve ser impedido.
14. Edge preferencialmente outbound-only.

---

# 152. Pergunta obrigatória de segurança

Antes de concluir uma funcionalidade, responder:

> Quem está executando esta ação, em qual tenant, sobre qual recurso, com qual permissão, através de qual canal e como a ação será auditada?

Se isso não estiver claro, a funcionalidade não está pronta.

---

# 153. Pergunta obrigatória de domínio

Antes de criar tabela ou campo:

> Esta informação pertence ao Asset, Device, Telemetry, Configuration ou Application?

Se a resposta não estiver clara, revisar a modelagem.

---

# 154. Pergunta obrigatória de arquitetura

Antes de adicionar dependência ou novo serviço:

> Isto resolve uma necessidade atual ou está adicionando complexidade para uma hipótese futura?

Evitar complexidade prematura.

---

# 155. Pergunta obrigatória para adapters

Antes de implementar lógica em Adapter:

> Esta lógica traduz equipamento/protocolo ou é uma regra de negócio?

Se for regra de negócio, não pertence ao Adapter.

---

# 156. Pergunta obrigatória para dashboards

Antes de adicionar tratamento específico no dashboard:

> O dado deveria estar normalizado no Core?

Se sim, corrigir o modelo, não o dashboard.

---

# 157. MVP Acceptance Criteria

O MVP deve demonstrar:

```text
Organization created

User authenticated

Site created

Asset hierarchy created

Gateway provisioned

Device registered

Telemetry collected

Telemetry normalized

Telemetry persisted

Current telemetry available

Historical telemetry available

Device connectivity detected

Offline Edge works

Store & Forward works

Alarm generated

Alarm acknowledged

Dashboard displays data

Audit records sensitive action

Cross-tenant access blocked
```

---

# 158. Piloto inicial

Após Release Candidate:

```text
1 Organization

1 Site

1 Gateway

1–5 Devices
```

Objetivo:

validar comportamento em instalação real.

---

# 159. Critérios de saída do piloto

```text
No critical data loss

No tenant leakage

No critical security issue

Stable Edge

Stable ingestion

Correct alarms

Acceptable dashboard operation
```

---

# 160. Scope Change

Durante o MVP, uma nova funcionalidade somente entra se:

1. for necessária para P0;
2. bloquear acceptance criteria;
3. corrigir arquitetura;
4. corrigir segurança crítica.

Caso contrário:

```text
POST-MVP BACKLOG
```

---

# 161. Regra contra scope creep

Uma funcionalidade não entra no MVP apenas porque:

```text
seria interessante
```

O MVP existe para validar a fundação da plataforma.

---

# 162. Pós-MVP

Prioridades naturais:

## Release 1.1

```text
Operational Commands

Notifications

Dashboard Templates

Basic Reports
```

## Release 1.2

```text
Device Twin

Remote Configuration

Advanced Diagnostics

Firmware Foundation
```

## Release 1.3

```text
OTA

Additional Protocols

Webhooks

External Integrations
```

## 2.x

```text
Analytics

Anomaly Detection

Predictive Maintenance

AI
```

---

# 163. Fonte oficial de verdade

Durante o desenvolvimento:

```text
Domain Model
        ↓
Contracts
        ↓
Implementation
```

Não:

```text
Implementation
        ↓
invent contract afterwards
```

---

# 164. Architecture Freeze

Após aprovação deste manual, mudanças nas seguintes áreas exigem ADR:

```text
Tenant model

Asset / Device separation

Telemetry Envelope

Metric Model

Security baseline

Edge contract

Core database strategy

Protocol boundaries
```

---

# 165. Qualidade esperada

O objetivo não é apenas entregar funcionalidades.

A plataforma deve possuir desde o início:

```text
predictability

traceability

testability

security

observability

maintainability
```

---

# 166. Regra de implementação

Quando houver conflito entre:

```text
implementação mais rápida
```

e:

```text
violação de boundary central
```

não violar o boundary.

Quando houver conflito entre:

```text
arquitetura perfeita
```

e:

```text
simplicidade suficiente para MVP
```

preferir a solução simples, desde que preserve os boundaries.

---

# 167. Critério final de desenvolvimento

Toda feature deve poder responder claramente:

```text
Who owns the data?

Who may access it?

Where is it persisted?

How is it validated?

How is it tested?

How is it observed?

How does it behave offline?

How does it fail?

How is it audited?
```

As perguntas que não se aplicam devem ser explicitamente identificadas como tal.

---

# 168. Resultado esperado do MVP

O MVP não deverá ser uma coleção de telas ou APIs desconectadas.

Ele deverá funcionar como um sistema industrial completo:

```text
FIELD
  ↓
EDGE
  ↓
CONNECTIVITY
  ↓
IoT CORE
  ↓
DATA
  ↓
OPERATIONS
```

com:

```text
Security
+
Observability
+
Resilience
```

atravessando todas as camadas.

---

# 169. Regra principal para o time

> Não desenvolver para uma tela específica, um fabricante específico ou uma demonstração específica. Desenvolver para o modelo da plataforma.

Equipamentos, dashboards e aplicações devem se conectar ao modelo comum.

Isso é o que permitirá que a solução evolua de alguns dispositivos para milhares deles sem reescrever o Core.

---

# 170. Baseline oficial

A partir da aprovação deste manual, ficam definidos como baseline do desenvolvimento:

### Arquitetura

```text
Edge + Cloud
Modular Monolith
API-first
Event-driven
Vendor-neutral
Multi-tenant
```

### Stack

```text
TypeScript
NestJS
React
PostgreSQL
TimescaleDB
Redis
EMQX
SQLite
Docker
```

### Dados

```text
Asset != Device

Semantic Metrics

RAW + Normalized

observed_at + received_at

UUID

UTC
```

### Segurança

```text
TLS
RBAC
RLS
Individual Device Identity
Audit
Least Privilege
```

### Operação Edge

```text
Adapters
Polling
Normalization
SQLite
Store & Forward
MQTT
Offline Operation
```

### Desenvolvimento

```text
Monorepo
Shared Contracts
Protected Branches
Pull Requests
Automated Tests
CI
ADR
Definition of Done
```

---

# 171. Início oficial do desenvolvimento

A primeira fase deve começar por:

```text
M0 — FOUNDATION
```

Sequência:

```text
1. Criar repositório

2. Criar monorepo

3. Criar estrutura de apps/packages

4. Configurar TypeScript

5. Configurar lint e formatter

6. Configurar testes

7. Criar Docker Compose

8. Subir PostgreSQL/TimescaleDB

9. Subir Redis

10. Subir EMQX

11. Criar skeleton NestJS

12. Criar skeleton React

13. Criar skeleton Edge

14. Criar packages/contracts

15. Configurar CI

16. Criar primeira migration

17. Implementar Organization
```

Não iniciar pelo dashboard.

Não iniciar por integrações específicas.

Não iniciar criando vários adapters.

Primeiro deve existir a fundação sobre a qual todo o restante será construído.

---

# 172. Definition of M0 Complete

M0 somente estará concluída quando um novo desenvolvedor puder:

```text
clone repository
↓
configure environment
↓
install dependencies
↓
start infrastructure
↓
start API
↓
start Web
↓
start Edge
↓
run tests
```

usando apenas a documentação do projeto.

Nenhum conhecimento tribal ou configuração não documentada poderá ser requisito.

---

# 173. Primeira entrega funcional

Depois de M0 e M1, a primeira entrega vertical deverá ser:

```text
Organization
↓
Site
↓
Asset
↓
Gateway
↓
Device
↓
Simulator
↓
Telemetry
↓
Current Value
↓
Simple Dashboard
```

Esse primeiro vertical slice deve funcionar completamente antes de aumentar significativamente o número de recursos.

---

# 174. Desenvolvimento por Vertical Slice

Depois da fundação, preferir desenvolvimento vertical.

Exemplo:

```text
Device registration
+
Edge acquisition
+
Ingestion
+
Storage
+
API
+
Simple UI
```

é preferível a implementar dezenas de tabelas e telas sem fluxo operacional validado.

---

# 175. Responsabilidade coletiva

Arquitetura, segurança, testes e documentação não pertencem exclusivamente a uma pessoa ou equipe específica.

Cada desenvolvedor é responsável por preservar:

```text
architecture

security

tests

contracts

documentation
```

em suas alterações.

---

# 176. Status do documento

**Documento:** Development Manual  
**Versão:** 1.0  
**Status:** Aprovável como baseline para início do desenvolvimento  
**Tipo:** Manual operacional consolidado  
**Aplicação:** Backend, Frontend, Edge, Infrastructure e QA  
**Escopo:** MVP da Plataforma Industrial IoT  

Este documento deve acompanhar o repositório em:

```text
/docs/DEVELOPMENT-MANUAL.md
```

e servir como referência primária para novos desenvolvedores, agentes de IA e revisões técnicas do projeto.