# Бібліографія та посилання — Лекція 6

## Офіційна документація AWS

| Ресурс | URL |
|--------|-----|
| EC2 Security Best Practices | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security.html |
| IMDSv2 User Guide | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html |
| AWS Systems Manager (SSM) | https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html |
| SSM Patch Manager | https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-patch.html |
| Amazon VPC Security | https://docs.aws.amazon.com/vpc/latest/userguide/infrastructure-security.html |
| VPC Endpoints | https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html |
| Amazon ECR Image Scanning | https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html |
| Amazon EKS Security Best Practices | https://docs.aws.amazon.com/eks/latest/userguide/security.html |
| EKS Best Practices Guide | https://aws.github.io/aws-eks-best-practices/security/docs/ |
| IRSA (IAM Roles for Service Accounts) | https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html |
| Amazon ECS Security | https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/security.html |
| AWS Fargate Security | https://docs.aws.amazon.com/AmazonECS/latest/userguide/fargate-security.html |
| AWS Lambda Security | https://docs.aws.amazon.com/lambda/latest/dg/lambda-security.html |
| AWS Secrets Manager | https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html |
| AWS Security Hub | https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html |
| Amazon GuardDuty | https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html |
| Amazon Inspector | https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html |

---

## Стандарти та нормативні документи

| Документ | Опис |
|----------|------|
| **NIST SP 800-190** | Application Container Security Guide (2017). Вичерпний посібник NIST з безпеки контейнерів. https://csrc.nist.gov/publications/detail/sp/800-190/final |
| **NIST EO 14028** | Executive Order on Improving the Nation's Cybersecurity (2021) — вимоги до SBOM. https://www.nist.gov/system/files/documents/2021/10/19/software_security_in_supply_chain_cisa.pdf |
| **CIS Docker Benchmark** | CIS Benchmark for Docker. https://www.cisecurity.org/benchmark/docker |
| **CIS Kubernetes Benchmark** | CIS Benchmark for Kubernetes. https://www.cisecurity.org/benchmark/kubernetes |
| **CIS Amazon EKS Benchmark** | https://www.cisecurity.org/benchmark/kubernetes |
| **PCI DSS v4.0** | Payment Card Industry Data Security Standard. Вимоги до контейнерних середовищ. https://www.pcisecuritystandards.org/ |
| **ISO/IEC 27001** | Information Security Management Systems. |
| **SLSA Framework** | Supply-chain Levels for Software Artifacts (Google). https://slsa.dev/ |

---

## Книги та навчальні матеріали

| Назва | Автор | Рік | Примітка |
|-------|-------|-----|----------|
| *Container Security* | Liz Rice | 2020 | O'Reilly. Найповніший посібник з безпеки контейнерів |
| *Hacking Kubernetes* | Andrew Martin, Michael Hausenblas | 2022 | O'Reilly. Атаки та захист K8s |
| *AWS Security Cookbook* | Heartin Kanikathottu | 2020 | Packt. Практичні рецепти безпеки AWS |
| *Kubernetes Security and Observability* | Brendan Creane, Amit Gupta | 2022 | O'Reilly |
| *Cloud Native Security* | Chris Binnie, Rory McCune | 2021 | Wiley |

---

## Інструменти

### Сканування образів та SBOM
| Інструмент | URL | Опис |
|-----------|-----|-------|
| **Trivy** | https://github.com/aquasecurity/trivy | CVE, secrets, IaC — комбінований сканер |
| **Grype** | https://github.com/anchore/grype | Швидкий CVE-сканер з SBOM-підтримкою |
| **Syft** | https://github.com/anchore/syft | Генерація SBOM (CycloneDX, SPDX) |
| **Snyk** | https://snyk.io | SaaS-платформа для security scanning |
| **Clair** | https://github.com/quay/clair | API-орієнтований сканер образів |

### Docker та Dockerfile
| Інструмент | URL | Опис |
|-----------|-----|-------|
| **Hadolint** | https://github.com/hadolint/hadolint | Lint для Dockerfile |
| **docker-bench-security** | https://github.com/docker/docker-bench-security | CIS Docker Benchmark автоматично |
| **Dockle** | https://github.com/goodwithtech/dockle | Кращі практики для образів |

### Kubernetes Security
| Інструмент | URL | Опис |
|-----------|-----|-------|
| **kube-bench** | https://github.com/aquasecurity/kube-bench | CIS Kubernetes Benchmark |
| **kube-hunter** | https://github.com/aquasecurity/kube-hunter | Пошук вразливостей кластера |
| **Falco** | https://falco.org | Runtime security для K8s/containers |
| **OPA Gatekeeper** | https://open-policy-agent.github.io/gatekeeper | Policy as Code для K8s |
| **Kyverno** | https://kyverno.io | K8s-нативний Policy as Code |
| **External Secrets Operator** | https://external-secrets.io | Синхронізація секретів AWS → K8s |

### IaC Scanning
| Інструмент | URL | Опис |
|-----------|-----|-------|
| **Checkov** | https://github.com/bridgecrewio/checkov | TF, CF, K8s, ARM — найбільша база |
| **tfsec** | https://github.com/aquasecurity/tfsec | Спеціалізований для Terraform |
| **cfn-guard** | https://github.com/aws-cloudformation/cloudformation-guard | AWS-нативний для CloudFormation |

### Image Signing
| Інструмент | URL | Опис |
|-----------|-----|-------|
| **cosign** | https://github.com/sigstore/cosign | Sigstore image signing |
| **Notary v2** | https://github.com/notaryproject/notation | CNCF image signing standard |

---

## CVE та інциденти, що розглядаються на лекції

| CVE / Інцидент | Рік | Опис |
|----------------|-----|-------|
| **CVE-2019-5736** | 2019 | runc container escape — перезапис бінарника через /proc/self/exe |
| **CVE-2021-44228** (Log4Shell) | 2021 | RCE у Apache Log4j 2.x через JNDI lookup |
| **SolarWinds supply chain attack** | 2020 | Компроміс збірки ПЗ — шкідливий код у підписаному оновленні |
| **Capital One breach** | 2019 | SSRF → IMDSv1 → AWS credentials витік |
| **Tesla cryptojacking** | 2018 | Відкритий Kubernetes dashboard → cryptomining |

---

## AWS Academy та лабораторні роботи

- **AWS Academy Cloud Foundations** — Module 6: Compute
- **AWS Academy Security Foundations** — Module 4: Container Security
- **AWS Skill Builder**: *Amazon EKS — Architecture and Security*
- **AWS Skill Builder**: *Amazon ECS — Security Best Practices*
- **AWS Skill Builder**: *AWS Lambda Security*

---

## Корисні онлайн-ресурси

- [AWS Security Blog](https://aws.amazon.com/blogs/security/) — офіційний блог безпеки AWS
- [CNCF Security TAG](https://github.com/cncf/tag-security) — документи з хмарної безпеки
- [Kubernetes Security Documentation](https://kubernetes.io/docs/concepts/security/)
- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- [Falco Rules Repository](https://github.com/falcosecurity/rules)
