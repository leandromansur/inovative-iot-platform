# Domain Model v1.0

## 1. Objetivo

Este documento define o modelo de domínio oficial da plataforma Industrial IoT.

O objetivo é estabelecer:

- entidades principais;
- responsabilidades;
- relacionamentos;
- cardinalidades;
- regras de propriedade;
- hierarquia operacional;
- separação entre ativos e dispositivos;
- modelo de métricas;
- estrutura de telemetria;
- eventos;
- alarmes;
- comandos;
- usuários e permissões;
- regras de multi-tenancy.

Este documento deve servir como referência para:

- banco de dados;
- migrations;
- APIs;
- contratos MQTT;
- frontend;
- Edge;
- integrações;
- testes.

---

# 2. Princípio central do domínio

A plataforma representa duas realidades diferentes:

```text
REALIDADE OPERACIONAL
+
REALIDADE DE CONECTIVIDADE
```

A realidade operacional contém:

```text
Organization
Site
Area
System
Asset
```

A realidade de conectividade contém:

```text
Gateway
Device
Device Model
Metric
Telemetry
```

Esses dois mundos se relacionam, mas não devem ser confundidos.

---

# 3. Visão geral

```text
Organization
│
├── Users / Memberships
│
├── Sites
│   │
│   ├── Areas
│   │   │
│   │   └── Systems
│   │       │
│   │       └── Assets
│   │           │
│   │           └── Devices
│   │
│   └── Gateways
│       │
│       └── Devices
│
├── Alarm Rules
├── Dashboards
├── Integrations
└── Audit Logs
```

---

# 4. Diagrama conceitual

```text
                    PLATFORM
                       │
                       │
                 Organization
                       │
            ┌──────────┼───────────┐
            │          │           │
        Membership    Site     Integrations
                       │
                      Area
                       │
                     System
                       │
                     Asset
                       │
                 ┌─────┴──────┐
                 │            │
              Device       Child Asset
                 │
           Device Model
                 │
          Metric Mapping
                 │
              Metric
                 │
             Telemetry
```

Gateway:

```text
Site
 │
 └── Gateway
       │
       └── Device
```

Um Device poderá simultaneamente estar associado a:

```text
Gateway
+
Asset
```

---

# 5. Organization

## Definição

Representa a unidade principal de isolamento de dados da plataforma.

Normalmente corresponde a:

- cliente;
- empresa;
- grupo empresarial;
- organização interna.

Exemplo:

```text
Empresa ABC
```

## Campos

```text
Organization

id UUID
name varchar
slug varchar
status enum
metadata jsonb

created_at timestamptz
updated_at timestamptz
```

## Status

```text
ACTIVE
SUSPENDED
DISABLED
```

## Regras

Todo dado pertencente a cliente deve estar associado direta ou indiretamente a uma Organization.

`organization_id` será também a principal chave de isolamento de tenant.

---

# 6. User

## Definição

Representa uma identidade humana capaz de acessar a plataforma.

```text
User

id UUID

name
email

password_hash

status

last_login_at
created_at
updated_at
```

## Status

```text
ACTIVE
INVITED
SUSPENDED
DISABLED
```

Um usuário poderá possuir acesso a várias organizações.

Por esse motivo:

```text
User
```

não possui obrigatoriamente um único:

```text
organization_id
```

O relacionamento será realizado através de Membership.

---

# 7. Membership

Relaciona:

```text
User
↔
Organization
```

## Campos

```text
Membership

id UUID
user_id UUID
organization_id UUID

role
status

created_at
```

## Exemplo

```text
Leandro
│
├── Organization A
│      role = OWNER
│
└── Organization B
       role = VIEWER
```

---

# 8. Role

Roles iniciais:

```text
PLATFORM_ADMIN

ORGANIZATION_OWNER
ORGANIZATION_ADMIN

ENGINEER
TECHNICIAN
OPERATOR
VIEWER
```

## Responsabilidade

Role determina permissões gerais.

Não deverá inicialmente representar permissões individuais extremamente granulares.

O sistema poderá evoluir posteriormente para:

```text
RBAC
+
ABAC
```

---

# 9. Site

## Definição

Representa uma instalação física ou lógica de uma Organization.

Exemplos:

```text
Fábrica Rio de Janeiro
Hospital Unidade Centro
Posto GNV Barra
Shopping Tijuca
```

## Campos

```text
Site

id UUID
organization_id UUID

name
code

timezone

latitude
longitude

address

status
metadata

created_at
updated_at
```

## Regra importante

`timezone` deve ser obrigatória.

Internamente:

```text
timestamps → UTC
```

Apresentação:

```text
UTC → timezone do Site
```

---

# 10. Area

## Definição

Subdivisão funcional ou física de um Site.

Exemplos:

```text
Utilidades
Casa de Máquinas
Produção
Subestação
HVAC
Estacionamento
```

## Campos

```text
Area

id UUID
organization_id UUID
site_id UUID

parent_area_id UUID nullable

name
code
description

created_at
updated_at
```

Áreas poderão possuir estrutura hierárquica.

Exemplo:

```text
Produção
├── Linha 01
└── Linha 02
```

---

# 11. System

## Definição

Representa um sistema funcional.

Exemplos:

```text
Sistema de Ar Comprimido
Sistema de Bombeamento
HVAC
Sistema Fotovoltaico
Sistema Elétrico
Carregamento EV
```

## Campos

```text
System

id UUID
organization_id UUID
site_id UUID
area_id UUID

name
code
system_type

description
metadata

created_at
updated_at
```

---

# 12. Asset

## Definição

Asset representa algo que o cliente considera um ativo operacional.

Exemplos:

```text
Compressor 01
Bomba 03
Chiller 02
Transformador 01
Painel QGBT
Gerador
Motor
Carregador EV
```

Asset não significa necessariamente um equipamento conectado.

---

# 13. Asset Fields

```text
Asset

id UUID

organization_id UUID
site_id UUID
area_id UUID nullable
system_id UUID nullable

parent_asset_id UUID nullable

asset_type_id UUID

name
code

manufacturer nullable
model nullable
serial_number nullable

installation_date nullable

status

metadata jsonb

created_at
updated_at
```

---

# 14. Hierarquia de Assets

Assets podem possuir outros Assets.

Exemplo:

```text
Compressor 01
│
├── Motor principal
├── Sistema de óleo
└── Resfriador
```

Outro exemplo:

```text
Painel QGBT
│
├── Alimentador 01
├── Alimentador 02
└── Medição Geral
```

Isso utiliza:

```text
parent_asset_id
```

---

# 15. Asset Type

Define a categoria funcional do Asset.

## Estrutura

```text
AssetType

id UUID
key
name
description
category

metadata
```

## Exemplos

```text
motor
pump
compressor

transformer
generator
ups

electrical_panel

solar_inverter

ev_charger

chiller
hvac_unit
```

---

# 16. Por que Asset Type é importante

Asset Type permitirá associar:

```text
Dashboard Template

Alarm Template

Metric Requirements

Analytics Model

Maintenance Template
```

Exemplo:

```text
AssetType
compressor

→ Compressor Dashboard
→ Pressure Metrics
→ Vibration Metrics
→ Compressor Alarm Rules
```

---

# 17. Gateway

## Definição

Gateway representa um dispositivo responsável por conectar um conjunto de equipamentos à plataforma.

Pode ser:

```text
Industrial PC
Raspberry Pi industrial
PLC
Dedicated Gateway
Embedded Linux
```

## Campos

```text
Gateway

id UUID
organization_id UUID
site_id UUID

name

serial_number
hardware_model

software_version
firmware_version

status

last_seen_at

metadata

created_at
updated_at
```

---

# 18. Gateway Status

```text
ONLINE
DEGRADED
OFFLINE
UNKNOWN
DISABLED
```

---

# 19. Device

## Definição

Device representa qualquer entidade capaz de produzir ou receber dados através da plataforma.

Exemplos:

```text
Energy Meter
PLC
Drive
Sensor
Controller
UPS
Protection Relay
```

---

# 20. Device Fields

```text
Device

id UUID

organization_id UUID
site_id UUID

asset_id UUID nullable
gateway_id UUID nullable

device_model_id UUID

name

serial_number
external_id

protocol

status

firmware_version

last_seen_at

metadata

created_at
updated_at
```

---

# 21. Relação Asset × Device

Um Asset poderá possuir:

```text
0..N Devices
```

Um Device poderá pertencer inicialmente a:

```text
0..1 Asset
```

Exemplo:

```text
Asset
Compressor 01

Devices
├── Drive CFW11
├── Meter MMW04
├── Pressure Sensor
└── Temperature Sensor
```

---

# 22. Device sem Asset

Permitido.

Exemplo:

```text
Gateway recém-provisionado
Sensor ainda não associado
Medidor aguardando configuração
```

Depois poderá ser associado a um Asset.

---

# 23. Device Model

Representa um modelo conhecido de equipamento.

```text
DeviceModel

id UUID

manufacturer
family
model

device_type

adapter_key

description

metadata

created_at
updated_at
```

Exemplo:

```text
manufacturer:
WEG

family:
MMW

model:
MMW04

device_type:
ENERGY_METER

adapter_key:
weg.mmw04
```

---

# 24. Device Type

Valores iniciais:

```text
SENSOR

ENERGY_METER

DRIVE

SOFT_STARTER

PLC

CONTROLLER

PROTECTION_RELAY

UPS

EV_CHARGER

GATEWAY

OTHER
```

---

# 25. Metric Definition

Metric Definition representa uma grandeza semanticamente conhecida pela plataforma.

Não pertence a um fabricante específico.

## Campos

```text
MetricDefinition

id UUID

key

name

quantity

canonical_unit

datatype

category

description

metadata
```

---

# 26. Metric Key

Metric Key deverá seguir nomenclatura hierárquica.

Exemplos:

```text
electrical.voltage.line_l1_l2

electrical.current.l1

electrical.frequency

electrical.active_power.total

electrical.energy.imported

pressure.oil

pressure.suction

pressure.discharge

temperature.motor

temperature.bearing

vibration.rms

rotation.speed
```

---

# 27. Datatypes

Inicialmente:

```text
FLOAT

INTEGER

BOOLEAN

STRING

ENUM
```

---

# 28. Unidade canônica

A plataforma deve armazenar cada grandeza em unidade canônica.

Exemplo:

```text
Voltage
→ V

Current
→ A

Pressure
→ bar

Temperature
→ °C

Power
→ kW

Energy
→ kWh
```

Adapters serão responsáveis pela conversão.

Exemplo:

```text
Device:
pressure = 250 kPa

Adapter:

250 kPa
   ↓
2.5 bar

Platform:
pressure = 2.5 bar
```

---

# 29. Device Metric Mapping

Define quais métricas um Device Model suporta.

```text
DeviceMetricMapping

id UUID

device_model_id UUID
metric_id UUID

source_address

source_datatype

scale
offset

poll_interval_ms

readable
writable

metadata
```

---

# 30. Exemplo de mapping

```text
Device Model:
MMW04

Source:
register 40101

Metric:
electrical.voltage.line_l1_l2

Scale:
0.1

Unit:
V
```

---

# 31. Telemetry

Telemetria representa valores medidos.

Tabela principal:

```text
Telemetry

organization_id UUID

device_id UUID
asset_id UUID nullable

metric_id UUID

observed_at timestamptz
received_at timestamptz

value_double nullable
value_integer nullable
value_boolean nullable
value_text nullable

quality

message_id
```

---

# 32. Observed At × Received At

`observed_at`

Momento em que o valor foi medido.

`received_at`

Momento em que a plataforma recebeu o valor.

Exemplo:

```text
Internet caiu

10:00 medição
10:01 medição
10:02 medição

Internet retorna 10:15

received_at = 10:15

observed_at permanece:
10:00
10:01
10:02
```

---

# 33. Quality

Qualidade da medição:

```text
GOOD
UNCERTAIN
BAD
STALE
UNKNOWN
```

---

# 34. Message

Mensagem enviada pelo Edge.

```text
Message

message_id UUID

organization_id

gateway_id

device_id

message_type

observed_at
received_at

sequence_number

payload
```

`message_id` será utilizado para idempotência.

---

# 35. Raw Message

Payload recebido antes da transformação.

```text
RawMessage

id UUID

message_id

organization_id
gateway_id
device_id

protocol

observed_at
received_at

payload jsonb

processing_status

error
```

---

# 36. Processing Status

```text
RECEIVED

PROCESSED

PARTIALLY_PROCESSED

FAILED
```

---

# 37. Event

Eventos representam ocorrências.

```text
Event

id UUID

organization_id

site_id nullable

asset_id nullable

device_id nullable

source_type
source_id

event_type

severity

occurred_at
received_at

message

payload
```

---

# 38. Event Severity

```text
INFO

NOTICE

WARNING

CRITICAL
```

---

# 39. Event Types

Exemplos:

```text
DEVICE_ONLINE
DEVICE_OFFLINE

DEVICE_DEGRADED

GATEWAY_ONLINE
GATEWAY_OFFLINE

HIGH_PRESSURE

LOW_PRESSURE

HIGH_TEMPERATURE

DEVICE_FAULT

COMMAND_EXECUTED

COMMAND_FAILED
```

---

# 40. Alarm Rule

Define quando um evento operacional deve virar alarme.

```text
AlarmRule

id UUID

organization_id

asset_type_id nullable
asset_id nullable
device_id nullable
metric_id

operator

threshold

duration_ms

severity

enabled

metadata
```

---

# 41. Operadores

```text
>

>=

<

<=

==

!=

BETWEEN

OUTSIDE
```

Futuramente:

```text
RATE_OF_CHANGE

DEVIATION

ANOMALY
```

---

# 42. Alarm

Instância de uma condição de alarme.

```text
Alarm

id UUID

organization_id

alarm_rule_id

asset_id
device_id

severity

status

opened_at
acknowledged_at
cleared_at

acknowledged_by

trigger_value

message
```

---

# 43. Alarm Status

```text
ACTIVE

ACKNOWLEDGED

CLEARED
```

---

# 44. Alarm Lifecycle

```text
NORMAL
  │
  ↓
CONDITION TRUE
  │
  ↓
ACTIVE
  │
  ├──── acknowledge ────→ ACKNOWLEDGED
  │
  ↓
CONDITION FALSE
  │
  ↓
CLEARED
```

Acknowledgement não significa eliminação da condição.

---

# 45. Command

Command representa uma ação enviada ao Edge ou Device.

```text
Command

id UUID

organization_id

gateway_id
device_id
asset_id nullable

command_type

payload

status

requested_by

requested_at
sent_at
acknowledged_at
completed_at

correlation_id

result
error
```

---

# 46. Command Status

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
```

---

# 47. Command Type

Não deve ser texto arbitrário enviado diretamente pelo frontend.

Deve existir catálogo conhecido.

Exemplos:

```text
START

STOP

RESET

SET_SPEED

SET_SETPOINT

SET_MODE

UPDATE_CONFIGURATION
```

---

# 48. Command Definition

```text
CommandDefinition

id UUID

key

name

target_type

schema

requires_confirmation

criticality
```

---

# 49. Safety para comandos

Comandos devem possuir classificação:

```text
LOW

MEDIUM

HIGH

CRITICAL
```

Exemplo:

```text
READ_CONFIGURATION
LOW

RESET_FAULT
MEDIUM

START_MOTOR
HIGH
```

Comandos críticos poderão posteriormente exigir:

```text
MFA
re-authentication
dual approval
```

---

# 50. Device Connectivity

Não criar um booleano simples chamado:

```text
is_online
```

Conectividade deve ser derivada.

Campos:

```text
last_seen_at

expected_interval

communication_state
```

Status:

```text
ONLINE

DEGRADED

OFFLINE

UNKNOWN

DISABLED
```

---

# 51. Device State

Device state é diferente de connectivity.

Exemplo:

```text
Connectivity:
ONLINE

Operational State:
FAULT
```

Por isso deve existir:

```text
connectivity_status
```

e:

```text
operational_status
```

separadamente.

---

# 52. Dashboard Template

```text
DashboardTemplate

id UUID

key
name

asset_type_id

version

definition jsonb

status

created_at
```

---

# 53. Dashboard Instance

Permite personalização futura.

```text
DashboardInstance

id UUID

organization_id

template_id

asset_id nullable
system_id nullable

configuration

created_at
updated_at
```

---

# 54. Notification Channel

```text
NotificationChannel

id UUID

organization_id

type

configuration

enabled
```

Tipos:

```text
EMAIL
WEBHOOK
WHATSAPP
SMS
PUSH
```

---

# 55. Notification Rule

Relaciona eventos/alarmes a canais.

```text
NotificationRule

id UUID

organization_id

event_type nullable
severity nullable
alarm_rule_id nullable

channel_id

enabled
```

---

# 56. Integration

Representa sistemas externos.

```text
Integration

id UUID

organization_id

type

name

configuration

status

created_at
```

Exemplos:

```text
ERP
CMMS
SCADA
Webhook
REST API
External MQTT
```

---

# 57. API Credential

Para sistemas externos.

```text
ApiCredential

id UUID

organization_id

name

client_id
secret_hash

scopes

status

expires_at

created_at
```

---

# 58. Device Credential

Identidade máquina-a-máquina.

```text
DeviceCredential

id UUID

organization_id

device_id nullable
gateway_id nullable

credential_type

identifier

secret_hash nullable
certificate_fingerprint nullable

status

created_at
expires_at
revoked_at
```

---

# 59. Firmware

Estrutura preparada para futuro OTA.

```text
Firmware

id UUID

device_model_id

version

storage_key

checksum

signature

created_at
```

---

# 60. Device Twin

Preparação para futuro gerenciamento.

```text
DeviceTwin

device_id

desired_state jsonb

reported_state jsonb

desired_version

reported_version

updated_at
```

---

# 61. Audit Log

Toda operação sensível deve gerar auditoria.

```text
AuditLog

id UUID

organization_id nullable

actor_type

actor_id

action

resource_type
resource_id

timestamp

ip_address

correlation_id

before jsonb
after jsonb

metadata
```

---

# 62. Actor Type

```text
USER

DEVICE

GATEWAY

SYSTEM

API_CLIENT
```

---

# 63. Regras de Multi-Tenancy

A seguinte regra será obrigatória:

```text
dados de Organization A
não podem ser acessados por
Organization B
```

Mesmo que haja falha na aplicação.

Portanto teremos:

```text
Application authorization
+
PostgreSQL RLS
```

---

# 64. Organization ID

Sempre que tecnicamente possível, entidades de tenant devem possuir explicitamente:

```text
organization_id
```

Mesmo que seja possível derivá-lo através de relacionamentos.

Isso simplifica:

- RLS;
- queries;
- auditoria;
- particionamento;
- troubleshooting.

---

# 65. IDs

Todos os IDs expostos externamente serão:

```text
UUID
```

Não utilizar IDs sequenciais nas APIs públicas.

---

# 66. Soft Delete

Não utilizar soft delete indiscriminadamente.

Entidades que exigem histórico poderão usar:

```text
deleted_at
```

ou:

```text
status = DISABLED
```

Telemetria e auditoria não devem ser apagadas por operação comum.

---

# 67. Metadata

Campos:

```text
metadata JSONB
```

podem ser utilizados para propriedades auxiliares.

Porém informações essenciais para:

- query;
- relacionamento;
- regras;
- segurança;

não devem existir apenas dentro de JSONB.

---

# 68. Naming Convention

Banco:

```text
snake_case
```

Exemplo:

```text
organization_id
observed_at
device_model_id
```

Código TypeScript:

```text
camelCase
```

Exemplo:

```text
organizationId
observedAt
deviceModelId
```

Tipos/classes:

```text
PascalCase
```

Exemplo:

```text
DeviceModel
AlarmRule
CommandDefinition
```

---

# 69. Timestamp

Todo timestamp de banco:

```text
TIMESTAMPTZ
```

Persistência:

```text
UTC
```

Interface:

```text
timezone do Site
```

---

# 70. Entidades principais v1

O domínio inicial contém:

```text
User
Organization
Membership

Site
Area
System

AssetType
Asset

Gateway

DeviceType
DeviceModel
Device

MetricDefinition
DeviceMetricMapping

RawMessage
Telemetry
Event

AlarmRule
Alarm

CommandDefinition
Command

DashboardTemplate
DashboardInstance

NotificationChannel
NotificationRule

Integration
ApiCredential
DeviceCredential

AuditLog
```

---

# 71. ER Model resumido

```text
User
  │
  └──< Membership >── Organization
                         │
                         ├──< Site
                         │     │
                         │     ├──< Area
                         │     │     │
                         │     │     └──< System
                         │     │           │
                         │     │           └──< Asset
                         │     │                  │
                         │     │                  └──< Device
                         │     │
                         │     └──< Gateway
                         │             │
                         │             └──< Device
                         │
                         ├──< AlarmRule
                         │
                         ├──< DashboardInstance
                         │
                         ├──< Integration
                         │
                         └──< AuditLog


DeviceModel
   │
   └──< DeviceMetricMapping >── MetricDefinition
   │
   └──< Device
            │
            ├──< Telemetry
            ├──< Event
            ├──< Alarm
            └──< Command
```

---

# 72. Cardinalidades principais

```text
Organization 1:N Site

Site 1:N Area

Area 1:N System

System 1:N Asset

Asset 1:N Asset

Asset 1:N Device

Site 1:N Gateway

Gateway 1:N Device

DeviceModel 1:N Device

DeviceModel N:N MetricDefinition
via DeviceMetricMapping

Device 1:N Telemetry

Device 1:N Event

Device 1:N Alarm

Device 1:N Command
```

---

# 73. Exemplo completo

```text
Organization
Hospital ABC

└── Site
    Unidade Copacabana

    └── Area
        Central de Utilidades

        └── System
            Sistema de Ar Comprimido

            └── Asset
                Compressor 01

                ├── Asset
                │   Motor Principal
                │
                ├── Device
                │   Inversor
                │
                ├── Device
                │   Medidor de Energia
                │
                └── Device
                    Sensor de Pressão
```

Gateway:

```text
Site

└── Gateway
    Gateway Utilidades 01

    ├── Device
    │   Inversor
    │
    ├── Device
    │   Medidor
    │
    └── Device
        Sensor
```

---

# 74. Exemplo de fluxo de dados

Sensor:

```text
4-20 mA
```

PLC/Gateway:

```text
7.3 bar
```

Device Adapter:

```text
pressure.discharge
=
7.3 bar
```

Mensagem:

```json
{
  "deviceId": "uuid",
  "observedAt": "2026-10-06T13:30:00Z",
  "metrics": {
    "pressure.discharge": 7.3
  }
}
```

Plataforma:

```text
authenticate

validate

deduplicate

raw storage

normalize

persist telemetry

publish event
```

Consumers:

```text
Alarm Engine

Dashboard

WebSocket

Analytics

Integrations
```

---

# 75. Regra de desacoplamento

Nenhum dashboard poderá depender diretamente de:

```text
Modbus register

manufacturer

raw payload
```

A cadeia obrigatória será:

```text
Device-specific data
        ↓
Adapter
        ↓
Semantic Metric
        ↓
Telemetry
        ↓
Application
```

---

# 76. Regra para adapters

Adapter nunca deverá implementar:

```text
business rule
dashboard logic
user permission
alarm notification
```

Adapter somente traduz:

```text
vendor/protocol data
        ↕
platform semantic model
```

---

# 77. Regra para Edge

Edge não será a fonte oficial de configuração global.

A nuvem será a autoridade sobre:

```text
device configuration

metric configuration

desired state
```

O Edge deverá manter cópia local para operação offline.

---

# 78. Source of Truth

Fontes oficiais:

```text
Identity
→ Core Database

Asset registry
→ Core Database

Device registry
→ Core Database

Telemetry
→ TimescaleDB

Audit
→ Audit storage

Desired configuration
→ Core Database

Reported configuration
→ Device/Edge
```

---

# 79. Consistência

Para operações administrativas:

```text
strong consistency
```

Exemplo:

```text
create device
change permission
send command
```

Para telemetria:

```text
eventual consistency
```

é aceitável.

---

# 80. Idempotência

Obrigatória para:

```text
telemetry ingestion

event ingestion

command results

Edge retransmission
```

Chave principal:

```text
message_id
```

---

# 81. Retenção

Política deverá ser configurável futuramente.

Modelo conceitual:

```text
RAW

high resolution telemetry

aggregated telemetry
```

Exemplo:

```text
Raw
30 dias

Telemetry original
12 meses

Hourly aggregates
5 anos
```

Os períodos ainda não ficam definidos nesta versão.

---

# 82. Aggregation

TimescaleDB poderá gerar:

```text
1 minute

5 minutes

15 minutes

1 hour

1 day
```

Com:

```text
min

max

avg

sum

count

first

last
```

de acordo com o tipo da métrica.

---

# 83. Energy Metrics

Energia acumulada não deve ser tratada como média.

Exemplo:

```text
electrical.energy.imported
```

é contador acumulativo.

Enquanto:

```text
electrical.active_power.total
```

é grandeza instantânea.

O catálogo de métricas deverá indicar:

```text
aggregation_type
```

---

# 84. Metric Aggregation Type

Adicionar futuramente ou desde a primeira migration:

```text
GAUGE

COUNTER

STATE
```

Exemplo:

```text
voltage
→ GAUGE

energy
→ COUNTER

motor.running
→ STATE
```

---

# 85. Metadata de Metric Definition

Recomendado:

```text
aggregation_type

precision

minimum_value

maximum_value

display_unit
```

---

# 86. Estado operacional

Estados operacionais devem utilizar métricas semânticas quando possível.

Exemplo:

```text
machine.running

machine.mode

machine.faulted
```

Em vez de colunas especiais para cada tipo de máquina.

---

# 87. Tags

Deverá existir suporte futuro a tags.

Exemplo:

```text
critical

production

utility

high_priority
```

Modelo:

```text
Tag

id
organization_id
name
```

Relacionamento:

```text
AssetTag
DeviceTag
```

Não é obrigatório no MVP inicial.

---

# 88. Custom Properties

Alguns Assets precisarão de propriedades específicas.

Exemplo:

Motor:

```text
power_hp
rated_voltage
rated_current
```

Compressor:

```text
stages
rated_pressure
```

Para evitar dezenas de tabelas específicas inicialmente:

```text
Asset.metadata
```

poderá ser utilizado.

Se uma propriedade se tornar importante para consultas ou regras, deverá migrar para estrutura tipada.

---

# 89. Scope v1.0

O Domain Model v1.0 define o modelo completo de referência.

Entretanto o MVP inicial deverá implementar prioritariamente:

```text
Organization
User
Membership

Site
Area
System

AssetType
Asset

Gateway

DeviceModel
Device

MetricDefinition
DeviceMetricMapping

RawMessage
Telemetry

Event

AlarmRule
Alarm

AuditLog
```

Command poderá entrar imediatamente depois da estabilização da telemetria.

---

# 90. Ordem recomendada de migrations

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

Posteriormente:

```text
022 command_definitions

023 commands

024 dashboard_templates

025 dashboard_instances

026 notifications

027 integrations

028 device_twins

029 firmware
```

---

# 91. Critérios para alteração do domínio

Uma nova entidade somente deverá ser criada quando existir:

1. identidade própria;
2. ciclo de vida próprio;
3. relacionamento próprio;
4. comportamento ou regra relevante.

Caso contrário, deve ser avaliado se é:

```text
Value Object

enum

configuration

metadata
```

---

# 92. Regra de compatibilidade

Mudanças futuras não devem quebrar contratos existentes sem versionamento.

APIs:

```text
/api/v1
```

MQTT:

```text
v1/
```

Schemas:

```text
schema_version
```

---

# 93. Decisões consolidadas

O Domain Model v1.0 estabelece:

1. Organization como tenant principal.
2. User independente de Organization.
3. Membership para associação de usuários.
4. Site como instalação.
5. Area como subdivisão.
6. System como agrupamento funcional.
7. Asset como representação operacional.
8. Device como entidade de conectividade.
9. Gateway separado de Device.
10. Device Model como definição técnica.
11. Metric Definition como ontologia semântica.
12. Device Mapping como tradução fabricante → plataforma.
13. RAW separado da telemetria normalizada.
14. `observed_at` separado de `received_at`.
15. Event separado de Telemetry.
16. Alarm separado de Event.
17. Command tratado como entidade auditável.
18. Tenant isolation através de `organization_id`.
19. UUID como identificador externo.
20. UTC como padrão interno.
21. Dashboard desacoplado do hardware.
22. Edge desacoplado das regras de aplicação.
23. Idempotência por `message_id`.
24. TimescaleDB para séries temporais.
25. domínio preparado para Digital Twin, OTA e analytics futuros.

---

# 94. Critério principal do domínio

Antes da criação de qualquer nova tabela, campo ou relacionamento, a equipe deve responder:

> Esta informação pertence ao ativo, ao dispositivo, à telemetria, à configuração ou à aplicação?

Essa classificação deverá acontecer antes da implementação.

O objetivo é impedir que responsabilidades diferentes sejam misturadas.

---

# 95. Status do documento

**Documento:** Domain Model  
**Versão:** 1.0  
**Status:** Baseline inicial  
**Dependência:** Architecture Blueprint v1.0  
**Próximo documento:** Telemetry & Messaging Specification v1.0

Este documento deve ser considerado a referência principal para modelagem do banco e das APIs de domínio.