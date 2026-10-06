# API & Edge Contracts v1.0

## 1. Objetivo

Este documento define os contratos oficiais de integração da plataforma Industrial IoT.

Abrange:

- API REST;
- autenticação;
- autorização;
- paginação;
- filtros;
- respostas;
- erros;
- versionamento;
- provisioning;
- Edge Agent;
- gateways;
- devices;
- telemetria;
- eventos;
- heartbeat;
- configuração;
- comandos;
- Device Twin;
- interfaces de adapters;
- contratos internos compartilhados.

O objetivo é permitir desenvolvimento independente e previsível entre:

```text
Frontend
Backend
Workers
Edge
Adapters
Integrações externas
```

---

# 2. Princípios

Todos os contratos devem seguir:

1. API versionada.
2. Schemas explícitos.
3. IDs estáveis.
4. UTC em timestamps.
5. JSON como formato principal.
6. Erros padronizados.
7. idempotência onde necessário.
8. compatibilidade retroativa.
9. validação de entrada.
10. autorização explícita.
11. tenant derivado da identidade.
12. nenhum detalhe de banco exposto.
13. Edge e Cloud desacoplados.
14. adapters independentes do transporte.

---

# 3. Base URL

Produção:

```text
https://api.example.com/api/v1
```

Ambientes:

```text
https://api-dev.example.com/api/v1
https://api-staging.example.com/api/v1
https://api.example.com/api/v1
```

O domínio definitivo será definido posteriormente.

---

# 4. Versionamento

Versão principal:

```text
/api/v1
```

Breaking changes:

```text
/api/v2
```

Alterações compatíveis permanecem dentro de:

```text
/api/v1
```

---

# 5. Content Type

Padrão:

```http
Content-Type: application/json
Accept: application/json
```

UTF-8 obrigatório.

---

# 6. Identificadores

IDs expostos pela API:

```text
UUID
```

Exemplo:

```text
550e8400-e29b-41d4-a716-446655440000
```

Não expor IDs sequenciais internos.

---

# 7. Timestamps

Formato:

```text
ISO 8601
UTC
```

Exemplo:

```text
2026-10-06T15:30:00.000Z
```

Campos:

```text
created_at
updated_at
observed_at
received_at
last_seen_at
```

---

# 8. Naming Convention

JSON:

```text
snake_case
```

Exemplo:

```json
{
  "organization_id": "uuid",
  "last_seen_at": "2026-10-06T15:30:00Z"
}
```

Internamente TypeScript poderá utilizar:

```text
camelCase
```

A conversão deve ocorrer na camada apropriada.

---

# 9. Envelope de sucesso

Objeto único:

```json
{
  "data": {
    "id": "uuid"
  }
}
```

Coleções:

```json
{
  "data": [],
  "meta": {}
}
```

---

# 10. Envelope de erro

Formato obrigatório:

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

# 11. Correlation ID

Toda requisição deve possuir:

```text
correlation_id
```

O servidor deve gerar um quando ausente.

Header recomendado:

```http
X-Correlation-ID
```

Resposta deve retornar o mesmo valor.

---

# 12. Request ID

Opcionalmente:

```http
X-Request-ID
```

Pode ser separado de:

```text
correlation_id
```

para tracing interno.

---

# 13. Status HTTP

Uso padrão:

```text
200 OK
201 Created
202 Accepted
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
429 Too Many Requests

500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

---

# 14. 400 vs 422

Usar:

```text
400
```

para requisição estruturalmente inválida.

Usar:

```text
422
```

para regra de domínio inválida.

Exemplo:

```text
speed = 1000 Hz
```

pode ser JSON válido, mas domínio inválido.

---

# 15. 401 vs 403

```text
401
```

usuário não autenticado ou credencial inválida.

```text
403
```

usuário autenticado sem permissão.

---

# 16. 404 e Tenant Isolation

Para determinados recursos cross-tenant, a API poderá responder:

```text
404
```

em vez de:

```text
403
```

para evitar revelar existência de recurso de outro tenant.

---

# 17. Paginação

Padrão recomendado:

```text
cursor-based pagination
```

Query:

```text
?limit=50&cursor=abc
```

Resposta:

```json
{
  "data": [],
  "meta": {
    "next_cursor": "abc",
    "has_more": true
  }
}
```

---

# 18. Limites

Default:

```text
50
```

Máximo inicial:

```text
200
```

Configurável.

---

# 19. Ordenação

Formato:

```text
?sort=created_at
```

Descendente:

```text
?sort=-created_at
```

---

# 20. Filtros

Formato:

```text
?status=ACTIVE
```

Múltiplos:

```text
?status=ACTIVE&site_id=uuid
```

Filtros complexos devem ser explicitamente definidos por endpoint.

---

# 21. Busca textual

Formato:

```text
?q=compressor
```

Busca não deve permitir query arbitrária de banco.

---

# 22. Seleção de campos

Não implementar inicialmente GraphQL-like field selection.

Retornar DTOs estáveis.

---

# 23. Expansão

Quando necessário:

```text
?include=devices
```

Usar com parcimônia.

Evitar payloads excessivamente aninhados.

---

# 24. Autenticação de usuário

Endpoint:

```text
POST /api/v1/auth/login
```

Request:

```json
{
  "email": "user@example.com",
  "password": "secret"
}
```

---

# 25. Login Response

```json
{
  "data": {
    "user": {
      "id": "uuid",
      "name": "User",
      "email": "user@example.com"
    },
    "mfa_required": false
  }
}
```

Tokens preferencialmente entregues via:

```text
secure cookies
```

quando interface web própria.

---

# 26. MFA Challenge

```text
POST /api/v1/auth/mfa/verify
```

Request:

```json
{
  "challenge_id": "uuid",
  "code": "123456"
}
```

---

# 27. Current User

```text
GET /api/v1/auth/me
```

Resposta:

```json
{
  "data": {
    "id": "uuid",
    "name": "User",
    "email": "user@example.com",
    "memberships": []
  }
}
```

---

# 28. Logout

```text
POST /api/v1/auth/logout
```

---

# 29. Sessions

```text
GET /api/v1/auth/sessions
DELETE /api/v1/auth/sessions/{id}
```

---

# 30. Organizations

```text
GET    /api/v1/organizations
POST   /api/v1/organizations
GET    /api/v1/organizations/{id}
PATCH  /api/v1/organizations/{id}
```

Exclusão física não deve ser exposta inicialmente.

---

# 31. Organization DTO

```json
{
  "id": "uuid",
  "name": "Empresa ABC",
  "slug": "empresa-abc",
  "status": "ACTIVE",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

# 32. Memberships

```text
GET    /api/v1/organizations/{id}/members
POST   /api/v1/organizations/{id}/members
PATCH  /api/v1/organizations/{id}/members/{member_id}
DELETE /api/v1/organizations/{id}/members/{member_id}
```

---

# 33. Sites

```text
GET    /api/v1/sites
POST   /api/v1/sites
GET    /api/v1/sites/{id}
PATCH  /api/v1/sites/{id}
```

Filtros:

```text
organization_id
status
q
```

---

# 34. Areas

```text
GET    /api/v1/areas
POST   /api/v1/areas
GET    /api/v1/areas/{id}
PATCH  /api/v1/areas/{id}
```

---

# 35. Systems

```text
GET    /api/v1/systems
POST   /api/v1/systems
GET    /api/v1/systems/{id}
PATCH  /api/v1/systems/{id}
```

---

# 36. Asset Types

```text
GET  /api/v1/asset-types
POST /api/v1/asset-types
```

Criação pode ser restrita a:

```text
PLATFORM_ADMIN
```

ou administradores específicos.

---

# 37. Assets

```text
GET    /api/v1/assets
POST   /api/v1/assets
GET    /api/v1/assets/{id}
PATCH  /api/v1/assets/{id}
```

---

# 38. Asset DTO

```json
{
  "id": "uuid",
  "organization_id": "uuid",
  "site_id": "uuid",
  "area_id": "uuid",
  "system_id": "uuid",
  "parent_asset_id": null,
  "asset_type": {
    "id": "uuid",
    "key": "compressor",
    "name": "Compressor"
  },
  "name": "Compressor 01",
  "manufacturer": "Example",
  "model": "X100",
  "serial_number": "ABC123",
  "status": "ACTIVE",
  "metadata": {},
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

# 39. Asset Tree

Endpoint específico:

```text
GET /api/v1/sites/{site_id}/asset-tree
```

Resposta otimizada para hierarquia.

Não usar consulta recursiva genérica em todos os endpoints.

---

# 40. Gateways

```text
GET    /api/v1/gateways
POST   /api/v1/gateways
GET    /api/v1/gateways/{id}
PATCH  /api/v1/gateways/{id}
```

---

# 41. Gateway DTO

```json
{
  "id": "uuid",
  "organization_id": "uuid",
  "site_id": "uuid",
  "name": "Gateway 01",
  "serial_number": "GW123",
  "hardware_model": "IPC-01",
  "software_version": "1.0.0",
  "status": "ONLINE",
  "last_seen_at": "timestamp"
}
```

---

# 42. Device Models

```text
GET    /api/v1/device-models
POST   /api/v1/device-models
GET    /api/v1/device-models/{id}
PATCH  /api/v1/device-models/{id}
```

---

# 43. Devices

```text
GET    /api/v1/devices
POST   /api/v1/devices
GET    /api/v1/devices/{id}
PATCH  /api/v1/devices/{id}
```

---

# 44. Device DTO

```json
{
  "id": "uuid",
  "organization_id": "uuid",
  "site_id": "uuid",
  "asset_id": "uuid",
  "gateway_id": "uuid",
  "device_model_id": "uuid",
  "name": "Medidor Principal",
  "serial_number": "123456",
  "external_id": "meter-01",
  "protocol": "MODBUS_TCP",
  "connectivity_status": "ONLINE",
  "operational_status": "RUNNING",
  "firmware_version": "1.2.0",
  "last_seen_at": "timestamp"
}
```

---

# 45. Device Capabilities

Endpoint:

```text
GET /api/v1/devices/{id}/capabilities
```

Resposta:

```json
{
  "data": {
    "metrics": [],
    "commands": [],
    "configuration": {}
  }
}
```

---

# 46. Metric Definitions

```text
GET    /api/v1/metrics
POST   /api/v1/metrics
GET    /api/v1/metrics/{id}
PATCH  /api/v1/metrics/{id}
```

---

# 47. Metric DTO

```json
{
  "id": "uuid",
  "key": "pressure.discharge",
  "name": "Pressão de descarga",
  "quantity": "pressure",
  "canonical_unit": "bar",
  "datatype": "FLOAT",
  "aggregation_type": "GAUGE"
}
```

---

# 48. Device Metric Mappings

```text
GET  /api/v1/device-models/{id}/metric-mappings
POST /api/v1/device-models/{id}/metric-mappings
```

---

# 49. Telemetry Ingestion

HTTP:

```text
POST /api/v1/ingestion/telemetry
```

Autenticação:

```text
Device Credential
Gateway Credential
```

Não aceitar sessão de usuário comum como credencial padrão para ingestão.

---

# 50. Telemetry Request

```json
{
  "schema_version": "1.0",
  "message_id": "uuid",
  "message_type": "telemetry",
  "organization_id": "uuid",
  "gateway_id": "uuid",
  "device_id": "uuid",
  "observed_at": "timestamp",
  "sequence": 123,
  "payload": {
    "metrics": {
      "pressure.discharge": 7.3
    }
  }
}
```

---

# 51. Telemetry Response

Novo:

```json
{
  "data": {
    "message_id": "uuid",
    "status": "PROCESSED"
  }
}
```

Duplicado:

```json
{
  "data": {
    "message_id": "uuid",
    "status": "ALREADY_PROCESSED"
  }
}
```

---

# 52. Batch Ingestion

```text
POST /api/v1/ingestion/telemetry/batch
```

Resposta deve permitir status individual.

```json
{
  "data": [
    {
      "message_id": "uuid-1",
      "status": "PROCESSED"
    },
    {
      "message_id": "uuid-2",
      "status": "REJECTED",
      "error_code": "INVALID_METRIC"
    }
  ]
}
```

---

# 53. Event Ingestion

```text
POST /api/v1/ingestion/events
```

Mesmo modelo de autenticação.

---

# 54. Heartbeat HTTP

Fallback:

```text
POST /api/v1/ingestion/heartbeat
```

MQTT continua preferencial.

---

# 55. Historical Telemetry

```text
GET /api/v1/devices/{id}/telemetry
```

Query:

```text
metrics
from
to
resolution
aggregation
limit
cursor
```

Exemplo:

```text
?metrics=pressure.discharge,temperature.motor
&from=...
&to=...
&resolution=5m
&aggregation=avg
```

---

# 56. Telemetry Response

```json
{
  "data": {
    "device_id": "uuid",
    "series": [
      {
        "metric": "pressure.discharge",
        "unit": "bar",
        "points": [
          {
            "timestamp": "timestamp",
            "value": 7.3
          }
        ]
      }
    ]
  }
}
```

---

# 57. Current Telemetry

Endpoint otimizado:

```text
GET /api/v1/devices/{id}/telemetry/current
```

Resposta:

```json
{
  "data": {
    "pressure.discharge": {
      "value": 7.3,
      "unit": "bar",
      "observed_at": "timestamp",
      "quality": "GOOD"
    }
  }
}
```

---

# 58. Asset Telemetry

```text
GET /api/v1/assets/{id}/telemetry/current
```

Agrega devices vinculados ao Asset.

Deve resolver conflitos semanticamente.

---

# 59. Events

```text
GET /api/v1/events
GET /api/v1/events/{id}
```

Filtros:

```text
site_id
asset_id
device_id
event_type
severity
from
to
```

---

# 60. Alarms

```text
GET    /api/v1/alarms
GET    /api/v1/alarms/{id}
POST   /api/v1/alarms/{id}/acknowledge
```

---

# 61. Acknowledge Alarm

Request:

```json
{
  "comment": "Verificado em campo"
}
```

---

# 62. Alarm Rules

```text
GET    /api/v1/alarm-rules
POST   /api/v1/alarm-rules
GET    /api/v1/alarm-rules/{id}
PATCH  /api/v1/alarm-rules/{id}
```

---

# 63. Commands

Criar comando:

```text
POST /api/v1/devices/{id}/commands
```

---

# 64. Command Request

```json
{
  "command_type": "SET_SPEED",
  "payload": {
    "value": 45,
    "unit": "Hz"
  }
}
```

O frontend não define:

```text
organization_id
gateway_id
command_id
criticality
```

Esses campos são resolvidos pelo backend.

---

# 65. Command Response

```json
{
  "data": {
    "id": "uuid",
    "device_id": "uuid",
    "command_type": "SET_SPEED",
    "status": "QUEUED",
    "requested_at": "timestamp",
    "expires_at": "timestamp"
  }
}
```

---

# 66. Command List

```text
GET /api/v1/devices/{id}/commands
GET /api/v1/commands/{id}
```

---

# 67. Command Cancellation

Se permitido:

```text
POST /api/v1/commands/{id}/cancel
```

Somente para:

```text
CREATED
QUEUED
```

Não garantir cancelamento depois de `SENT`.

---

# 68. Command Result

Edge envia preferencialmente via MQTT.

Fallback HTTP:

```text
POST /api/v1/ingestion/command-results
```

---

# 69. Command Idempotency

Criação de comandos deverá aceitar:

```http
Idempotency-Key
```

para evitar comando duplicado por retry do frontend.

---

# 70. Idempotency-Key

Exemplo:

```http
Idempotency-Key: 5dca...
```

Mesmo usuário + endpoint + key:

```text
mesma operação
```

durante janela configurada.

---

# 71. Dashboard Templates

```text
GET    /api/v1/dashboard-templates
POST   /api/v1/dashboard-templates
GET    /api/v1/dashboard-templates/{id}
PATCH  /api/v1/dashboard-templates/{id}
```

---

# 72. Dashboard Instances

```text
GET    /api/v1/dashboards
POST   /api/v1/dashboards
GET    /api/v1/dashboards/{id}
PATCH  /api/v1/dashboards/{id}
```

---

# 73. Notifications

```text
GET    /api/v1/notification-channels
POST   /api/v1/notification-channels

GET    /api/v1/notification-rules
POST   /api/v1/notification-rules
```

---

# 74. Integrations

```text
GET    /api/v1/integrations
POST   /api/v1/integrations
GET    /api/v1/integrations/{id}
PATCH  /api/v1/integrations/{id}
```

---

# 75. API Credentials

```text
POST   /api/v1/api-credentials
GET    /api/v1/api-credentials
DELETE /api/v1/api-credentials/{id}
```

Secret só deve ser exibido:

```text
uma única vez
```

na criação.

---

# 76. Audit Logs

```text
GET /api/v1/audit-logs
```

Somente leitura.

Filtros:

```text
actor_id
action
resource_type
resource_id
from
to
```

---

# 77. Edge Provisioning

Fluxo inicial:

```text
Admin
↓
Create Gateway
↓
Generate Activation Token
↓
Edge starts
↓
Provisioning API
↓
Credential issued
↓
Activation Token invalidated
```

---

# 78. Generate Activation Token

```text
POST /api/v1/gateways/{id}/activation-token
```

Response:

```json
{
  "data": {
    "activation_token": "secret",
    "expires_at": "timestamp"
  }
}
```

Token exibido uma única vez.

---

# 79. Provision Endpoint

Edge:

```text
POST /api/v1/edge/provision
```

Request:

```json
{
  "gateway_id": "uuid",
  "activation_token": "secret",
  "hardware": {
    "model": "IPC-X",
    "serial_number": "ABC123"
  },
  "software_version": "1.0.0"
}
```

---

# 80. Provision Response

Modelo mTLS:

```json
{
  "data": {
    "gateway_id": "uuid",
    "certificate": "PEM",
    "certificate_chain": "PEM",
    "mqtt": {
      "host": "mqtt.example.com",
      "port": 8883
    }
  }
}
```

Chave privada idealmente deve ser gerada no próprio Edge.

Preferência futura:

```text
CSR
```

em vez de transferência de private key.

---

# 81. CSR Provisioning

Modelo preferido:

```text
Edge generates private key
↓
Edge sends CSR
↓
Platform signs CSR
↓
Certificate returned
```

A private key:

```text
never leaves Edge
```

---

# 82. Edge Registration

Após provisionamento:

```text
POST /api/v1/edge/register
```

ou processo equivalente implícito.

Campos:

```text
software_version
capabilities
hardware_info
```

---

# 83. Edge Bootstrap Config

Endpoint:

```text
GET /api/v1/edge/configuration
```

Autenticação de Gateway.

Resposta:

```json
{
  "data": {
    "version": 12,
    "gateway": {},
    "devices": [],
    "polling": {},
    "mqtt": {}
  }
}
```

---

# 84. Configuration Version

Toda configuração recebida pelo Edge deve possuir:

```text
version
```

Exemplo:

```text
12
```

Edge reporta:

```text
reported_version
```

---

# 85. Edge Configuration Poll

Mesmo usando MQTT, deve existir mecanismo de recuperação via API.

```text
GET /api/v1/edge/configuration
```

permite sincronização após restart ou perda de mensagem.

---

# 86. Configuration Push

Via MQTT:

```text
v1/{organization}/gateway/{gateway}/configuration
```

Payload:

```json
{
  "schema_version": "1.0",
  "configuration_id": "uuid",
  "version": 13,
  "desired": {}
}
```

---

# 87. Configuration Result

```text
v1/{organization}/gateway/{gateway}/configuration-result
```

---

# 88. Edge Health

```text
GET /api/v1/gateways/{id}/health
```

Resposta possível:

```json
{
  "data": {
    "connectivity": "ONLINE",
    "last_seen_at": "timestamp",
    "software_version": "1.0.0",
    "queue_depth": 42,
    "cpu_percent": 18.5,
    "memory_percent": 37.2,
    "disk_percent": 21.3
  }
}
```

---

# 89. Edge Heartbeat Payload

```json
{
  "schema_version": "1.0",
  "message_id": "uuid",
  "message_type": "heartbeat",
  "gateway_id": "uuid",
  "observed_at": "timestamp",
  "payload": {
    "uptime_seconds": 12345,
    "software_version": "1.0.0",
    "queue_depth": 10,
    "system": {
      "cpu_percent": 10,
      "memory_percent": 20,
      "disk_percent": 30
    }
  }
}
```

---

# 90. Device Discovery

O Edge poderá futuramente reportar dispositivos descobertos.

Endpoint:

```text
POST /api/v1/edge/discovery
```

Importante:

```text
discovered ≠ registered
```

Nenhum Device deve entrar automaticamente em produção sem aprovação.

---

# 91. Discovery DTO

```json
{
  "devices": [
    {
      "protocol": "MODBUS_TCP",
      "address": "192.168.1.10",
      "metadata": {}
    }
  ]
}
```

---

# 92. Adapter Architecture

Existem dois tipos principais:

```text
Protocol Adapter
Device Adapter
```

---

# 93. Protocol Adapter Contract

Interface conceitual:

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

# 94. Protocol Adapter Responsibilities

Pode:

```text
connect
disconnect
read
write
retry protocol operation
report protocol health
```

Não pode:

```text
publish MQTT

persist telemetry

evaluate alarm

know tenant permissions
```

---

# 95. RawReadResult

Conceito:

```json
{
  "success": true,
  "observed_at": "timestamp",
  "values": {
    "40101": 2217,
    "40102": 2205
  }
}
```

---

# 96. Device Adapter Contract

Interface conceitual:

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

# 97. Normalized Metric

```json
{
  "metric_key": "electrical.voltage.line_l1_l2",
  "value": 221.7,
  "quality": "GOOD",
  "observed_at": "timestamp"
}
```

---

# 98. Device Capabilities

```json
{
  "metrics": [
    "electrical.voltage.line_l1_l2"
  ],
  "commands": [
    "RESET"
  ],
  "configuration": {
    "supports_poll_interval": true
  }
}
```

---

# 99. Adapter Metadata

Cada adapter deve possuir:

```text
key
version
manufacturer
supported_models
protocol
```

Exemplo:

```json
{
  "key": "weg.mmw04",
  "version": "1.0.0",
  "manufacturer": "WEG",
  "supported_models": ["MMW04"],
  "protocol": "MODBUS_RTU"
}
```

---

# 100. Adapter Versioning

Adapters devem possuir versionamento independente.

Mudança no adapter não deve alterar automaticamente:

```text
MetricDefinition
```

---

# 101. Adapter Errors

Modelo:

```text
CONNECTION_FAILED

TIMEOUT

INVALID_RESPONSE

CRC_ERROR

UNSUPPORTED_REGISTER

COMMAND_REJECTED
```

---

# 102. Adapter Health

```json
{
  "status": "HEALTHY",
  "last_success_at": "timestamp",
  "last_error_at": null,
  "error_count": 0
}
```

Status:

```text
HEALTHY
DEGRADED
UNHEALTHY
UNKNOWN
```

---

# 103. Edge Internal Modules

```text
edge/
├── core
├── config
├── protocols
├── devices
├── polling
├── normalization
├── outbox
├── mqtt
├── commands
├── health
└── diagnostics
```

---

# 104. Edge Polling Engine

Responsável por:

```text
schedule
read
normalize
enqueue
```

Fluxo:

```text
Scheduler
↓
Protocol Adapter
↓
Device Adapter
↓
Normalized Metrics
↓
Telemetry Envelope
↓
Outbox
```

---

# 105. Edge Outbox Contract

```text
enqueue(message)

peek(batchSize)

markSent(messageId)

markAcknowledged(messageId)

markFailed(messageId, error)
```

Persistência:

```text
SQLite
```

---

# 106. Edge Command Processor

Fluxo:

```text
Receive Command
↓
Validate Schema
↓
Validate Target
↓
Validate Expiration
↓
Check Duplicate
↓
Device Adapter
↓
Protocol Adapter
↓
Execute
↓
Create Command Result
↓
Publish Result
```

---

# 107. Edge Command Deduplication

Manter histórico local mínimo de:

```text
command_id
result
processed_at
```

Ao receber o mesmo comando:

```text
return previous result
```

sem executar novamente.

---

# 108. Edge Config Store

SQLite:

```text
configuration

current_version
payload
applied_at
```

A configuração anterior pode ser mantida para rollback local.

---

# 109. Edge Startup

Fluxo recomendado:

```text
Start
↓
Load Local Configuration
↓
Initialize Database
↓
Initialize Adapters
↓
Connect MQTT
↓
Sync Configuration
↓
Start Polling
↓
Start Command Consumer
↓
Start Heartbeat
```

---

# 110. Edge Offline Startup

Se Cloud indisponível:

```text
load last valid configuration
```

e continuar aquisição local.

---

# 111. Edge Safe Configuration

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

Se falhar:

```text
rollback previous config
```

---

# 112. Configuration Validation

Antes de aplicar:

```text
schema valid

adapter exists

device exists

protocol config valid

metric mapping valid
```

---

# 113. HTTP Retry

Edge pode efetuar retry em:

```text
408
429
500
502
503
504
```

Não repetir automaticamente operações não idempotentes sem proteção.

---

# 114. Retry-After

Quando API retornar:

```http
Retry-After
```

cliente deve respeitar.

---

# 115. API Rate Limit Headers

Opcionalmente:

```text
X-RateLimit-Limit
X-RateLimit-Remaining
X-RateLimit-Reset
```

---

# 116. Concurrency Control

Para updates administrativos, utilizar:

```text
updated_at
```

ou preferencialmente:

```text
version
```

para optimistic locking.

---

# 117. Resource Version

Exemplo:

```json
{
  "id": "uuid",
  "version": 4
}
```

PATCH:

```json
{
  "version": 4,
  "name": "Novo Nome"
}
```

Se versão mudou:

```text
409 CONFLICT
```

---

# 118. PATCH Semantics

Utilizar partial update.

Campos ausentes:

```text
não alterar
```

Campos `null`:

```text
limpar
```

somente quando campo permitir.

---

# 119. DELETE Semantics

DELETE deve ser utilizado com cautela.

Recursos operacionais devem preferir:

```text
DISABLED
```

ou:

```text
ARCHIVED
```

quando histórico precisar ser preservado.

---

# 120. Bulk Operations

Não implementar genericamente.

Criar endpoints específicos quando houver necessidade real.

---

# 121. Export

Futuramente:

```text
POST /api/v1/exports
```

Processamento assíncrono.

Resposta:

```text
202 ACCEPTED
```

---

# 122. Long-running Operations

Utilizar recurso de Job.

```text
Job

id
type
status
progress
result
```

---

# 123. Job Status

```text
QUEUED
RUNNING
SUCCESS
FAILED
CANCELLED
```

---

# 124. OpenAPI

API REST deve possuir especificação:

```text
OpenAPI 3.x
```

Gerada ou validada pelo backend.

---

# 125. Swagger

Disponível em:

```text
development
staging
```

Produção:

```text
restricted
```

ou protegida por autenticação.

---

# 126. Contracts Package

Estrutura:

```text
packages/contracts/
│
├── api/
├── telemetry/
├── events/
├── commands/
├── configuration/
├── errors/
└── common/
```

---

# 127. Common Types

```text
UUID
Timestamp
Pagination
ErrorResponse
CorrelationId
```

---

# 128. Validation

Recomendação:

```text
Zod
```

ou JSON Schema como fonte comum.

O importante é evitar schemas divergentes.

---

# 129. Source of Truth

Contratos compartilhados devem possuir uma fonte principal.

Não manter manualmente:

```text
DTO backend
+
frontend interface
+
Edge interface
```

com definições independentes.

---

# 130. Contract Tests

Obrigatórios:

```text
backend ↔ frontend

backend ↔ edge

edge ↔ broker

adapter ↔ edge
```

---

# 131. API Compatibility Test

Novas versões devem verificar:

```text
removed field

changed datatype

changed required field

enum incompatibility
```

---

# 132. Edge Compatibility

Cloud deve conhecer:

```text
edge software version
```

Configuração incompatível não deve ser enviada silenciosamente.

---

# 133. Minimum Edge Version

Config poderá conter:

```text
minimum_edge_version
```

---

# 134. Feature Capability

Edge deve anunciar capabilities.

Exemplo:

```json
{
  "capabilities": {
    "configuration_v1": true,
    "commands_v1": true,
    "device_twin_v1": false
  }
}
```

---

# 135. Feature Negotiation

Cloud não deve enviar recurso não suportado.

---

# 136. Device Twin API

Preparação:

```text
GET   /api/v1/devices/{id}/twin
PATCH /api/v1/devices/{id}/twin/desired
```

---

# 137. Twin DTO

```json
{
  "data": {
    "device_id": "uuid",
    "desired": {},
    "reported": {},
    "desired_version": 5,
    "reported_version": 5
  }
}
```

---

# 138. Firmware API

Preparação:

```text
GET  /api/v1/firmware
POST /api/v1/firmware

POST /api/v1/devices/{id}/firmware-deployments
```

Não faz parte do MVP inicial.

---

# 139. Health Endpoints

Aplicação:

```text
GET /health/live
GET /health/ready
```

---

# 140. Liveness

Indica:

```text
process is alive
```

Não deve testar todas as dependências.

---

# 141. Readiness

Verifica dependências essenciais:

```text
database
redis
broker connection
```

conforme serviço.

---

# 142. Internal Metrics

Endpoint:

```text
/metrics
```

Formato Prometheus.

Não expor publicamente.

---

# 143. WebSocket Endpoint

```text
wss://api.example.com/ws
```

Autenticação obrigatória.

---

# 144. WebSocket Subscription

Cliente solicita:

```json
{
  "action": "subscribe",
  "channel": "device",
  "id": "uuid"
}
```

Servidor valida autorização antes de registrar subscription.

---

# 145. WebSocket Event

```json
{
  "event": "telemetry.updated",
  "resource_id": "uuid",
  "timestamp": "timestamp",
  "data": {}
}
```

---

# 146. WebSocket Reconnect

Frontend deve suportar:

```text
disconnect
↓
reconnect
↓
refetch current state
```

Não depender de replay ilimitado de WebSocket.

---

# 147. Webhooks

Futuro:

```text
POST /api/v1/webhooks
```

Eventos:

```text
alarm.opened
alarm.cleared
device.offline
command.completed
```

---

# 148. Webhook Security

Assinar payload:

```text
HMAC
```

Headers:

```text
X-Signature
X-Timestamp
X-Delivery-ID
```

---

# 149. Webhook Retry

Usar:

```text
exponential backoff
```

com limite.

---

# 150. Webhook Idempotency

Cada entrega:

```text
delivery_id
```

Consumidor poderá deduplicar.

---

# 151. External API Clients

Machine-to-machine:

```text
client_id
client_secret
```

ou OAuth2 client credentials.

Scopes obrigatórios.

---

# 152. API Credential Secret

Armazenar:

```text
hash
```

quando possível.

Secret só exibido no momento da criação.

---

# 153. Security Rules dos contratos

Nenhum endpoint pode aceitar:

```text
tenant_id arbitrário
```

sem validação.

Nenhum comando pode aceitar:

```text
raw Modbus register write
```

pela API pública genérica.

---

# 154. Raw Protocol Access

Se necessário para engenharia:

```text
separate diagnostic endpoint
```

com permissão elevada e auditoria.

Não fazer parte da API operacional comum.

---

# 155. Diagnostic Access

Exemplo futuro:

```text
POST /api/v1/devices/{id}/diagnostics/read-register
```

Somente:

```text
ENGINEER
```

ou superior.

Obrigatório:

```text
audit
rate limit
feature flag
```

---

# 156. API Domain Boundaries

Controllers não devem implementar regras de domínio.

Fluxo:

```text
Controller
↓
Application Service / Use Case
↓
Domain
↓
Repository
```

---

# 157. Error Mapping

Exemplo:

```text
DeviceNotFound
→ 404

PermissionDenied
→ 403

InvalidMetric
→ 422

VersionConflict
→ 409
```

---

# 158. Edge API Boundaries

Edge não acessa diretamente banco cloud.

Comunicação exclusivamente por:

```text
MQTT
HTTPS API
```

---

# 159. Database Contracts

Não são API pública.

Nenhum consumer externo deve depender diretamente do schema SQL.

---

# 160. Contract Ownership

Responsabilidades:

```text
API contracts
→ platform team

MQTT contracts
→ platform + edge team

Protocol adapters
→ edge/integration team

Device adapters
→ device integration team
```

---

# 161. Contract Change Process

Mudança deve seguir:

```text
proposal
↓
impact analysis
↓
schema update
↓
contract tests
↓
documentation
↓
implementation
```

Breaking change exige versionamento.

---

# 162. ADR

Decisões que alterem contratos centrais devem gerar:

```text
Architecture Decision Record
```

---

# 163. API MVP

Endpoints mínimos para MVP:

```text
Auth

Organizations

Memberships

Sites

Areas

Systems

Asset Types

Assets

Gateways

Device Models

Devices

Metrics

Telemetry ingestion

Telemetry current/history

Events

Alarm Rules

Alarms

Audit
```

---

# 164. Edge MVP

Implementar:

```text
Provisioning

Configuration sync

Heartbeat

Modbus adapter

Device adapter interface

Polling

Normalization

SQLite outbox

MQTT publish

Store & Forward

Command receiver infrastructure
```

Command execution pode ser ativado na etapa seguinte.

---

# 165. MVP Contracts prioritários

Prioridade P0:

```text
Authentication

Tenant context

Gateway provisioning

Device registry

Metric registry

Telemetry envelope

HTTP ingestion

MQTT ingestion

Heartbeat

Historical query
```

P1:

```text
Events

Alarms

WebSocket

Commands
```

P2:

```text
Device Twin

OTA

Webhooks

External integrations
```

---

# 166. Estrutura sugerida no repositório

```text
packages/contracts/
│
├── common/
│   ├── identifiers.ts
│   ├── timestamps.ts
│   └── pagination.ts
│
├── api/
│   ├── auth/
│   ├── assets/
│   ├── devices/
│   ├── gateways/
│   ├── telemetry/
│   ├── alarms/
│   └── commands/
│
├── messaging/
│   ├── telemetry/
│   ├── events/
│   ├── heartbeat/
│   ├── commands/
│   └── configuration/
│
└── adapters/
    ├── protocol.ts
    └── device.ts
```

---

# 167. Contratos que não devem divergir

As seguintes estruturas são consideradas canônicas:

```text
UUID

Timestamp

MetricKey

TelemetryEnvelope

EventEnvelope

HeartbeatEnvelope

CommandEnvelope

CommandResult

ConfigurationEnvelope

ErrorResponse

PaginationResponse
```

---

# 168. Exemplo de fluxo completo

Provisionamento:

```text
Admin
↓
POST /gateways
↓
POST /activation-token
↓
Edge receives token
↓
POST /edge/provision
↓
Certificate
↓
MQTT connect
↓
GET /edge/configuration
↓
Edge operational
```

---

# 169. Fluxo de telemetria

```text
Device
↓
Protocol Adapter
↓
Device Adapter
↓
TelemetryEnvelope
↓
SQLite Outbox
↓
MQTT
↓
Broker
↓
Ingestion
↓
Validation
↓
Persistence
↓
Realtime/Event Consumers
```

---

# 170. Fluxo de comando

```text
Frontend
↓
POST /devices/{id}/commands
↓
Authorization
↓
Command Definition
↓
Validation
↓
Command DB
↓
MQTT
↓
Edge
↓
Device Adapter
↓
Protocol Adapter
↓
Device
↓
Command Result
↓
Cloud
```

---

# 171. Regras não negociáveis

1. Frontend nunca acessa broker diretamente para comando.
2. Edge nunca acessa banco cloud.
3. Device Adapter nunca conhece permissões.
4. Protocol Adapter nunca conhece tenant.
5. API nunca confia em `organization_id` enviado pelo cliente.
6. Telemetria usa contratos versionados.
7. comando sempre possui ID.
8. comando sempre possui expiração.
9. provisioning token é single-use.
10. secrets não são retornados novamente após criação.
11. APIs administrativas exigem autenticação humana.
12. APIs de ingestão exigem identidade de Device/Gateway.
13. schemas devem ser compartilhados.
14. alterações incompatíveis exigem nova versão.
15. todo erro relevante possui correlation ID.

---

# 172. Decisões consolidadas

O API & Edge Contracts v1.0 estabelece:

1. REST API versionada em `/api/v1`.
2. JSON como formato inicial.
3. UUID em IDs externos.
4. timestamps UTC.
5. envelope padrão de respostas.
6. erro padronizado.
7. correlation ID obrigatório.
8. paginação cursor-based.
9. DTOs estáveis.
10. autenticação separada por tipo de identidade.
11. OpenAPI como especificação REST.
12. contratos compartilhados em package comum.
13. HTTP e MQTT usando estruturas compatíveis.
14. Edge provisionado via activation token.
15. preferência por CSR + X.509.
16. configuração Edge versionada.
17. Edge opera offline com última configuração válida.
18. Protocol Adapter separado de Device Adapter.
19. SQLite Outbox obrigatório.
20. comando idempotente e deduplicado.
21. optimistic locking em updates relevantes.
22. WebSocket somente para realtime.
23. REST para histórico.
24. Device Twin previsto, mas não obrigatório no MVP.
25. OTA previsto, mas fora do MVP.
26. contract tests obrigatórios.
27. compatibility checks no CI.
28. feature negotiation Edge ↔ Cloud.
29. raw protocol access fora da API operacional.
30. ADR obrigatório para mudança estrutural de contrato.

---

# 173. Critério principal

Todo novo contrato deve responder claramente:

> Quem chama, como se autentica, qual recurso pode acessar, qual schema envia, qual schema recebe, qual versão utiliza, como lida com retry e como evita duplicidade?

Se qualquer uma dessas respostas estiver indefinida, o contrato não deve ser considerado pronto.

---

# 174. Status

**Documento:** API & Edge Contracts  
**Versão:** 1.0  
**Status:** Baseline inicial  
**Dependências:** Architecture Blueprint v1.0, Domain Model v1.0, Telemetry & Messaging Specification v1.0, Security Architecture v1.0  
**Próximo documento:** MVP Scope & Delivery Plan v1.0