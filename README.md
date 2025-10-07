
# **МОДУЛЬ 1. ОСНОВЫ ЛОГИРОВАНИЯ**

### Цель

Понять, зачем нужны логи, какие они бывают и как их обрабатывать на базовом уровне.
После этого модуля ты сможешь уверенно читать, фильтровать и направлять системные логи Linux-машины.

---

## 1. Что такое лог и зачем он нужен

Лог (от англ. log — «журнал») — это структурированная запись о событии в системе.
Каждый лог имеет:

* **источник** (программа, сервис, ядро);
* **время** события;
* **уровень важности** (от `DEBUG` до `CRITICAL`);
* **сообщение** — текст или структура данных.

**Главные цели логирования:**

1. Диагностика — понять, что пошло не так.
2. Аудит — кто, когда и что сделал.
3. Метрики — понимание нагрузки, задержек, ошибок.
4. Форензика — восстановление цепочки событий при инциденте.

---

## 2. Типы логов

| Тип                  | Примеры                     | Где хранится                           |
| -------------------- | --------------------------- | -------------------------------------- |
| **Системные**        | kernel, systemd, cron       | `/var/log/syslog`, `/var/log/messages` |
| **Приложений**   | nginx, postgres, python-app | `/var/log/nginx/access.log`            |
| **Безопасность**     | auditd, auth.log            | `/var/log/auth.log`                    |
| **Сетевые**          | firewall, VPN               | `/var/log/iptables.log`                |
| **Инфраструктурные** | docker, k8s, CI/CD          | через stdout/stderr → агрегаторы       |

---

## 🔍 Примеры логов по типам

---

### **2.1. Системные логи**

Журналирует события ядра, systemd, cron, загрузку и выключение машины.

**Файл:** `/var/log/syslog` или `/var/log/messages`

**Примеры:**

```
Oct 07 09:32:11 debian kernel: [  105.544229] eth0: link up, 1000 Mbps, full-duplex
Oct 07 09:33:00 debian systemd[1]: Started Session 42 of user igor.
Oct 07 09:34:11 debian CRON[2764]: (root) CMD (/usr/local/bin/backup.sh)
Oct 07 09:34:15 debian systemd[1]: Stopping Nginx Web Server...
Oct 07 09:34:16 debian systemd[1]: nginx.service: Succeeded.
```

> **Комментарий:**
> Такие логи полезны для отладки загрузки интерфейсов, зависаний systemd-сервисов, cron-тасков и событий ядра.

---

### **2.2. Логи приложений**

Журналирует работу конкретных программ и сервисов (nginx, PostgreSQL, Python-приложения, backend-сервисы).

**Файлы:**

* `/var/log/nginx/access.log`
* `/var/log/nginx/error.log`
* `/var/log/postgresql/postgresql-15-main.log`

**Примеры:**

`access.log`

```
192.168.1.10 - - [07/Oct/2025:10:14:55 +0300] "GET /api/v1/users HTTP/1.1" 200 532 "-" "curl/7.88.1"
```

`error.log`

```
2025/10/07 10:15:12 [error] 2148#2148: *42 upstream timed out (110: Connection timed out) while reading response header from upstream, client: 192.168.1.10, server: api.local, request: "GET /api/v1/users HTTP/1.1", upstream: "http://127.0.0.1:8000/users"
```

`application.log` (пример структурированного Python-лога)

```json
{"timestamp": "2025-10-07T10:16:00Z", "level": "INFO", "service": "billing", "user_id": 312, "message": "Payment processed successfully"}
```

> **Комментарий:**
> Приложенческие логи чаще всего требуют агрегации и фильтрации по полям (`user_id`, `service`, `status_code`).

---

### **2.3. Логи безопасности**

Фиксируют попытки входа, sudo, изменения прав, запуск демонов.

**Файл:** `/var/log/auth.log` или `/var/log/secure`

**Примеры:**

```
Oct 07 10:21:18 debian sshd[3181]: Failed password for root from 185.241.56.77 port 51822 ssh2
Oct 07 10:21:22 debian sshd[3181]: Accepted password for igor from 192.168.1.20 port 51244 ssh2
Oct 07 10:21:25 debian sudo:     igor : TTY=pts/0 ; PWD=/home/igor ; USER=root ; COMMAND=/bin/systemctl restart nginx
Oct 07 10:21:26 debian su[3210]: (to postgres) igor on pts/0
```

> **Комментарий:**
> Эти логи — сокровище для SOC и forensics. Они показывают, кто, когда и как входил в систему, что запускал и с какими привилегиями.

---

### **2.4. Сетевые логи**

Отражают трафик, фильтрацию пакетов, NAT, VPN и события сетевых служб.

**Файлы:**

* `/var/log/iptables.log`
* `/var/log/openvpn.log`
* `/var/log/dnsmasq.log`

**Примеры:**

`iptables.log`

```
Oct 07 10:25:09 debian kernel: [ 132.3121] IPTABLES DROP IN=eth0 OUT= MAC=02:42:ac:11:00:02 SRC=5.61.12.34 DST=192.168.0.2 LEN=60 TOS=0x00 PREC=0x00 TTL=51 ID=54321 DF PROTO=TCP SPT=58214 DPT=22 WINDOW=65535 RES=0x00 SYN URGP=0
```

`openvpn.log`

```
Tue Oct 07 10:26:00 2025 [client1] Peer Connection Initiated with [AF_INET]203.0.113.45:1194
Tue Oct 07 10:26:10 2025 client1/203.0.113.45 Data Channel: TLSv1.3, cipher TLS_AES_256_GCM_SHA384
```

> **Комментарий:**
> Для сетевой диагностики эти логи показывают дропнутые пакеты, VPN-сессии и аномальную активность (например, flood на порт 22).

---

### **2.5. Инфраструктурные (контейнеры и оркестраторы)**

Регистрируют события контейнеров, деплойментов, пайплайнов CI/CD и оркестрации (Docker, Kubernetes, GitLab CI).

**Источники:**

* `docker logs`
* `/var/log/containers/*.log`
* `/var/log/pods/*.log`
* Jenkins, GitLab Runner

**Примеры:**

`docker logs webapp`

```
2025-10-07T10:30:42Z INFO  Starting HTTP server on :8080
2025-10-07T10:31:01Z ERROR DB connection failed: timeout
2025-10-07T10:31:03Z INFO  Retrying in 5s...
```

`kubelet.log`

```
Oct 07 10:32:10 kubelet[8123]: Started container nginx in pod web-frontend-54fd8
Oct 07 10:32:25 kubelet[8123]: Liveness probe failed: HTTP probe failed with statuscode: 500
```

`gitlab-runner.log`

```
Running with gitlab-runner 16.4.0 (12b6e5f8)
[INFO] Job succeeded in 3m12s (stage: deploy)
```

> **Комментарий:**
> Эти логи — основа для мониторинга состояния CI/CD и микросервисов. Они идут через stdout/stderr и собираются Fluent Bit/Fluentd из `/var/lib/docker/containers`.


---

> **Best practice:**
> Храни хотя бы 7–14 дней логов каждого типа в централизованной системе (ELK, Loki, Graylog).
> Настрой индексирование по типу (`system-*`, `app-*`, `security-*`), чтобы фильтровать миллионы строк за миллисекунды.

---


## ⚙️ 3. Форматы логов 

### 3.1. Plain text — “дедовский” формат

**Описание:**
Самый простой вариант — обычная строка текста без структуры. Каждый компонент лога (время, процесс, сообщение) разделён пробелами или табами, но формат никак не стандартизирован.

**Пример:**

```
[07/Oct/2025 10:15:24] Service started successfully on port 8080
[07/Oct/2025 10:16:02] Connection timeout for user 312
```

**Плюсы:**

* Читается глазами, даже без инструментов.
* Минимальная нагрузка на CPU при записи.

**Минусы:**

* Парсинг зависит от контекста: для одного сервиса формат “YYYY-MM-DD”, для другого “DD/Mon/YYYY”.
* Поиск и фильтрация через grep и regex медленные и ненадёжные.
* Централизованные системы (Fluentd, Graylog) тратят ресурсы на парсинг.

**Когда использовать:**
Либо в простых CLI-утилитах, либо как временное решение в песочнице. В продакшне — избегать.

---

### 3.2. Syslog-формат — промышленный стандарт RFC 3164 / RFC 5424

**Описание:**
Классика UNIX-мира. Каждое сообщение начинается с приоритета `<PRI>` (в нём кодируются facility и severity), затем идёт временная метка, хост, процесс и сообщение.
Используется демоном **rsyslog**, **syslog-ng**, **journald**, сетевыми устройствами и сервисами.

**Пример (RFC 3164):**

```
<34>Oct 07 10:20:15 web01 nginx[2148]: access from 192.168.1.10 status=200 path=/api/v1/users
```

**Пример (RFC 5424):**

```
<165>1 2025-10-07T10:21:00Z web01 nginx 2148 ID47 [origin ip="192.168.1.10"] request="GET /api"
```

**Пояснение:**

* `<34>` — приоритет: facility *daemon*=3, severity *info*=4 → (3×8)+4=28, но тут пример 34.
* `Oct 07 10:20:15` — timestamp.
* `web01` — имя хоста.
* `nginx[2148]` — процесс и PID.
* Сообщение — произвольный текст или структурированные пары `key=value`.

**Плюсы:**

* Универсальный формат, поддерживается почти всем софтом.
* Позволяет маршрутизировать сообщения по уровню и facility (`auth`, `daemon`, `kern` и т.д.).
* Поддерживает сетевую передачу (UDP/TCP/RELP).

**Минусы:**

* Не содержит строгой структуры — парсинг может отличаться у разных систем.
* Старый RFC 3164 не поддерживает миллисекунды и таймзоны.
* Непредсказуемо ведёт себя при Unicode-символах.

**Best practice:**
Переходи на RFC 5424, где формат фиксированный и время указано в ISO 8601 (UTC).

---

### 3.3. JSON — структурированный формат “новой школы”

**Описание:**
Каждое сообщение — JSON-объект. Это идеальный вариант для централизованных систем логирования (Fluentd, ELK, Loki, Vector).
Позволяет фильтровать, индексировать и агрегировать логи по полям, а не по строкам.

**Пример:**

```json
{
  "timestamp": "2025-10-07T10:23:14Z",
  "level": "ERROR",
  "service": "auth-api",
  "user_id": 314,
  "message": "Failed login attempt",
  "ip": "192.168.1.10"
}
```

**Плюсы:**

* Машиночитаемый и самодокументирующийся формат.
* Идеально подходит для ElasticSearch и Grafana Loki.
* Простая фильтрация по ключам (`level:error`, `service:auth-api`).
* Удобен для слияния данных из разных сервисов.

**Минусы:**

* Чуть выше нагрузка на CPU и диск.
* Требует строгой сериализации: ошибка в JSON ломает ingestion.

**Best practice:**

1. Используй поля `timestamp`, `level`, `message`, `service`, `trace_id`, `user_id`.
2. Всегда в UTC (`Z` на конце времени).
3. Избегай вложенных объектов глубже 2 уровней — Elasticsearch их flatten’ит.

---

### 3.4. CEF / LEEF — стандарты для безопасности (SIEM-ориентированные)

**CEF (Common Event Format)** и **LEEF (Log Event Extended Format)** используются SIEM-платформами (ArcSight, QRadar, Splunk).
Они позволяют хранить метаданные о событии в структурированной, но лёгкой для парсинга форме.

**Пример (CEF):**

```
CEF:0|nginx|webserver|1.0|100|Access denied|5|src=192.168.1.10 spt=443 request=/admin method=GET
```

**Пояснение:**

* `CEF:0` — версия.
* `nginx|webserver|1.0` — вендор, продукт, версия.
* `100|Access denied|5` — ID, описание, уровень.
* После `|` идут поля `key=value`.

**Плюсы:**

* Оптимизирован для SIEM: легко разбирается, логично индексируется.
* Поддерживает поля типа `src`, `dst`, `spt`, `dpt`, `proto`.

**Минусы:**

* Не гибкий — фиксированный набор полей.
* Не подходит для обычных приложений (избыточен).

**Best practice:**
Если ты работаешь в команде безопасности — конвертируй syslog в CEF перед отправкой в SIEM.

---

### 3.5. CSV / TSV

**Описание:**
Табличный формат, часто используется в старых приложениях и отчётных логах.

**Пример:**

```
timestamp,level,user_id,action,result
2025-10-07T10:30:15Z,INFO,312,login,success
2025-10-07T10:30:17Z,ERROR,312,login,invalid_password
```

**Плюсы:**

* Простой, открывается в Excel и BI-системах.
* Удобен для экспорта в аналитику.

**Минусы:**

* Не поддерживает вложенность и типизацию.
* При запятых в тексте ломается структура.
* Непрактичен для realtime-анализа.


---

### 3.6. Binary (протобаф, Avro, Parquet)

**Описание:**
Логи в бинарном виде для высоконагруженных систем и data-pipeline’ов (Kafka, Spark, ClickHouse).

**Пример:**

```
(Двоичные данные, сериализованные через Protobuf)
```

**Плюсы:**

* Экономия места в 2-5 раз.
* Быстрая десериализация.
* Полная типизация и схема.

**Минусы:**

* Не читаются человеком.
* Требуют строгого соответствия схеме.

---

## ⚙️ 4. Потоки stdout и stderr

Каждый процесс в Unix получает три дескриптора — каналы, через которые идёт ввод-вывод.
Они создаются ядром при запуске программы и наследуются дочерними процессами.

| Поток      | Номер дескриптора | Назначение                   | Пример использования       |
| ---------- | ----------------: | ---------------------------- | -------------------------- |
| **stdin**  |                 0 | стандартный ввод             | `cat` читает из клавиатуры |
| **stdout** |                 1 | стандартный вывод результата | `echo "OK"`                |
| **stderr** |                 2 | сообщения об ошибках         | `ls /root 2>&1`            |

> **Важно:** даже если программа падает, `stderr` остаётся отдельным каналом — это позволяет анализировать сбой независимо от полезного вывода.

##### 🔹 Разделение потоков

```bash
python app.py > app.log 2> errors.log
```

1. `>` перенаправляет дескриптор 1 (stdout) в `app.log`.
2. `2>` направляет дескриптор 2 (stderr) в `errors.log`.

Так ты получаешь чистый лог и отдельный файл ошибок — это золотой стандарт для cron-тасков и systemd-сервисов.

##### 🔹 Объединение потоков

```bash
python app.py > all.log 2>&1
```

Команда `2>&1` означает: *«отправь stderr туда, куда сейчас идёт stdout».*
Результат — оба потока записываются в один файл, сохраняя порядок по времени.

##### 🔹 Под капотом

Когда ты выполняешь перенаправление, shell (bash, zsh) вызывает системный вызов `dup2()`, создавая копию файлового дескриптора.
Это низкоуровневая операция ядра, обеспечивающая гибкость UNIX-пайплайнов (`|`).


##### 🔹 Для контейнеров

В Docker stdout и stderr автоматически собираются `containerd` и передаются лог-драйверу (json-file, journald, Fluent Bit).
Поэтому внутри контейнера **всегда логируй только в stdout/stderr** — не в файлы.

> 💡 **Best practice:** в systemd-юнитах используй
> `StandardOutput=journal` и `StandardError=journal`.
> Это позволит journald подхватывать логи без дополнительного rsyslog-конфига.
---

### 4.1. Проверим на примере

```bash
#!/bin/bash
echo "OK: service started"
>&2 echo "ERROR: database connection failed"
```

При выполнении:

```bash
bash script.sh > out.log 2> err.log
```

**out.log:**

```
OK: service started
```

**err.log:**

```
ERROR: database connection failed
```

---

### 4.2. Типичные ошибки

1. **Переопределение stderr в /dev/null**

   ```bash
   ./app 2>/dev/null
   ```

   Это подавит все ошибки. Часто делают “чтобы не мешали”, но потом невозможно понять, почему сервис упал.

2. **Случайное объединение потоков**
   Иногда операторы CI/CD (например, Jenkins) сливают всё в один поток.
   Результат — при алертинге по ключевому слову “ERROR” может сработать ложный сигнал, если слово встречается в обычном выводе.

3. **Вывод бинарных данных в stdout**
   При логировании бинарного потока (например, gzip stdout) лучше явно указывать отдельный файл, чтобы не повредить консольный вывод.

---

### 4.3. Best practices

* Всегда разделяй stdout и stderr в сервисах, особенно в скриптах резервного копирования, обновления, деплоя.
* В контейнерах Docker **пиши всё в stdout/stderr** — контейнерный рантайм сам соберёт эти потоки и отправит Fluent Bit или journald.
* Не используй `2>/dev/null` в продакшне.
* Если необходимо отфильтровать шум, перенаправь stderr во временный файл и проверь его перед удалением.
* Для cron-тасков логируй оба потока:

  ```bash
  /usr/local/bin/backup.sh > /var/log/backup.log 2>&1
  ```

---

## 🧭 5. Systemd и journald

Современные дистрибутивы Linux больше не используют классический syslog как единственный источник логов.
С переходом на **systemd** появился собственный движок логов — **journald**.

---

### 5.1. Что делает journald

`journald` собирает события из:

* ядра (Linux kernel messages),
* systemd-сервисов,
* стандартных потоков stdout/stderr,
* классического syslog (через совместимость),
* приложений, использующих API systemd.

Он хранит логи в бинарной форме (каталог `/var/log/journal/`), а читает их утилита **journalctl**.

---

### 5.2. Основные команды

```bash
journalctl -xe
```

Показать последние ошибки, включая контекст.
Полезно после падения сервиса или systemctl restart.

```bash
journalctl -u nginx.service
```

Вывод логов конкретного юнита (службы).

```bash
journalctl -f
```

Следить за логом в реальном времени (аналог `tail -f`).

```bash
journalctl --since "2025-10-07 10:00" --until "2025-10-07 11:00"
```

Фильтрация по времени. Удобно при расследовании инцидента.

```bash
journalctl -p err -b
```

Показать все сообщения уровня *error* после последней загрузки системы.

#### 🔹 Как systemd связывает stdout/stderr с journald

Когда сервис стартует через systemd, PID процесса прикрепляется к сокету `/run/systemd/journal/stdout`.
Все его потоки перехватываются journald автоматически.
Параметры `StandardOutput=` и `StandardError=` задают, куда именно писать:
`inherit`, `journal`, `syslog`, `null` или `kmsg`.

#### 🔹 Пример анализа конкретного PID

```bash
journalctl _PID=1524
```

Покажет все сообщения только от этого процесса — удобно для расследования “зомби-сервисов”.
---

### 5.3. Практическая польза

Journald автоматически подхватывает вывод сервисов systemd: всё, что программа пишет в stdout или stderr, попадает в журнал.
Поэтому для системных сервисов **не нужно указывать файлы логов вручную** — systemd сам собирает всё необходимое.

**Пример:**
Юнит `/etc/systemd/system/webapp.service`

```ini
[Service]
ExecStart=/usr/local/bin/webapp
Restart=always
StandardOutput=journal
StandardError=journal
```

Логи можно потом просмотреть:

```bash
journalctl -u webapp.service -f
```

---

### 5.4. Настройки journald

Файл конфигурации: `/etc/systemd/journald.conf`

Ключевые параметры:

```
[Journal]
Storage=persistent      # хранить логи между перезагрузками
SystemMaxUse=500M       # ограничить общий объём
RuntimeMaxUse=100M      # ограничить память под временные логи
MaxFileSec=1month       # максимальный возраст файла
Compress=yes             # сжимать старые записи
```

После изменений — перезапуск:

```bash
systemctl restart systemd-journald
```

---

### 5.5. Архитектура хранения

journald хранит логи в бинарных файлах (`/var/log/journal/<machine-id>/system.journal`).
Каждое сообщение содержит метаданные:

* `_PID` — идентификатор процесса,
* `_UID` — пользователь,
* `_COMM` — команда,
* `_SYSTEMD_UNIT` — юнит,
* `_EXE` — путь к исполняемому файлу.

Эти поля позволяют делать точную фильтрацию:

```bash
journalctl _UID=0 _SYSTEMD_UNIT=nginx.service
```

---

### 5.6. Интеграция journald и rsyslog

По умолчанию journald может пересылать логи в rsyslog для дальнейшей маршрутизации:

```
ForwardToSyslog=yes
```

Это позволяет строить гибридные схемы: journald собирает локально, rsyslog отправляет дальше по TCP → Fluentd → ELK.

---

### 5.7. Best practices

1. **Ограничивай размер журнала:**
   `SystemMaxUse=` или `RuntimeMaxUse=`. Иначе `/var` можно переполнить за день при активных сервисах.
2. **Храни логи постоянно:**
   Установи `Storage=persistent`, иначе они исчезнут после перезагрузки.
3. **Не сохраняй файлы вручную внутри сервисов под systemd.**
   Systemd сам перехватывает stdout/stderr.
4. **Используй фильтры по юниту и времени,** а не по grep — это быстрее и надёжнее.
5. **Отправляй journald → Fluent Bit → Elasticsearch**, если нужна централизованная аналитика.

---


## 🔧 Практические задания

### Задание 1. Базовая работа с логами

Выведи последние 100 строк журнала ядра и сохрани их в отдельный файл.

```bash
journalctl -k -n 100 > kernel.log
```

### Задание 2. Python-логгер

Создай скрипт:

```python
import logging
logging.basicConfig(filename='app.log',
                    level=logging.INFO,
                    format='%(asctime)s %(levelname)s:%(message)s')
logging.info('Сервис стартовал')
logging.warning('Память на исходе')
logging.error('Ошибка подключения к БД')
```

Проверь, что файл `app.log` создался и содержит время + уровень + сообщение.

### Задание 3. Перенаправление stdout/stderr

Создай bash-скрипт, который пишет в оба потока, и перенаправь их в разные файлы.

```bash
echo "ok"  
>&2 echo "error"  
```

---

## 📊 Тест для самопроверки

1. Чем отличается stdout от stderr?
2. Какой уровень логирования выше — INFO или DEBUG?
3. Что делает `journalctl -u ssh -f`?
4. Почему временные зоны важны в логах?
5. Почему формат JSON удобен для агрегации?

---

## 🧠 Best Practices

1. Используй **один формат логов** во всей инфраструктуре.
2. Добавляй **таймзону и UTC-метку** в каждое сообщение.
3. Не храни логи в /var/log долго — делай ротацию.
4. Пиши логи в stdout в контейнерах — их заберёт агрегатор.
5. Структурируй сообщения и добавляй ключи: `user_id`, `session`, `service`.

---

**После этого модуля ты уже можешь:**

* фильтровать и сохранять системные логи;
* создавать структурированные логи в своих приложениях;
* понимать, как stdout/stderr идут в агрегатор.

---


# **МОДУЛЬ 2. СИСТЕМНОЕ ЛОГИРОВАНИЕ С RSYSLOG**

### Цель

Освоить работу демона **rsyslog**, научиться принимать, фильтровать и отправлять системные логи на другие узлы.
После модуля ты сможешь построить устойчивый базовый лог-пайплайн Linux-сервера.

---

## 1. Что такое rsyslog

**rsyslog** — современный демон системных логов, совместимый с классическим **syslog** (RFC 5424).
Он выполняет три роли:

1. Слушает входящие сообщения от ядра, systemd, приложений.
2. Фильтрует их по правилам (facility, severity, source).
3. Записывает локально или пересылает дальше — в файлы, сокеты, TCP, HTTP, Kafka и т. д.

---

## 2. Архитектура

```
Input  →  Filter / Parser  →  Queue  →  Output
```

* **Input** — откуда приходят логи (kernel, UDP, journald, imfile).
* **Filter/Parser** — по каким условиям обрабатывать.
* **Queue** — очередь, гарантирующая доставку.
* **Output** — куда отправлять (файл, удалённый сервер, Elastic, Kafka).

Модули rsyslog подключаются директивой `module(load="имя")`.

---

## 3. Конфигурация rsyslog

Основной файл: `/etc/rsyslog.conf`
Дополнительные правила: `/etc/rsyslog.d/*.conf`

Каждая строка — правило:

```
facility.level   action
```

**Пример:**

```
authpriv.*    /var/log/secure
*.info;mail.none;authpriv.none;cron.none   /var/log/messages
```

* `facility` — тип источника (auth, daemon, cron, kern).
* `level` — минимальный уровень (info, warn, err).
* `action` — куда писать.

---

## 4. Основные facilities и уровни

| Facility        | Значение                  |
| --------------- | ------------------------- |
| auth / authpriv | аутентификация            |
| cron            | планировщик заданий       |
| daemon          | системные демоны          |
| kern            | ядро                      |
| mail            | почтовые сервисы          |
| user            | пользовательские процессы |

| Уровень | Число | Описание                   |
| ------- | ----- | -------------------------- |
| emerg   | 0     | система неработоспособна   |
| alert   | 1     | критическая ситуация       |
| crit    | 2     | ошибка ядра или приложения |
| err     | 3     | обычная ошибка             |
| warning | 4     | предупреждение             |
| notice  | 5     | информационное сообщение   |
| info    | 6     | общая информация           |
| debug   | 7     | отладочные данные          |

---

## 5. Примеры правил

### 5.1. Запись всех сообщений в один файл

```
*.*   /var/log/all.log
```

### 5.2. Исключить почту и cron

```
*.info;mail.none;cron.none   /var/log/messages
```

### 5.3. Разделить по уровням

```
*.err      /var/log/errors.log
*.warning  /var/log/warnings.log
```

### 5.4. Логи конкретного приложения

```
:programname, isequal, "nginx"  /var/log/nginx-rsyslog.log
```

### 5.5. Отправка на удалённый сервер

```
*.*  @@10.0.0.10:514
```

Один `@` — UDP, два `@@` — TCP.

---

## 6. Принятие удалённых логов

### 6.1. Включаем модуль и порт

```
module(load="imudp")
input(type="imudp" port="514")

module(load="imtcp")
input(type="imtcp" port="514")
```

### 6.2. Проверяем сокеты

```bash
sudo ss -lnpt | grep rsyslog
```

### 6.3. Разрешаем в firewall

```bash
sudo ufw allow 514/tcp
sudo ufw allow 514/udp
```

Теперь сервер готов принимать сообщения с других машин.

---

## 7. Отправка логов с клиента

На клиенте добавь в `/etc/rsyslog.d/90-remote.conf`:

```
*.*  @@logserver.example.com:514
```

Перезапусти демон:

```bash
systemctl restart rsyslog
```

Проверка:

```bash
logger "Test message from $(hostname)"
```

На сервере появится запись в `/var/log/all.log`.

---

## 8. Использование шаблонов

rsyslog может форматировать строки перед записью:

```
template(name="jsonTemplate" type="list") {
  constant(value="{")
  constant(value="\"timestamp\":\"") property(name="timereported" dateFormat="rfc3339")
  constant(value="\",\"host\":\"") property(name="hostname")
  constant(value="\",\"message\":\"") property(name="msg")
  constant(value="\"}\n")
}
*.* action(type="omfile" file="/var/log/all.json" template="jsonTemplate")
```

→ Логи будут структурированы в JSON-виде — удобно для Fluent Bit/ELK.

---

## 9. Очереди и надёжность

rsyslog имеет встроенные очереди для гарантированной доставки:

* **In-memory** — быстро, но данные теряются при падении.
* **Disk-assisted** — хранит бэкап на диске.
* **Disk** — полная надёжность.

**Пример:**

```
main_queue(
  type="LinkedList"
  fileName="mainq"
  maxDiskSpace="1g"
  queue.saveonshutdown="on"
)
```

---

## 🔧 Практические задания

### Задание 1. Собери локальные логи в единый файл

Создай `/etc/rsyslog.d/10-all.conf`:

```
*.*  /var/log/all.log
```

Перезапусти rsyslog и проверь:

```bash
logger "test message"
tail -n 5 /var/log/all.log
```

### Задание 2. Отправь логи на удалённый сервер

1. На сервере A включи приём TCP 514.
2. На клиенте B добавь строку `*.* @@serverA:514`.
3. Проверь передачу через `logger`.

### Задание 3. Создай JSON-шаблон

1. Добавь шаблон из примера в конфиг.
2. Сгенерируй пару сообщений.
3. Убедись, что файл `/var/log/all.json` валиден (JSON Lint).

---

## 📊 Тест по модулю 2

1. Какой порт используется для syslog по TCP?
2. Чем отличаются один `@` и два `@@` в правиле отправки?
3. Где находятся дополнительные файлы настроек rsyslog?
4. Что такое facility и severity?
5. Какая директива включает приём UDP-сообщений?

---

## 🧠 Best Practices

1. **Разделяй логи по типам:** system, auth, app — отдельные файлы.
2. **Используй TCP вместо UDP,** если нужна гарантированная доставка.
3. **Добавь очередь на диск,** чтобы не терять логи при рестартах.
4. **Храни логи в JSON,** если планируешь отправлять в Fluent Bit или Elasticsearch.
5. **Проверяй права на /var/log/** — ошибки прав часто приводят к молчанию rsyslog.

---

**После этого модуля ты уже умеешь:**

* конфигурировать rsyslog,
* фильтровать и маршрутизировать логи,
* пересылать их по TCP/UDP,
* строить структурированные форматы для агрегации.

---


---

# **МОДУЛЬ 3. FLUENTD И FLUENT BIT — СОВРЕМЕННЫЙ СБОР И МАРШРУТИЗАЦИЯ ЛОГОВ**

---

## 🎯 Цель

Научиться собирать, фильтровать и отправлять логи с серверов, контейнеров и приложений в централизованное хранилище.
После модуля студент сможет самостоятельно построить пайплайн:
**Fluent Bit → Fluentd → Elasticsearch.**

---

## 1. Что такое Fluent Bit и Fluentd

Fluentd — это универсальный маршрутизатор логов,
а Fluent Bit — его младший, лёгкий “агент”, который стоит ближе к источникам данных.

Думай о них так:

* **Fluent Bit** живёт на каждом сервере и собирает логи (из journald, файлов, docker).
* **Fluentd** живёт в центре (на выделенном узле) и решает, куда всё это отправить — Elasticsearch, Kafka, S3 и т.д.

| Параметр     | Fluent Bit                   | Fluentd                   |
| ------------ | ---------------------------- | ------------------------- |
| Цель         | сбор и доставка логов        | маршрутизация и обработка |
| Память       | 5–10 МБ                      | 50–100 МБ                 |
| Язык         | C                            | Ruby                      |
| Конфиг       | `fluent-bit.conf`            | `fluent.conf`             |
| Где ставится | на каждом хосте / контейнере | в центре стека            |

> **Best practice:**
> В продакшне всегда используй связку:
> **Fluent Bit** как агент → **Fluentd** как приёмник и маршрутизатор.

---

## 2. Установка

### Fluent Bit (Ubuntu/Debian)

```bash
# добавляем репозиторий
curl https://packages.fluentbit.io/fluentbit.key | sudo apt-key add -

# подключаем официальный apt-репозиторий
echo "deb https://packages.fluentbit.io/ubuntu/noble noble main" | sudo tee /etc/apt/sources.list.d/fluentbit.list

# устанавливаем пакет агента
sudo apt update && sudo apt install td-agent-bit
```

**Что происходит:**

1. Импортируется ключ подписи пакетов.
2. Добавляется репозиторий.
3. Устанавливается бинарник `td-agent-bit` и создаётся systemd-сервис.

---

### Fluentd (агрегатор)

```bash
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-jammy-td-agent4.sh | sudo sh
```

Этот скрипт устанавливает **td-agent** — официальный пакет Fluentd.
После установки создаётся конфиг `/etc/td-agent/td-agent.conf` и сервис `td-agent.service`.

---

## 3. Архитектура Fluent Bit и Fluentd

```
[INPUT] → [FILTER] → [BUFFER] → [OUTPUT]
```

Каждый блок выполняет свою задачу:

* **INPUT** — получает логи (например, tail — читает файлы).
* **FILTER** — модифицирует или фильтрует данные (например, убирает DEBUG).
* **BUFFER** — временно хранит данные (в памяти или на диске).
* **OUTPUT** — отправляет в нужную систему (stdout, Elasticsearch, Fluentd и т.д.).

---

## 4. Минимальный конфиг Fluent Bit

**Файл:** `/etc/fluent-bit/fluent-bit.conf`

```ini
[SERVICE]
    Flush        5
    Daemon       Off
    Log_Level    info
```

* `[SERVICE]` — общий раздел для глобальных настроек агента.
* `Flush 5` — отправлять данные каждые 5 секунд.
* `Daemon Off` — работать на переднем плане (удобно для отладки).
* `Log_Level info` — уровень внутреннего логирования Fluent Bit.

---

```ini
[INPUT]
    Name         tail
    Path         /var/log/nginx/*.log
    Tag          nginx.*
```

* `[INPUT]` — источник логов.
* `Name tail` — тип входа: **tail** читает файлы построчно (как `tail -f`).
* `Path /var/log/nginx/*.log` — откуда читать.
* `Tag nginx.*` — тег нужен для маршрутизации (match по тегу).

---

```ini
[FILTER]
    Name         grep
    Match        nginx.*
    Regex        message .*error.*
```

* `[FILTER]` — фильтрующий блок.
* `Name grep` — фильтр “по шаблону” (оставляет только строки, где message содержит “error”).
* `Match nginx.*` — применить фильтр только к тегам nginx.
* `Regex message .*error.*` — регулярка для фильтрации.

---

```ini
[OUTPUT]
    Name         stdout
    Match        *
```

* `[OUTPUT]` — куда выводить.
* `Name stdout` — писать на экран (в консоль).
* `Match *` — брать все теги.

---

**Результат:**
Fluent Bit будет каждые 5 секунд читать логи nginx, фильтровать ошибки и печатать их в консоль.
Это идеальный старт для отладки.

---

## 5. Fluentd — агрегатор

**Файл:** `/etc/td-agent/td-agent.conf`

```conf
<source>
  @type forward
  port 24224
</source>
```

* `<source>` — входной источник.
* `@type forward` — приём данных по протоколу Forward (от Fluent Bit).
* `port 24224` — стандартный порт для связи Bit → Fluentd.

---

```conf
<match nginx.*>
  @type elasticsearch
  host 127.0.0.1
  port 9200
  logstash_format true
  include_tag_key true
  flush_interval 5s
</match>
```

* `<match nginx.*>` — выбирает все сообщения с тегом `nginx.*`.
* `@type elasticsearch` — подключает плагин отправки в Elasticsearch.
* `logstash_format true` — создаёт индексы по дате, как у Logstash (`fluentd-YYYY.MM.DD`).
* `flush_interval 5s` — каждые 5 секунд сбрасывает буфер в Elasticsearch.

> ⚠️ При первом запуске убедись, что Elasticsearch уже работает — иначе Fluentd будет бесконечно ретраить отправку (и засорит логи).

---

## 6. Буферизация и надёжность

При перебоях сеть может “рвать” поток логов.
Fluent Bit и Fluentd умеют **буферизировать** (запоминать) сообщения до успешной отправки.

---

### Буфер Fluentd

```conf
<buffer>
  @type file
  path /var/log/td-agent/buffer
  flush_interval 10s
  retry_type exponential_backoff
</buffer>
```

* `@type file` — хранить буфер на диске.
* `path` — каталог для временных файлов.
* `flush_interval 10s` — сбрасывать каждые 10 секунд.
* `retry_type exponential_backoff` — увеличивать интервалы между повторами при ошибках (чтобы не перегрузить сеть).

---

### Буфер Fluent Bit

```ini
[OUTPUT]
    Name es
    Match *
    Host 127.0.0.1
    Port 9200
    Retry_Limit False
    storage.total_limit_size 512M
```

* `Retry_Limit False` — бесконечно пытаться доставить (пока не получится).
* `storage.total_limit_size 512M` — лимит дискового буфера.

---

## 7. Routing по тегам

Fluentd маршрутизирует потоки по тегам.
Пример:

```conf
<source>
  @type tail
  path /var/log/auth.log
  tag auth.*
</source>

<match auth.*>
  @type file
  path /var/log/central/auth.log
</match>
```

Здесь:

* Источник `auth.log` получает тег `auth.*`.
* Все записи с этим тегом записываются в `/var/log/central/auth.log`.

Это даёт гибкость — можно одним Fluentd управлять логами десятков сервисов.

---

## 8. Парсинг и структурирование

Fluentd умеет превращать сырые текстовые строки в структурированные JSON-записи.

### Regex-пример

```conf
<parse>
  @type regexp
  expression /(?<ip>\d+\.\d+\.\d+\.\d+) .* "(?<method>[A-Z]+) (?<path>[^"]+)"/
  time_format %d/%b/%Y:%H:%M:%S %z
</parse>
```

* `expression` — регулярка с именованными группами (`?<ip>`, `?<method>`).
* `time_format` — шаблон времени из строки.

Результат:

```json
{
  "ip": "192.168.1.10",
  "method": "GET",
  "path": "/index.html"
}
```

### JSON-пример

Если логи уже в JSON:

```conf
<parse>
  @type json
</parse>
```

Fluentd автоматически распознает и добавит поля в индекс.

---

## 9. Сбор логов из Docker и systemd

### Docker

```ini
[INPUT]
    Name  tail
    Path  /var/lib/docker/containers/*/*.log
    Parser docker
    Tag docker.*
```

* `Parser docker` — встроенный парсер для JSON-логов контейнеров.
* `Tag docker.*` — помечаем всё, что пришло от контейнеров.

### journald

```ini
[INPUT]
    Name  systemd
    Tag   host.*
    Systemd_Filter _SYSTEMD_UNIT=nginx.service
```

* `Name systemd` — входной плагин для `journald`.
* `Systemd_Filter` — фильтр по юниту (здесь nginx).

---

## 10. Связка Fluent Bit → Fluentd

### Конфиг на агенте (Fluent Bit)

```ini
[OUTPUT]
    Name  forward
    Match *
    Host  log-aggregator
    Port  24224
```

* `Name forward` — выходной плагин, отправляющий данные по TCP на Fluentd.
* `Host log-aggregator` — IP или hostname агрегатора.
* `Port 24224` — стандартный порт Fluentd.

---

### Конфиг на сервере (Fluentd)

```conf
<source>
  @type forward
  port 24224
</source>

<match **>
  @type stdout
</match>
```

Fluentd принимает поток и печатает всё на экран — удобно для отладки связки.
Если всё работает — можно заменить `stdout` на `elasticsearch` или `file`.

---

## 🔧 Практические задания

### Задание 1. Минимальный пайплайн

Настрой Fluent Bit, чтобы:

* читать `/var/log/syslog`,
* фильтровать `error`,
* печатать результат в stdout.

### Задание 2. Связка Bit → Fluentd → Elasticsearch

1. Подними локально Elasticsearch (можно через docker).
2. Настрой Fluent Bit как агент.
3. Настрой Fluentd как приёмник (`@type elasticsearch`).
4. Проверь, что в Kibana появились логи nginx.

### Задание 3. Journald → Fluent Bit

1. Подключи вход `systemd`.
2. Фильтруй `_SYSTEMD_UNIT=sshd.service`.
3. Выведи результат в файл `/var/log/sshd-fluent.log`.

---

## 📊 Тест

1. Чем Fluent Bit отличается от Fluentd?
2. Что делает директива `Match *`?
3. Как включить буферизацию на диске?
4. На каком порту Fluentd принимает данные?
5. Какой плагин используется для сбора логов journald?

---

## 🧠 Best Practices

1. **Fluent Bit → Fluentd** — классическая производственная схема.
2. **JSON как стандарт формата.** Если можешь — логируй в JSON.
3. **Дисковый буфер обязателен** в продакшне.
4. **Используй TLS и аутентификацию** между агентом и сервером.
5. **Тестируй regex-парсеры на edge-сервере,** чтобы не ломать продакшн-агрегатор.
6. **Контролируй flush_interval.** Слишком короткий — нагружает сеть, слишком длинный — задерживает логи.

---

После этого модуля ты:

* Понимаешь архитектуру сбора логов.
* Можешь настроить Fluent Bit и Fluentd.
* Разбираешь и фильтруешь логи из journald, файлов и контейнеров.
* Можешь построить рабочий пайплайн до Elasticsearch.

---

# **МОДУЛЬ 4. ХРАНЕНИЕ И ИНДЕКСАЦИЯ ЛОГОВ В ELASTICSEARCH**

---

## 🎯 Цель

Понять, как устроен Elasticsearch, научиться хранить, индексировать и эффективно искать миллионы логов без потери производительности.
После модуля ты сможешь спроектировать отказоустойчивое хранилище логов уровня production.

---

## 1. Что такое Elasticsearch

**Elasticsearch (ES)** — это распределённая поисковая и аналитическая система, оптимизированная под текстовые и структурированные данные.

Когда Fluentd или Fluent Bit отправляют логи в ES, они превращаются в документы JSON, которые индексируются — то есть разбиваются на ключевые слова для быстрого поиска.

Логически Elasticsearch состоит из:

* **Cluster** — совокупность всех узлов.
* **Node** — отдельный сервер с ролью (data, master, ingest).
* **Index** — “таблица” данных (например, `nginx-logs-2025.10.07`).
* **Shard** — кусочек индекса, физически хранящийся на диске.
* **Replica** — копия шардов для отказоустойчивости.

---

## 2. Установка Elasticsearch (одиночный узел)

Для лабораторной среды:

```bash
# 1. Скачать и установить Elasticsearch
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.14.0-amd64.deb
sudo dpkg -i elasticsearch-8.14.0-amd64.deb

# 2. Запустить сервис
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

Проверка:

```bash
curl -k -u elastic:пароль https://localhost:9200
```

Если всё ок — увидишь JSON с версией кластера.

---

## 3. Основные настройки (elasticsearch.yml)

Файл: `/etc/elasticsearch/elasticsearch.yml`

```yaml
cluster.name: logs-cluster
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
xpack.security.enabled: false
```

**Что это значит:**

* `cluster.name` — имя кластера, под которым соединяются узлы.
* `node.name` — имя конкретного узла (используется в интерфейсе Kibana).
* `network.host` — адрес, на котором слушает API.
* `discovery.type: single-node` — одиночный режим (без мастеров).
* `xpack.security.enabled: false` — отключаем авторизацию для локальной лаборатории.

> ⚠️ В продакшне безопасность включается обязательно: с TLS и API-токенами.

---

## 4. Структура индексов и rollover

Каждый индекс содержит миллионы документов.
Для логов принято создавать **дневные или недельные индексы**, чтобы:

* не перегружать один индекс,
* легко удалять старые,
* управлять retention (сроком хранения).

Пример имени:

```
logs-nginx-2025.10.07
```

**Rollover policy** — автоматическое создание нового индекса, когда старый достигает лимита.

**Пример настройки:**

```json
PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "7d",
            "max_size": "5gb"
          }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

* `hot` — активный индекс (приём логов).
* `rollover` — создаёт новый каждые 7 дней или при 5 ГБ.
* `delete` — удаляет старые через 30 дней.

---

## 5. Mapping и анализаторы

**Mapping** — это “схема” индекса: какие поля есть и какого они типа.
ES может сам определить типы, но лучше задавать явно.

```json
PUT logs-nginx
{
  "mappings": {
    "properties": {
      "timestamp": { "type": "date" },
      "level":      { "type": "keyword" },
      "service":    { "type": "keyword" },
      "message":    { "type": "text" },
      "ip":         { "type": "ip" },
      "user_id":    { "type": "integer" }
    }
  }
}
```

* `keyword` — поле для фильтрации (точное совпадение).
* `text` — поле для полнотекстового поиска.
* `date`, `ip`, `integer` — структурированные типы.

> **Best practice:**
> Используй `keyword` для полей уровня (`INFO`, `ERROR`, `service`), а `text` — только для `message`. Это экономит RAM и ускоряет фильтры.

---

## 6. Загрузка логов вручную (пример)

Создадим тестовый документ:

```bash
curl -X POST "localhost:9200/logs-nginx/_doc" -H 'Content-Type: application/json' -d '{
  "timestamp": "2025-10-07T10:33:21Z",
  "level": "ERROR",
  "service": "nginx",
  "message": "Upstream connection timed out",
  "ip": "192.168.1.10"
}'
```

Проверим:

```bash
curl "localhost:9200/logs-nginx/_search?q=service:nginx"
```

---

## 7. Поиск и фильтрация

Elasticsearch поддерживает язык запросов DSL.
Пример простого запроса:

```json
POST logs-nginx/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "ERROR" }},
        { "range": { "timestamp": { "gte": "now-1h" }}}
      ]
    }
  },
  "sort": [{ "timestamp": "desc" }]
}
```

→ Покажет все ошибки за последний час, отсортированные по времени.

---

## 8. Индексация через Fluentd

Чтобы Fluentd писал логи в Elasticsearch, используется плагин `@type elasticsearch`.

```conf
<match **>
  @type elasticsearch
  host 127.0.0.1
  port 9200
  logstash_format true
  include_tag_key true
  flush_interval 5s
</match>
```

**Разбор:**

* `logstash_format true` — создаёт индексы вида `fluentd-YYYY.MM.DD`.
* `include_tag_key true` — добавляет тег как отдельное поле.
* `flush_interval` — сбрасывает буфер каждые 5 секунд.

---

## 9. Мониторинг состояния кластера

Основные API:

```bash
# общее состояние
curl -X GET "localhost:9200/_cluster/health?pretty"

# список индексов
curl -X GET "localhost:9200/_cat/indices?v"

# проверка шардов
curl -X GET "localhost:9200/_cat/shards?v"
```

**Типичные состояния:**

* `green` — всё отлично.
* `yellow` — отсутствуют реплики, но данные есть.
* `red` — шардов не хватает, возможна потеря данных.

---

## 10. Оптимизация хранения

1. **Hot-warm-cold архитектура**

   * hot — активные данные;
   * warm — старые, но доступные;
   * cold — архив (S3, Glacier).
     Пример: за 7 дней данные “остывают” и переносятся на дешёвые узлы.

2. **Сжатие и refresh**
   Уменьшай `refresh_interval` до 30–60s (по умолчанию 1s), чтобы сократить нагрузку.

   ```
   PUT logs-nginx/_settings
   {
     "index": { "refresh_interval": "30s" }
   }
   ```

3. **Удаляй старые индексы автоматически** (см. ILM policy выше).

---

## 🔧 Практические задания

### Задание 1. Разверни Elasticsearch и создай индекс

1. Установи ES и запусти его.
2. Создай индекс `logs-test` с полями `timestamp`, `level`, `message`.
3. Отправь 3 тестовых документа.
4. Сделай поиск по уровню `ERROR`.

### Задание 2. Подключи Fluentd → Elasticsearch

1. Настрой Fluentd с `@type elasticsearch`.
2. Отправь логи nginx.
3. Убедись, что индекс `fluentd-*` появился.

### Задание 3. Настрой rollover policy

1. Создай ILM-политику с rollover каждые 5 минут (для теста).
2. Наблюдай, как создаются новые индексы.
3. Проверь удаление старых через `DELETE`.

---

## 📊 Тест по модулю 4

1. Что такое shard и replica в Elasticsearch?
2. Для чего используется mapping?
3. Что делает rollover policy?
4. Как изменить время обновления индекса?
5. Почему `keyword` лучше для фильтрации, чем `text`?

---

## 🧠 Best Practices

1. **Дневные индексы** — стандарт для логов, недельные — для архивов.
2. **Храни не больше 30–90 дней** данных в горячем кластере.
3. **Используй поля типа keyword** для меток, а не text.
4. **Следи за refresh_interval** — частые обновления убивают производительность.
5. **Включай реплики** (1 минимум) — спасут при падении узла.
6. **Не пиши напрямую в один индекс годами.** Используй rollover.

---

После этого модуля ты умеешь:

* устанавливать и настраивать Elasticsearch;
* создавать индексы и mapping;
* выполнять запросы и фильтрацию;
* использовать rollover и ILM для автоматического управления хранением;
* принимать логи от Fluentd в реальном времени.

---

# **МОДУЛЬ 5. ВИЗУАЛИЗАЦИЯ И АНАЛИЗ ЛОГОВ (KIBANA / GRAFANA)**

---

## 🎯 Цель

Научиться искать, фильтровать и визуализировать логи, строить дашборды, находить закономерности и настраивать алерты.
После этого модуля ты сможешешь работать с Kibana и Grafana так же уверенно, как с `grep`.

---

## 1. Что такое Kibana и Grafana

| Инструмент  | Назначение                           | Источник данных                                                           |
| ----------- | ------------------------------------ | ------------------------------------------------------------------------- |
| **Kibana**  | визуальный интерфейс к Elasticsearch | только Elasticsearch                                                      |
| **Grafana** | дашборды для метрик и логов          | поддерживает десятки источников (ES, Loki, Prometheus, PostgreSQL и т.д.) |

> **Best practice:**
> Используй Kibana для глубокой аналитики и фильтров, Grafana — для общих дашбордов и алертов.

---

## 2. Установка Kibana

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.14.0-amd64.deb
sudo dpkg -i kibana-8.14.0-amd64.deb
sudo systemctl enable kibana
sudo systemctl start kibana
```

Проверка:
Открой браузер → `http://localhost:5601`
(если Elasticsearch без аутентификации — Kibana поднимется сразу).

**Конфиг / etc/kibana/kibana.yml**

```yaml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://127.0.0.1:9200"]
```

---

## 3. Первый запуск и Index Pattern

1. Зайди в Kibana → **Discover**.
2. Нажми **Create index pattern**.
3. Укажи имя, соответствующее твоему индексу, например `fluentd-*`.
4. В поле времени выбери `@timestamp`.

Теперь Kibana “понимает”, что каждая запись в этом индексе имеет временную ось — можно строить графики и фильтры.

---

## 4. Поиск и фильтрация в Kibana

### Примеры KQL (Kibana Query Language)

```
level : "ERROR"
service : "nginx" and message : "timeout"
ip : 192.168.* and not level : "INFO"
```

**KQL** понимает операторы `and`, `or`, `not`, `*`, и автоматически подсвечивает типы полей.

> 🔎 Если ты ищешь текст в `message`, пиши без кавычек — Kibana найдёт по подстроке.

---

## 5. Визуализация в Kibana

### 5.1 Создание дашборда

1. Перейди → **Visualize Library** → **Create visualization**.
2. Выбери тип, например **Area**, **Bar**, **Pie** или **Metric**.
3. Укажи поле времени (`@timestamp`).
4. Настрой агрегацию (например, `Count of records` по `level.keyword`).
5. Сохрани и добавь на дашборд.

### 5.2 Пример – ошибки по сервисам

* **X-ось:** `@timestamp`
* **Разделение по цвету:** `service.keyword`
* **Фильтр:** `level : ERROR`
  → Получишь график количества ошибок по времени для каждого сервиса.

---

## 6. Алерты и оповещения в Kibana

Kibana умеет создавать **Rule-based alerts** — срабатывают, если условие в Elasticsearch выполнено.

**Пример:** “Если за 5 минут > 10 ошибок в auth-сервисе — шлём в Slack”.

1. Перейди → **Stack Management → Rules and Connectors**
2. Создай новое правило `Elasticsearch query`.
3. Запрос:

   ```
   level:"ERROR" and service:"auth"
   ```
4. Период: каждые 1 мин.
5. Действие: Slack или Email.

---

## 7. Grafana — вторая половина монеты

Grafana часто ставится поверх Kibana, когда нужно объединить логи и метрики.

### 7.1 Установка

```bash
sudo apt install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/oss/release/grafana_10.3.0_amd64.deb
sudo dpkg -i grafana_10.3.0_amd64.deb
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Интерфейс — `http://localhost:3000`, логин/пароль по умолчанию `admin/admin`.

---

## 8. Подключение Elasticsearch к Grafana

1. **Configuration → Data Sources → Add data source**
2. Выбери **Elasticsearch**
3. Введи `http://localhost:9200`
4. Укажи Index Pattern: `fluentd-*`
5. Time field name: `@timestamp`

Теперь можно строить дашборды поверх тех же логов, что в Kibana.

---

## 9. Пример дашборда в Grafana

**Панель 1.** Количество ошибок по уровню:

```json
Metric: Count  
Group by: level.keyword  
Filter: level != "INFO"
```

**Панель 2.** Топ 10 IP с наибольшим числом ошибок:

```json
Metric: Count  
Group by: ip.keyword  
Order: desc  
Limit: 10
```

**Панель 3.** Частота ошибок во времени (“heatmap”):

```json
Metric: Count  
Group by: Date histogram (@timestamp)
```

> **Совет:** в Grafana ставь “auto-refresh = 1 min” для реального времени.

---

## 10. Алертинг в Grafana

Grafana умеет создавать гибкие триггеры по любому запросу.

**Пример:**
если за 5 минут > 5 ошибок из одного IP — отправить в Telegram.

1. В панели нажми **Alert → Create alert rule**
2. Выбери метрику Count of records (`level="ERROR"`)
3. Условие: `IS ABOVE 5`
4. Интервал: `5m`
5. Действие: Webhook/Telegram.

---

## 🔧 Практические задания

### Задание 1. Создай дашборд ошибок

1. Собери логи в Elasticsearch.
2. В Kibana создай дашборд “Ошибки по времени”.
3. Добавь фильтр по `service.keyword`.

### Задание 2. Построй топ-IP в Grafana

1. Подключи Elasticsearch в Grafana.
2. Создай панель “Top 10 IP по ошибкам”.
3. Настрой автообновление раз в минуту.

### Задание 3. Настрой оповещение

1. Создай alert в Kibana для `service:auth`.
2. Добавь уведомление в Slack или Email.

---

## 📊 Тест по модулю 5

1. Как связаны Kibana и Elasticsearch?
2. Что такое Index Pattern и зачем он нужен?
3. Какой язык используется в поиске Kibana?
4. Можно ли Grafana подключить к нескольким источникам логов?
5. Как создать alert в Kibana?

---

## 🧠 Best Practices

1. **Один дашборд — одна тема.** Разделяй: ошибки, производительность, аутентификация.
2. **Используй фильтры по `service` и `level`.** Это ускоряет работу.
3. **Включай auto-refresh** для оперативного мониторинга.
4. **Ограничивай Retention** в Kibana — не строить графики за год.
5. **Делай дашборды многоуровневыми:** общие → детализация по сервису.

---

После модуля 5 ты умеешь:
* работать в Kibana и Grafana;
* искать и фильтровать логи по KQL;
* строить дашборды и алерты;
* объединять логи и метрики в одном виде;
* переходить от ошибки в графике к конкретному сообщению в логах.

---

# **МОДУЛЬ 6. GRAYLOG И АЛЬТЕРНАТИВНЫЕ РЕШЕНИЯ**

---

## 🎯 Цель

Освоить развёртывание и настройку Graylog, понять архитектуру, фильтры и потоки, научиться собирать логи от rsyslog, Fluent Bit и Docker.
После модуля ты сможешь построить готовый продакшн-уровень стек логов без ELK.

---

## 1. Что такое Graylog

Graylog — это **лог-платформа “всё-в-одном”**, объединяющая:

* сбор логов (через Syslog, GELF, Beats, HTTP),
* хранение (в Elasticsearch или OpenSearch),
* поиск и визуализацию (в своём веб-интерфейсе),
* маршрутизацию и обработку (Streams и Pipelines).

**Архитектура:**

```
Input → Journal (Kafka-подобная очередь) → Processing → Elasticsearch → UI
```

Логи сначала попадают в встроенный **journal** — надёжную очередь на диске,
а затем обрабатываются и индексируются в Elasticsearch.

---

## 2. Установка (Docker Compose)

Создадим простой кластер: MongoDB (метаданные), Elasticsearch (хранилище), Graylog (обработка).

**docker-compose.yml**

```yaml
version: '3'
services:
  mongo:
    image: mongo:6
    restart: always
    volumes:
      - ./mongo_data:/data/db

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.14.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./es_data:/usr/share/elasticsearch/data

  graylog:
    image: graylog/graylog:6.0
    environment:
      - GRAYLOG_HTTP_EXTERNAL_URI=http://localhost:9000/
      - GRAYLOG_PASSWORD_SECRET=SomeRandomLongString
      - GRAYLOG_ROOT_PASSWORD_SHA2=$(echo -n "admin" | sha256sum | awk '{print $1}')
      - GRAYLOG_ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - GRAYLOG_MONGODB_URI=mongodb://mongo/graylog
    entrypoint: /usr/bin/tini -- wait-for-it elasticsearch:9200 -- /docker-entrypoint.sh
    depends_on:
      - mongo
      - elasticsearch
    ports:
      - "9000:9000"   # Web UI
      - "1514:1514/udp" # Syslog input
      - "12201:12201/udp" # GELF input
```

**Что здесь происходит:**

* **mongo** хранит конфигурации (inputs, users, дашборды);
* **elasticsearch** — хранит сами логи;
* **graylog** — центральный узел приёма и анализа логов;
* порты `1514/udp` и `12201/udp` слушают входящие логи (Syslog и GELF);
* веб-панель доступна на `http://localhost:9000`.

---

## 3. Первичный вход

После запуска:

```bash
docker-compose up -d
```

Перейди в браузере:
`http://localhost:9000`
Логин: `admin`, пароль: `admin`

Graylog автоматически создаст дефолтные индексы и кэш очереди.

---

## 4. Inputs — приём логов

Graylog умеет принимать десятки форматов. Основные:

* **Syslog UDP/TCP**
* **GELF (Graylog Extended Log Format)** — родной формат JSON.
* **Beats / HTTP / Raw TCP**

### Пример настройки Syslog-input:

1. В UI → **System → Inputs → Syslog UDP → Launch new input**
2. Port: `1514`
3. Bind address: `0.0.0.0`
4. Назови input, например `syslog-input`.

Теперь сервер слушает порт 1514.

---

## 5. Отправка логов с клиента (rsyslog)

На клиенте создаём файл `/etc/rsyslog.d/90-graylog.conf`:

```
*.*  @graylog-server:1514
```

Перезапуск:

```bash
systemctl restart rsyslog
```

Теперь все системные логи с клиента будут стекаться в Graylog.

---

## 6. Streams — потоки логов

**Streams** — мощная фича, которая позволяет разделять потоки логов по условиям.
Каждый stream — это как фильтр: “всё, где `facility=auth` — в поток Security”.

**Пример:**

1. Создай Stream `Auth Logs`.
2. Условие: `source` содержит `auth`.
3. Назначь индекс `auth-logs`.

Теперь все логи с “auth” будут храниться отдельно и иметь свои дашборды.

---

## 7. Extractors — извлечение данных

**Extractors** превращают сырые строки в структурированные поля.

Пример лог-строки:

```
Oct 07 10:21:18 debian sshd[3181]: Failed password for root from 185.241.56.77 port 51822 ssh2
```

Можно добавить extractor типа **Regex**:

```
Failed password for (?<user>\S+) from (?<ip>\d+\.\d+\.\d+\.\d+)
```

→ Graylog автоматически создаст поля:

```
user=root
ip=185.241.56.77
```

---

## 8. Pipelines — обработка логов

Pipelines позволяют выполнять сложную логику обработки:

* удалять поля,
* нормализовать уровни (`WARN` → `WARNING`),
* перенаправлять потоки.

Пример pipeline-правила:

```pseudocode
rule "normalize nginx levels"
when
  has_field("level") && to_lowercase($message.level) == "warn"
then
  set_field("level", "WARNING");
end
```

---

## 9. Визуализация и поиск

Graylog имеет встроенный **Search** и **Dashboards**.

Пример запросов (Lucene-синтаксис):

```
level:ERROR
source:nginx AND message:"timeout"
ip:192.168.*
```

Можно сохранять запросы как “виджеты” и собирать из них дашборды:

* Errors per Service
* Top IPs
* SSH Brute-Force Attempts

---

## 10. Алерты в Graylog

Алерты (Event Definitions) срабатывают по условиям: “если за 1 минуту > 10 ERROR”.

1. **Alerts → Event Definitions → Create Event Definition**
2. Условие:

   ```
   level:ERROR AND service:auth
   ```
3. Frequency: 1 минута
4. Action: Email / HTTP Webhook / Slack.

---

## 🔧 Практические задания

### Задание 1. Разверни Graylog в Docker

1. Склонируй compose-файл.
2. Подними кластер.
3. Проверь веб-панель на `http://localhost:9000`.

### Задание 2. Настрой приём Syslog

1. Создай input Syslog UDP (порт 1514).
2. Настрой клиент rsyslog для отправки логов.
3. Убедись, что логи появляются в “Search”.

### Задание 3. Создай Stream и Extractor

1. Создай поток `SSH Logs`.
2. Настрой extractor по полю `ip`.
3. Сделай график количества попыток входа по IP.

---

## 📊 Тест по модулю 6

1. Из чего состоит архитектура Graylog?
2. Для чего нужен journal?
3. Чем Streams отличаются от Pipelines?
4. Как создать extractor и что он делает?
5. На каком порту Graylog слушает Syslog UDP по умолчанию?

---

## 🧠 Best Practices

1. **Включи журнал (journal):** он спасает логи при падении Elasticsearch.
2. **Разделяй потоки по типам:** системные, безопасность, приложения.
3. **Не отправляй в Graylog “DEBUG”-уровень** — экономь место.
4. **Используй GELF** вместо plain-syslog — быстрее, без потерь формата.
5. **Регулярно чисти индексы** (через Index Sets), чтобы не переполнить ES.
6. **Храни Extractors и Pipelines в Git** — это код инфраструктуры.

---

После этого модуля ты умеешь:
* Разворачивать Graylog;
* Принимать логи через Syslog и GELF;
* Создавать Streams, Extractors и Pipelines;
* Строить дашборды и алерты;
* Работать без Fluentd, если нужна лёгкая альтернатива ELK.

---
