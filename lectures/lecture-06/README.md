# Лекція 6: Безпека обчислювальних сервісів і контейнерів

**Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
**Модуль:** 2 — Захист сервісів, інциденти та відповідність  
**Аудиторія:** Магістри спеціальності «Кібербезпека»  
**Тривалість:** 2 академічні години (90 хв)

---

## Мета лекції

Сформувати у студентів систематичне розуміння безпеки хмарних обчислювальних сервісів: від класичних віртуальних машин EC2 до сучасних контейнерних платформ та serverless-архітектур. Студенти навчаться застосовувати принципи захисту на кожному рівні стеку AWS.

---

## Цілі навчання

Після завершення лекції студент зможе:

- Пояснити модель спільної відповідальності для EC2, ECS, EKS, Fargate та Lambda
- Налаштувати безпечну EC2-інфраструктуру з IMDSv2, SSM та Patch Manager
- Написати безпечний Dockerfile з non-root, multi-stage build та .dockerignore
- Впровадити сканування вразливостей образів у CI/CD pipeline (Trivy, ECR)
- Налаштувати RBAC, Pod Security Standards та Network Policies у Kubernetes
- Застосувати Admission Controllers (Kyverno/OPA) для Policy as Code
- Безпечно управляти секретами через AWS Secrets Manager та External Secrets Operator
- Описати типові атаки на контейнери: container escape, supply chain, SSRF→IMDS
- Організувати реагування на інциденти в ефемерних контейнерних середовищах
- Застосувати принципи Zero Trust та Service Mesh для мікросегментації

---

## Структура лекції (37 слайдів)

| # | Тема | Ключові концепції |
|---|------|-------------------|
| 1 | Титульний слайд | — |
| 2 | План лекції | 16 тем |
| 3 | Огляд обчислювальних сервісів AWS | EC2, ECS, EKS, Fargate, Lambda |
| 4 | Модель спільної відповідальності | AWS vs клієнт для кожного сервісу |
| 5 | Безпека EC2 | AMI, IMDSv2, SSH best practices |
| 6 | EC2: SSM та Patch Manager | Session Manager, Patch Baseline |
| 7 | Security Groups та NACL | Stateful vs stateless, Defense in Depth |
| 8 | VPC: безпечна архітектура | Subnets, VPC Endpoints, PrivateLink |
| 9 | Основи контейнеризації | Namespaces, cgroups, seccomp, VM vs Container |
| 10 | Docker: архітектура та безпека | Daemon, socket, non-root, capabilities |
| 11 | Безпечний Dockerfile | non-root, ENV secrets, помилки |
| 12 | Multi-stage Build та .dockerignore | distroless, секрети у build context |
| 13 | Сканування образів | Trivy, ECR, Inspector, pipeline |
| 14 | SBOM та Image Signing | CycloneDX, cosign, SLSA |
| 15 | DevSecOps CI/CD Pipeline | Shift Left, GitHub Actions, SARIF |
| 16 | Kubernetes: архітектура та RBAC | Control Plane, YAML-приклади |
| 17 | Pod Security та Network Policies | PSS, default-deny |
| 18 | Admission Controllers | OPA Gatekeeper, Kyverno |
| 19 | Kubernetes Secrets шифрування | Envelope encryption, External Secrets |
| 20 | Amazon EKS | IRSA, приватний endpoint, kube-bench |
| 21 | EKS: безпека вузлів | Bottlerocket OS, node IAM role |
| 22 | Amazon ECS та Fargate | Task Definition, mVM ізоляція |
| 23 | Lambda: безпека | Firecracker, ризики, валідація |
| 24 | Lambda: VPC та Resource Policy | Мережева ізоляція, SourceArn |
| 25 | Управління секретами | Secrets Manager vs SSM, антипатерни |
| 26 | IaC безпека | Checkov, tfsec, AWS Config Rules |
| 27 | Типові атаки на контейнери | Таблиця: escape, poisoning, SSRF, crypto |
| 28 | Container Escape | CVE-2019-5736, seccomp, gVisor |
| 29 | Supply Chain: Log4Shell | CVE-2021-44228, SBOM-захист |
| 30 | Моніторинг та виявлення загроз | GuardDuty, Falco, CloudTrail |
| 31 | Incident Response для контейнерів | IR workflow, forensics |
| 32 | Zero Trust | mTLS, Istio, AWS Verified Access |
| 33 | CIS Benchmarks та Compliance | NIST SP 800-190, kube-bench, чек-лист |
| 34 | Кращі практики | EC2, Containers, Lambda |
| 35 | Практичне завдання | 3 завдання |
| 36 | Підсумок | Зв'язок з іншими лекціями |
| 37 | Питання | — |

---

## Файли

| Файл | Опис |
|------|------|
| [`_06.html`](./_06.html) | Reveal.js презентація (37 слайдів) |
| [`README.md`](./README.md) | Цей файл — огляд та навігація |
| [`references.md`](./references.md) | Бібліографія, посилання, стандарти |
| [`notes.md`](./notes.md) | Конспект лекції з детальними поясненнями (Markdown) |
| [`notes.html`](./notes.html) | Конспект лекції у форматі HTML |

---

## Як відкрити презентацію

```bash
# Локальний HTTP-сервер (Python)
cd lectures/lecture-06
python3 -m http.server 8080
# Відкрити: http://localhost:8080/_06.html
```

Або відкрийте `_06.html` безпосередньо в браузері.

**Навігація в презентації:**
- `→` або `Space` — наступний слайд
- `←` — попередній слайд
- `F` — повноекранний режим
- `S` — режим доповідача
- `Esc` або `O` — огляд усіх слайдів

---

## Передумови

Студенти мають бути знайомі з:
- Базовими концепціями AWS (IAM, VPC, S3) — Лекції 1–4
- Принципами мережевої безпеки — Лекція 3
- Основами шифрування та KMS — Лекція 5

---

## Зв'язки з іншими темами

- **← Лекція 5:** Захист даних і криптографія (KMS для шифрування томів та секретів)
- **→ Лекція 7:** Безпека веб- та телеком-застосунків (WAF, API Gateway)
- **→ Лекція 8:** Моніторинг, логування та виявлення загроз (GuardDuty, CloudTrail, Falco)
