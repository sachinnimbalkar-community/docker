End-to-End Enterprise Docker Lab Assignment
Enterprise AI Infrastructure — Install, Configure & Test Docker Services
Objective
Build and validate a complete Enterprise AI / Agentic AI infrastructure stack on Ubuntu WSL2 running on Windows.
You will install the required Docker images, create the required containers, configure networking and persistent storage, verify service-to-service communication, and test every service through its browser/API/CLI interface.
1. Environment
Required platform
Windows 10/11
      │
      ▼
WSL2
      │
      ▼
Ubuntu
      │
      ▼
Docker Engine / Docker Desktop
      │
      ▼
Enterprise AI Containers

Minimum recommended resources
Resource	Recommended
CPU	8 cores
RAM	16 GB+
Disk	50 GB+
OS	Ubuntu 22.04/24.04
Docker	Current stable
WSL	WSL2


2. Services to Install
The student must deploy the following services.
#	Service	Docker Image	Purpose
1	Nginx	nginx:1.29-alpine	API Gateway
2	PostgreSQL	pgvector/pgvector:pg16	Database + Vector DB
3	pgAdmin	dpage/pgadmin4	DB Administration
4	Redis	redis:latest	Cache
5	Qdrant	qdrant/qdrant	Vector Database
6	Ollama	ollama/ollama	Local LLM
7	Kafka	Kafka image of choice	Event Streaming
8	Prometheus	prom/prometheus	Metrics
9	Grafana	grafana/grafana	Monitoring
10	Loki	grafana/loki	Centralized Logging
11	Portainer	portainer/portainer-ce	Container Management


3. Assignment Architecture
Build the following environment:
                         WINDOWS
                            │
                           WSL2
                            │
                         UBUNTU
                            │
                         DOCKER
                            │
             ┌──────────────┴──────────────┐
             │                             │
          Gateway                      Management
             │                             │
          NGINX                       Portainer
             │
     ┌───────┼────────┬──────────┐
     │       │        │          │
     ▼       ▼        ▼          ▼
 PostgreSQL Redis    Kafka      Ollama
     │                  │
 pgvector               │
     │                  │
 pgAdmin             Agents
     │
     └──────────┐
                ▼
             Qdrant

          OBSERVABILITY
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
  Prometheus Grafana   Loki

PART 1 — Docker Installation
Task 1: Verify WSL
Execute:
wsl --version

Then:
lsb_release -a

Evidence
Take a screenshot showing:
Ubuntu version
WSL version

PART 2 — Docker Verification
Task 2: Check Docker
Run:
docker --version

docker info

docker ps

Expected result
Docker should respond without errors.
Record
Create:
docker-environment.txt

containing:
OS:
WSL:
Docker:
CPU:
Memory:
Docker Root Dir:

PART 3 — Create Project Structure
Task 3
Create:
mkdir -p ~/enterprise-ai-platform
cd ~/enterprise-ai-platform

Then:
mkdir -p \
docker \
nginx \
postgres \
pgadmin \
redis \
qdrant \
ollama \
kafka \
prometheus \
grafana \
loki \
portainer \
scripts \
tests \
docs

Verify:
tree

PART 4 — Docker Network
Task 4
Create an enterprise network:
docker network create ai-platform-net

Verify:
docker network ls

Inspect:
docker network inspect ai-platform-net

Question
Explain why all application containers should not communicate using localhost.
PART 5 — PostgreSQL + pgvector
Task 5 — Pull Image
docker pull pgvector/pgvector:pg16

Verify:
docker images | grep pgvector

Task 6 — Create PostgreSQL Volume
docker volume create postgres_data

Verify:
docker volume ls

Task 7 — Start PostgreSQL
docker run -d \
  --name postgres \
  --network ai-platform-net \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=agentdb \
  -p 5432:5432 \
  -v postgres_data:/var/lib/postgresql/data \
  pgvector/pgvector:pg16

Check:
docker ps

Task 8 — Test PostgreSQL
docker exec -it postgres psql -U postgres -d agentdb

Execute:
SELECT version();

Then:
CREATE EXTENSION IF NOT EXISTS vector;

Verify:
SELECT * FROM pg_extension WHERE extname = 'vector';

Exit:
\q

Evidence
Capture successful:
PostgreSQL connection
vector extension

PART 6 — pgAdmin
Task 9 — Pull Image
docker pull dpage/pgadmin4

Task 10 — Run pgAdmin
docker run -d \
  --name pgadmin \
  --network ai-platform-net \
  -p 5050:80 \
  -e PGADMIN_DEFAULT_EMAIL=admin@example.com \
  -e PGADMIN_DEFAULT_PASSWORD=Admin@12345 \
  dpage/pgadmin4

Check:
docker ps

Browser test
Open:
http://localhost:5050

Login:
Email:
admin@example.com

Password:
Admin@12345

PART 7 — pgAdmin → PostgreSQL
Task 11
In pgAdmin create a PostgreSQL server.
Use:
Host: postgres
Port: 5432
Database: agentdb
Username: postgres
Password: postgres

Important
Do not use:
localhost

for the PostgreSQL hostname from inside the pgAdmin container.
Test
Run:
SELECT current_database();

Then:
SELECT version();

Then:
SELECT extname
FROM pg_extension;

PART 8 — Redis
Task 12
Pull:
docker pull redis:latest

Run:
docker run -d \
  --name redis \
  --network ai-platform-net \
  -p 6379:6379 \
  redis:latest

Verify:
docker ps

Test:
docker exec -it redis redis-cli ping

Expected:
PONG

Then:
docker exec -it redis redis-cli SET enterprise "AI Platform"

docker exec -it redis redis-cli GET enterprise

Expected:
"AI Platform"

PART 9 — Qdrant
Task 13
Pull:
docker pull qdrant/qdrant

Create volume:
docker volume create qdrant_storage

Run:
docker run -d \
  --name qdrant \
  --network ai-platform-net \
  -p 6333:6333 \
  -p 6334:6334 \
  -v qdrant_storage:/qdrant/storage \
  qdrant/qdrant

Test:
http://localhost:6333

API test:
curl http://localhost:6333/collections

Expected response should indicate the Qdrant service is responding.
PART 10 — Ollama
Task 14
Pull:
docker pull ollama/ollama

Create volume:
docker volume create ollama_data

Run:
docker run -d \
  --name ollama \
  --network ai-platform-net \
  -p 11434:11434 \
  -v ollama_data:/root/.ollama \
  ollama/ollama

Check:
docker ps

Test:
curl http://localhost:11434/api/tags

Model test
Inside the container:
docker exec -it ollama ollama list

Pull an appropriate model for your hardware:
docker exec -it ollama ollama pull <model>

Test:
docker exec -it ollama ollama run <model>

Ask:
Explain enterprise microservices architecture.

Deliverable
Record:
Model:
Model size:
Response time:
RAM usage:

PART 11 — Nginx
Task 15
Pull:
docker pull nginx:1.29-alpine

Run:
docker run -d \
  --name api-gateway-nginx \
  --network ai-platform-net \
  -p 80:80 \
  nginx:1.29-alpine

Test:
http://localhost

Expected:
Welcome to nginx!

PART 12 — Kafka
Task 16
Deploy Kafka using an enterprise-appropriate Kafka image/configuration.
Requirements:
Kafka Broker
Kafka Controller
Persistent Storage
Internal Docker networking
External client access

Create topics:
orders.created
orders.updated
payments.requested
payments.completed
payments.failed
inventory.reservation.requested
inventory.reserved
inventory.rejected
notifications.requested
integration.retry
integration.dlq
audit.events

Verify:
docker ps

Kafka test
Produce:
Hello Enterprise AI

Consume the message.
Deliverable
Demonstrate:
Producer
   ↓
Kafka
   ↓
Consumer

PART 13 — Prometheus
Task 17
Pull:
docker pull prom/prometheus

Create configuration:
prometheus/prometheus.yml

Configure scraping for services that expose Prometheus-compatible metrics.
Run:
docker run -d \
  --name prometheus \
  --network ai-platform-net \
  -p 9090:9090 \
  prom/prometheus

Test:
http://localhost:9090

PART 14 — Grafana
Task 18
Pull:
docker pull grafana/grafana

Run:
docker volume create grafana_data

docker run -d \
  --name grafana \
  --network ai-platform-net \
  -p 3000:3000 \
  -v grafana_data:/var/lib/grafana \
  grafana/grafana

Open:
http://localhost:3000

Configure Prometheus as a datasource.
Create a dashboard containing:
CPU
Memory
Container availability
HTTP requests
HTTP latency
Database metrics
Redis metrics
Kafka metrics

PART 15 — Loki
Task 19
Pull:
docker pull grafana/loki

Deploy Loki with persistent storage/configuration.
Verify its HTTP endpoint.
Integrate:
Docker
  ↓
Loki
  ↓
Grafana

Create a Grafana dashboard showing application logs.
PART 16 — Portainer
Task 20
Pull:
docker pull portainer/portainer-ce

Create:
docker volume create portainer_data

Run:
docker run -d \
  --name portainer \
  --restart=always \
  -p 8000:8000 \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce

Open:
https://localhost:9443

Use Portainer to inspect:
- Containers
- Images
- Networks
- Volumes
- Logs
- CPU
- Memory
PART 17 — Final Service Verification
Create:
tests/service-test.sh

The script must test every service.
Minimum checks:
curl http://localhost
curl http://localhost:5050
curl http://localhost:6333/collections
curl http://localhost:11434/api/tags
curl http://localhost:9090
curl http://localhost:3000

Redis:
docker exec redis redis-cli ping

PostgreSQL:
docker exec postgres pg_isready

Kafka:
Kafka broker health check

Portainer:
https://localhost:9443

PART 18 — Enterprise Health Dashboard
Create a table:
Service	Port	Status	Test
Nginx	80	☐	HTTP
PostgreSQL	5432	☐	pg_isready
pgAdmin	5050	☐	Browser
Redis	6379	☐	PING
Qdrant	6333	☐	REST API
Ollama	11434	☐	API
Kafka	9092*	☐	Producer/Consumer
Prometheus	9090	☐	Browser
Grafana	3000	☐	Browser
Loki	3100	☐	API
Portainer	9443	☐	Browser


*Use the actual Kafka port from your deployment.
PART 19 — Failure Testing
This is an important enterprise-level requirement.
Test 1 — PostgreSQL failure
docker stop postgres

Verify that pgAdmin/database operations fail.
Restart:
docker start postgres

Verify recovery.
Test 2 — Redis failure
docker stop redis

Test the application.
Restart:
docker start redis

Test 3 — Ollama failure
docker stop ollama

Verify that the AI API reports the dependency failure correctly.
Restart:
docker start ollama

Test 4 — Container restart
Restart all application containers and verify persistent data.
For example:
docker restart postgres
docker restart redis
docker restart qdrant
docker restart grafana

Verify that previously created data remains available.
PART 20 — Persistence Test
Create PostgreSQL data:
CREATE TABLE enterprise_test (
    id SERIAL PRIMARY KEY,
    message TEXT
);

INSERT INTO enterprise_test(message)
VALUES ('Enterprise AI Platform Test');

Restart PostgreSQL:
docker restart postgres

Verify:
SELECT * FROM enterprise_test;

Expected
The data must still exist.
Repeat similar persistence testing for:
- Redis
- Qdrant
- Ollama
- Grafana
- Portainer
where applicable.
PART 21 — Docker Inspection
Run:
docker ps -a

docker images

docker volume ls

docker network ls

docker stats --no-stream

docker system df

Student must explain
1. Difference between image and container.
2. Difference between volume and bind mount.
3. Docker bridge/network concepts.
4. Container-to-container communication.
5. Port mapping.
6. Restart policies.
7. Health checks.
8. Persistent storage.
9. Container resource limits.
10. Docker security risks.
PART 22 — Final Enterprise Integration Test
The final workflow must demonstrate:
                    USER
                      │
                      ▼
                    NGINX
                      │
                      ▼
                  AI API
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       Redis       PostgreSQL   Kafka
          │           │           │
          │        pgvector       │
          │           │           │
          │           ▼           ▼
          │         RAG         Events
          │
          ▼
       Ollama
          │
          ▼
      AI Response

And observability:
Services
   │
   ├── Metrics ──→ Prometheus ──→ Grafana
   │
   └── Logs ─────→ Loki ────────→ Grafana

Final Deliverables
Students must submit:
enterprise-ai-docker-lab/
│
├── README.md
├── architecture/
│   └── enterprise-ai-architecture.png
│
├── docker/
│   ├── docker-compose.yml
│   └── .env.example
│
├── nginx/
│   └── nginx.conf
│
├── postgres/
│   └── init.sql
│
├── prometheus/
│   └── prometheus.yml
│
├── grafana/
│   └── dashboards/
│
├── loki/
│   └── loki-config.yml
│
├── kafka/
│   └── topics.sh
│
├── tests/
│   ├── service-test.sh
│   ├── persistence-test.sh
│   └── failure-test.sh
│
├── docs/
│   ├── installation.md
│   ├── troubleshooting.md
│   ├── security.md
│   └── operations-runbook.md
│
└── screenshots/
    ├── docker.png
    ├── pgadmin.png
    ├── redis.png
    ├── qdrant.png
    ├── ollama.png
    ├── kafka.png
    ├── prometheus.png
    ├── grafana.png
    ├── loki.png
    └── portainer.png

Assessment
Component	Marks
WSL/Docker installation	5
Docker networking	5
PostgreSQL + pgvector	10
pgAdmin	5
Redis	5
Qdrant	5
Ollama	10
Nginx	5
Kafka	10
Prometheus	5
Grafana	5
Loki	5
Portainer	5
Persistence testing	5
Failure/recovery testing	5
Documentation	10
Total	100


Final Acceptance Criteria
The assignment is considered complete only when the student can demonstrate:
✅ All required Docker images pulled
✅ All required containers running
✅ No unintended port conflicts
✅ All services accessible
✅ PostgreSQL + pgvector working
✅ pgAdmin connected to PostgreSQL
✅ Redis PING → PONG
✅ Qdrant API responding
✅ Ollama responding to an LLM request
✅ Kafka producer/consumer working
✅ Prometheus collecting metrics
✅ Grafana displaying metrics
✅ Loki receiving logs
✅ Portainer managing containers
✅ Persistent volumes verified
✅ Failure/recovery demonstrated
✅ Complete architecture documented# docker
