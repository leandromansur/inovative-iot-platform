# Telemetry & Messaging Specification v1.0

## 1. Objetivo

Este documento define o padrão oficial de comunicação de telemetria, eventos, status, comandos e respostas entre:

- dispositivos;
- gateways;
- Edge Agent;
- broker MQTT;
- APIs HTTP;
- IoT Core;
- serviços consumidores.

O objetivo é garantir:

- consistência;
- versionamento;
- idempotência;
- rastreabilidade;
- interoperabilidade;
- segurança;
- evolução futura sem quebra de compatibilidade.

---

# 2. Princípios

A comunicação da plataforma deve obedecer aos seguintes princípios:

1. Toda mensagem relevante deve possuir identificador único.
2. Toda mensagem deve possuir versão de schema.
3. Todo timestamp deve estar em UTC.
4. `observed_at` deve ser diferente de `received_at`.
5. Telemetria normalizada não deve carregar detalhes específicos do fabricante.
6. Payloads devem ser validados antes da persistência.
7. Retransmissão não pode gerar duplicidade.
8. MQTT será o canal principal para Edge ↔ Cloud.
9. HTTPS será suportado para ingestão e integração.
10. Comandos devem possuir confirmação explícita.
11. Eventos devem ser separados de telemetria.
12. Estado operacional deve ser separado de conectividade.
13. Todos os contratos devem ser versionados.

---

# 3. Tipos de mensagem

A plataforma define inicialmente:

```text
TELEMETRY
EVENT
STATUS
HEARTBEAT
COMMAND
COMMAND_RESULT
CONFIGURATION
CONFIGURATION_RESULT
```

Futuramente:

```text
FIRMWARE
OTA_STATUS
DIAGNOSTIC
LOG
```

---

# 4. Envelope padrão

Toda mensagem deve seguir um envelope comum.

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
  "payload": {}
}
```

Campos:

```text
schema_version
message_id
message_type
organization_id
gateway_id
device_id
observed_at
sequence
payload
```

---

# 5. Message ID

Formato:

```text
UUID
```

Função:

- idempotência;
- deduplicação;
- troubleshooting;
- correlação;
- auditoria.

A mesma mensagem retransmitida deve manter:

```text
message_id
```

Nunca gerar novo ID para uma retransmissão da mesma medição.

---

# 6. Sequence

Cada produtor poderá manter contador crescente.

Exemplo:

```text
1001
1002
1003
1004
```

Uso:

- detecção de perda;
- diagnóstico;
- ordenação;
- identificação de gaps.

A sequência não substitui `message_id`.

---

# 7. Timestamps

## observed_at

Momento da medição no Edge ou Device.

## received_at

Gerado pelo servidor no momento do recebimento.

Exemplo:

```text
observed_at
10:00

received_at
10:15
```

Isso indica atraso de 15 minutos.

---

# 8. Timestamp format

Padrão obrigatório:

```text
ISO 8601
UTC
```

Exemplo:

```text
2026-10-06T13:30:00.000Z
```

---

# 9. Telemetry Message

Formato:

```json
{
  "schema_version": "1.0",
  "message_id": "9ffb2df6-...",
  "message_type": "telemetry",
  "organization_id": "uuid",
  "gateway_id": "uuid",
  "device_id": "uuid",
  "observed_at": "2026-10-06T13:30:00.000Z",
  "sequence": 12345,
  "payload": {
    "metrics": {
      "electrical.voltage.line_l1_l2": 221.7,
      "electrical.current.l1": 5.3,
      "electrical.active_power.total": 3.2
    }
  }
}
```

---

# 10. Métricas

Formato:

```text
metric_key → value
```

Exemplo:

```json
{
  "pressure.discharge": 7.3
}
```

Não utilizar:

```json
{
  "register_40101": 73
}
```

nem:

```json
{
  "schneider_pressure": 7.3
}
```

O Device Adapter deve transformar dados do fabricante em métricas semânticas.

---

# 11. Tipos de valor

Permitidos:

```text
number
integer
boolean
string
enum
```

Exemplos:

```json
{
  "temperature.motor": 74.2,
  "machine.running": true,
  "machine.mode": "AUTO"
}
```

---

# 12. Qualidade por métrica

Opcionalmente uma métrica poderá possuir qualidade.

Exemplo:

```json
{
  "metrics": {
    "pressure.discharge": {
      "value": 7.3,
      "quality": "GOOD"
    }
  }
}
```

Qualidades:

```text
GOOD
UNCERTAIN
BAD
STALE
UNKNOWN
```

---

# 13. Forma compacta

Para reduzir tráfego, métricas com qualidade GOOD podem utilizar forma simples:

```json
{
  "pressure.discharge": 7.3
}
```

Forma expandida somente quando necessário:

```json
{
  "pressure.discharge": {
    "value": 7.3,
    "quality": "UNCERTAIN"
  }
}
```

---

# 14. Event Message

Formato:

```json
{
  "schema_version": "1.0",
  "message_id": "uuid",
  "message_type": "event",
  "organization_id": "uuid",
  "gateway_id": "uuid",
  "device_id": "uuid",
  "observed_at": "2026-10-06T13:32:00.000Z",
  "payload": {
    "event_type": "DEVICE_FAULT",
    "severity": "WARNING",
    "code": "E01",
    "message": "Falha detectada",
    "data": {}
  }
}
```

---

# 15. Eventos não são telemetria

Não representar evento desta forma:

```json
{
  "fault": 1
}
```

quando existe significado operacional.

Preferir:

```text
event_type = DEVICE_FAULT
```

Telemetria representa estado ou grandeza.

Evento representa ocorrência.

---

# 16. Status Message

Formato:

```json
{
  "schema_version": "1.0",
  "message_id": "uuid",
  "message_type": "status",
  "organization_id": "uuid",
  "gateway_id": "uuid",
  "device_id": "uuid",
  "observed_at": "2026-10-06T13:33:00.000Z",
  "payload": {
    "connectivity": "ONLINE",
    "operational_status": "RUNNING"
  }
}
```

---

# 17. Connectivity

Estados:

```text
ONLINE
DEGRADED
OFFLINE
UNKNOWN
DISABLED
```

---

# 18. Operational Status

Estados não devem ser universais para todo equipamento.

Preferir semântica por tipo de ativo/dispositivo.

Exemplo genérico:

```text
STOPPED
STARTING
RUNNING
STOPPING
FAULT
MAINTENANCE
UNKNOWN
```

---

# 19. Heartbeat

Heartbeat é diferente de telemetria.

Formato:

```json
{
  "schema_version": "1.0",
  "message_id": "uuid",
  "message_type": "heartbeat",
  "organization_id": "uuid",
  "gateway_id": "uuid",
  "observed_at": "2026-10-06T13:34:00.000Z",
  "payload": {
    "uptime_seconds": 86400,
    "software_version": "1.0.0"
  }
}
```

---

# 20. Heartbeat interval

Deve ser configurável.

Valor inicial recomendado:

```text
30 segundos
```

Pode variar conforme:

- rede;
- criticidade;
- tipo de equipamento;
- custo de comunicação.

---

# 21. Device Status Derivation

O servidor poderá calcular:

```text
ONLINE
DEGRADED
OFFLINE
```

a partir de:

```text
last heartbeat

last telemetry

expected interval
```

Exemplo:

```text
expected = 30 s

last_seen <= 60 s
ONLINE

last_seen <= 180 s
DEGRADED

last_seen > 180 s
OFFLINE
```

Esses valores devem ser configuráveis.

---

# 22. MQTT Namespace

Padrão:

```text
v1/{organization_id}/{device_id}/telemetry

v1/{organization_id}/{device_id}/event

v1/{organization_id}/{device_id}/status

v1/{organization_id}/{device_id}/command

v1/{organization_id}/{device_id}/command-result
```

---

# 23. Gateway Topics

```text
v1/{organization_id}/gateway/{gateway_id}/heartbeat

v1/{organization_id}/gateway/{gateway_id}/status

v1/{organization_id}/gateway/{gateway_id}/event
```

---

# 24. MQTT Topic Rules

Não colocar:

```text
cliente_nome
empresa_nome
cidade
fabricante
```

nos tópicos.

Utilizar IDs estáveis.

Errado:

```text
v1/hospital-rio/mmw04-01/telemetry
```

Correto:

```text
v1/{organization_uuid}/{device_uuid}/telemetry
```

---

# 25. MQTT QoS

Recomendação:

Telemetria:

```text
QoS 1
```

Eventos críticos:

```text
QoS 1
```

Comandos:

```text
QoS 1
```

Command Result:

```text
QoS 1
```

Heartbeat:

```text
QoS 0 ou QoS 1
```

dependendo do cenário.

Não utilizar QoS 2 inicialmente.

---

# 26. MQTT Retain

Regras:

```text
telemetry
retain = false

event
retain = false

command
retain = false
```

Status poderá utilizar:

```text
retain = true
```

quando fizer sentido.

---

# 27. Last Will and Testament

Gateways devem utilizar MQTT Last Will.

Exemplo:

```text
topic:
v1/{organization}/gateway/{gateway}/status
```

Payload:

```json
{
  "connectivity": "OFFLINE"
}
```

---

# 28. Clean Session

Preferência:

```text
persistent session
```

quando necessário para comandos e mensagens importantes.

Configuração exata deve ser validada no broker.

---

# 29. HTTPS Ingestion

Endpoint:

```text
POST /api/v1/ingestion/telemetry
```

Payload:

mesmo envelope utilizado no MQTT.

Objetivo:

não manter dois formatos diferentes.

---

# 30. HTTP Event Ingestion

```text
POST /api/v1/ingestion/events
```

---

# 31. Batch Telemetry

Suportar múltiplas medições em uma chamada.

Exemplo:

```json
{
  "schema_version": "1.0",
  "messages": [
    {
      "message_id": "uuid-1",
      "device_id": "uuid",
      "observed_at": "2026-10-06T13:30:00Z",
      "metrics": {
        "pressure.discharge": 7.2
      }
    },
    {
      "message_id": "uuid-2",
      "device_id": "uuid",
      "observed_at": "2026-10-06T13:31:00Z",
      "metrics": {
        "pressure.discharge": 7.3
      }
    }
  ]
}
```

---

# 32. Batch Size

O limite deve ser configurável.

Valor inicial sugerido:

```text
500 mensagens
```

Não permitir payloads indefinidamente grandes.

---

# 33. Payload Size

Definir limite inicial.

Sugestão:

```text
256 KB por mensagem MQTT
```

Payloads maiores devem ser rejeitados ou tratados por canal específico.

---

# 34. Store & Forward

Fluxo:

```text
Device
↓
Edge
↓
SQLite queue
↓
Publish
↓
Broker
↓
ACK
↓
Remove from local queue
```

---

# 35. Queue local

Tabela conceitual:

```text
outbox_message

id
message_id

topic
payload

created_at

attempt_count
last_attempt_at

status
```

---

# 36. Status de Outbox

```text
PENDING
SENT
ACKNOWLEDGED
FAILED
```

---

# 37. Retry Policy

Retry deve utilizar backoff.

Exemplo:

```text
1 s
2 s
5 s
10 s
30 s
60 s
```

Depois:

```text
intervalo configurável
```

---

# 38. Dead Letter

Mensagens permanentemente inválidas não devem bloquear fila.

Criar:

```text
dead_letter
```

Campos:

```text
message_id
payload
error
attempt_count
created_at
```

---

# 39. Ordenação

Mensagens podem chegar fora de ordem.

A plataforma nunca deve assumir:

```text
received_at == order
```

Ordenação histórica deve considerar:

```text
observed_at
```

---

# 40. Deduplicação

Ao receber:

```text
message_id já processado
```

o servidor deve:

```text
ignorar duplicata
```

sem retornar erro destrutivo.

Resposta deve indicar:

```text
already_processed
```

ou comportamento equivalente.

---

# 41. Device Authentication

Cada conexão MQTT deve possuir identidade própria.

Preferência:

```text
mTLS / X.509
```

Alternativa inicial:

```text
client_id
username
password/secret
```

---

# 42. MQTT Client ID

Formato recomendado:

```text
gateway:{gateway_uuid}
```

ou:

```text
device:{device_uuid}
```

---

# 43. ACL MQTT

Um Device não deve publicar fora do próprio namespace.

Exemplo:

Device A pode publicar:

```text
v1/org-a/device-a/#
```

Não pode publicar:

```text
v1/org-a/device-b/#
```

Muito menos:

```text
v1/org-b/#
```

---

# 44. Gateway ACL

Gateway poderá publicar para Devices explicitamente vinculados a ele.

ACL deve considerar:

```text
gateway
→ allowed devices
```

---

# 45. Cloud to Edge

Command Topic:

```text
v1/{organization}/{device}/command
```

Payload:

```json
{
  "schema_version": "1.0",
  "command_id": "uuid",
  "correlation_id": "uuid",
  "command_type": "SET_SPEED",
  "requested_at": "2026-10-06T13:40:00Z",
  "expires_at": "2026-10-06T13:41:00Z",
  "payload": {
    "value": 45.0,
    "unit": "Hz"
  }
}
```

---

# 46. Command Expiration

Todo comando deve possuir:

```text
expires_at
```

Um comando antigo não deve ser executado depois que a conexão retornar.

---

# 47. Command Result

```json
{
  "schema_version": "1.0",
  "message_type": "command_result",
  "command_id": "uuid",
  "correlation_id": "uuid",
  "device_id": "uuid",
  "observed_at": "2026-10-06T13:40:05Z",
  "status": "SUCCESS",
  "result": {},
  "error": null
}
```

---

# 48. Command States

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

---

# 49. Command Acknowledgement

Diferenciar:

```text
RECEIVED
```

de:

```text
SUCCESS
```

Receber o comando não significa executá-lo com sucesso.

---

# 50. Correlation ID

Utilizado para rastrear uma operação completa.

Exemplo:

```text
API request
↓
command
↓
MQTT
↓
Edge
↓
Device
↓
command result
```

Todos os registros devem compartilhar:

```text
correlation_id
```

---

# 51. Configuration Message

Cloud → Edge:

```json
{
  "schema_version": "1.0",
  "message_type": "configuration",
  "configuration_id": "uuid",
  "device_id": "uuid",
  "version": 12,
  "desired": {
    "sample_interval": 10
  }
}
```

---

# 52. Configuration Result

```json
{
  "schema_version": "1.0",
  "message_type": "configuration_result",
  "configuration_id": "uuid",
  "device_id": "uuid",
  "version": 12,
  "status": "APPLIED",
  "reported": {
    "sample_interval": 10
  }
}
```

---

# 53. Device Twin Preparation

Modelo:

```text
desired
reported
```

Não implementar sincronização completa no primeiro MVP, mas preservar compatibilidade.

---

# 54. Schema Validation

Todos os payloads devem possuir schema formal.

Recomendação:

```text
JSON Schema
```

Uso:

- Edge;
- backend;
- testes;
- documentação;
- SDKs.

---

# 55. Contract Package

Estrutura:

```text
packages/contracts/

schemas/
├── telemetry.schema.json
├── event.schema.json
├── status.schema.json
├── command.schema.json
└── command-result.schema.json
```

---

# 56. Type Generation

Os tipos TypeScript devem ser derivados dos schemas sempre que possível.

Objetivo:

evitar:

```text
JSON Schema ≠ TypeScript Interface
```

---

# 57. Versionamento

Contrato:

```text
schema_version
```

Exemplo:

```text
1.0
```

Breaking change:

```text
2.0
```

Mudança compatível:

```text
1.1
```

---

# 58. Compatibilidade

Novos campos opcionais:

```text
compatível
```

Remoção ou mudança semântica de campo:

```text
breaking change
```

---

# 59. MQTT Version Namespace

Além de `schema_version`, o tópico possui:

```text
v1/
```

Isso permite evolução de protocolo independente da aplicação.

---

# 60. Ingestion Pipeline

Fluxo recomendado:

```text
MQTT / HTTP
    ↓
Ingress
    ↓
Authentication
    ↓
Authorization
    ↓
Schema Validation
    ↓
Deduplication
    ↓
Raw Persistence
    ↓
Semantic Validation
    ↓
Telemetry Persistence
    ↓
Internal Event
    ↓
Consumers
```

---

# 61. Semantic Validation

Schema válido não significa dado válido.

Exemplo:

```text
frequency = 50000 Hz
```

pode ser JSON válido, mas provavelmente inválido semanticamente.

Validar:

```text
minimum
maximum
datatype
unit
allowed enum
```

---

# 62. Raw Persistence

Payload recebido deve ser preservado antes ou durante processamento.

Campos:

```text
message_id
raw_payload
received_at
processing_status
```

---

# 63. Internal Event Bus

Após persistência:

```text
TelemetryReceived
```

poderá ser publicado internamente.

Consumidores:

```text
Alarm Engine
WebSocket
Analytics
Integrations
Aggregation
```

---

# 64. Evitar acoplamento

Ingestion Service não deve:

```text
enviar WhatsApp
renderizar dashboard
calcular relatório
executar lógica de manutenção
```

Ele apenas:

```text
recebe
valida
persiste
publica evento interno
```

---

# 65. Alarm Processing

Fluxo:

```text
TelemetryStored
↓
Alarm Engine
↓
Rule Evaluation
↓
Alarm State Change
↓
Alarm Event
↓
Notification
```

---

# 66. Realtime Dashboard

Fluxo:

```text
TelemetryStored
↓
Realtime Publisher
↓
WebSocket
↓
Frontend
```

Frontend não deve consumir MQTT diretamente na primeira versão.

---

# 67. WebSocket Channels

Exemplos:

```text
organization:{id}

site:{id}

asset:{id}

device:{id}
```

---

# 68. WebSocket Events

Inicialmente:

```text
telemetry.updated

device.status_changed

alarm.opened

alarm.acknowledged

alarm.cleared

command.updated
```

---

# 69. Histórico

Consulta histórica deve ser realizada via REST.

Exemplo:

```text
GET /api/v1/devices/{id}/telemetry
```

Filtros:

```text
metrics
from
to
resolution
aggregation
```

---

# 70. Resolution

Exemplos:

```text
raw
1m
5m
15m
1h
1d
```

---

# 71. Aggregation

Exemplos:

```text
avg
min
max
sum
first
last
```

A API não deve permitir agregações incoerentes com o tipo da métrica.

---

# 72. Gauge

Exemplo:

```text
voltage
temperature
pressure
```

Agregações:

```text
min
max
avg
first
last
```

---

# 73. Counter

Exemplo:

```text
energy.imported
runtime.total
```

Agregações preferenciais:

```text
first
last
delta
increase
```

Não utilizar média como valor principal.

---

# 74. State

Exemplo:

```text
machine.running
machine.mode
```

Agregações:

```text
first
last
duration_by_state
```

---

# 75. Sampling

Dois conceitos separados:

```text
poll_interval
```

e:

```text
publish_interval
```

Exemplo:

```text
poll = 1 s
publish = 10 s
```

---

# 76. Edge Aggregation

Permitido futuramente.

Exemplo:

```text
1-second samples
↓
10-second average
↓
cloud
```

Somente se configurado.

O Edge não deve alterar semanticamente o dado sem registrar isso.

---

# 77. Deadband

Suporte futuro:

```text
publish only if change > X
```

Exemplo:

```text
temperature deadband = 0.5 °C
```

---

# 78. Critical Metrics

Algumas métricas não devem utilizar deadband.

Exemplo:

```text
fault state
emergency stop
protection trip
```

---

# 79. Compression

Não é obrigatória no protocolo v1.

Pode ser adotada posteriormente em conexões de baixa largura de banda.

---

# 80. Binary Protocol

Não utilizar protocolo binário próprio inicialmente.

Padrão inicial:

```text
JSON
```

Motivos:

- debug;
- interoperabilidade;
- desenvolvimento;
- documentação.

Futuramente poderá ser avaliado:

```text
Protobuf
```

---

# 81. Protocol Adapters

Responsabilidade:

```text
industrial protocol
        ↓
raw device values
```

Exemplo:

```text
Modbus register
→ integer
```

---

# 82. Device Adapter

Responsabilidade:

```text
raw device value
        ↓
semantic metric
```

Exemplo:

```text
register 40101 = 2217
scale 0.1
↓
electrical.voltage.line_l1_l2 = 221.7
```

---

# 83. Separação obrigatória

Protocol Adapter não deve saber:

```text
qual dashboard existe
```

Device Adapter não deve saber:

```text
qual cliente possui o equipamento
```

Alarm Engine não deve saber:

```text
qual protocolo originou a métrica
```

---

# 84. Error Model

Formato padrão:

```json
{
  "code": "INVALID_SCHEMA",
  "message": "Payload validation failed",
  "details": {},
  "correlation_id": "uuid"
}
```

---

# 85. Error Codes iniciais

```text
INVALID_SCHEMA

INVALID_METRIC

UNKNOWN_DEVICE

UNAUTHORIZED_DEVICE

TENANT_MISMATCH

DUPLICATE_MESSAGE

INVALID_TIMESTAMP

PAYLOAD_TOO_LARGE

RATE_LIMITED

INTERNAL_ERROR
```

---

# 86. Unknown Metric

Se o Edge enviar métrica não cadastrada:

```text
reject metric
```

ou:

```text
quarantine
```

Nunca criar Metric Definition automaticamente em produção.

---

# 87. Unknown Device

Device desconhecido:

```text
reject
```

Não efetuar autocadastro silencioso.

---

# 88. Clock Drift

A plataforma deve verificar diferença entre:

```text
observed_at
received_at
```

Se exceder limite:

```text
mark quality = UNCERTAIN
```

ou gerar evento.

---

# 89. Future Timestamp

Medição muito no futuro:

```text
reject
```

ou:

```text
quarantine
```

Limite deve ser configurável.

---

# 90. Late Data

Dados atrasados são permitidos.

Exemplo:

```text
observed_at = 10:00
received_at = 14:00
```

Desde que válidos.

---

# 91. Telemetry Retention

Definir políticas separadas:

```text
raw messages

raw telemetry

aggregated telemetry
```

Valores concretos serão definidos no documento de operação/capacidade.

---

# 92. Backpressure

Caso o backend esteja sobrecarregado:

```text
broker
↓
queue
↓
consumer
```

deve absorver picos.

Não descartar telemetria silenciosamente.

---

# 93. Rate Limiting

Aplicar limites por:

```text
device
gateway
organization
API client
```

Objetivo:

- proteção;
- estabilidade;
- prevenção de configuração incorreta.

---

# 94. Regras de segurança

Nunca aceitar `organization_id` como verdade apenas porque veio no payload.

A plataforma deve validar:

```text
credential
↓
device
↓
organization
```

e verificar coerência.

---

# 95. Trust Model

Exemplo:

```text
credential identifies device A

device A belongs to organization X

payload claims organization Y
```

Resultado:

```text
reject
+
security event
```

---

# 96. Command Security

Antes de publicar comando:

```text
User authenticated
↓
Tenant authorized
↓
Role authorized
↓
Asset/Device access authorized
↓
Command allowed
↓
Safety checks
↓
Audit record
↓
Publish
```

---

# 97. Edge Command Validation

O Edge também deve validar:

```text
command type

expiration

target device

configuration

local safety conditions
```

Nunca confiar cegamente na Cloud.

---

# 98. Offline Command Policy

Por padrão:

```text
não executar comando expirado
```

Fila de comandos deve respeitar `expires_at`.

---

# 99. Safety-Critical Control

A plataforma não deve inicialmente assumir responsabilidade por funções de segurança funcional.

Intertravamentos críticos permanecem em:

```text
PLC
relay
drive
safety controller
```

Cloud/Edge não substituem sistemas certificados de segurança.

---

# 100. Observabilidade

Cada mensagem deve ser rastreável através de:

```text
message_id
correlation_id
device_id
gateway_id
organization_id
```

---

# 101. Logging

Logs não devem incluir:

```text
password
secret
private key
token
```

---

# 102. Metrics operacionais

Monitorar:

```text
messages_received_total

messages_rejected_total

duplicate_messages_total

processing_latency

ingestion_rate

queue_depth

device_online_count

device_offline_count

command_success_rate

command_timeout_rate
```

---

# 103. Tracing

Fluxos críticos devem possuir tracing distribuído.

Exemplo:

```text
HTTP Request
↓
Command Service
↓
MQTT
↓
Edge
↓
Result
```

---

# 104. Testes obrigatórios

Telemetria:

```text
valid payload
invalid payload
duplicate payload
late payload
unknown metric
unknown device
tenant mismatch
```

---

# 105. Testes de Store & Forward

Cenários:

```text
internet online

internet offline

restart do Edge

retorno da internet

duplicação

fila parcialmente enviada

mensagem inválida
```

---

# 106. Testes de comando

Cenários:

```text
success

device offline

timeout

command expired

unauthorized user

invalid payload

duplicate result
```

---

# 107. Contratos canônicos v1.0

A plataforma considera canônicos:

```text
TelemetryEnvelope

EventEnvelope

StatusEnvelope

HeartbeatEnvelope

CommandEnvelope

CommandResultEnvelope

ConfigurationEnvelope

ConfigurationResultEnvelope
```

---

# 108. Estrutura no repositório

```text
packages/contracts/
│
├── telemetry/
│   ├── telemetry.schema.json
│   └── telemetry.types.ts
│
├── events/
│   ├── event.schema.json
│   └── event.types.ts
│
├── status/
│
├── commands/
│
└── configuration/
```

---

# 109. Edge SDK

Criar futuramente:

```text
packages/device-sdk
```

Responsabilidades:

- geração de envelope;
- UUID;
- timestamp;
- sequence;
- publish;
- retry;
- buffering;
- validation.

---

# 110. Protocol SDK

Criar:

```text
packages/protocol-sdk
```

Para padronizar:

```text
connect
read
write
health
disconnect
```

---

# 111. Adapter Interface

Exemplo conceitual:

```text
read()
normalize()
write()
health()
```

Não acoplar diretamente ao MQTT.

---

# 112. Fluxo completo de telemetria

```text
Physical Device
      ↓
Industrial Protocol
      ↓
Protocol Adapter
      ↓
Device Adapter
      ↓
Semantic Metric
      ↓
Telemetry Envelope
      ↓
Edge Outbox
      ↓
MQTT
      ↓
EMQX
      ↓
Ingestion
      ↓
Authentication
      ↓
Schema Validation
      ↓
Tenant Validation
      ↓
Deduplication
      ↓
Raw Storage
      ↓
Telemetry Storage
      ↓
Internal Event Bus
      ↓
┌─────────────┬─────────────┬──────────────┐
│             │             │              │
Alarm      Realtime      Analytics    Integration
Engine      Gateway
```

---

# 113. Fluxo completo de comando

```text
User
 ↓
Frontend
 ↓
API
 ↓
Authorization
 ↓
Command Service
 ↓
Audit
 ↓
MQTT
 ↓
Gateway
 ↓
Device
 ↓
Execution
 ↓
Command Result
 ↓
MQTT
 ↓
Command Service
 ↓
Database
 ↓
WebSocket
 ↓
Frontend
```

---

# 114. Decisões consolidadas

A Telemetry & Messaging Specification v1.0 estabelece:

1. MQTT como protocolo principal Edge ↔ Cloud.
2. HTTPS como canal alternativo.
3. JSON como formato inicial.
4. Envelope padrão para mensagens.
5. UUID para `message_id`.
6. `schema_version` obrigatório.
7. `observed_at` em UTC.
8. `received_at` gerado pelo servidor.
9. QoS 1 como padrão de telemetria e comandos.
10. Store & Forward obrigatório no Edge.
11. SQLite como fila persistente inicial.
12. deduplicação por `message_id`.
13. sequence para diagnóstico e gaps.
14. RAW separado de telemetria normalizada.
15. eventos separados de métricas.
16. status separado de telemetria.
17. heartbeat explícito.
18. comandos com `expires_at`.
19. confirmação de comando obrigatória.
20. schema validation obrigatória.
21. semantic validation obrigatória.
22. ACL por Device/Gateway.
23. tenant validado pelas credenciais.
24. WebSocket apenas para realtime frontend.
25. REST para histórico.
26. compatibilidade preparada para Device Twin.
27. adapter desacoplado da aplicação.
28. Edge não substitui safety controller.
29. observabilidade baseada em IDs correlacionáveis.
30. contratos versionados em pacote compartilhado.

---

# 115. Status

**Documento:** Telemetry & Messaging Specification  
**Versão:** 1.0  
**Status:** Baseline inicial  
**Dependências:** Architecture Blueprint v1.0, Domain Model v1.0  
**Próximo documento:** Security Architecture v1.0