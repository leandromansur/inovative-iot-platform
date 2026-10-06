# Security Architecture v1.0

## 1. Objetivo

Este documento define a arquitetura de segurança oficial da plataforma Industrial IoT.

O objetivo é proteger:

- usuários;
- organizações;
- dispositivos;
- gateways;
- APIs;
- telemetria;
- comandos;
- credenciais;
- segredos;
- sessões;
- dados históricos;
- integrações externas;
- infraestrutura.

A segurança deve ser aplicada em múltiplas camadas e não depender de um único mecanismo.

---

# 2. Princípios de segurança

A plataforma deve seguir:

```text
Zero Trust
Least Privilege
Defense in Depth
Secure by Default
Fail Secure
Explicit Authorization
Tenant Isolation
Full Auditability
Credential Separation
```

Nenhuma comunicação deve ser considerada confiável apenas por estar dentro da infraestrutura.

---

# 3. Modelo de confiança

A plataforma possui cinco classes principais de identidade:

```text
Human User

API Client

Device

Gateway

Internal Service
```

Cada uma deve possuir identidade, credencial e regras de autorização próprias.

---

# 4. Arquitetura geral de segurança

```text
                       USER
                        │
                    HTTPS/TLS
                        │
                 Identity Layer
                        │
                Authentication
                        │
                  Authorization
                        │
                 Tenant Context
                        │
                       API
                        │
              Application Security
                        │
                 PostgreSQL RLS


FIELD DEVICE
     │
 Industrial Protocol
     │
   EDGE
     │
Device Identity
     │
   mTLS
     │
    MQTT
     │
 Broker ACL
     │
IoT Ingestion
     │
Tenant Validation
     │
Core Platform
```

---

# 5. Identidade humana

Entidade:

```text
User
```

Identificador:

```text
UUID
```

Login principal:

```text
email
```

A identidade do usuário deve ser global.

A associação do usuário às organizações ocorre através de:

```text
Membership
```

---

# 6. Autenticação humana

A primeira versão deve suportar:

```text
email
+
password
```

Preparada para:

```text
OIDC
OAuth 2.0
SSO
Microsoft Entra ID
Google Workspace
SAML
```

A implementação não deve acoplar autorização diretamente ao método de login.

---

# 7. Senhas

Password hashing obrigatório:

```text
Argon2id
```

Nunca armazenar:

```text
password
encrypted_password
reversible_password
```

Somente:

```text
password_hash
```

---

# 8. Política de senha

A plataforma não deve depender somente de complexidade artificial.

Requisitos iniciais:

```text
mínimo: 12 caracteres

bloquear senhas conhecidas como comprometidas

permitir passphrases

não obrigar troca periódica sem motivo
```

Troca obrigatória em caso de:

- suspeita de comprometimento;
- recuperação de conta;
- ação administrativa;
- incidente de segurança.

---

# 9. MFA

MFA deve fazer parte da arquitetura desde o início.

Métodos recomendados:

```text
TOTP
WebAuthn / Passkeys
```

Evitar SMS como mecanismo principal.

---

# 10. MFA obrigatório

Deve ser obrigatório para:

```text
PLATFORM_ADMIN
ORGANIZATION_OWNER
```

Recomendado para:

```text
ORGANIZATION_ADMIN
ENGINEER
TECHNICIAN
```

---

# 11. Step-up Authentication

Operações críticas poderão exigir nova autenticação.

Exemplos:

```text
alterar permissões

revogar credencial

emitir certificado

executar comando crítico

alterar configuração crítica

deletar organização

iniciar OTA
```

Fluxo:

```text
Authenticated User
      ↓
Sensitive Action
      ↓
Step-up MFA
      ↓
Authorization
      ↓
Execution
```

---

# 12. Sessões

A plataforma deve utilizar sessões seguras.

Cada sessão:

```text
Session

id
user_id

created_at
last_seen_at
expires_at

ip_address
user_agent

revoked_at
```

---

# 13. Tokens

Para aplicações web:

preferência:

```text
short-lived access token
+
rotating refresh token
```

Ou sessão server-side equivalente.

Access token:

```text
5–15 minutos
```

Refresh token:

```text
vida maior
+
rotação
+
revogação
```

Valores exatos devem ser configuráveis.

---

# 14. Refresh Token Rotation

Ao utilizar refresh token:

```text
Refresh A
   ↓
Access Token
+
Refresh B
```

Após uso:

```text
Refresh A
→ invalid
```

Reutilização de token antigo deve indicar possível comprometimento.

---

# 15. Cookie Security

Quando utilizados cookies:

```text
Secure
HttpOnly
SameSite
```

Nunca expor refresh token para JavaScript quando puder ser evitado.

---

# 16. Logout

Logout deve invalidar:

```text
current session
```

A plataforma também deve permitir:

```text
logout all sessions
```

---

# 17. Sessões administrativas

Administradores devem conseguir visualizar:

```text
active sessions
```

e revogar sessões suspeitas.

---

# 18. RBAC

Modelo inicial:

```text
Role-Based Access Control
```

Roles:

```text
PLATFORM_ADMIN

ORGANIZATION_OWNER
ORGANIZATION_ADMIN

ENGINEER
TECHNICIAN
OPERATOR
VIEWER
```

---

# 19. Platform Admin

Permissões:

```text
gerenciar plataforma

gerenciar tenants

suporte técnico

operações administrativas
```

Não deve automaticamente acessar toda telemetria dos clientes sem necessidade operacional.

Acesso excepcional deve ser auditado.

---

# 20. Organization Owner

Pode:

```text
gerenciar organização

gerenciar usuários

gerenciar sites

gerenciar ativos

gerenciar dispositivos

gerenciar permissões

visualizar dados

configurar integrações
```

---

# 21. Organization Admin

Semelhante ao Owner, mas não deve:

```text
transferir ownership

deletar organização

executar determinadas operações críticas
```

---

# 22. Engineer

Pode:

```text
configurar devices

configurar gateways

configurar métricas

configurar alarmes

visualizar telemetria

executar comandos permitidos
```

---

# 23. Technician

Pode:

```text
visualizar dispositivos

diagnosticar

executar comandos operacionais autorizados

reconhecer alarmes

consultar históricos
```

---

# 24. Operator

Pode:

```text
monitorar

reconhecer determinados alarmes

executar comandos operacionais limitados
```

---

# 25. Viewer

Somente leitura.

```text
dashboards
telemetria
histórico
relatórios
```

---

# 26. Permission Model

Internamente, roles devem ser traduzidas para permissions.

Exemplo:

```text
asset.read

asset.write

device.read

device.configure

telemetry.read

alarm.read

alarm.acknowledge

command.execute

command.execute.high

user.manage

integration.manage
```

Isso evita regras codificadas diretamente em:

```text
if role == ADMIN
```

---

# 27. Evolução futura

Preparar arquitetura para:

```text
RBAC
+
ABAC
```

Exemplo:

```text
User has role TECHNICIAN

AND

site_id IN allowed_sites
```

---

# 28. Tenant Isolation

O tenant principal é:

```text
Organization
```

Toda requisição autenticada deve possuir contexto de tenant explícito.

Fluxo:

```text
User
↓
Session
↓
Membership
↓
Organization Context
↓
Authorization
↓
Database
```

---

# 29. Organization Context

Nunca confiar diretamente em:

```text
organization_id
```

enviado pelo frontend.

O contexto deve ser validado contra:

```text
authenticated user
+
membership
```

---

# 30. PostgreSQL RLS

Row Level Security deve ser utilizada nas tabelas sensíveis.

Exemplo conceitual:

```sql
organization_id = current_tenant()
```

O objetivo é impedir vazamento entre tenants mesmo em caso de erro da aplicação.

---

# 31. RLS Defense in Depth

Fluxo:

```text
API Authorization
       ↓
Tenant Context
       ↓
Repository
       ↓
PostgreSQL RLS
```

Nunca depender apenas da cláusula:

```text
WHERE organization_id = ?
```

---

# 32. Service Database Roles

Criar roles distintas para:

```text
migration

application

read-only

monitoring

backup
```

A aplicação não deve utilizar credencial de superuser.

---

# 33. Bypass RLS

`BYPASSRLS` deve ser restrito.

A conta normal do backend não deve possuir:

```text
SUPERUSER
BYPASSRLS
```

---

# 34. Device Identity

Cada Device ou Gateway deve possuir identidade própria.

Nunca utilizar:

```text
uma senha global
```

para todos os equipamentos.

---

# 35. Modelo preferencial

Preferência:

```text
X.509
+
mTLS
```

Cada identidade possuirá certificado individual.

---

# 36. Device Certificate

Relacionado a:

```text
device_id
```

ou:

```text
gateway_id
```

Armazenar no core:

```text
certificate_fingerprint

serial_number

issued_at

expires_at

revoked_at

status
```

A chave privada nunca deve ser armazenada no backend depois de provisionada.

---

# 37. Provisionamento inicial

Fluxo recomendado:

```text
Device/Gateway created
        ↓
Activation Token generated
        ↓
Edge authenticates
        ↓
Platform validates token
        ↓
Certificate issued
        ↓
Activation token invalidated
```

---

# 38. Activation Token

Deve ser:

```text
single use

short lived

cryptographically random
```

Nunca reutilizável.

---

# 39. Bootstrap Credential

A credencial de bootstrap serve apenas para provisionamento.

Depois:

```text
bootstrap credential
→ revoked
```

e entra:

```text
operational certificate
```

---

# 40. Certificados

A arquitetura deve possuir:

```text
Root CA

Intermediate CA

Device Certificates
```

Preferência:

```text
offline Root CA
+
online Intermediate CA
```

para produção madura.

---

# 41. Certificate Rotation

Certificados devem possuir expiração.

A plataforma deve suportar:

```text
renewal
rotation
revocation
```

Sem necessidade de intervenção manual para cada dispositivo.

---

# 42. Certificate Revocation

Em caso de comprometimento:

```text
DeviceCredential.status = REVOKED
```

Broker e APIs devem rejeitar imediatamente novas conexões.

---

# 43. Alternativa inicial

Caso mTLS atrase demasiadamente o MVP:

```text
client_id
+
device secret
```

Pode ser usado temporariamente.

Porém:

- um segredo por Device/Gateway;
- hash no servidor;
- TLS obrigatório;
- rotação suportada;
- migração futura para certificado.

---

# 44. MQTT Security

Toda comunicação cloud MQTT deve utilizar:

```text
MQTTS
```

Preferência:

```text
TLS 1.3
```

Mínimo:

```text
TLS 1.2
```

---

# 45. MQTT Authentication

Broker autentica:

```text
Device
Gateway
Internal Service
```

Nenhum cliente anônimo.

---

# 46. MQTT Authorization

ACL por namespace.

Device:

```text
publish:
v1/{organization}/{device}/telemetry
v1/{organization}/{device}/event
v1/{organization}/{device}/status

subscribe:
v1/{organization}/{device}/command
v1/{organization}/{device}/configuration
```

---

# 47. Gateway ACL

Gateway poderá publicar somente para Devices vinculados a ele.

Isso deve ser derivado do Device Registry.

---

# 48. Topic Injection

IDs utilizados nos tópicos devem ser validados.

Não aceitar valores arbitrários contendo:

```text
+
#
/
```

onde possam alterar a ACL.

---

# 49. HTTP Security

Toda comunicação externa:

```text
HTTPS
```

Não expor HTTP não criptografado em produção.

---

# 50. API Authentication

Human API:

```text
session/access token
```

Machine API:

```text
OAuth client credentials
```

ou credencial equivalente.

Device API:

```text
mTLS
```

ou Device Credential.

---

# 51. API Scopes

API Clients devem utilizar scopes.

Exemplos:

```text
telemetry:read

telemetry:write

assets:read

devices:read

alarms:read

commands:execute
```

---

# 52. Rate Limiting

Obrigatório em:

```text
login

password reset

MFA

API

device ingestion

commands

provisioning
```

---

# 53. Login Protection

Implementar:

```text
progressive delay

rate limit

IP/device signals

security logging
```

Evitar bloquear permanentemente contas apenas por tentativa externa, para não permitir DoS simples.

---

# 54. Brute Force Detection

Gerar evento em:

```text
repeated login failures

repeated MFA failures

device authentication failures
```

---

# 55. Secrets Management

Segredos incluem:

```text
database passwords

JWT signing keys

API secrets

SMTP credentials

broker credentials

cloud credentials

private keys

webhook secrets
```

---

# 56. Regra principal

Nunca armazenar segredo em:

```text
Git

source code

Dockerfile

frontend

logs

documentation

plaintext database fields
```

---

# 57. Development

No desenvolvimento:

```text
.env
```

permitido localmente.

Deve existir:

```text
.env.example
```

sem segredos reais.

`.env` deve estar no:

```text
.gitignore
```

---

# 58. Production Secrets

Produção deve evoluir para:

```text
Secrets Manager
```

Exemplos:

```text
HashiCorp Vault

cloud-native secrets manager
```

---

# 59. Secret Rotation

Credenciais importantes devem suportar rotação sem downtime significativo.

Especialmente:

```text
database credentials

API secrets

signing keys

device credentials
```

---

# 60. Signing Keys

JWT ou tokens assinados devem possuir:

```text
key_id
```

para permitir múltiplas chaves durante rotação.

---

# 61. Encryption at Rest

Produção deve utilizar criptografia de armazenamento.

Inclui:

```text
database disk

object storage

backups

snapshots
```

---

# 62. Sensitive Fields

Dados particularmente sensíveis podem possuir criptografia no nível da aplicação.

Exemplo:

```text
integration credentials
```

---

# 63. Logs

Nunca registrar:

```text
password

access token

refresh token

private key

full secret

authorization header
```

---

# 64. Audit Architecture

Audit Log é diferente de application log.

Application Log:

```text
debug
error
performance
```

Audit Log:

```text
quem
fez o quê
quando
sobre qual recurso
```

---

# 65. Audit Events

Obrigatórios:

```text
LOGIN_SUCCESS
LOGIN_FAILURE

MFA_ENABLED
MFA_DISABLED

USER_CREATED
USER_DISABLED

ROLE_CHANGED

DEVICE_CREATED
DEVICE_DELETED

DEVICE_CREDENTIAL_CREATED
DEVICE_CREDENTIAL_REVOKED

COMMAND_CREATED
COMMAND_EXECUTED
COMMAND_FAILED

ALARM_ACKNOWLEDGED

INTEGRATION_CREATED

SECURITY_CONFIGURATION_CHANGED
```

---

# 66. Audit Fields

```text
id

timestamp

organization_id

actor_type
actor_id

action

resource_type
resource_id

ip_address

user_agent

correlation_id

before
after

metadata
```

---

# 67. Audit Immutability

Audit Logs não devem ser alteráveis por usuários comuns.

Preferência futura:

```text
append-only storage
```

---

# 68. Sensitive Audit Data

Não incluir segredo completo no:

```text
before
after
```

Exemplo:

correto:

```text
api_secret_changed = true
```

incorreto:

```text
old_secret = ...
new_secret = ...
```

---

# 69. Command Security

Command é uma das áreas mais sensíveis da plataforma.

Nenhum comando deve ir:

```text
Frontend → MQTT
```

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
Command Validation
↓
Safety Policy
↓
Audit
↓
Command Queue
↓
MQTT
```

---

# 70. Command Definitions

Somente comandos cadastrados podem ser executados.

Exemplo:

```text
SET_SPEED
```

possui:

```text
schema
allowed range
required role
criticality
timeout
```

---

# 71. Command Parameter Validation

Exemplo:

```text
SET_SPEED
```

Não basta validar:

```text
number
```

Deve validar:

```text
minimum

maximum

unit

target capability
```

---

# 72. Command Criticality

Categorias:

```text
LOW

MEDIUM

HIGH

CRITICAL
```

---

# 73. High Risk Commands

Podem exigir:

```text
recent authentication

MFA

explicit confirmation

specific permission
```

---

# 74. Critical Commands

Preparar arquitetura para:

```text
dual authorization
```

Exemplo futuro:

```text
Technician requests

Engineer approves
```

---

# 75. Command Expiration

Obrigatório:

```text
expires_at
```

Comandos expirados:

```text
must not execute
```

---

# 76. Replay Protection

Command possui:

```text
command_id
correlation_id
expires_at
```

Edge deve manter cache de comandos processados por período suficiente.

Comando repetido com mesmo ID:

```text
não executar novamente
```

---

# 77. Command Result

Resultado deve ser autenticado através do canal Device/Gateway.

Nunca aceitar atualização arbitrária de:

```text
Command.status
```

via API pública comum.

---

# 78. Local Safety

Cloud não deve substituir:

```text
interlock

emergency stop

safety relay

safety PLC

drive safety function
```

O Edge e equipamento devem continuar aplicando regras locais de segurança.

---

# 79. Secure Edge

O Edge deve seguir:

```text
minimal services

automatic security updates

firewall

non-root execution

read-only filesystem where possible

protected credentials
```

---

# 80. Edge User

Processo Edge não deve executar como:

```text
root
```

sem necessidade técnica explícita.

---

# 81. Edge Credentials

Certificados e secrets devem possuir permissões de arquivo restritivas.

Exemplo Linux:

```text
0600
```

---

# 82. Edge Configuration

Configurações devem ser assinadas ou autenticadas pelo canal seguro.

O Edge não deve aceitar configuração arbitrária via rede local sem autenticação.

---

# 83. Edge Local API

Caso exista:

```text
localhost only
```

por padrão.

Exposição LAN deve exigir decisão explícita.

---

# 84. Protocol Security

Protocolos industriais frequentemente não possuem segurança nativa.

Exemplo:

```text
Modbus RTU
Modbus TCP
```

Por isso:

```text
industrial network
```

deve ser considerada uma zona de confiança limitada.

Gateway deve funcionar como boundary.

---

# 85. Network Segmentation

Arquitetura recomendada:

```text
Industrial Network
       │
     Edge
       │
Outbound TLS
       │
    Internet
       │
     Cloud
```

Evitar necessidade de conexões inbound da Cloud para a planta.

---

# 86. Outbound-only Edge

Preferência forte:

```text
Edge inicia conexão
```

Cloud não abre conexão diretamente para rede industrial.

Isso reduz:

- firewall complexity;
- attack surface;
- NAT issues.

---

# 87. Threat Model

A plataforma deve considerar pelo menos os seguintes agentes:

```text
external attacker

malicious tenant user

compromised user account

compromised device

compromised gateway

malicious API client

insider

supply-chain compromise
```

---

# 88. Ameaça: Credential Theft

Cenário:

```text
user password stolen
```

Mitigações:

```text
MFA

session management

rate limiting

security events

step-up auth
```

---

# 89. Ameaça: Tenant Data Leakage

Cenário:

```text
Organization A
accesses Organization B
```

Mitigações:

```text
tenant context

RBAC

RLS

security tests

audit
```

---

# 90. Ameaça: Compromised Device

Cenário:

Device envia telemetria falsa ou excessiva.

Mitigações:

```text
individual credential

rate limiting

schema validation

semantic validation

device revocation

anomaly detection
```

---

# 91. Ameaça: Compromised Gateway

Cenário:

Gateway tenta publicar como outros dispositivos.

Mitigações:

```text
ACL

gateway-device registry

certificate identity

rate limits

security events
```

---

# 92. Ameaça: Replay

Cenário:

reenvio de:

```text
telemetry

command

authentication data
```

Mitigações:

```text
TLS

message_id

command_id

expires_at

deduplication

sequence
```

---

# 93. Ameaça: Command Injection

Cenário:

usuário ou atacante envia comando fora dos limites.

Mitigações:

```text
command definitions

schema validation

RBAC

range validation

device capabilities

audit

local safety
```

---

# 94. Ameaça: Broker Abuse

Cenário:

cliente MQTT publica ou assina tópicos indevidos.

Mitigações:

```text
authentication

per-client ACL

topic validation

rate limiting

connection limits
```

---

# 95. Ameaça: Secret Leakage

Mitigações:

```text
secret manager

log redaction

Git scanning

rotation

least privilege
```

---

# 96. Ameaça: SQL Injection

Mitigações:

```text
ORM / parameterized queries

validation

no raw SQL from user input

least privilege DB user
```

---

# 97. Ameaça: XSS

Frontend:

```text
escape output

Content Security Policy

avoid unsafe HTML
```

---

# 98. Ameaça: CSRF

Quando autenticação utilizar cookies:

```text
SameSite

CSRF protection

origin validation
```

---

# 99. Ameaça: SSRF

Integrações e webhooks devem validar destinos.

Bloquear acesso arbitrário a:

```text
localhost

metadata services

internal networks
```

quando aplicável.

---

# 100. Ameaça: Malicious File

Uploads como:

```text
firmware

attachments

imports
```

devem possuir:

```text
file size validation

content validation

checksum

storage isolation
```

---

# 101. Firmware Security

Firmware futuro deverá possuir:

```text
checksum
+
digital signature
```

Edge não deve instalar firmware sem assinatura válida.

---

# 102. OTA Authorization

OTA exige:

```text
authorized user

compatible device model

signed firmware

audit log

deployment tracking
```

---

# 103. Dependency Security

Pipeline CI deverá executar:

```text
dependency vulnerability scan
```

e controlar:

```text
lockfile
```

---

# 104. Container Security

Containers:

```text
non-root

minimal base image

fixed versions

no unnecessary packages

read-only where possible
```

---

# 105. Image Registry

Somente imagens provenientes do registry autorizado devem ser utilizadas em produção.

---

# 106. Security Scan CI/CD

Pipeline:

```text
lint
↓
tests
↓
SAST
↓
dependency scan
↓
secret scan
↓
build
↓
container scan
↓
deploy
```

---

# 107. Branch Protection

Branches principais:

```text
main
develop
```

ou modelo equivalente devem possuir proteção.

Exigir:

```text
review

passing tests

no direct push
```

para produção.

---

# 108. Production Access

Acesso a produção:

```text
named accounts

MFA

least privilege

audited access
```

Evitar contas compartilhadas.

---

# 109. Environment Separation

Separar:

```text
development

staging

production
```

Cada ambiente deve possuir:

```text
different database

different secrets

different credentials

different broker
```

---

# 110. Nunca reutilizar

Não reutilizar credenciais de:

```text
development
```

em:

```text
production
```

---

# 111. Backup Security

Backups devem possuir:

```text
encryption

access control

retention

restore tests
```

Backup sem teste de restauração não deve ser considerado estratégia de recuperação válida.

---

# 112. Database Backup

Cobrir:

```text
PostgreSQL

TimescaleDB

security configuration
```

---

# 113. Object Storage Backup

Quando necessário:

```text
firmware
reports
critical documents
```

---

# 114. Availability

Segurança também inclui disponibilidade.

Mitigar:

```text
resource exhaustion

message flood

connection flood

large payload

runaway device
```

---

# 115. Quotas

Preparar para limites por Organization:

```text
devices

API rate

telemetry rate

storage

connections
```

---

# 116. Security Event

Criar categoria própria:

```text
SECURITY_EVENT
```

Exemplos:

```text
INVALID_DEVICE_CREDENTIAL

TENANT_MISMATCH

CERTIFICATE_REVOKED

REPEATED_LOGIN_FAILURE

FORBIDDEN_COMMAND

ACL_VIOLATION
```

---

# 117. Alertas de segurança

Eventos críticos devem poder gerar alerta administrativo.

Exemplo:

```text
100 authentication failures
```

ou:

```text
device publishing outside normal rate
```

---

# 118. Correlation ID

Ações sensíveis devem preservar:

```text
correlation_id
```

entre:

```text
API
service
broker
edge
database
audit
```

---

# 119. Security Headers

Frontend/API:

```text
Content-Security-Policy

Strict-Transport-Security

X-Content-Type-Options

Referrer-Policy

frame restrictions
```

---

# 120. CORS

Nunca utilizar em produção:

```text
Access-Control-Allow-Origin: *
```

para APIs autenticadas.

Origens devem ser explicitamente permitidas.

---

# 121. Error Handling

Não expor:

```text
stack trace

SQL query

filesystem path

secret

internal topology
```

ao cliente.

---

# 122. Production Debug

Debug detalhado:

```text
disabled
```

em produção.

---

# 123. Data Classification

Classificação inicial:

```text
PUBLIC

INTERNAL

CONFIDENTIAL

SECRET
```

---

# 124. Exemplos

```text
Product documentation
→ PUBLIC

Operational metadata
→ INTERNAL

Customer telemetry
→ CONFIDENTIAL

Credentials/private keys
→ SECRET
```

---

# 125. Data Access

Acesso deve ser proporcional à classificação.

---

# 126. Telemetry Ownership

Telemetria de uma Organization pertence ao contexto daquela Organization.

Administradores da plataforma não devem tratá-la como dado público ou global.

---

# 127. Privacy by Design

Evitar coletar informações pessoais que não sejam necessárias para a função industrial da plataforma.

---

# 128. Development Rules

Nunca aceitar código com:

```text
hard-coded password

hard-coded API secret

disabled authentication

temporary admin bypass

wildcard MQTT ACL

disabled TLS verification
```

mesmo "temporariamente" sem registro técnico explícito.

---

# 129. Security Exceptions

Qualquer exceção deve gerar ADR ou registro equivalente.

Contendo:

```text
reason

risk

mitigation

owner

expiration/review date
```

---

# 130. Security Testing

Obrigatório:

```text
unit tests

authorization tests

tenant isolation tests

RLS tests

API security tests

MQTT ACL tests

device authentication tests

command authorization tests
```

---

# 131. Tenant Isolation Test

Teste fundamental:

```text
User A
belongs to Organization A
```

deve receber:

```text
403
```

ou equivalente ao tentar acessar recurso de:

```text
Organization B
```

Mesmo conhecendo o UUID.

---

# 132. RLS Test

Mesmo se camada da API falhar, query executada no contexto do tenant A não deve retornar registros do tenant B.

---

# 133. Command Security Tests

Testar:

```text
viewer executes command
→ deny

operator executes allowed command
→ allow

technician sends invalid setpoint
→ deny

expired command
→ deny

wrong tenant
→ deny
```

---

# 134. Device Security Tests

Testar:

```text
invalid certificate

expired certificate

revoked certificate

wrong tenant

unauthorized topic

duplicate message

oversized payload
```

---

# 135. Security Observability

Dashboard interno deve mostrar:

```text
authentication failures

revoked credentials

device authentication failures

ACL violations

rate limit blocks

security events

active sessions
```

---

# 136. Incident Response

Preparar processo mínimo:

```text
Detect
↓
Contain
↓
Revoke
↓
Investigate
↓
Recover
↓
Review
```

---

# 137. Compromised User

Procedimento:

```text
disable user

revoke sessions

reset credentials

review audit logs

review permissions
```

---

# 138. Compromised Device

Procedimento:

```text
revoke credential

block broker access

mark device compromised

preserve logs

issue new credential only after investigation
```

---

# 139. Compromised Gateway

Como gateway pode representar vários Devices:

```text
revoke gateway

quarantine associated communication

investigate affected devices

rotate credentials
```

---

# 140. Security Baseline para MVP

Obrigatório antes do MVP entrar em produção:

```text
TLS

Argon2id

secure sessions

RBAC

PostgreSQL RLS

tenant isolation tests

individual device credentials

MQTT authentication

MQTT ACL

rate limiting

audit logs

secret separation

production environment separation

backup

security logging
```

---

# 141. Pode entrar após MVP inicial

```text
full PKI automation

WebAuthn

dual approval

advanced anomaly detection

SIEM integration

automatic secret rotation

signed OTA

hardware secure elements

ABAC
```

A arquitetura deve, contudo, permitir essas evoluções.

---

# 142. Segurança mínima do Edge MVP

```text
TLS

individual gateway credential

local encrypted/restricted secret

non-root process

SQLite protected

outbound-only cloud connection

command expiration validation

duplicate command protection
```

---

# 143. Segurança mínima da Cloud MVP

```text
HTTPS only

authenticated APIs

secure password hashing

RBAC

RLS

audit

MQTT ACL

rate limits

secret isolation

environment separation

encrypted backup
```

---

# 144. Requisitos críticos

Os seguintes pontos são considerados não negociáveis:

1. nenhuma senha armazenada em texto puro;
2. nenhuma credencial global de equipamentos;
3. nenhum acesso cross-tenant;
4. nenhum comando direto frontend → broker;
5. nenhum MQTT anônimo;
6. nenhum HTTP em produção;
7. nenhum segredo no Git;
8. nenhum backend executando como DB superuser;
9. nenhuma função crítica de segurança industrial dependente da Cloud;
10. nenhuma alteração sensível sem auditoria.

---

# 145. Matriz resumida de controle

| Área | Controle principal |
|---|---|
| Usuário | MFA + sessão segura |
| Tenant | RBAC + RLS |
| API | TLS + auth + scopes |
| Device | credencial individual |
| Gateway | certificado/secret individual |
| MQTT | TLS + ACL |
| Banco | least privilege + RLS |
| Command | RBAC + validação + audit |
| Edge | outbound-only + credentials |
| Secrets | secret manager |
| Audit | append-oriented logging |
| CI/CD | security scanning |
| Firmware | assinatura futura |
| Backup | criptografia + restore test |

---

# 146. Trust Boundaries

A plataforma reconhece como fronteiras de confiança:

```text
Browser
│
┆ TRUST BOUNDARY
│
Cloud API


Cloud
│
┆ TRUST BOUNDARY
│
Internet
│
┆ TRUST BOUNDARY
│
Edge


Edge
│
┆ TRUST BOUNDARY
│
Industrial Network
│
Device
```

Toda travessia de boundary exige validação.

---

# 147. Security Architecture resumida

```text
USER
 │
 MFA
 │
Identity
 │
Session
 │
RBAC
 │
Tenant Context
 │
API
 │
RLS
 │
DATABASE


DEVICE
 │
Identity
 │
Certificate
 │
mTLS
 │
MQTT ACL
 │
INGESTION
 │
Schema Validation
 │
Tenant Validation
 │
CORE


USER
 │
Command Request
 │
Authentication
 │
Authorization
 │
Validation
 │
Safety Policy
 │
Audit
 │
MQTT
 │
EDGE
 │
Local Safety
 │
DEVICE
```

---

# 148. Decisões consolidadas

A Security Architecture v1.0 estabelece:

1. Zero Trust como princípio.
2. Defense in Depth.
3. User global com Membership por Organization.
4. Argon2id para password hashing.
5. MFA previsto desde o início.
6. MFA obrigatório para perfis administrativos.
7. sessões revogáveis.
8. access tokens de curta duração.
9. refresh token rotation quando aplicável.
10. RBAC baseado em permissions.
11. evolução futura para ABAC.
12. Organization como boundary de tenant.
13. PostgreSQL RLS obrigatório.
14. backend sem BYPASSRLS.
15. identidade individual por Device/Gateway.
16. preferência por X.509/mTLS.
17. bootstrap credential de uso único.
18. certificate rotation e revocation.
19. MQTTS obrigatório.
20. MQTT ACL por Device/Gateway.
21. HTTPS obrigatório.
22. scopes para API clients.
23. rate limiting.
24. secret management.
25. logs sem secrets.
26. auditoria separada de logs técnicos.
27. comandos sempre autorizados e auditados.
28. comandos possuem expiração.
29. replay de comando deve ser impedido.
30. safety industrial permanece local.
31. Edge preferencialmente outbound-only.
32. separação dev/staging/prod.
33. security scans no CI/CD.
34. encrypted backups.
35. threat model formal.
36. testes obrigatórios de isolamento de tenants.
37. security events explícitos.
38. incident response definido.
39. credenciais comprometidas devem ser revogáveis.
40. nenhuma exceção de segurança sem registro.

---

# 149. Critério arquitetural de segurança

Toda nova funcionalidade deve responder:

> Qual identidade está executando esta ação, sobre qual recurso, em qual tenant, através de qual canal, com qual permissão e como isso será auditado?

Se essas respostas não estiverem claras, a funcionalidade não deve ser considerada pronta para implementação.

---

# 150. Status

**Documento:** Security Architecture  
**Versão:** 1.0  
**Status:** Baseline inicial  
**Dependências:** Architecture Blueprint v1.0, Domain Model v1.0, Telemetry & Messaging Specification v1.0  
**Próximo documento:** API & Edge Contracts v1.0