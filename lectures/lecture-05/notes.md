# Лекція 5 — Захист даних і криптографія в AWS: Детальний конспект

> **Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
> **Лекція:** 5 з 11 | **Модуль:** 1

---

## 1. Три стани даних та їх захист

Кожні дані в системі існують в одному з трьох станів, і для кожного стану існують специфічні механізми захисту.

### 1.1 Дані в спокої (Data at Rest)

**Визначення:** Дані, що зберігаються на фізичних або логічних носіях і наразі не передаються.

**Приклади:** файли на дисках, записи в БД, архіви, резервні копії, snapshots.

**Механізми захисту:**
- Шифрування дисків (Full Disk Encryption)
- Шифрування на рівні файлів або об'єктів
- Шифрування на рівні БД або таблиці
- Управління ключами (KMS, CloudHSM)

**Ризики без захисту:** фізична крадіжка диску, несанкціонований доступ адміністратора, витік через backup.

### 1.2 Дані у русі (Data in Transit)

**Визначення:** Дані, що передаються мережею між компонентами, сервісами або кінцевими точками.

**Приклади:** HTTP-запити, SQL-запити до БД, передача файлів між сервісами, API-виклики.

**Механізми захисту:**
- TLS/SSL (Transport Layer Security) — стандартний протокол шифрування каналу зв'язку
- VPN (Virtual Private Network) — зашифрований тунель між мережами
- HTTPS — HTTP поверх TLS
- mTLS (Mutual TLS) — двостороння автентифікація

**Ризики без захисту:** man-in-the-middle атаки, прослуховування трафіку, replay attacks.

### 1.3 Дані у використанні (Data in Use)

**Визначення:** Дані, що активно обробляються процесором або завантажені в оперативну пам'ять.

**Механізми захисту:**
- Confidential Computing (TEE — Trusted Execution Environment)
- AWS Nitro Enclaves — ізольоване середовище виконання
- Intel SGX, AMD SEV — апаратні технології конфіденційних обчислень

**Ризики без захисту:** атаки на пам'ять (memory scraping), privileged access abuse, cold boot attacks.

### 1.4 Тріада CIA

| Властивість | Визначення | Механізм захисту |
|-------------|-----------|------------------|
| **Конфіденційність** | Лише авторизовані суб'єкти мають доступ | Шифрування, IAM, ACL |
| **Цілісність** | Дані не змінено несанкціоновано | Хешування, цифрові підписи, HMAC |
| **Доступність** | Дані доступні авторизованим суб'єктам у потрібний час | Резервування, Replication, Backup |

---

## 2. Симетрична криптографія

### 2.1 Принцип роботи

Симетрична криптографія використовує **один і той самий ключ** для шифрування та дешифрування. Основна математична операція — побітові перетворення з ключем.

```
Шифрування: Encrypt(Plaintext, Key) → Ciphertext
Дешифрування: Decrypt(Ciphertext, Key) → Plaintext
```

### 2.2 AES (Advanced Encryption Standard)

AES — міжнародний стандарт симетричного шифрування, затверджений NIST у 2001 р. (FIPS 197).

**Характеристики:**
- Блоковий шифр з фіксованим розміром блоку 128 біт
- Підтримує ключі 128, 192, 256 біт
- Замінив DES (56 біт) та 3DES (112/168 біт)

**Режими роботи:**
- **AES-GCM (Galois/Counter Mode)** — найрекомендованіший, поєднує шифрування та автентифікацію (AEAD). Використовується в AWS KMS та TLS 1.3.
- **AES-CBC (Cipher Block Chaining)** — блоковий режим, потребує padding, вразливий до padding oracle атак без HMAC.
- **AES-CTR (Counter Mode)** — перетворює блоковий шифр в потоковий, паралелізується.

**Апаратна підтримка:** процесори Intel (Haswell+), AMD та ARM мають AES-NI інструкції, які прискорюють AES в 3-10 разів.

### 2.3 Проблема розповсюдження ключів

Головна проблема симетричного шифрування: якщо N сторін хочуть спілкуватися парами, потрібно N×(N-1)/2 унікальних ключів. Для 100 сторін — 4950 ключів. Безпечний обмін ключами через незахищений канал — нетривіальна задача, вирішена алгоритмом Діффі-Геллмана.

---

## 3. Асиметрична криптографія

### 3.1 Принцип роботи

Асиметрична криптографія використовує **пару ключів**: відкритий (public) та закритий (private). Математична основа — односторонні функції (задачі, легкі в один бік, практично нерозв'язні у зворотному).

- **Відкритий ключ** — можна публічно поширювати; використовується для шифрування або перевірки підпису
- **Закритий ключ** — зберігається в таємниці; використовується для дешифрування або підпису

### 3.2 RSA

RSA базується на складності факторизації великих цілих чисел. Ключ RSA-2048 вважається безпечним для цивільних застосувань, RSA-4096 — для довгострокового зберігання.

**Проблема RSA:** повільний, не підходить для шифрування великих даних. На практиці RSA шифрує лише симетричний ключ (Key Encapsulation).

### 3.3 Еліптичні криві (ECC)

ECDSA (Elliptic Curve Digital Signature Algorithm) та ECDH (Elliptic Curve Diffie-Hellman) базуються на задачі дискретного логарифму на еліптичних кривих.

**Переваги перед RSA:**
- Менший розмір ключа при однаковому рівні безпеки: EC-256 ≈ RSA-3072
- Швидша генерація підписів та обмін ключами
- AWS KMS підтримує: ECC_NIST_P256, ECC_NIST_P384, ECC_NIST_P521

### 3.4 Алгоритм Діффі-Геллмана

Дозволяє двом сторонам виробити спільний секрет через **незахищений канал**, не передаючи сам секрет:

1. Аліса та Боб домовляються про публічні параметри (g, p)
2. Аліса вибирає секрет `a`, обчислює `A = g^a mod p`, надсилає Бобу `A`
3. Боб вибирає секрет `b`, обчислює `B = g^b mod p`, надсилає Алісі `B`
4. Аліса обчислює `K = B^a mod p`, Боб обчислює `K = A^b mod p`
5. Обидва отримали однаковий спільний секрет `K = g^(ab) mod p`

**ECDH** — ефективніша версія на еліптичних кривих, використовується в TLS 1.3.

---

## 4. Хешування та цілісність даних

### 4.1 Криптографічні хеш-функції

**Властивості:**
- **Детермінованість:** однакові вхідні дані → однаковий хеш
- **Необоротність:** з хешу практично неможливо відновити вхідні дані
- **Лавинний ефект:** 1 біт зміни вхідних даних → ~50% бітів хешу змінюються
- **Стійкість до колізій:** практично неможливо знайти два різних входи з однаковим хешем

### 4.2 Алгоритми

| Алгоритм | Розмір хешу | Статус | Примітка |
|----------|------------|--------|----------|
| MD5 | 128 біт | **Застарілий** | Колізії знайдено у 2004 р. |
| SHA-1 | 160 біт | **Застарілий** | Практична колізія у 2017 р. (SHAttered) |
| SHA-256 | 256 біт | ✅ Рекомендований | Частина SHA-2, використовується в Bitcoin |
| SHA-384 | 384 біт | ✅ Рекомендований | Для більш чутливих застосувань |
| SHA-512 | 512 біт | ✅ Рекомендований | Максимальний рівень безпеки SHA-2 |
| SHA3-256 | 256 біт | ✅ Найсучасніший | Keccak, NIST 2015, різна архітектура від SHA-2 |

### 4.3 HMAC (Hash-based Message Authentication Code)

HMAC поєднує хешування з симетричним ключем для забезпечення як **цілісності**, так і **автентичності**:

```
HMAC(key, message) = Hash(key ⊕ opad || Hash(key ⊕ ipad || message))
```

**Застосування в AWS:**
- **SigV4:** підпис API-запитів до AWS використовує HMAC-SHA256. Кожен запит підписується з включенням timestamp, щоб запобігти replay атакам.
- **CloudTrail Log File Validation:** SHA-256 хеші для кожного лог-файлу, ланцюжок хешів для виявлення модифікації.

### 4.4 Цифровий підпис

```
Підпис:    Sign(PrivateKey, Hash(Message)) → Signature
Перевірка: Verify(PublicKey, Message, Signature) → true/false
```

Цифровий підпис надає **аутентичність** (підписав власник приватного ключа) та **невідмовність** (підписувач не може заперечити факт підпису).

---

## 5. Конвертне шифрування (Envelope Encryption)

### 5.1 Проблема прямого шифрування через KMS

AWS KMS API має обмеження:
- Максимальний розмір даних для прямого шифрування: **4 КБ**
- Кожен KMS API-виклик платний (~$0.03 за 10,000 викликів)
- Мережева затримка при кожному виклику: 5-20 мс

### 5.2 Рішення: дворівневе шифрування

**Ключі:**
- **DEK (Data Encryption Key)** — симетричний AES-256 ключ для шифрування самих даних
- **KEK (Key Encryption Key)** / **CMK** — KMS-ключ для шифрування DEK

**Алгоритм шифрування:**
```
1. KMS.GenerateDataKey(CMK) → (PlaintextDEK, EncryptedDEK)
2. AES_GCM_Encrypt(PlaintextDEK, PlaintextData) → CiphertextData
3. Зберегти: CiphertextData + EncryptedDEK (PlaintextDEK видалити з пам'яті!)
```

**Алгоритм дешифрування:**
```
1. KMS.Decrypt(CMK, EncryptedDEK) → PlaintextDEK
2. AES_GCM_Decrypt(PlaintextDEK, CiphertextData) → PlaintextData
3. PlaintextDEK видалити з пам'яті
```

**Ключові переваги:**
- Дані будь-якого розміру шифруються локально (швидко, без мережевих затримок)
- KMS викликається лише 1 раз при дешифруванні (незалежно від розміру даних)
- При компрометації DEK — достатньо перешифрувати лише DEK через `ReEncrypt`

---

## 6. AWS Key Management Service (KMS)

### 6.1 Огляд

AWS KMS — централізований сервіс управління криптографічними ключами, інтегрований із понад 100 AWS-сервісами. Побудований на апаратних модулях безпеки (HSM), сертифікованих за FIPS 140-2 Level 3.

### 6.2 Типи KMS Keys

| Тип | Хто керує | Ротація | Ваша Key Policy | CloudTrail | Ціна |
|-----|-----------|---------|-----------------|------------|------|
| Customer Managed Key (CMK) | Ви | Ручна/авто (1 рік) | ✅ | ✅ | $1/міс + API |
| AWS Managed Key | AWS | Авто (1 рік) | ❌ | ✅ | Безкоштовно |
| AWS Owned Key | AWS (внутрішній) | Авто | ❌ | ❌ | Безкоштовно |

**Коли використовувати CMK:**
- Потрібен аудит (CloudTrail) кожного шифрування/дешифрування
- Потрібно надати крос-акаунт доступ до зашифрованих даних
- Потрібна власна key policy (обмеження, умови)
- Відповідність вимогам: GDPR, PCI DSS, HIPAA

### 6.3 Key Policy

Key Policy — це документ JSON, прикріплений до ключа, що визначає хто і що може робити з цим ключем. На відміну від IAM-політик, Key Policy обов'язково повинна явно дозволяти кореневому акаунту доступ — інакше ключ стане недоступним.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::123456789012:root" },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow App to use the key",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::123456789012:role/AppRole" },
      "Action": ["kms:Decrypt", "kms:GenerateDataKey"],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": "s3.eu-west-1.amazonaws.com"
        }
      }
    }
  ]
}
```

### 6.4 Ротація ключів

При автоматичній ротації AWS генерує **новий матеріал ключа**, але Key ID залишається незмінним. Старий матеріал зберігається і використовується для дешифрування раніше зашифрованих даних. Немає необхідності перешифровувати існуючі дані при ротації.

### 6.5 KMS Multi-Region Keys

Multi-Region Keys (MRK) — один логічний ключ, матеріал якого реплікується в кілька регіонів AWS. Ключові властивості:

- Однаковий Key ID (`mrk-...`) в усіх регіонах
- Дані, зашифровані в одному регіоні, можна розшифрувати в будь-якому регіоні з реплікою
- Підходить для Active-Active DR та cross-region backup без перешифрування

---

## 7. Шифрування Amazon S3

### 7.1 Опції серверного шифрування (SSE)

**SSE-S3 (Server-Side Encryption with S3-Managed Keys)**
- AWS генерує унікальний ключ для кожного об'єкта
- Ключ шифрується майстер-ключем, яким керує AWS
- AES-256; включено за замовчуванням з 2023 р.
- Безкоштовно; немає видимості ключів у CloudTrail

**SSE-KMS (Server-Side Encryption with KMS Keys)**
- Кожен об'єкт шифрується DEK, захищеним KMS-ключем
- Підтримує AWS Managed Key або CMK
- Кожне дешифрування — виклик KMS API (платно, ~$0.03/10K)
- Аудит кожного дешифрування в CloudTrail
- Bucket Key оптимізація: зменшує кількість KMS API викликів у 99%

**SSE-C (Server-Side Encryption with Customer-Provided Keys)**
- Клієнт передає ключ у заголовку кожного запиту
- AWS шифрує об'єкт та зразу видаляє ключ
- Потребує HTTPS (обов'язково)
- Немає управління ключами з боку AWS

**CSE (Client-Side Encryption)**
- Дані шифруються до відправлення в S3
- AWS ніколи не бачить plaintext дані
- Використовується AWS Encryption SDK або власна реалізація
- Максимальний контроль та конфіденційність

### 7.2 S3 Object Lock

Object Lock реалізує модель **WORM (Write Once, Read Many)**:

- **Governance mode:** захист від видалення, але адміністратори з `s3:BypassGovernanceRetention` можуть обійти
- **Compliance mode:** ніхто (навіть root) не може видалити або змінити об'єкт до закінчення retention period
- Використовується для виконання вимог регуляторів щодо незмінності даних (SEC 17a-4, CFTC)

---

## 8. AWS Encryption SDK

AWS Encryption SDK — клієнтська бібліотека для реалізації конвертного шифрування. Забезпечує стандартизований формат зашифрованого повідомлення, який містить:

- Зашифровані копії DEK (по одній для кожного CMK)
- Алгоритм шифрування та параметри
- Authenticated encryption data (GCM authentication tag)

**Key Commitment** — гарантія, що зашифровані дані можуть бути розшифровані лише ключем, яким були зашифровані. Захищає від атак на підміну ключа (key confusion attacks).

---

## 9. Шифрування EBS та RDS

### 9.1 Amazon EBS Encryption

Шифрування EBS-томів відбувається на рівні інфраструктури AWS, **прозоро для EC2-інстансу**. Для EC2 том виглядає як звичайний блоковий пристрій.

**Що шифрується:**
- Дані на томі
- Усі snapshots тому
- Усі томи, створені з зашифрованого snapshot

**Важливо:** Шифрування вмикається **при створенні** тому і не може бути змінене пізніше. Для шифрування існуючого тому: snapshot → зашифрований snapshot → новий том.

**Увімкнення за замовчуванням:** в Налаштуваннях EC2 регіону можна увімкнути "EBS Encryption by default" — усі нові томи автоматично шифруватимуться.

### 9.2 Amazon RDS Encryption

Аналогічно EBS, шифрування вмикається при створенні інстансу. Шифруються: сховище, automated backups, snapshots, read replicas.

**Примусовий SSL/TLS у PostgreSQL:**
```sql
-- Параметр в Parameter Group:
rds.force_ssl = 1

-- Перевірка поточного з'єднання:
SELECT ssl_is_used();
```

**Примусовий SSL у MySQL/Aurora MySQL:**
```sql
-- Параметр в Parameter Group:
require_secure_transport = ON

-- Перевірка:
SHOW STATUS LIKE 'Ssl_cipher';
```

---

## 10. Захист даних у русі: TLS/SSL

### 10.1 TLS Handshake

TLS 1.3 (RFC 8446) спрощено порівняно з TLS 1.2 — handshake займає лише 1 round-trip замість 2:

```
Client                              Server
  |-- ClientHello (TLS 1.3) -------> |
  |   (key_share: public ECDH key)   |
  |                                  |
  | <-- ServerHello ---------------  |
  |     (key_share: server ECDH key) |
  |     Certificate + CertVerify     |
  |     Finished                     |
  |                                  |
  |-- Finished (encrypted) -------->  |
  |== Application Data (encrypted) == |
```

### 10.2 Cipher Suites для TLS 1.3

TLS 1.3 підтримує лише 5 безпечних cipher suites (усі з Forward Secrecy):
- `TLS_AES_256_GCM_SHA384`
- `TLS_AES_128_GCM_SHA256`
- `TLS_CHACHA20_POLY1305_SHA256`

### 10.3 Perfect Forward Secrecy (PFS)

PFS — властивість, при якій компрометація довгострокового приватного ключа (сертифіката) не дозволяє розшифрувати раніше захоплений трафік. Забезпечується за рахунок тимчасових (ephemeral) ECDH-ключів для кожної сесії.

---

## 11. AWS Certificate Manager (ACM)

### 11.1 Публічні сертифікати

ACM видає безкоштовні публічні сертифікати DV (Domain Validated) для доменів, підтверджених через:
- **DNS validation** (рекомендовано): додавання CNAME-запису до DNS
- **Email validation:** підтвердження на email домену

**Обмеження ACM-сертифікатів:**
- Не можна завантажити (export) — private key залишається в ACM
- Не підходять для встановлення на EC2 напряму (для EC2 — Let's Encrypt або власний CA)
- Підходять для: ALB, NLB, CloudFront, API Gateway, App Runner, AppSync

### 11.2 ACM Private CA

Власний ієрархічний PKI для внутрішніх сертифікатів. Архітектура:

```
Root CA (офлайн або CloudHSM)
    └── Intermediate CA (ACM Private CA)
            ├── Service certificates (mTLS)
            ├── Client certificates (VPN)
            └── Device certificates (IoT)
```

---

## 12. AWS Secrets Manager

### 12.1 Автоматична ротація

Ротація відбувається за Lambda-функцією, яка виконує 4 кроки:

1. **createSecret** — генерація нового значення секрету
2. **setSecret** — встановлення нового значення в цільовому сервісі (наприклад, зміна пароля RDS)
3. **testSecret** — перевірка, що нове значення працює
4. **finishSecret** — позначення нового значення як AWSCURRENT (старе стає AWSPREVIOUS)

AWS надає готові Lambda-шаблони для ротації: RDS MySQL, RDS PostgreSQL, RDS Oracle, Redshift, DocumentDB.

### 12.2 Найкращі практики отримання секретів

```python
import boto3
import json
from functools import lru_cache

# Кешувати секрет всередині Lambda execution environment
# (не викликати Secrets Manager на кожний запит)
@lru_cache(maxsize=1)
def get_secret(secret_name: str) -> dict:
    client = boto3.client('secretsmanager', region_name='eu-west-1')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])
```

**Антипатерн:** жорстке кодування секретів у змінних середовища або коді.

---

## 13. AWS CloudHSM

### 13.1 Коли використовувати CloudHSM замість KMS

CloudHSM — виділений HSM-пристрій (мінімум LUNA HSM від Thales або Cavium) у вашій VPC. **Ніхто з AWS не має доступу до ключів**, лише ви.

| Критерій | KMS | CloudHSM |
|----------|-----|----------|
| Контроль над ключовим матеріалом | AWS + ви | Лише ви |
| Відповідність FIPS 140-2 L3 | ✅ | ✅ |
| Власний PKI | Обмежено (ACM Private CA) | Повний (PKCS#11) |
| Продуктивність (RSA operations/s) | Обмежена | Тисячі на секунду |
| HA | Managed by AWS | Кластер 2+ HSM |
| Ціна | ~$1/міс за ключ | ~$1.45/год (~$1050/міс) |

**Сценарії для CloudHSM у телекомі:**
- PKI-інфраструктура оператора (сертифікати для базових станцій, eNB, gNB)
- Підписання кореневих сертифікатів (Root CA)
- Захист SIM-карт та eSIM (ключі Ki, OPc)
- Відповідність вимогам PCI DSS HSM (point-of-sale термінали)

---

## 14. Amazon Macie

Macie використовує ML для автоматичного **виявлення, класифікації та захисту** чутливих даних в Amazon S3.

**Типи знахідок (findings):**
- `SensitiveData:S3Object/Personal` — PII (ПІБ, email, телефони, паспортні дані)
- `SensitiveData:S3Object/Financial` — номери карток, IBAN, рахунки
- `SensitiveData:S3Object/Credentials` — AWS Access Keys, паролі у файлах
- `Policy:IAMUser/S3BlockPublicAccessDisabled` — публічний доступ до бакету

**Інтеграція з EventBridge та Security Hub:**
```json
{
  "source": ["aws.macie"],
  "detail-type": ["Macie Finding"],
  "detail": {
    "severity": { "description": ["High", "Critical"] }
  }
}
```

---

## 15. Post-Quantum Cryptography (PQC)

### 15.1 Загроза

Квантові комп'ютери загрожують двом класам задач:
- **Факторизація великих чисел** (Алгоритм Шора): ламає RSA та DSA
- **Дискретний логарифм** (Алгоритм Шора): ламає ECDH, ECDSA, DH

Алгоритм Гровера пришвидшує пошук у N^0.5 разів, що знижує ефективну безпеку симетричних шифрів удвічі — **AES-256 залишається безпечним** (ефективно AES-128 проти квантового комп'ютера).

### 15.2 NIST PQC Стандарти (2024)

- **FIPS 203 (ML-KEM / CRYSTALS-Kyber):** обмін ключами та KEM, заміна ECDH
- **FIPS 204 (ML-DSA / CRYSTALS-Dilithium):** цифровий підпис, заміна ECDSA
- **FIPS 205 (SLH-DSA / SPHINCS+):** підпис на основі хешів, резервний варіант

### 15.3 Hybrid PQC в AWS

AWS TLS підтримує гібридне узгодження ключів: класичний ECDH + постквантовий Kyber одночасно. Гібридний підхід гарантує, що навіть якщо один алгоритм виявиться вразливим, безпека забезпечується другим.

### 15.4 Концепція "Harvest Now, Decrypt Later"

Зловмисники можуть записувати зашифрований трафік сьогодні та розшифрувати його у майбутньому, коли з'являться квантові комп'ютери. **Тому мігрувати до PQC потрібно вже зараз**, особливо для даних, що мають зберігатися таємницею 10+ років.

---

## 16. Zero Trust та криптографія

### 16.1 Модель Zero Trust

Zero Trust — архітектурний підхід, що базується на принципах:
1. **Ніколи не довіряй, завжди перевіряй** (Never Trust, Always Verify)
2. **Мінімальний доступ** (Least Privilege)
3. **Припускай порушення** (Assume Breach)

Криптографія є фундаментом Zero Trust — кожне з'єднання та кожна операція має бути аутентифікована та авторизована за допомогою криптографічних доказів.

### 16.2 SPIFFE та SPIRE

SPIFFE (Secure Production Identity Framework For Everyone) — відкритий стандарт для криптографічної ідентичності workloads.

- Кожен workload отримує **SVID (SPIFFE Verifiable Identity Document)** — X.509-сертифікат з SPIFFE URI (`spiffe://trust-domain/workload-name`)
- SPIRE — reference implementation SPIFFE
- AWS IAM Roles Anywhere підтримує SPIFFE/SVID для on-premise workloads

---

## 17. Відповідність регуляторним вимогам

### 17.1 GDPR (General Data Protection Regulation)

**Article 32 — Security of processing:** вимагає "шифрування персональних даних" як технічний захід, де це доречно. GDPR не вказує конкретні алгоритми, але орган DPA (Data Protection Authority) очікує AES-256 або еквівалентний.

**AWS допомагає:** KMS, SSE-S3/KMS, RDS encryption. Але **відповідальність за увімкнення** цих механізмів — ваша.

### 17.2 PCI DSS v4.0

- **Requirement 3.5:** PAN (Primary Account Number) має зберігатися у зашифрованому вигляді (AES-256 або 3DES-2K)
- **Requirement 4.2.1:** Передача PAN через публічні мережі лише по TLS 1.2+

### 17.3 HIPAA Technical Safeguards

- Шифрування PHI (Protected Health Information) — "addressable" (рекомендується, але з обґрунтуванням за відсутності)
- AWS підписує BAA (Business Associate Agreement) для HIPAA-eligible сервісів

---

## 18. Defence in Depth (Ешелонована оборона)

Концепція ешелонованої оборони: **жоден один механізм захисту не є достатнім**. Захист організовується у кілька незалежних рівнів, кожен з яких захищає від різних векторів атак.

### Рівні захисту для даних

| Рівень | Механізми | Що захищає |
|--------|-----------|-----------|
| Ідентичність | IAM, MFA, SCP, IAM Roles Anywhere | Хто може отримати доступ |
| Мережа | VPC, SG, NACLs, PrivateLink, WAF | Мережевий доступ |
| Шифрування | KMS, SSE, TLS/mTLS, ACM, CloudHSM | Конфіденційність даних |
| Застосунок | Input validation, parameterized queries | Логічні вразливості |
| Виявлення | GuardDuty, Macie, CloudTrail, Config | Виявлення атак |
| Відновлення | Backup, DR, multi-region | Відновлення після інциденту |

**Принцип:** навіть якщо зловмисник обійшов ідентичність (вкрав ключі IAM), шифрування гарантує, що дані залишаться нечитабельними без KMS key policy.

---

## 19. Практичне завдання: CDR система в AWS

### Архітектура

Телеком-оператор переносить систему зберігання CDR (Call Detail Records) в AWS. CDR — структуровані записи про дзвінки, що містять MSISDN абонентів, час, тривалість, геолокацію. Відповідно до GDPR, ці дані є персональними.

### Покроковий план реалізації

**Крок 1: Створення CMK в KMS**
```bash
aws kms create-key \
  --description "CDR encryption key - prod" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS \
  --tags TagKey=Environment,TagValue=prod TagKey=DataClass,TagValue=confidential

aws kms enable-key-rotation --key-id <key-id>
```

**Крок 2: S3 бакет з SSE-KMS**
```bash
aws s3api create-bucket \
  --bucket telecom-cdr-prod-2024 \
  --region eu-west-1

aws s3api put-bucket-encryption \
  --bucket telecom-cdr-prod-2024 \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "<key-arn>"
      },
      "BucketKeyEnabled": true
    }]
  }'

aws s3api put-public-access-block \
  --bucket telecom-cdr-prod-2024 \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

**Крок 3: RDS PostgreSQL з шифруванням**
```bash
aws rds create-db-instance \
  --db-instance-identifier telecom-billing-prod \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --master-username dbadmin \
  --storage-encrypted \
  --kms-key-id <key-arn> \
  --ca-certificate-identifier rds-ca-rsa2048-g1
```

**Крок 4: Secrets Manager для пароля БД**
```bash
aws secretsmanager create-secret \
  --name prod/telecom/rds-password \
  --kms-key-id <key-arn> \
  --secret-string '{"username":"dbadmin","password":"..."}'

aws secretsmanager rotate-secret \
  --secret-id prod/telecom/rds-password \
  --rotation-rules AutomaticallyAfterDays=30
```

**Крок 5: ACM сертифікат**
```bash
aws acm request-certificate \
  --domain-name api.telecom-cdr.example.com \
  --validation-method DNS \
  --key-algorithm EC_prime256v1
```

---

## Ключові терміни

| Термін | Визначення |
|--------|-----------|
| **AES-256-GCM** | Advanced Encryption Standard, 256-бітний ключ, режим Galois/Counter — з вбудованою автентифікацією |
| **HMAC** | Hash-based Message Authentication Code — хешування з ключем |
| **DEK** | Data Encryption Key — ключ для шифрування даних |
| **KEK** | Key Encryption Key — ключ для шифрування DEK |
| **CMK** | Customer Managed Key — ключ KMS, яким управляє клієнт |
| **FIPS 140-2** | Американський стандарт безпеки криптографічних модулів |
| **WORM** | Write Once, Read Many — модель незмінного зберігання |
| **PQC** | Post-Quantum Cryptography — криптографія, стійка до квантових комп'ютерів |
| **SPIFFE** | Secure Production Identity Framework For Everyone |
| **mTLS** | Mutual TLS — двостороння TLS-автентифікація |
| **SSE** | Server-Side Encryption — серверне шифрування |
| **CSE** | Client-Side Encryption — клієнтське шифрування |
| **CDR** | Call Detail Record — запис деталей дзвінка |
| **PII** | Personally Identifiable Information — персонально ідентифікована інформація |
| **PAN** | Primary Account Number — номер платіжної картки |
| **PHI** | Protected Health Information — захищена медична інформація |
