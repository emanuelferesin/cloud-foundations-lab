# Qué levantamos en clase 1

Este documento explica qué es cada componente del entorno y por qué existe.

---

## El entorno: GitHub Codespaces

Cuando abrís el repo en Codespaces, GitHub construye un container usando la
configuración en `.devcontainer/devcontainer.json`. Ese container incluye:

- Python 3.12
- Docker (para correr los servicios)
- Las extensiones de VS Code ya instaladas (Python, SQLTools, Docker, Terraform)

No instalaste nada a mano. El entorno es idéntico para todos.

---

## Los servicios: Docker Compose

`compose.yaml` define 5 servicios. Los levantás con:

```bash
docker compose up -d postgres minio redis   # los tres que usamos hoy
docker compose up -d                         # todos
```

### PostgreSQL — puerto 5432

Base de datos relacional. Es el equivalente local de **Amazon RDS** o **Aurora**.

Usada para guardar datos estructurados con SQL: usuarios, eventos, transacciones.

```bash
# Conectarse
docker exec -it cloud-foundations-postgres psql -U postgres -d course
```

### MinIO — puertos 9000 / 9001

Object storage compatible con la API de S3. Es el equivalente local de **Amazon S3**.

- Puerto 9000: API (donde los scripts suben y bajan archivos)
- Puerto 9001: consola web — abrilo desde el panel Ports de VS Code

Credenciales: `minioadmin` / `minioadmin`

### Redis — puerto 6379

Almacén de clave-valor en memoria. Es el equivalente local de **Amazon ElastiCache**.

Usado para cache, sesiones, colas simples y comunicación rápida entre procesos.

```bash
docker exec -it cloud-foundations-redis redis-cli ping
# → PONG
```

### LocalStack — puerto 4566

Emula las APIs de AWS localmente: S3, SQS, SNS, Lambda, DynamoDB, EventBridge.
No necesitás cuenta de AWS ni pagar nada.

```bash
# Verificar que está up
curl http://localhost:4566/_localstack/health
```

Lo usamos a partir de clase 12 (colas, eventos).

### Redpanda — puerto 9092

Broker de mensajes compatible con la API de Kafka. Es el equivalente local de
**Amazon MSK** o **Kinesis**.

Usado para streaming de eventos en tiempo real.

```bash
docker exec -it cloud-foundations-redpanda rpk topic list
```

Lo usamos a partir de clase 15 (streaming).

---

## Los scripts

| Script | Qué hace |
|--------|----------|
| `scripts/bootstrap.sh` | Verifica dependencias, crea `.env`, directorios y datos de ejemplo. Corre una sola vez al inicio. |
| `scripts/check.sh` | Verifica que los servicios estén up y que los archivos esperados existan. Corre al inicio de cada clase. |
| `scripts/load_postgres.py` | Crea el schema `events` y la tabla `signups` en PostgreSQL, e inserta datos de ejemplo. |
| `scripts/process_events.py` | Lee `data/raw/events.jsonl`, filtra los eventos de tipo `signup` y los guarda en `data/processed/signups.json`. |

---

## Los datos

```
data/
  raw/
    events.jsonl      ← eventos sin procesar (generados por bootstrap.sh)
  processed/
    signups.json      ← salida de process_events.py (la generás vos)
```

`events.jsonl` tiene un evento por línea en formato JSON. Hay signups, logins,
purchases y logouts. El script de procesamiento filtra solo los signups.

---

## El repo como evidencia

Cada cambio que hacés va en un commit. Cada decisión va en `docs/decisions.md`.

El stack puede ser cualquiera — lo que se evalúa es el razonamiento documentado.

```bash
git add docs/decisions.md
git commit -m "lab-01: entorno levantado en Codespaces"
git push
```
