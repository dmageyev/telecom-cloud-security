# Лекція 5 — Список літератури та джерел

## Офіційна документація AWS

### AWS Key Management Service (KMS)
- [AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/)
- [KMS Key Policy Examples](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-examples.html)
- [KMS Multi-Region Keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html)
- [AWS KMS Pricing](https://aws.amazon.com/kms/pricing/)

### AWS Certificate Manager (ACM)
- [ACM User Guide](https://docs.aws.amazon.com/acm/latest/userguide/)
- [ACM Private CA](https://docs.aws.amazon.com/privateca/latest/userguide/)

### Шифрування сховищ
- [S3 Encryption](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html)
- [EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html)
- [RDS Encryption](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html)
- [DynamoDB Encryption at Rest](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EncryptionAtRest.html)

### AWS Secrets Manager та Parameter Store
- [Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/)
- [SSM Parameter Store User Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)

### AWS CloudHSM
- [CloudHSM User Guide](https://docs.aws.amazon.com/cloudhsm/latest/userguide/)
- [CloudHSM vs KMS — When to use which](https://aws.amazon.com/blogs/security/aws-cloudhsm-best-practices-for-planning-deployment/)

### AWS Nitro Enclaves
- [Nitro Enclaves User Guide](https://docs.aws.amazon.com/enclaves/latest/user/)
- [AWS Nitro Enclaves с KMS](https://docs.aws.amazon.com/enclaves/latest/user/kms.html)

### Encryption SDK та Crypto Tools
- [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)
- [AWS Crypto Tools на GitHub](https://github.com/aws/aws-encryption-sdk-python)

### Amazon Macie
- [Amazon Macie User Guide](https://docs.aws.amazon.com/macie/latest/user/)

### Моніторинг та відповідність
- [AWS Config Managed Rules](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
- [AWS Security Hub FSBP Standard](https://docs.aws.amazon.com/securityhub/latest/userguide/fsbp-standard.html)
- [AWS Artifact](https://aws.amazon.com/artifact/)

---

## Стандарти та RFC

### Криптографія
- [NIST FIPS 197 — AES](https://csrc.nist.gov/publications/detail/fips/197/final)
- [NIST SP 800-57 — Key Management](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final)
- [NIST SP 800-175B — Cryptographic Standards](https://csrc.nist.gov/publications/detail/sp/800-175b/rev-1/final)
- [RFC 5246 — TLS 1.2](https://datatracker.ietf.org/doc/html/rfc5246)
- [RFC 8446 — TLS 1.3](https://datatracker.ietf.org/doc/html/rfc8446)

### Post-Quantum Cryptography
- [NIST PQC Standardization — FIPS 203 (ML-KEM/Kyber)](https://csrc.nist.gov/pubs/fips/203/final)
- [NIST PQC Standardization — FIPS 204 (ML-DSA/Dilithium)](https://csrc.nist.gov/pubs/fips/204/final)
- [NIST PQC Standardization — FIPS 205 (SLH-DSA/SPHINCS+)](https://csrc.nist.gov/pubs/fips/205/final)

### Телекомунікації та безпека
- [3GPP TS 33.501 — 5G Security](https://www.3gpp.org/DynaReport/33501.htm)
- [GSMA FS.16 — Cloud Security Guidelines for MVNO](https://www.gsma.com/newsroom/all-documents/fs-16-cloud-security-guidelines/)
- [ETSI TS 102 165-2 — TVRA Telecom Security](https://www.etsi.org/deliver/etsi_ts/102100_102199/10216502/)

### Регуляторні вимоги
- [GDPR Офіційний текст](https://gdpr.eu/tag/gdpr/)
- [PCI DSS v4.0](https://www.pcisecuritystandards.org/document_library/)
- [ISO/IEC 27001:2022](https://www.iso.org/standard/82875.html)
- [HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html)

---

## Рекомендована література

### Книги
- *AWS Security Cookbook* — Heartin Kanikathottu (Packt, 2021)
- *Hacking the Cloud: AWS Security Fundamentals* — Scott Piper (O'Reilly, 2023)
- *Cryptography and Network Security* — William Stallings (Pearson, 8th ed., 2022)
- *Serious Cryptography* — Jean-Philippe Aumasson (No Starch Press, 2017)
- *Cloud Security Alliance: Security Guidance for Critical Areas of Focus in Cloud Computing v4.0*

### Онлайн-курси та сертифікації
- [AWS Certified Security – Specialty (SCS-C02)](https://aws.amazon.com/certification/certified-security-specialty/)
- [AWS Well-Architected Framework — Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
- [AWS re:Invent talks on YouTube: KMS & Data Protection](https://www.youtube.com/@AWSEventsChannel)

---

## Корисні блоги та ресурси

- [AWS Security Blog](https://aws.amazon.com/blogs/security/)
- [AWS What's New — KMS & Security](https://aws.amazon.com/about-aws/whats-new/security-identity-compliance/)
- [CloudSecDocs.com — AWS Encryption Best Practices](https://cloudsecdocs.com/)
- [OWASP Cryptographic Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [Krebs on Security — Cloud Breaches Analysis](https://krebsonsecurity.com/)
