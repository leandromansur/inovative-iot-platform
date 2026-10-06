\# Architecture Blueprint v1.0



\## 1. Objetivo



Projetar uma plataforma industrial IoT própria, escalável e multitenant, capaz de integrar equipamentos, sensores, controladores e gateways de diferentes fabricantes, permitindo:



\- aquisição de telemetria;

\- monitoramento em tempo real;

\- histórico;

\- alarmes e eventos;

\- dashboards;

\- gestão de ativos;

\- comandos remotos;

\- gerenciamento de dispositivos;

\- relatórios;

\- analytics;

\- integração com sistemas externos;

\- evolução futura para manutenção preditiva e IA.



A arquitetura deve permitir operação:



\- Cloud;

\- Edge + Cloud;

\- futuramente, instalação on-premise.



\---



\# 2. Princípios arquiteturais



\## 2.1 Vendor-neutral



A plataforma não deve depender de WEG, Schneider, Siemens ou qualquer outro fabricante.



Os equipamentos específicos são tratados através de adapters.



```text

WEG

Schneider

Siemens

ABB

Danfoss

Sensores genéricos

CLPs

Carregadores EV

&#x20;       │

&#x20;       ▼

Protocol / Device Adapters

&#x20;       │

&#x20;       ▼

Modelo normalizado da plataforma

```



\---



\## 2.2 Asset ≠ Device



Um ativo físico não deve ser confundido com o dispositivo responsável pela comunicação.



Exemplo:



```text

Asset

Compressor 01



Devices

├── Inversor

├── Medidor de energia

├── Sensor de pressão

├── Sensor de vibração

└── Gateway

```



O usuário gerencia \*\*ativos\*\*.



A infraestrutura IoT gerencia \*\*devices\*\*.



\---



\## 2.3 Edge-first



A perda da comunicação com a nuvem não deve impedir a operação local.



O Edge deve ser capaz de:



\- coletar dados;

\- armazenar temporariamente;

\- executar regras críticas;

\- registrar timestamps;

\- retransmitir dados;

\- executar comandos;

\- operar offline quando necessário.



\---



\## 2.4 API-first



Nenhuma funcionalidade central deve depender diretamente da interface web.



```text

Frontend

Mobile App

External API

Edge

Integrações

&#x20;      │

&#x20;      ▼

&#x20;    APIs

&#x20;      │

&#x20;      ▼

IoT Platform

```



\---



\## 2.5 Event-driven



Telemetria, alarmes, comandos e eventos devem poder circular de maneira assíncrona.



\---



\## 2.6 Security by design



Segurança não será implementada posteriormente.



Identidade, segregação, auditoria e criptografia fazem parte da arquitetura inicial.



\---



\# 3. Arquitetura de alto nível



```text

┌───────────────────────────────────────────────────────────┐

│                    APPLICATION LAYER                      │

│                                                           │

│ Dashboards │ Reports │ Alarms │ Analytics │ Mobile │ API │

└────────────────────────────▲──────────────────────────────┘

&#x20;                            │

┌────────────────────────────┴──────────────────────────────┐

│                     IoT CORE PLATFORM                     │

│                                                           │

│ Identity                                                  │

│ Tenant Management                                         │

│ Asset Registry                                            │

│ Device Registry                                           │

│ Telemetry                                                 │

│ Alarm Engine                                              │

│ Command Service                                           │

│ Notification Service                                      │

│ Reporting                                                 │

│ Audit                                                     │

│ Integration API                                           │

└────────────────────────────▲──────────────────────────────┘

&#x20;                            │

┌────────────────────────────┴──────────────────────────────┐

│                  MESSAGE / INGESTION LAYER                │

│                                                           │

│ MQTT │ HTTP │ WebSocket │ Event Bus │ Normalization      │

└────────────────────────────▲──────────────────────────────┘

&#x20;                            │

┌────────────────────────────┴──────────────────────────────┐

│                       EDGE LAYER                          │

│                                                           │

│ Gateway                                                   │

│ Protocol Adapters                                         │

│ Store \& Forward                                           │

│ Local Rules                                               │

│ Local Commands                                            │

│ Edge Diagnostics                                          │

└────────────────────────────▲──────────────────────────────┘

&#x20;                            │

┌────────────────────────────┴──────────────────────────────┐

│                  CONNECTED PRODUCTS                       │

│                                                           │

│ Sensors │ Meters │ Drives │ PLC │ UPS │ EV │ HVAC        │

│ Machines │ Protection │ Industrial Equipment             │

└───────────────────────────────────────────────────────────┘

```



\---



\# 4. Camadas



\## 4.1 Connected Products



Representa tudo que existe fisicamente no campo.



Exemplos:



\- medidores de energia;

\- inversores;

\- soft-starters;

\- CLPs;

\- sensores;

\- relés de proteção;

\- UPS;

\- chillers;

\- compressores;

\- carregadores EV;

\- controladores HVAC.



Protocolos iniciais recomendados:



\- Modbus RTU;

\- Modbus TCP;

\- MQTT;

\- HTTP/REST;

\- OPC UA.



Futuros:



\- BACnet;

\- CAN;

\- IEC 61850;

\- SNMP;

\- PROFINET;

\- EtherNet/IP;

\- OCPP.



\---



\# 5. Edge Layer



\## 5.1 Edge Agent



Software responsável pela comunicação entre campo e plataforma.



Responsabilidades:



```text

Protocol acquisition

&#x20;       ↓

Parsing

&#x20;       ↓

Normalization

&#x20;       ↓

Timestamp

&#x20;       ↓

Local buffer

&#x20;       ↓

Cloud transmission

```



\---



\## 5.2 Protocol Adapter



Cada protocolo será implementado como um adapter independente.



Exemplo:



```text

adapters/

├── modbus-rtu

├── modbus-tcp

├── mqtt

├── opcua

└── http

```



\---



\## 5.3 Device Adapter



Traduz informações específicas de determinado equipamento.



Exemplo:



```text

devices/

├── weg/

│   ├── mmw04

│   └── cfw11

│

├── schneider/

│   ├── pm5000

│   └── atv630

│

└── siemens/

&#x20;   └── pac3200

```



Exemplo:



```text

Registrador 40101

&#x20;       ↓

Schneider adapter

&#x20;       ↓

electrical.voltage.l1\_l2

```



\---



\# 6. IoT Core



O IoT Core será o núcleo da plataforma.



Inicialmente deve ser implementado como \*\*modular monolith\*\*.



Não iniciar com microserviços.



Separação lógica:



```text

modules/



identity

tenancy

assets

devices

telemetry

alarms

commands

notifications

reports

audit

integrations

```



Cada módulo deve possuir:



```text

domain

application

infrastructure

api

```



\---



\# 7. Módulos principais



\## Identity



Responsável por:



\- usuários;

\- login;

\- sessões;

\- MFA;

\- recuperação de senha;

\- autenticação API.



\---



\## Tenancy



Responsável por:



\- organizações;

\- clientes;

\- isolamento de dados;

\- memberships;

\- permissões.



\---



\## Assets



Responsável pela representação operacional dos equipamentos.



Hierarquia:



```text

Organization

&#x20;  └── Site

&#x20;      └── Area

&#x20;          └── System

&#x20;              └── Asset

```



Exemplo:



```text

Cliente ABC

└── Unidade Rio

&#x20;   └── Utilidades

&#x20;       └── Sistema de Ar Comprimido

&#x20;           └── Compressor 01

```



\---



\## Devices



Representa dispositivos conectados.



Exemplos:



```text

device

gateway

sensor

meter

PLC

drive

controller

```



Um asset pode possuir vários devices.



\---



\## Telemetry



Responsável por:



\- ingestão;

\- normalização;

\- validação;

\- armazenamento;

\- consulta histórica;

\- agregações.



\---



\## Alarms



Responsável por:



```text

Rule

Condition

Alarm

Acknowledgement

Event

Escalation

```



\---



\## Commands



Responsável pela comunicação plataforma → campo.



```text

User

&#x20;↓

Command API

&#x20;↓

Authorization

&#x20;↓

Command Queue

&#x20;↓

Gateway

&#x20;↓

Device

&#x20;↓

ACK

&#x20;↓

Command Result

```



\---



\## Notifications



Canais futuros:



\- e-mail;

\- WhatsApp;

\- push;

\- SMS;

\- webhook.



\---



\## Audit



Deve registrar ações administrativas e operacionais.



Exemplo:



```text

user

action

resource

timestamp

source\_ip

before

after

correlation\_id

```



\---



\# 8. Modelo principal de entidades



\## Organization



```text

id

name

slug

status

created\_at

```



\---



\## Site



```text

id

organization\_id

name

timezone

location

status

```



\---



\## Area



```text

id

site\_id

name

```



\---



\## System



Representa um conjunto funcional.



Exemplos:



\- sistema HVAC;

\- sistema elétrico;

\- sistema de bombeamento;

\- compressor;

\- geração solar.



```text

id

area\_id

name

type

```



\---



\## Asset



```text

id

system\_id

parent\_asset\_id

asset\_type\_id

name

manufacturer

model

serial\_number

status

metadata

```



\---



\## Device



```text

id

organization\_id

asset\_id

gateway\_id

device\_model\_id



serial\_number

external\_id



status

firmware\_version



last\_seen\_at

created\_at

```



\---



\## Gateway



```text

id

organization\_id

site\_id



serial\_number

software\_version



status

last\_seen\_at

```



\---



\## Device Model



```text

id

manufacturer

family

model

device\_type

adapter

```



\---



\# 9. Modelo de métricas



Um catálogo semântico global deve existir.



\## Metric Definition



```text

id

key

name

quantity

canonical\_unit

datatype

category

```



Exemplo:



```text

key:

electrical.voltage.line\_l1\_l2



quantity:

voltage



canonical\_unit:

V



datatype:

float

```



Outros exemplos:



```text

electrical.current.l1

electrical.active\_power.total

electrical.energy.imported



pressure.oil

pressure.discharge



temperature.motor

temperature.bearing



vibration.rms

speed.rotation

```



\---



\# 10. Telemetria



Formato conceitual:



```json

{

&#x20; "device\_id": "UUID",

&#x20; "observed\_at": "timestamp",

&#x20; "metrics": {

&#x20;   "electrical.voltage.line\_l1\_l2": 221.7,

&#x20;   "electrical.current.l1": 5.3,

&#x20;   "electrical.active\_power.total": 3.2

&#x20; }

}

```



Internamente:



```text

telemetry



tenant\_id

device\_id

metric\_id



observed\_at

received\_at



value



quality

```



\---



\# 11. Dados RAW



Payloads originais devem poder ser preservados.



```text

raw\_messages



id

tenant\_id

device\_id

gateway\_id



protocol



observed\_at

received\_at



payload



processing\_status

```



Objetivos:



\- auditoria;

\- troubleshooting;

\- reprocessamento;

\- análise de adapters.



\---



\# 12. Fluxo de telemetria



```text

DEVICE

&#x20;  ↓

Protocol

&#x20;  ↓

EDGE

&#x20;  ↓

Protocol Adapter

&#x20;  ↓

Device Adapter

&#x20;  ↓

Normalization

&#x20;  ↓

Local buffer

&#x20;  ↓

MQTT / HTTPS

&#x20;  ↓

INGESTION SERVICE

&#x20;  ↓

Authentication

&#x20;  ↓

Validation

&#x20;  ↓

Raw storage

&#x20;  ↓

Normalization validation

&#x20;  ↓

Telemetry storage

&#x20;  ↓

Event Bus

&#x20;  ├── Alarm Engine

&#x20;  ├── Dashboard

&#x20;  ├── Analytics

&#x20;  └── Integrations

```



\---



\# 13. Store \& Forward



O Edge deve manter fila persistente local.



```text

measurement

↓

local database

↓

cloud delivery

↓

ACK

↓

remove from queue

```



Cada mensagem deverá possuir identificador único.



```text

message\_id

```



Isso permitirá idempotência.



\---



\# 14. Estados de conectividade



Não utilizar apenas:



```text

online

offline

```



Adotar:



```text

ONLINE

DEGRADED

OFFLINE

UNKNOWN

DISABLED

```



Cálculo baseado em:



\- heartbeat;

\- telemetria;

\- expectativa de comunicação;

\- falhas de protocolo.



\---



\# 15. Event Model



Eventos devem ter estrutura independente da telemetria.



```text

event



id

tenant\_id

source\_type

source\_id



event\_type

severity



occurred\_at

received\_at



payload

```



Exemplos:



```text

DEVICE\_ONLINE

DEVICE\_OFFLINE



HIGH\_PRESSURE

LOW\_PRESSURE



OVERCURRENT



COMMAND\_EXECUTED

COMMAND\_FAILED



CONFIGURATION\_CHANGED

```



\---



\# 16. Alarm Engine



Regra:



```text

metric

operator

threshold

duration

severity

```



Exemplo:



```text

pressure.oil < 2.5 bar

durante 5 segundos

```



Resultado:



```text

ACTIVE

ACKNOWLEDGED

CLEARED

```



\---



\# 17. Command Architecture



Comandos nunca devem ser enviados diretamente pelo frontend.



```text

Frontend

&#x20;  ↓

Command API

&#x20;  ↓

RBAC check

&#x20;  ↓

Safety validation

&#x20;  ↓

Command record

&#x20;  ↓

MQTT broker

&#x20;  ↓

Gateway

&#x20;  ↓

Device

```



Resposta:



```text

RECEIVED

EXECUTING

SUCCESS

FAILED

TIMEOUT

```



\---



\# 18. Segurança



\## Usuários



Utilizar:



\- OIDC/OAuth2;

\- MFA;

\- sessões seguras;

\- refresh tokens;

\- password hashing Argon2id.



\---



\## Devices



Cada gateway/device deve possuir identidade própria.



Preferência:



```text

X.509 certificate

```



Alternativa inicial:



```text

device\_id

\+

client\_id

\+

secret

```



Nunca usar uma única senha compartilhada para todos os dispositivos.



\---



\# 19. Comunicação



Obrigatório:



```text

TLS 1.2+

```



Preferencial:



```text

TLS 1.3

```



MQTT:



```text

MQTTS

port 8883

```



HTTP:



```text

HTTPS

```



\---



\# 20. Multi-tenancy



Todas as entidades pertencentes a clientes devem possuir:



```text

organization\_id

```



Defesa em múltiplas camadas:



```text

Authentication

&#x20;     ↓

Application authorization

&#x20;     ↓

Tenant context

&#x20;     ↓

PostgreSQL RLS

```



\---



\# 21. RBAC



Roles iniciais:



```text

platform\_admin



organization\_owner

organization\_admin



engineer

technician

operator

viewer

```



Depois poderá evoluir para RBAC + ABAC.



\---



\# 22. Auditoria



Ações sensíveis precisam ser auditadas:



```text

LOGIN

CREATE\_USER

DELETE\_DEVICE

CHANGE\_CONFIGURATION

SEND\_COMMAND

ACK\_ALARM

CHANGE\_PERMISSION

UPDATE\_FIRMWARE

```



\---



\# 23. Segredos



Não armazenar segredos:



\- no código;

\- no Git;

\- no banco em texto puro.



Inicialmente:



```text

Docker secrets / environment

```



Evolução:



```text

HashiCorp Vault

ou

Cloud Secrets Manager

```



\---



\# 24. Stack tecnológica recomendada



\## Backend



Recomendação:



```text

TypeScript

Node.js

NestJS

```



Motivos:



\- arquitetura modular;

\- forte ecossistema;

\- excelente suporte a API;

\- MQTT;

\- WebSocket;

\- filas;

\- validação;

\- facilidade de manutenção.



\---



\## Frontend



```text

React

TypeScript

Vite

```



Bibliotecas recomendadas:



```text

TanStack Query

React Router

Zod

ECharts

```



UI:



```text

Tailwind CSS

\+

component library própria

```



\---



\# 25. Banco principal



```text

PostgreSQL

```



Para séries temporais:



```text

TimescaleDB

```



Estrutura:



```text

PostgreSQL

├── tenants

├── users

├── assets

├── devices

├── alarms

├── commands

└── configuration



TimescaleDB

├── telemetry

└── events

```



\---



\# 26. Edge Database



Recomendação inicial:



```text

SQLite

```



Funções:



\- buffer;

\- configuração;

\- estado local;

\- fila de mensagens.



\---



\# 27. MQTT Broker



Recomendação:



```text

EMQX

```



Alternativa:



```text

Mosquitto

```



EMQX é preferível devido a:



\- clustering;

\- gerenciamento;

\- autenticação;

\- ACL;

\- observabilidade;

\- maior capacidade de evolução.



\---



\# 28. Cache



```text

Redis

```



Usos:



\- sessão;

\- cache;

\- rate limiting;

\- locks;

\- presença;

\- filas rápidas.



\---



\# 29. Filas e eventos



Na primeira versão:



```text

Redis Streams

ou

BullMQ

```



Não iniciar diretamente com Kafka.



Kafka poderá entrar caso volume e integração justifiquem.



\---



\# 30. Object Storage



Para:



\- relatórios;

\- firmware;

\- anexos;

\- exports;

\- imagens.



Utilizar API compatível com:



```text

S3

```



Inicialmente:



```text

MinIO

```



\---



\# 31. Observabilidade



Desde o início:



```text

OpenTelemetry

```



Stack recomendada:



```text

Prometheus

Grafana

Loki

Tempo

```



Permitir:



```text

Metrics

Logs

Traces

```



\---



\# 32. Infraestrutura



Inicialmente:



```text

Docker

Docker Compose

```



Produção pequena/média:



```text

Docker

Reverse Proxy

PostgreSQL

EMQX

Redis

Object Storage

```



Não introduzir Kubernetes inicialmente.



Kubernetes somente quando houver necessidade operacional real.



\---



\# 33. CI/CD



```text

GitHub

GitHub Actions

Docker Registry

```



Pipeline:



```text

lint

↓

unit tests

↓

integration tests

↓

build

↓

security scan

↓

docker image

↓

deploy

```



\---



\# 34. Estrutura do repositório



Recomendação:



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



\---



\# 35. Monorepo



Recomendado utilizar:



```text

pnpm

\+

Turborepo

```



Permite compartilhar:



```text

types

schemas

contracts

SDK

validation

domain models

```



entre:



```text

frontend

backend

worker

edge

```



\---



\# 36. Organização interna do backend



```text

apps/api/src/modules/



identity/

tenancy/

sites/

assets/

devices/

telemetry/

alarms/

commands/

notifications/

audit/

integrations/

```



Dentro de cada módulo:



```text

domain/

application/

infrastructure/

presentation/

```



Exemplo:



```text

devices/



domain/

&#x20; device.entity.ts

&#x20; device.repository.ts



application/

&#x20; register-device.usecase.ts



infrastructure/

&#x20; postgres-device.repository.ts



presentation/

&#x20; device.controller.ts

```



\---



\# 37. Contratos compartilhados



Os contratos devem ser versionados.



Exemplo:



```text

packages/contracts/



telemetry/

command/

event/

device/

authentication/

```



Nunca permitir que cada serviço invente sua própria estrutura.



\---



\# 38. MQTT Topics



Estrutura inicial sugerida:



```text

v1/{tenant}/{device}/telemetry



v1/{tenant}/{device}/event



v1/{tenant}/{device}/status



v1/{tenant}/{device}/command



v1/{tenant}/{device}/command-result

```



Gateway:



```text

v1/{tenant}/gateway/{gateway}/status

```



\---



\# 39. Device Twin



Não implementar completamente no primeiro MVP, mas reservar o conceito.



Separar:



```text

desired

reported

```



Exemplo:



```json

{

&#x20; "desired": {

&#x20;   "sample\_interval": 10

&#x20; },

&#x20; "reported": {

&#x20;   "sample\_interval": 10

&#x20; }

}

```



Isso permitirá gerenciamento remoto futuro.



\---



\# 40. Firmware Management



Preparar entidades:



```text

Firmware

FirmwareVersion

DeviceFirmware

Deployment

DeploymentTarget

```



OTA pode entrar posteriormente.



\---



\# 41. Dashboards



Dashboard não deve definir o modelo de dados.



Fluxo correto:



```text

Device

↓

Semantic Metric

↓

Telemetry API

↓

Dashboard

```



Não:



```text

Device

↓

Dashboard-specific payload

```



\---



\# 42. Dashboard Templates



Criar conceito:



```text

DashboardTemplate

```



Associado inicialmente a:



```text

AssetType

```



Exemplo:



```text

Compressor

Energy Meter

Pump

HVAC

EV Charger

```



\---



\# 43. Asset Types



Criar catálogo.



Exemplos:



```text

compressor

pump

motor

electrical\_panel

transformer

generator

ups

ev\_charger

hvac\_unit

solar\_inverter

```



Isso permitirá reutilização de dashboards e analytics.



\---



\# 44. APIs externas



Versionamento desde o início:



```text

/api/v1/

```



Exemplo:



```text

GET /api/v1/assets

GET /api/v1/devices

GET /api/v1/telemetry

GET /api/v1/alarms



POST /api/v1/commands

```



\---



\# 45. WebSocket



WebSocket apenas para informações realmente em tempo real:



```text

telemetry

alarm

device status

command status

```



Histórico permanece REST.



\---



\# 46. Arquitetura de implantação inicial



```text

&#x20;                   INTERNET

&#x20;                       │

&#x20;                 Reverse Proxy

&#x20;                       │

&#x20;          ┌────────────┴────────────┐

&#x20;          │                         │

&#x20;       Frontend                   API

&#x20;                                    │

&#x20;                 ┌──────────────────┼──────────────────┐

&#x20;                 │                  │                  │

&#x20;            PostgreSQL           Redis              EMQX

&#x20;            TimescaleDB                             MQTT

&#x20;                 │

&#x20;               MinIO

```



\---



\# 47. Edge



```text

Industrial Device

&#x20;     │

&#x20;   Modbus

&#x20;     │

&#x20;     ▼

&#x20;Edge Agent

&#x20;     │

&#x20;┌────┴─────┐

&#x20;│ SQLite   │

&#x20;│ Buffer   │

&#x20;└────┬─────┘

&#x20;     │

&#x20;    MQTT

&#x20;     │

&#x20;     ▼

&#x20;    EMQX

&#x20;     │

&#x20;     ▼

&#x20;IoT Platform

```



\---



\# 48. MVP — versão 0.1



A primeira versão não deve tentar implementar todo o blueprint.



Escopo:



```text

Identity

Tenancy

Sites

Assets

Devices



MQTT

HTTP ingestion



Telemetry

History



Device status



Basic alarms



Basic dashboard



Audit



Edge Agent

Store \& Forward

```



\---



\# 49. Não implementar no primeiro MVP



Evitar inicialmente:



```text

Kubernetes

Kafka

Microservices

AI

Machine Learning

Digital Twin completo

OTA complexo

Mobile App nativo

Workflow engine

Rule engine complexo

```



Esses componentes devem ser arquiteturalmente possíveis, não imediatamente implementados.



\---



\# 50. Roadmap arquitetural



\## V0.1 — Foundation



```text

Identity

Tenant

Site

Asset

Device

Telemetry

Edge

```



\## V0.2 — Monitoring



```text

Dashboard

Alarm

Events

Notifications

Reports

```



\## V0.3 — Control



```text

Commands

Command audit

Remote configuration

```



\## V0.4 — Device Management



```text

Provisioning

Digital Twin

Firmware

OTA

Diagnostics

```



\## V0.5 — Intelligence



```text

Analytics

Anomaly detection

Predictive maintenance

AI

```



\---



\# 51. Arquitetura resumida



A plataforma deve obedecer ao fluxo:



```text

FIELD

│

├── Assets

├── Devices

└── Sensors

&#x20;      ↓

EDGE

│

├── Protocol adapters

├── Device adapters

├── Local buffer

└── Local logic

&#x20;      ↓

CONNECTIVITY

│

├── MQTT

└── HTTPS

&#x20;      ↓

IoT CORE

│

├── Identity

├── Tenancy

├── Asset Registry

├── Device Registry

├── Telemetry

├── Events

├── Alarms

├── Commands

└── Audit

&#x20;      ↓

DATA

│

├── PostgreSQL

├── TimescaleDB

├── Redis

└── Object Storage

&#x20;      ↓

APPLICATIONS

│

├── Dashboard

├── Reports

├── Notifications

├── Analytics

├── API

└── Integrations

```



\---



\# 52. Decisões arquiteturais v1.0



As seguintes decisões ficam estabelecidas:



1\. Plataforma multitenant.

2\. Plataforma multimarcas.

3\. Asset separado de Device.

4\. Arquitetura Edge + Cloud.

5\. Backend API-first.

6\. Arquitetura event-driven.

7\. Modular monolith inicialmente.

8\. Monorepo.

9\. TypeScript como linguagem predominante.

10\. NestJS no backend.

11\. React no frontend.

12\. PostgreSQL como banco principal.

13\. TimescaleDB para telemetria.

14\. MQTT como principal protocolo IoT cloud.

15\. EMQX como broker MQTT.

16\. Redis para cache e processamento assíncrono.

17\. SQLite no Edge.

18\. S3/MinIO para objetos.

19\. Docker como padrão de implantação inicial.

20\. Kubernetes não será utilizado inicialmente.

21\. Modelo semântico global de métricas.

22\. Payload bruto + telemetria normalizada.

23\. TLS obrigatório.

24\. Credencial individual por device/gateway.

25\. RLS para isolamento entre tenants.

26\. Auditoria de ações sensíveis.

27\. Comandos remotos sempre registrados.

28\. Store \& Forward obrigatório no Edge.

29\. Dashboards desacoplados dos equipamentos.

30\. Arquitetura preparada para Digital Twin, OTA e analytics futuros.



\---



\# 53. Critério arquitetural principal



Toda nova funcionalidade deve responder positivamente à seguinte pergunta:



> O componente pode evoluir, ser substituído ou escalar sem obrigar a redefinir o modelo central de ativos, dispositivos e telemetria?



Se a resposta for não, a implementação deve ser reavaliada antes de entrar no core.



\---



\*\*Documento:\*\* Architecture Blueprint  

\*\*Versão:\*\* 1.0  

\*\*Status:\*\* Baseline inicial  

\*\*Objetivo:\*\* servir como referência técnica para implementação da nova plataforma Industrial IoT.

