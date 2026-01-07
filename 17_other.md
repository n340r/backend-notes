# Other things that you need to know on a high-level

## Security

_Main things to prevent:_

- DDoS
- Unauthorized requests

### Basic app security

_Gateway (Nginx) level_

- `limit_req` (RPS for IP)
- `limit_conn` (connections for IP),
- `client_max_body_size`
- read / write timeouts `client_body_timeout`, `keepalive_timeout`.

_APP level_

- `@nestjs/throttler` (global throttler), `AuthGuard(Jwt)`, `RolesGuard`

<!-- WIP: -->

### Basic authorization

- JWT, Refresh, Session
- Bearer
- separate prefix for admin endpoints and more strict auth rules

## Devops, clouds, other DBs

**Events:**

- `EventBridge` (AWS) - rules-based **event bus**. Routes events between Lambdas, SQS, other AWS services
- `SQS` (AWS) - simple queue
- `NATS` - Simple messaging system. Very fast, lightweight. Popular in Go (Golang) world.

**Databases:**

- `DynamoDB` (AWS) - Database key–value / NoSQL, serverless, auto-scaling
- `BigQuery` (GCP) - Database OLAP. similar to clickhouse, SQL-like syntax
- `Memcached` - Simple Redis. Basically distributed hashmap. Only SET, GET, DEL. Only strings
- `Cassandra` - NoSQL, designed for huge-scale, mostly storing ledacy data. No joins, No transactions

**Other things:**

- `Terraform` - Infrastructure as service "I want 2 servers, 1 queue, connected like this". Used with AWS, GCP, Azure
- `Lambda` (AWS) - Serverless function. Runs only when triggered, stateless, needs no dedicated always-on server.
- `Edge-function` - Lambda that runs georgraphically closest to you.
- `Kinesis` (AWS) - collect, process, analyze logs, clicks (large data **streams**)

## Deployment strategies

- `Canary Release` - Deploy new version to small % of users first. `1%`, then `5%` and if something breaks, roll back.
- `Blue / Green` - There are two environments: `Blue` (current), `Green` (new one)
- `Rolling deployment` - update servers one by one (often in K8S) so that users dont notice downtime.

## Devops

Понимать что такое:

**K8S**

вот так должно выглядеть запущенное приложение (столько подов тут, там)

**Terraform**

какая вообще инфра есть. Создает кластеры внутри GCP / AWS

**Help charts**

Настройки для разных окружений (кол-во подов, env, версии образов, и тд.)

```
my-backend/
  Chart.yaml --> passport of a chart
  values.yaml --> main settings (changed often)
  templates/
    deployment.yaml --> k8s template with placeholders for values from values.yaml
    service.yaml -->
```

Helm чарты пишут руками либо берут готовые для:

- БД
- Redis
- Kafka
- Prometheus

**Терминология:**

- Pod - запущенный инстанс приложения (умирают, пересоздаются)
- Deployment - упралвяет подами (скока нужно, версия image, как обновлять)
- Service (сетевой объект) - тк поды умирают/создаются, нужен стабильынй service. он принимает запросы, роутит на подыё
- configMap / secrets - env для конфига и секретов соответственно
- Ingress - без него прила была бы доступна только в кластере. принимает HTTP, делает TLS, роутит на реальные сервисы

```
Твой код
 ↓
Dockerfile
 ↓
Docker image
 ↓
Deployment (сколько инстансов)
 ↓
Pod (реальный процесс)
 ↓
Service (внутренний адрес)
 ↓
Ingress (доступ извне)
```
