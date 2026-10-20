# SOC Stand

Стенд из 3 виртуальных машин для развёртывания SOC-инструментов.

## Содержание

- [Цель](#цель)
- [Архитектура](#архитектура)
- [Этапы работы](#этапы-работы)

## Цель

Построить стенд из **3 виртуальных машин** для развёртывания и тестирования **SOC-инструментов** (Falco, OPA, cosign).

**Примечание:** файл `Vagrantfile` находится в корне репозитория. Все команды ниже выполняются в папке проекта.

## Архитектура

| VM | IP | Роль | Ресурсы |
|---|---|---|---|
| **docker** | 192.168.56.10 | запуск контейнеров | 2 CPU, 2 GB |
| **docker_registry** | 192.168.56.11 | хранение образов | 2 CPU, 2 GB |
| **storage** | 192.168.56.12 | хранение логов | 2 CPU, 2 GB |

## Этапы работы

### Этап 1: Проверка Vagrant

**Зачем:** убедиться, что Vagrant установлен и готов к работе.

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant --version
```

<img width="1097" height="119" alt="image" src="https://github.com/user-attachments/assets/d3dcd8fb-8187-47be-b1e1-7ba17ff21e85" />

Vagrant установлен, версия 2.4.9. Если команда не найдена — Vagrant не установлен в PATH.

### Этап 2: Проверка Vagrantfile

**Зачем:** убедиться, что `Vagrantfile` уже создан в проекте и не требует повторной инициализации.

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant init bento/ubuntu-24.04
```

<img width="1348" height="171" alt="image" src="https://github.com/user-attachments/assets/8c9b6f8d-293a-47c4-97f8-b91a7f3e2c74" />

`Vagrantfile` уже создан в проекте, поэтому команда `init` не требуется. Vagrant предупреждает и не перезаписывает существующий файл.

### Этап 3: Запуск стенда

**Зачем:** создать и запустить все три VM из бокса.

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant up
```

Vagrant читает `Vagrantfile`, создаёт три VM из бокса и запускает их. При первом запуске скачивается бокс (~600 МБ), поэтому процесс может занять 5-10 минут. При последующих запусках VM поднимаются за 1-2 минуты.

### Этап 4: Статус VM

**Зачем:** проверить, что все три VM успешно запустились.

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant status
```

<img width="1336" height="413" alt="image" src="https://github.com/user-attachments/assets/778cb626-6528-4b45-aacc-13b750eb6591" />

Все три VM запущены (`running`). Стенд готов к работе.

### Этап 5: Вход в VM docker

**Зачем:** подключиться к VM `docker` для дальнейшей настройки.

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant ssh docker
```

<img width="1503" height="798" alt="image" src="https://github.com/user-attachments/assets/2df1a6f1-016d-46ad-968d-d010042a61df" />

Vagrant подключается к VM по SSH-ключу, пароль не запрашивается. Приглашение `vagrant@docker` подтверждает успешный вход в виртуальную машину. Аналогично можно зайти в `docker_registry` и `storage` — команды `vagrant ssh docker_registry` и `vagrant ssh storage`.

### Этап 6: Проверка /etc/hosts

**Зачем:** убедиться, что все VM видят друг друга по имени, а не только по IP.

**Где:** VM `docker`.

**Команда:**

```bash
cat /etc/hosts
```

<img width="1124" height="567" alt="image" src="https://github.com/user-attachments/assets/8decc3bb-d226-4976-a823-bef7bf5e8cb0" />

Все три VM прописаны в локальном справочнике имён. Это позволяет обращаться к ним по имени, а не по IP-адресу — удобнее и безопаснее при изменениях адресации. Файл `/etc/hosts` нужно привести к такому виду **на всех трёх VM** — `docker`, `docker_registry` и `storage`.

### Этап 7: Ping между VM

**Зачем:** проверить, что сеть между VM работает и имена разрешаются в правильные IP.

**Где:** VM `docker`.

**Команда:**

```bash
ping docker-registry
```

<img width="1518" height="389" alt="image" src="https://github.com/user-attachments/assets/e3dc8dfb-70e7-4462-b756-9836b7206a6a" />

Потери пакетов 0%, имя разрешилось в IP `192.168.56.11`. Сеть между VM работает, `/etc/hosts` настроен корректно. Для полной проверки стоит также пингануть `storage` с `docker` и наоборот.

### Этап 8: Список боксов

**Зачем:** убедиться, что нужный бокс находится в кэше и не требует повторной загрузки.

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant box list
```

<img width="1053" height="135" alt="image" src="https://github.com/user-attachments/assets/2705640b-37c8-49ab-9fe3-2174f2f67115" />

Бокс `bento/ubuntu-24.04` находится в локальном кэше Vagrant. Именно он используется как основа для всех трёх VM, и его не нужно скачивать повторно.

### Этап 9: Версия Docker

**Зачем:** убедиться, что Docker установлен и работает.

**Где:** VM `docker`.

**Команда:**

```bash
docker --version
```

<img width="1034" height="117" alt="image" src="https://github.com/user-attachments/assets/74d4e65d-5229-455d-b112-1381e91f2b83" />

Docker установлен, версия 29.1.3. Если выводится ошибка — Docker не установлен или не добавлен в PATH. **Установку Docker нужно выполнить также на `docker_registry`** (на `storage` Docker не нужен).

### Этап 10: Добавление в группу docker

**Зачем:** дать пользователю `vagrant` право работать с Docker без `sudo`.

**Где:** VM `docker`.

**Команда:**

```bash
sudo usermod -aG docker vagrant
```

<img width="977" height="109" alt="image" src="https://github.com/user-attachments/assets/c583a7a3-dee5-450f-89e8-f18680276f4a" />

Пользователь `vagrant` добавлен в группу `docker`. Это позволит работать с Docker без `sudo`. Флаг `-aG` добавляет в группу, не удаляя из остальных. Изменения вступают в силу после повторного входа в систему. **Команду нужно выполнить также на `docker_registry`.**

### Этап 11: Статус Docker

**Зачем:** убедиться, что Docker-демон запущен и включён в автозагрузку.

**Где:** VM `docker`.

**Команда:**

```bash
sudo systemctl status docker
```

<img width="2539" height="898" alt="image" src="https://github.com/user-attachments/assets/77d550a3-f688-456c-b877-6bec68546596" />

Docker-демон работает (`active (running)`) и включён в автозагрузку (`enabled`). Служба запустится автоматически при перезагрузке VM — не нужно запускать Docker вручную.

### Этап 12: Проверка docker ps

**Зачем:** убедиться, что Docker работает и показывает список контейнеров.

**Где:** VM `docker`.

**Команда:**

```bash
docker ps
```

<img width="1356" height="125" alt="image" src="https://github.com/user-attachments/assets/8ddab2b8-cae4-4327-96a3-03fb0bc719f2" />

Список пуст — запущенных контейнеров нет. Это нормально для чистой установки: мы ещё ничего не запускали. Флаг `-a` показал бы и остановленные контейнеры.

### Этап 13: Запуск Registry

**Зачем:** развернуть приватный Docker Registry для хранения образов.

**Где:** VM `docker_registry`.

**Команда:**

```bash
docker run -d --restart=always --name registry -p 5000:5000 -e REGISTRY_STORAGE_DELETE_ENABLED=true registry:2
```

<img width="2538" height="176" alt="image" src="https://github.com/user-attachments/assets/0c98b805-14c0-4973-8333-4c2605f3f315" />

Docker вернул ID контейнера — запуск успешен. Флаги: `-d` (в фоне), `--restart=always` (автозапуск), `-p 5000:5000` (проброс порта), `-e REGISTRY_STORAGE_DELETE_ENABLED=true` (разрешить удаление образов через API).

### Этап 14: Проверка Registry

**Зачем:** убедиться, что контейнер `registry` запущен и порт проброшен.

**Где:** VM `docker_registry`.

**Команда:**

```bash
docker ps
```

<img width="2497" height="243" alt="image" src="https://github.com/user-attachments/assets/d5211db5-650e-4b8c-aa02-cf8fd8e360c6" />

Контейнер `registry` запущен. Порт 5000 проброшен на хост VM. Registry готов принимать образы.

### Этап 15: Проверка API Registry

**Зачем:** убедиться, что Registry отвечает на HTTP-запросы.

**Где:** VM `docker_registry`.

**Команда:**

```bash
curl http://localhost:5000/v2/_catalog
```

<img width="1305" height="124" alt="image" src="https://github.com/user-attachments/assets/2180d35c-fc68-429f-92c6-da67a37755f0" />

Registry отвечает на HTTP-запросы. Пустой массив означает, что Registry работает, но образов пока нет. Эндпоинт `/v2/_catalog` — стандартный API Docker Registry для получения списка образов.

### Этап 16: Настройка insecure-registries

**Зачем:** разрешить Docker работать с Registry по HTTP, так как по умолчанию требуется HTTPS.

**Где:** VM `docker`.

**Команда:**

```bash
cat /etc/docker/daemon.json
```

<img width="957" height="204" alt="image" src="https://github.com/user-attachments/assets/f9683982-69df-4baa-b63b-ddc75396cb7e" />

Docker настроен доверять Registry по HTTP. Без этой настройки `docker push` завершится ошибкой `http: server gave HTTP response to HTTPS client`, поскольку по умолчанию Docker требует HTTPS. После изменения файла нужно перезапустить Docker: `sudo systemctl restart docker`. 

### Этап 17: Pull alpine

**Зачем:** скачать тестовый образ, который будем пушить в Registry.

**Где:** VM `docker`.

**Команда:**

```bash
docker pull alpine
```

<img width="1534" height="259" alt="image" src="https://github.com/user-attachments/assets/cf44700a-3211-4875-a59f-adb30c0ad613" />

Образ `alpine` скачан из публичного Docker Hub. Digest — уникальный хеш образа, по которому можно проверить целостность. Этот образ используется как тестовый для проверки Registry.

### Этап 18: Tag alpine

**Зачем:** добавить к образу тег с адресом Registry, чтобы Docker знал, куда пушить.

**Где:** VM `docker`.

**Команда:**

```bash
docker tag alpine docker-registry:5000/alpine
```

<img width="1227" height="85" alt="image" src="https://github.com/user-attachments/assets/1038040f-cdd2-459e-a43c-8b9bf02b146e" />

Docker определяет, куда пушить образ, по имени тега. Формат `хост:порт/имя` указывает на приватный Registry. Тег не копирует образ — он добавляет второе имя тому же образу.

### Этап 19: Push alpine

**Зачем:** отправить образ в приватный Registry.

**Где:** VM `docker`.

**Команда:**

```bash
docker push docker-registry:5000/alpine
```

<img width="2512" height="381" alt="image" src="https://github.com/user-attachments/assets/245d1ac7-20fa-4a39-b4f7-32742c014056" />

Образ отправлен в Registry (`Pushed`). Предупреждение о multiplatform — норма для однослойных образов, оно не влияет на результат. Образ теперь хранится в Registry.

### Этап 20: Проверка каталога

**Зачем:** убедиться, что образ действительно появился в Registry.

**Где:** VM `docker`.

**Команда:**

```bash
curl http://docker-registry:5000/v2/_catalog
```

<img width="1226" height="115" alt="image" src="https://github.com/user-attachments/assets/eb055e82-9667-428e-92fa-36fcf11f90d3" />

Образ `alpine` появился в Registry. Push прошёл успешно, Registry хранит образ и готов отдавать его другим клиентам.

### Этап 21: Pull из Registry

**Зачем:** проверить, что образ можно скачать из Registry — полный цикл.

**Где:** VM `docker`.

**Команда:**

```bash
docker pull docker-registry:5000/alpine
```

<img width="1529" height="263" alt="image" src="https://github.com/user-attachments/assets/66dbf972-abf5-4e98-9cb6-43d5ad92fdaf" />

Образ успешно скачан из Registry (`Downloaded`). Полный цикл `push → pull` работает — Registry выполняет свою роль хранилища образов.

### Этап 22: Проверка образов

**Зачем:** убедиться, что образ есть локально и с исходным именем, и с именем из Registry.

**Где:** VM `docker`.

**Команда:**

```bash
docker images
```

<img width="2501" height="229" alt="image" src="https://github.com/user-attachments/assets/d710ddf9-ea5c-4892-8ade-f1ad235c172f" />

Оба тега ссылаются на один и тот же образ. Локально образ есть и с исходным именем, и с именем из Registry. Это подтверждает, что образ успешно прошёл полный цикл.

### Этап 23: Создание папки для сертификатов

**Зачем:** подготовить место, где будут храниться ключ и сертификат.

**Где:** VM `docker_registry`.

**Команды:**

```bash
mkdir -p /home/vagrant/certs
cd /home/vagrant/certs
```

<img width="1095" height="134" alt="image" src="https://github.com/user-attachments/assets/4af1bffb-299b-407a-aacc-76c94b9b2e7f" />

Папка `/home/vagrant/certs` создана. Флаг `-p` создаёт все родительские папки и не ругается, если папка уже существует.

### Этап 24: Генерация самоподписанного сертификата

**Зачем:** создать SSL-сертификат, чтобы Registry мог работать по HTTPS.

**Где:** VM `docker_registry`.

**Команда:**

```bash
openssl req -newkey rsa:4096 -nodes -sha256 -keyout domain.key -x509 -days 365 -out domain.crt -subj "/C=RU/CN=docker-registry" -addext "subjectAltName = DNS:docker-registry, DNS:docker-registry.local, IP:192.168.56.11"
```

<img width="2515" height="824" alt="image" src="https://github.com/user-attachments/assets/ca26c8a0-9cdd-4961-80da-271d270d0fad" />

Созданы два файла: `domain.key` — приватный ключ, `domain.crt` — публичный сертификат. Флаг `-x509` создаёт самоподписанный сертификат. `-addext "subjectAltName = ..."` добавляет SAN — без него Docker не примет сертификат (ошибка `legacy Common Name`).

### Этап 25: Проверка файлов сертификата

**Зачем:** убедиться, что оба файла созданы и лежат на месте.

**Где:** VM `docker_registry`.

**Команда:**

```bash
ls -la
```

<img width="1133" height="294" alt="image" src="https://github.com/user-attachments/assets/b4aeeecf-4b94-4f79-a395-7a547f0e8e1f" />

В папке два файла: `domain.crt` (1858 байт) и `domain.key` (3272 байта). Число `1` в выводе — это количество ссылок на файл. Обычная ситуация.

### Этап 26: Перезапуск Registry с TLS

**Зачем:** запустить Registry так, чтобы он использовал наш сертификат для HTTPS.

**Где:** VM `docker_registry`.

**Команды:**

```bash
docker stop registry
docker rm registry
docker run -d --restart=always --name registry -p 5000:5000 -v /home/vagrant/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key -e REGISTRY_STORAGE_DELETE_ENABLED=true registry:2
docker ps
```

<img width="2540" height="526" alt="image" src="https://github.com/user-attachments/assets/4ec3e3c2-317d-443d-bfd7-d42d618d6265" />

Registry запущен с TLS. Флаг `-v` подключает папку с сертификатами внутрь контейнера. Переменные `REGISTRY_HTTP_TLS_CERTIFICATE` и `REGISTRY_HTTP_TLS_KEY` указывают пути к сертификату и ключу.

### Этап 27: Проверка HTTPS

**Зачем:** убедиться, что Registry отвечает по HTTPS.

**Где:** VM `docker_registry`.

**Команды:**

```bash
curl https://localhost:5000/v2/_catalog
curl -k https://localhost:5000/v2/_catalog
```

<img width="1454" height="381" alt="image" src="https://github.com/user-attachments/assets/8ad8b949-3f79-40ed-832e-865898943784" />

Первая команда — ошибка `self-signed certificate`: curl не доверяет нашему сертификату. Вторая — с флагом `-k` (игнорировать проверку) — успешный ответ `{"repositories":[]}`. Значит, HTTPS работает.

### Этап 28: Создание папки доверия на docker

**Зачем:** подготовить место, куда Docker будет смотреть при подключении к Registry.

**Где:** VM `docker`.

**Команда:**

```bash
sudo mkdir -p /etc/docker/certs.d/docker-registry:5000
scp vagrant@docker-registry:/home/vagrant/certs/domain.crt /tmp/domain.crt
```

<img width="2445" height="380" alt="image" src="https://github.com/user-attachments/assets/25caff9d-c2be-470e-b552-eda778dcfb05" />

Папка создана. Имя папки — это адрес Registry. Docker ищет сертификат именно здесь. На скриншоте видно сразу две команды: `mkdir` и `scp` — потому что выполнялись подряд.

### Этап 29: Копирование сертификата с docker_registry

**Зачем:** перенести публичный сертификат с сервера на клиент.

**Где:** VM `docker`.

**Команда:**

```bash
sudo cp /tmp/domain.crt /etc/docker/certs.d/docker-registry:5000/ca.crt
```

<img width="1739" height="96" alt="image" src="https://github.com/user-attachments/assets/2ba29222-d70b-495d-87f6-cf581629be1a" />

Сертификат скопирован в `/tmp/domain.crt`. При первом подключении SSH спросил про fingerprint — подтвердили `yes`. Ввели пароль `vagrant`. Копируем через `/tmp`, потому что `scp` не имеет прав `sudo` — потом перенесём в нужное место.

### Этап 30: Перезапуск Docker

**Зачем:** Docker не видит новый сертификат, пока не перезапустится служба.

**Где:** VM `docker`.

**Команда:**

```bash
sudo systemctl restart docker
```

<img width="941" height="93" alt="image" src="https://github.com/user-attachments/assets/68d7d170-d104-461b-bdea-cf361a1c38f6" />

Docker перезапущен. Теперь он применит настройки доверия к сертификату, который мы положили в /etc/docker/certs.d/.

## Этап 31: Удаление insecure-registries

**Зачем:** HTTPS уже работает, поэтому режим `insecure-registries` (без шифрования) больше не нужен. Оставлять его — снижать безопасность.

**Где:** VM `docker`.

**Команда:**

```bash
sudo nano /etc/docker/daemon.json
cat /etc/docker/daemon.json
```

<img width="1003" height="164" alt="image" src="https://github.com/user-attachments/assets/fd428812-4100-485e-89bf-f649113ee57b" />

Файл daemon.json теперь пустой — строка с insecure-registries удалена. Docker будет работать только по HTTPS.

### Этап 32: Ошибка с CN — проверка сертификата

**Зачем:** Docker отказывается работать с сертификатом, в котором нет поля SAN (Subject Alternative Name). Нужно увидеть ошибку и понять причину.

**Где:** VM `docker`.

**Команды:**

```bash
docker pull docker-registry:5000/alpine
```

<img width="2513" height="235" alt="image" src="https://github.com/user-attachments/assets/7d3ed2e3-3828-4155-8e97-f2e54a84e46e" />

Ошибка: tls: failed to verify certificate: x509: certificate relies on legacy Common Name field, use SANs instead. Docker (начиная с версии 1.15) требует, чтобы в сертификате было поле SAN, а не только CN. Придётся пересоздать сертификат.


### Этап 33: Пересоздание сертификата с SAN

**Зачем:** Docker требует наличие поля SAN (Subject Alternative Name). Пересоздаём сертификат с правильным полем, чтобы Docker мог его принять.

**Где:** VM `docker_registry`.

**Команда:**

```bash
docker stop registry
docker rm registry
docker run -d --restart=always --name registry -p 5000:5000 -v /home/vagrant/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key -e REGISTRY_STORAGE_DELETE_ENABLED=true registry:2
docker ps
```

<img width="2508" height="518" alt="image" src="https://github.com/user-attachments/assets/f96dbc4b-e24e-4f29-8914-176414f97b36" />

Сертификат пересоздан с полем subjectAltName. Registry перезапущен с новым сертификатом. Теперь Docker должен принять его.


### Этап 34: Обновление сертификата на docker

**Зачем:** заменить старый сертификат новым — с SAN.

**Где:** VM `docker`.

**Команды:**

```bash
scp vagrant@docker-registry:/home/vagrant/certs/domain.crt /tmp/domain.crt
sudo cp /tmp/domain.crt /etc/docker/certs.d/docker-registry:5000/ca.crt
sudo systemctl restart docker
```

<img width="1784" height="166" alt="image" src="https://github.com/user-attachments/assets/a93d000f-ca53-4770-a2a3-a706d56a7df9" />
<img width="1730" height="118" alt="image" src="https://github.com/user-attachments/assets/6071336e-19eb-4d59-b3a8-fe099518781b" />

Новый сертификат скопирован на `docker` и заменил старый. Docker перезапущен.

### Этап 35: Push по HTTPS

**Зачем:** убедиться, что push работает по HTTPS без insecure-режима.

**Где:** VM `docker`.

**Команда:**

```bash
docker push docker-registry:5000/alpine:latest
```

<img width="2544" height="468" alt="image" src="https://github.com/user-attachments/assets/825fe507-6a63-4bef-89d4-7fe37d2631ac" />

Образ успешно отправлен: `Pushed`. Push по HTTPS работает — Docker доверяет сертификату.

### Этап 36: Pull по HTTPS

**Зачем:** проверить полный цикл — push и pull по HTTPS.

**Где:** VM `docker`.

**Команды:**

```bash
docker rmi docker-registry:5000/alpine
docker pull docker-registry:5000/alpine
```

<img width="1543" height="381" alt="image" src="https://github.com/user-attachments/assets/d074d45b-155c-42a6-8b52-a8243d490931" />

Образ удалён локально (`Deleted`), потом скачан из Registry (`Downloaded`). Полный цикл по HTTPS работает: push → удаление → pull.


### Этап 37: Скачивание ключа Falco

**Зачем:** Получить публичный ключ разработчиков Falco, чтобы `apt` мог проверять подлинность пакетов.

**Где:** VM `docker`.

**Команды:**

```bash
curl -fsSL https://falco.org/repo/falcosecurity-packages.asc | sudo gpg --dearmor -o /usr/share/keyrings/falco-archive-keyring.gpg
ls -la /usr/share/keyrings/falco-archive-keyring.gpg
```

<img width="1259" height="137" alt="image" src="https://github.com/user-attachments/assets/601c64a8-7f2d-484d-85f1-d400dd16d810" />

Ключ успешно скачан и сохранен в системную директорию `/usr/share/keyrings/`. Файл существует и имеет размер 1189 байт.

### Этап 38: Добавление репозитория Falco

**Зачем:** Указать системе, откуда скачивать пакет Falco и каким ключом проверять подпись.

**Где:** VM `docker`.

**Команда:**

```bash
sudo nano /etc/apt/sources.list.d/falco.sources
```

*(Вставлены 5 строк конфигурации репозитория)*

```bash
cat /etc/apt/sources.list.d/falco.sources
```

<img width="1278" height="310" alt="image" src="https://github.com/user-attachments/assets/5b22e97b-bb3d-45c1-8a07-565fd833ee43" />


Конфигурационный файл репозитория создан. Теперь `apt` знает, где искать Falco и каким ключом проверять его подлинность.

### Этап 39: Обновление списков пакетов

**Зачем:** Заставить `apt` прочитать новый файл репозитория и скачать метаданные о пакетах Falco.

**Где:** VM `docker`.

**Команда:**

```bash
sudo apt update
```

<img width="1024" height="607" alt="image" src="https://github.com/user-attachments/assets/31c2a26e-153f-49f2-bc60-5efb2e61a3f1" />

`apt` успешно скачал данные из нового репозитория. В строках `Get:4` и `Get:9` видно обращение к `d20hasrqv82i0q.cloudfront.net` — это CDN, который раздает пакеты Falco.

### Этап 40: Установка Falco

**Зачем:** Установить программу Falco на виртуальную машину.

**Где:** VM `docker`.

**Команда:**

```bash
sudo apt install -y falco
```

<img width="1024" height="170" alt="image" src="https://github.com/user-attachments/assets/b672fc1a-317d-402e-9e39-64d136b3129f" />

Установщик автоматически определил ядро, выбрал современный драйвер `falco-modern-bpf` и запустил службу. Строки `Created symlink` подтверждают, что служба включена в автозагрузку.

### Этап 41: Проверка статуса Falco

**Зачем:** Убедиться, что служба Falco работает, использует eBPF и загрузила правила безопасности.

**Где:** VM `docker`.

**Команда:**

```bash
sudo systemctl status falco
```
*(Выйти из статуса — `q`)*

<img width="1076" height="426" alt="image" src="https://github.com/user-attachments/assets/159460a4-bbe0-4feb-a046-308bf494c73e" />

Служба в статусе `active (running)`. В логах видно загрузку правил `/etc/falco/falco_rules.yaml` и использование `modern_ebpf`. Память занимает ~51 МБ.


### Этап 42: Настройка вывода Falco в файл

**Зачем:** По умолчанию Falco пишет в системный журнал. Для отправки через `rsyslog` нужно перенаправить вывод в отдельный файл.

**Где:** VM `docker`.

**Команда:**

```bash
sudo nano /etc/falco/falco.yaml
```

*(Изменены блоки: `syslog_output` выключен, `file_output` включен, путь указан `/var/log/falco/events.log`)*

<img width="1541" height="817" alt="image" src="https://github.com/user-attachments/assets/7ef5ba11-dc08-4326-bf52-8bc19f9d0c72" />

Файл конфигурации отредактирован. Теперь Falco будет писать события в `/var/log/falco/events.log`.

### Этап 43: Создание директории для логов Falco

**Зачем:** Falco не создаёт папку для логов автоматически. Без неё он не сможет записать файл `events.log`.

**Где:** VM `docker`.

**Команда:**

```bash
sudo mkdir -p /var/log/falco
```

<img width="943" height="91" alt="image" src="https://github.com/user-attachments/assets/9276f8af-c2cf-4b21-b48d-738c7910e382" />

Директория `/var/log/falco` создана. Теперь Falco сможет писать туда события.

### Этап 44: Перезапуск и проверка статуса Falco

**Зачем:** Применить новые настройки конфигурации (вывод в файл) и убедиться, что служба успешно запустилась.

**Где:** VM `docker`.

**Команды:**

```bash
sudo systemctl restart falco
sudo systemctl status falco
```
*(Выйти из статуса — `q`)*

<img width="2078" height="898" alt="image" src="https://github.com/user-attachments/assets/a70096b6-aba2-4ce9-a5c9-eb19526afabc" />

Служба успешно перезапущена и находится в статусе `active (running)`. В логах видно, что Falco загрузил правила безопасности и подключился к ядру через `modern_ebpf`.

### Этап 45: Генерация тестового события

**Зачем:** Проверить, что Falco реагирует на подозрительные действия и создает файл с логами.

**Где:** VM `docker`.

**Команда:**

```bash
sudo cat /etc/shadow > /dev/null
```

<img width="998" height="91" alt="image" src="https://github.com/user-attachments/assets/f5ea3e45-c983-4116-a3ad-fd88ca0bea58" />

Мы попытались прочитать файл с паролями. Вывод погашен в `/dev/null`, но Falco зафиксировал факт доступа. Теперь должен появиться файл `/var/log/falco/events.log`.

### Этап 46: Проверка лога Falco

**Зачем:** Убедиться, что сгенерированное событие действительно записалось в файл `/var/log/falco/events.log`.

**Где:** VM `docker`.

**Команда:**

```bash
sudo cat /var/log/falco/events.log
```

<img width="2519" height="232" alt="image" src="https://github.com/user-attachments/assets/53395706-94c7-4545-9044-919f41a38b1f" />

В логе появилось событие `Warning Sensitive file opened for reading...`. Falco зафиксировал чтение `/etc/shadow` командой `cat` от имени `root`. Контейнер не используется (`container_id=host`).

### Этап 47: Генерация события внутри контейнера

**Зачем:** Убедиться, что Falco видит подозрительную активность не только на хосте, но и внутри Docker-контейнеров.

**Где:** VM `docker`.

**Команды:**

```bash
docker run --rm -it alpine /bin/sh
```
*(внутри контейнера написать `exit`)*

```bash
sudo cat /var/log/falco/events.log
```

<img width="2528" height="524" alt="image" src="https://github.com/user-attachments/assets/b489c82e-940b-44f3-bc2a-da0bf27e40e8" />

В логе появилось второе событие: `Notice A shell was spawned in a container...`. В нем указано имя контейнера (`dreamy_brahmagupta`) и образ (`alpine`). Это доказывает, что Falco отслеживает процессы внутри контейнеров.

### Этап 48: Создание конфига rsyslog на docker

**Зачем:** Настроить `rsyslog` на чтение файла логов Falco и отправку этих событий по сети на сервер `storage`.

**Где:** VM `docker`.

**Команда:**

```bash
sudo nano /etc/rsyslog.d/50-falco.conf
```

*(Вставлены модуль `imfile`, `input` с указанием файла Falco и `action` для отправки по TCP)*

```bash
cat /etc/rsyslog.d/50-falco.conf
```

<img width="1729" height="253" alt="image" src="https://github.com/user-attachments/assets/95e78a10-febd-4c38-8865-f1e255e3d0f1" />

Конфиг создан. Rsyslog теперь будет следить за файлом `/var/log/falco/events.log` и при появлении новых строк отправлять их на IP `192.168.56.12` (порт 514) по протоколу TCP.

### Этап 49: Включение приема TCP-логов на storage

**Зачем:** `storage` должен уметь принимать логи от других машин по сети (TCP). По умолчанию эта функция отключена.

**Где:** VM `storage`.

**Команда:**

```bash
sudo nano /etc/rsyslog.conf
```

*(В конфиге раскомментированы строки `module(load="imtcp")` и `input(type="imtcp" port="514")`)*

<img width="1511" height="883" alt="image" src="https://github.com/user-attachments/assets/9b779b9c-004d-4688-9f90-abca3b319fe1" />

Модуль приема TCP-логов активирован. Теперь `rsyslog` на `storage` готов слушать порт 514 и принимать логи от `docker` и `docker_registry`.

### Этап 50: Создание шаблона для раскладки логов

**Зачем:** Настроить `rsyslog` так, чтобы входящие логи от разных машин и программ автоматически раскладывались по отдельным папкам и файлам.

**Где:** VM `storage`.

**Команда:**

```bash
sudo nano /etc/rsyslog.d/50-remote.conf
```

*(Вставлен шаблон `RemoteLogs` и правило `*.* ?RemoteLogs` с остановкой `& stop`)*

<img width="1632" height="213" alt="image" src="https://github.com/user-attachments/assets/63d01c93-dd7f-4b04-8d86-5c1111064241" />

Создан шаблон `$template RemoteLogs`. Теперь все входящие логи будут сохраняться в `/var/log/remote/<имя_хоста>/<имя_программы>.log`. Это позволяет не смешивать логи от `docker` и `docker_registry` в одну кучу.

### Этап 51: Создание директории для приема логов

**Зачем:** Создать папку, куда `rsyslog` будет складывать логи с других машин. Выдать права пользователю `syslog`, от имени которого работает служба, иначе она не сможет туда писать.

**Где:** VM `storage`.

**Команды:**

```bash
sudo mkdir -p /var/log/remote
sudo chown syslog:syslog /var/log/remote
```

<img width="1171" height="148" alt="image" src="https://github.com/user-attachments/assets/7612497f-0e07-447f-b506-b3741c6315b4" />

Директория `/var/log/remote` создана, и права на нее переданы пользователю `syslog`. Теперь `rsyslog` сможет сохранять входящие логи в эту папку.


### Этап 52: Перезапуск rsyslog на storage

**Зачем:** Применить новые настройки (включенный TCP-прием и созданный шаблон) к службе логирования.

**Где:** VM `storage`.

**Команды:**

```bash
sudo systemctl restart rsyslog
sudo systemctl status rsyslog
```
*(Выйти из статуса — `q`)*

<img width="2548" height="903" alt="image" src="https://github.com/user-attachments/assets/b04b7d58-9d9e-4f07-a933-ad42b83e547a" />

Служба `rsyslog` перезапущена и находится в статусе `active (running)`. Ошибок в логах нет, значит, конфигурация синтаксически верна, и порт 514 теперь слушается.

### Этап 53: Первая попытка перезапуска rsyslog на docker (Ошибка)

**Зачем:** Применить настройки отправки логов. Выяснить, что мешает службе работать корректно.

**Где:** VM `docker`.

**Команды:**

```bash
sudo systemctl restart rsyslog
sudo systemctl status rsyslog
```

<img width="2559" height="1064" alt="image" src="https://github.com/user-attachments/assets/996c9c5a-bc46-467c-8387-d5d811aa4626" />

В логах видно две ошибки:
1. `error during parsing file ... parameter 'state...'` — синтаксическая ошибка в конфиге.
2. `imfile: error accessing file ... Permission denied` — `rsyslog` не может прочитать файл `/var/log/falco/events.log`, так как он принадлежит `root`.

### Этап 54: Исправление конфига и прав на файл

**Зачем:** Устранить две ошибки, найденные на предыдущем этапе: убрать неподдерживаемый параметр `StateFile` из конфига и дать `rsyslog` права на чтение файла логов Falco.

**Где:** VM `docker`.

**Команды:**

```bash
sudo nano /etc/rsyslog.d/50-falco.conf
```
*(Из конфига убран параметр `StateFile="falco-state"`)*

```bash
cat /etc/rsyslog.d/50-falco.conf
sudo chmod 644 /var/log/falco/events.log
```

<img width="1431" height="263" alt="image" src="https://github.com/user-attachments/assets/d3e6bc4f-57ed-425a-8ad1-a346c75c57d5" />

Конфиг исправлен. Параметр `StateFile` удален (в версии rsyslog 8.23 он не поддерживается внутри `input`). Также выполнена команда `chmod 644`, которая дает права на чтение файла всем пользователям, включая `syslog`.

### Этап 55: Повторный перезапуск и проверка статуса rsyslog

**Зачем:** Применить исправленный конфиг и убедиться, что ошибки устранены, а служба работает стабильно.

**Где:** VM `docker`.

**Команды:**

```bash
sudo systemctl restart rsyslog
sudo systemctl status rsyslog
```
*(Выйти из статуса — `q`)*

<img width="2543" height="891" alt="image" src="https://github.com/user-attachments/assets/be7efc26-5d8b-424c-9a36-6de7171f9983" />

Служба в статусе `active (running)`. В логах больше нет ошибок `Permission denied` или ошибок парсинга. Теперь `rsyslog` успешно читает файл Falco и отправляет его на `storage`.

### Этап 56: Отправка тестового события на storage

**Зачем:** Сгенерировать новое событие, чтобы проверить, что цепочка Falco → rsyslog → storage работает после исправления всех ошибок.

**Где:** VM `docker`.

**Команда:**

```bash
sudo cat /etc/shadow > /dev/null
```

<img width="977" height="90" alt="image" src="https://github.com/user-attachments/assets/98bbeac4-9fe5-4b72-b799-7da7d28ab46b" />

Мы снова прочитали файл `/etc/shadow`, чтобы Falco сгенерировал алерт. Этот алерт должен быть отправлен через `rsyslog` на `storage`.

### Этап 57: Проверка логов Falco на storage

**Зачем:** Убедиться, что логи, сгенерированные Falco на `docker`, успешно дошли до `storage` и сохранились в правильном файле.

**Где:** VM `storage`.

**Команда:**

```bash
sudo cat /var/log/remote/docker/falco.log
```

<img width="2521" height="528" alt="image" src="https://github.com/user-attachments/assets/58887810-104f-4fe3-b090-872493a64324" />

В логе на `storage` видны все три события, которые мы сгенерировали ранее:
1. Чтение `/etc/shadow` (первый раз).
2. Запуск оболочки `sh` внутри контейнера `alpine`.
3. Чтение `/etc/shadow` (второй раз, после исправления конфига).

Это доказывает, что цепочка `Falco` → `rsyslog` → `storage` работает корректно.

### Этап 59: Создание кастомного правила Falco

**Зачем:** Штатные правила не всегда гарантируют детект запуска привилегированного контейнера. Создадим правило, которое ловит сам факт вызова `docker run --privileged`.

**Где:** VM `docker`.

**Команды:**

```bash
sudo nano /etc/falco/falco_rules.local.yaml
```


<img src="https://github.com/user-attachments/assets/b982b969-0675-468a-a118-79904bc65c1a" alt="image" />

Правило добавлено в файл локальных правил. Оно отслеживает запуск процесса `docker` с аргументами `run` и `--privileged`, используя макрос `spawned_process`.


### Этап 60: Проверка кастомного правила

**Зачем:** Убедиться, что созданное правило детектирует запуск привилегированного контейнера после перезапуска сервиса.

**Где:** VM `docker`.

**Команды:**

```bash
sudo systemctl restart falco
docker run --rm --privileged alpine true
```

<img src="https://github.com/user-attachments/assets/57747ff9-4560-4251-a263-3d1ed58696f7" alt="image" />

Falco перезапущен с новым конфигом. Выполнен запуск контейнера с флагом `--privileged`. Правило отслеживает сам вызов команды, поэтому контейнер завершается мгновенно, но событие фиксируется Falco.


### Этап 61: Проверка доставки кастомного события на storage

**Зачем:** Убедиться, что событие от нового правила Falco успешно передано через rsyslog и сохранено на центральном сервере.

**Где:** VM `storage`.

**Команды:**

```bash
sudo tail -n 5 /var/log/remote/docker/falco.log
```

<img src="https://github.com/user-attachments/assets/9db17713-4ddf-400a-ac5d-62abab7d0b68" alt="image" />

В логе на `storage` отображается событие `Warning Привилегированный контейнер запущен`, сгенерированное на `docker`. Цепочка Falco → rsyslog → storage работает корректно для кастомных правил.


### Этап 62: Загрузка утилиты Falcosidekick

**Зачем:** Скачать бинарный файл Falcosidekick, который будет выступать в роли промежуточного звена для пересылки алертов Falco во внешние системы (например, Telegram).

**Где:** VM `docker`.

**Команды:**

```bash
curl -L -o falcosidekick.tar.gz https://github.com/falcosecurity/falcosidekick/releases/download/2.31.0/falcosidekick_2.31.0_linux_amd64.tar.gz
```

<img src="https://github.com/user-attachments/assets/b01ecbeb-1dfa-4194-b1fd-ba44d1f4ca4c" alt="image" />

Архив с утилитой успешно скачан с официального репозитория GitHub. Размер файла составляет 47.8 МБ.

### Этап 63: Распаковка и установка Falcosidekick

**Зачем:** Извлечь бинарный файл из архива, переместить его в системную директорию и проверить работоспособность утилиты.

**Где:** VM `docker`.

**Команды:**

```bash
tar -xzf falcosidekick.tar.gz
sudo mv falcosidekick /usr/local/bin/
falcosidekick --version
```

<img src="https://github.com/user-attachments/assets/62c1dcf8-1c7b-4e3f-a161-2b9da88530fd" alt="image" />

Команда `tar` распаковала архив, после чего файл был перемещён в `/usr/local/bin/` для глобального доступа. Вызов с флагом `--version` подтвердил успешную установку (версия 2.31.0, платформа linux/amd64).


### Этап 64: Создание конфигурационного файла Falcosidekick

**Зачем:** Создать директорию и файл настроек, в котором будут указаны токен Telegram-бота и Chat ID получателя для пересылки алертов.

**Где:** VM `docker`.

**Команды:**

```bash
sudo mkdir -p /etc/falcosidekick
sudo nano /etc/falcosidekick/config.yaml
```
*(В файл добавлен блок настроек `telegram` с токеном и Chat ID)*

<img src="https://github.com/user-attachments/assets/7459c85e-2699-4cbc-9c52-fc5fe5e0a444" alt="image" />
<img width="1663" height="259" alt="image" src="https://github.com/user-attachments/assets/5a0fd0f2-05ad-44b6-85b0-9eea23ec8d26" />


Создана директория `/etc/falcosidekick`. В редакторе `nano` открыт файл `config.yaml` для внесения параметров интеграции с Telegram.


### Этап 65: Создание Telegram-бота для алертов

**Зачем:** Зарегистрировать нового бота в Telegram через BotFather, который будет выступать в роли получателя уведомлений от системы Falco.

**Где:** Telegram (хост-машина).

**Команды:**

*(В чате с `@BotFather` отправлены команды)*
```text
/newbot
SOC Stand Alerts
my_soc_stand_alerts_bot
```

<img src="https://github.com/user-attachments/assets/8f29b5e1-f53a-4e0c-9934-1126e0db5600" alt="image" />

Бот успешно создан. Имя бота — `SOC Stand Alerts`, username — `my_soc_stand_alerts_bot`. (Первый предложенный username `my_soc_alerts_bot` был отклонен, так как уже занят). BotFather выдал токен для дальнейшей настройки интеграции.


### Этап 66: Создание службы systemd для Falcosidekick

**Зачем:** Превратить Falcosidekick в системный демон (службу), который будет работать в фоне и автоматически запускаться при старте виртуальной машины, а также перезапускаться в случае сбоев.

**Где:** VM `docker`.

**Команды:**

```bash
sudo nano /etc/systemd/system/falcosidekick.service
```
*(В файл добавлен блок конфигурации службы)*


<img src="https://github.com/user-attachments/assets/3aa74bd3-f9e1-4f6f-92ef-3c2da4ee775a" alt="image" />

Файл службы создан. Параметр `After=network.target` гарантирует, что Falcosidekick запустится только после поднятия сети (нужно для работы с Telegram). `Restart=always` обеспечивает автоматический перезапуск при сбоях, а `WantedBy=multi-user.target` добавляет службу в автозагрузку системы.


### Этап 67: Запуск службы Falcosidekick

**Зачем:** Активировать службу в systemd, запустить её и убедиться, что она успешно работает в фоне, прослушивает порт и видит нужный Output (Telegram).

**Где:** VM `docker`.

**Команды:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable falcosidekick
sudo systemctl start falcosidekick
sudo systemctl status falcosidekick
```

<img src="https://github.com/user-attachments/assets/8feb97e5-7cfd-46d1-8caf-f32c23eceec6" alt="image" />

Служба успешно запущена (`active (running)`) и добавлена в автозагрузку (`enabled`). В логах видно, что Falcosidekick версии 2.31.0 успешно инициализировал вывод в Telegram (`Enabled Outputs: [Telegram]`) и начал прослушивать порт `2801` для приема событий от Falco.

### Этап 68: Включение JSON-вывода в Falco

**Зачем:** Настроить Falco на формирование логов в формате JSON, чтобы Falcosidekick мог корректно их парсить и передавать в Telegram в читаемом виде.

**Где:** VM `docker`.

**Команды:**

```bash
sudo nano /etc/falco/falco.yaml
```
*(В файле включены параметры `json_output` и `json_include_output_property`)*

```yaml
json_output: true
json_include_output_property: true
```

<img src="https://github.com/user-attachments/assets/ae0f48d3-14d3-40ad-a047-3d329bcc4262" alt="image" />

В конфигурационном файле `falco.yaml` активирован вывод в формате JSON (`json_output: true`). Также включен параметр `json_include_output_property`, который добавляет в JSON-объект текстовое поле `output` с описанием события, чтобы уведомление в Telegram было информативным.

### Этап 69: Включение HTTP-вывода в Falco

**Зачем:** Настроить Falco на отправку событий по HTTP на локальный порт, где их будет принимать Falcosidekick для дальнейшей пересылки в Telegram.

**Где:** VM `docker`.

**Команды:**

```bash
sudo nano /etc/falco/falco.yaml
```
*(В файле активирован блок `http_output`)*

```yaml
http_output:
  enabled: true
  url: "http://localhost:2801/"
```

<img src="https://github.com/user-attachments/assets/ef148ee2-b144-4730-a406-3239410e0fea" alt="image" />

В конфигурации Falco включен `http_output`. Теперь при генерации алерта Falco будет отправлять JSON-запрос на `localhost:2801`, где его ожидает запущенный сервис Falcosidekick.<img 

### Этап 70: Генерация тестового события для Telegram

**Зачем:** Перезапустить Falco с обновленным конфигом и сгенерировать событие, чтобы проверить сквозную цепочку отправки алерта в мессенджер.

**Где:** VM `docker`.

**Команды:**

```bash
sudo systemctl restart falco
docker run --rm --privileged alpine true
```

<img src="https://github.com/user-attachments/assets/f9fbe882-7373-47d2-b63b-9d826ef78ffb" alt="image" />

Falco перезапущен, применены настройки `http_output`. Выполнен запуск привилегированного контейнера. Сгенерированный алерт должен быть отправлен в Falcosidekick и далее перенаправлен в Telegram-бот.


## Безопасность цепочки поставок (Cosign)

### Этап 1: Загрузка утилиты Cosign

**Зачем:** Скачать официальный бинарный файл Cosign (v3.1.3) для подписи и проверки контейнерных образов.

**Где:** VM `docker`.

**Команды:**

```bash
curl -L -o cosign https://github.com/sigstore/cosign/releases/download/v3.1.3/cosign-linux-amd64
```

<img src="https://github.com/user-attachments/assets/fcbf6567-35a8-4ed2-acfe-d7e82f828978" alt="image" />

Выбран и скачан бинарник `cosign-linux-amd64` с официального репозитория Sigstore. Версия `amd64` соответствует архитектуре процессора виртуальной машины (x86_64). Размер файла — 134 МБ.


### Этап 2: Установка и проверка Cosign

**Зачем:** Сделать бинарный файл исполняемым, переместить его в системную директорию (чтобы команда была доступна глобально) и убедиться в работоспособности утилиты.

**Где:** VM `docker`.

**Команды:**

```bash
chmod +x cosign
sudo mv cosign /usr/local/bin/
cosign version
```
*(Примечание: команда `cosign --version` выдает ошибку, так как Cosign использует синтаксис без дефисов).*

<img src="https://github.com/user-attachments/assets/5d85b3f1-b6bd-4fff-bece-ba2faba8a47e" alt="image" />

Утилита Cosign успешно установлена. Вывод команды `cosign version` подтверждает, что версия (v3.1.3) установлена корректно для платформы `linux/amd64`.

### Этап 3: Генерация ключевой пары

**Зачем:** Создать пару криптографических ключей: приватный (`cosign.key`) для подписи образов и публичный (`cosign.pub`) для последующей проверки подписей.

**Где:** VM `docker`.

**Команды:**

```bash
cosign generate-key-pair
```
*(При выполнении команды потребуется дважды ввести пароль для защиты приватного ключа)*

<img src="https://github.com/user-attachments/assets/65008a7a-c970-4a42-94e8-3030beabc49c" alt="image" />

Утилита успешно сгенерировала ключи. В текущей директории созданы два файла: `cosign.key` (приватный ключ, защищенный паролем) и `cosign.pub` (публичный ключ). Публичный ключ будет использоваться для проверки подлинности образов.


### Этап 4: Создание Dockerfile для тестового образа

**Зачем:** Подготовить минимальный Dockerfile для сборки собственного образа, который в дальнейшем будет подписан и загружен в локальный Registry.

**Где:** VM `docker` (в директории `/home/vagrant/my-app`).

**Команды:**

```bash
mkdir my-app
cd my-app
nano Dockerfile
cat Dockerfile
```

*(Содержимое файла `Dockerfile`)*
```dockerfile
FROM alpine:latest
LABEL description="Test image for cosign signing"
CMD ["echo", "Hello from signed container!"]
```

<img src="https://github.com/user-attachments/assets/7bb87892-5422-4284-9774-1cf2a29964f5" alt="image" />

Создана рабочая директория `my-app`. В файл `Dockerfile` добавлена инструкция сборки на базе легковесного образа `alpine:latest`. Команда `cat` подтверждает корректность синтаксиса.


### Этап 5: Сборка и отправка тестового образа в Registry

**Зачем:** Собрать Docker-образ на основе подготовленного Dockerfile и загрузить его в локальный реестр (Registry) для последующей подписи.

**Где:** VM `docker` (в директории `/home/vagrant/my-app`).

**Команды:**

```bash
docker build -t docker-registry:5000/my-signed-app:v1 .
docker push docker-registry:5000/my-signed-app:v1
```

<img src="https://github.com/user-attachments/assets/72372312-8fa7-4815-b7b3-91dff3f94ed1" alt="image" />

Образ успешно собран (3 шага) и загружен в локальный Registry по защищенному протоколу HTTPS. В выводе отображается уникальный дайджест образа (`sha256:c2545a...`), который будет использоваться для криптографической привязки подписи.

### Этап 6: Подпись образа по дайджесту

**Зачем:** Обеспечить строгую неизменяемость подписи. Подпись по дайджесту (`sha256:...`) гарантирует, что подписан точный математический отпечаток содержимого образа, исключая подмену через перезапись тега.

**Где:** VM `docker`.

**Команды:**

```bash
cosign sign --key cosign.key --registry-cacert /etc/docker/certs.d/docker-registry:5000/ca.crt docker-registry:5000/my-signed-app@sha256:c2545a1e4506e0ef3dd1c31151a44044a610862eed38d1683cef83c555b94672
```

<img src="https://github.com/user-attachments/assets/27d96cf9-4867-4d55-8160-7676d6a9a016" alt="image" />

Образ успешно подписан по его уникальному дайджесту. Подпись (signature) сформирована и загружена в Registry.


### Этап 7: Проверка подписи по дайджесту

**Зачем:** Убедиться, что подпись привязана к неизменяемому математическому отпечатку образа (дайджесту). Это исключает риск подмены тега (`v1`) и гарантирует целостность артефакта.

**Где:** VM `docker`.

**Команды:**

```bash
cosign verify --key cosign.pub --registry-cacert /etc/docker/certs.d/docker-registry:5000/ca.crt docker-registry:5000/my-signed-app@sha256:c2545a1e4506e0ef3dd1c31151a44044a610862eed38d1683cef83c555b94672
```

<img src="https://github.com/user-attachments/assets/4c3baea1-1069-44eb-94cd-fa9efdc33bbe" alt="image" />

Проверка подписи по дайджесту прошла успешно. Вывод подтверждает, что подпись расшифрована публичным ключом (`The signatures were verified against the specified public key`), а дайджест подписанного образа (`docker-manifest-digest`) в точности совпадает с тем, который мы указали в команде. Целостность образа подтверждена.


### Этап 8: Демонстрация уязвимости (запуск неподписанного контейнера)

**Зачем:** На практике доказать, что стандартный Docker не проверяет цифровые подписи образов и позволяет запускать любой контейнер, даже если он не был подписан.

**Где:** VM `docker`.

**Команды:**

```bash
docker run --name unsigned-test alpine echo "Я неподписанный контейнер, но я работаю!"
docker ps -a
```

<img src="https://github.com/user-attachments/assets/3e9e0f07-0677-481d-a9d2-1227c6464571" alt="image" />

Контейнер на базе неподписанного образа `alpine` успешно запустился, выполнил команду и завершился с кодом `0`. Вывод `docker ps -a` подтверждает, что Docker не проверяет наличие подписи перед запуском. Это обосновывает необходимость внедрения механизма принудительной проверки (Enforcement).

### Этап 9: Создание скрипта-обертки для проверки подписи (Enforcement)

**Зачем:** Реализовать механизм принудительной проверки. Скрипт будет перехватывать команду запуска контейнера, проверять цифровую подпись через Cosign и блокировать запуск, если образ не подписан.

**Где:** VM `docker` (в директории `/home/vagrant`).

**Команды:**

```bash
nano safe-run.sh
chmod +x safe-run.sh
```
<img src="https://github.com/user-attachments/assets/db748f5b-9f07-4974-97cf-284e1000ca10" alt="image" />
<img width="797" height="126" alt="image" src="https://github.com/user-attachments/assets/bb142b03-18e9-4cf9-94fb-6bad08e64957" />

Создан скрипт `safe-run.sh`. Команда `chmod +x` делает файл исполняемым, позволяя запускать его как программу. Скрипт анализирует переданные аргументы, вызывает `cosign verify` для проверки подписи и принимает решение: при успешной проверке вызывает `docker run`, при ошибке — выводит сообщение о блокировке и завершает работу с кодом `1`.


### Этап 10: Тестирование блокировки неподписанного контейнера

**Зачем:** Проверить, что созданный скрипт-обертка корректно перехватывает запуск и блокирует контейнеры, не имеющие действительной цифровой подписи в Registry.

**Где:** VM `docker`.

**Команды:**

```bash
./safe-run.sh alpine echo "Тест блокировки"
```

<img src="https://github.com/user-attachments/assets/c34b3381-716d-4148-83fa-2cff8789ed99" alt="image" />

Скрипт успешно выполнил проверку через Cosign. Так как образ `alpine` не подписан, проверка вернула ошибку. Запуск контейнера был заблокирован (выведено сообщение "ОШИБКА БЕЗОПАСНОСТИ"), что доказывает работоспособность механизма Enforcement.

### Этап 11: Глобальный перехват команды docker run

**Зачем:** Обеспечить автоматическую проверку подписей для всех запускаемых контейнеров без необходимости использовать скрипт-обертку `./safe-run.sh`.

**Где:** VM `docker`.

**Команды:**

```bash
sudo nano /usr/local/bin/docker
sudo chmod +x /usr/local/bin/docker
```

<img width="2552" height="1299" alt="image" src="https://github.com/user-attachments/assets/ab3535ce-120a-459f-9414-7180caf46684" />
<img width="1042" height="121" alt="image" src="https://github.com/user-attachments/assets/5a6a7fcd-abe9-4a84-b9d9-bb6596d9e27f" />


Создан скрипт-перехватчик в директории /usr/local/bin/ (которая имеет приоритет над /usr/bin/). Команда chmod +x сделала файл исполняемым. Теперь при вызове команды docker run система сначала запускает скрипт: он анализирует аргументы, находит имя образа, проверяет подпись через Cosign и вызывает оригинальный Docker (/usr/bin/docker) только в случае успешной проверки



### Этап 12: Тестирование блокировки неподписанного контейнера через глобальный перехват

**Зачем:** Убедиться, что механизм глобального перехвата (Enforcement) корректно блокирует запуск контейнеров, образы которых не имеют действительной цифровой подписи.

**Где:** VM `docker`.

**Команды:**

```bash
hash -r
docker run --rm alpine echo "Я неподписанный контейнер!"
```

<img src="https://github.com/user-attachments/assets/37ce3e9e-dcf9-4bc6-9805-99ace227832e" alt="image" />

Команда `hash -r` сбросила кэш путей оболочки Bash, после чего система начала использовать наш скрипт из `/usr/local/bin/docker`. При попытке запуска неподписанного образа `alpine` сработал механизм проверки: была вызвана ошибка безопасности и запуск контейнера заблокирован.

### Этап 13: Тестирование запуска подписанного контейнера

**Зачем:** Убедиться, что созданный механизм перехвата не блокирует легитимные контейнеры, имеющие валидную цифровую подпись.

**Где:** VM `docker`.

**Команды:**

```bash
docker run --rm docker-registry:5000/my-signed-app:v1
```

<img src="https://github.com/user-attachments/assets/82776382-f8e2-4f00-acb0-6b2c50f8925b" alt="image" />

При запуске подписанного образа скрипт-перехватчик успешно проверил цифровую подпись через Cosign. Вывод подтверждает, что проверка пройдена (`Подпись валидна`), после чего управление было передано оригинальному Docker. Контейнер успешно запустился и вывел приветствие (`Hello from signed container!`).

### Этап 14: Добавление правил Falco для контроля недоверенных и неподписанных контейнеров

**Зачем:** Расширить покрытие Falco. Теперь он должен фиксировать попытки запуска контейнеров из недоверенных источников, а также события блокировки неподписанных образов, инициированные скриптом-перехватчиком.

**Где:** VM `docker`.

**Команды:**

```bash
sudo nano /etc/falco/falco_rules.local.yaml
```

<img src="https://github.com/user-attachments/assets/36c212d5-059b-494c-8f37-8204cca205d5" alt="image" />

В конфигурацию Falco добавлены два новых правила. Правило «Запуск контейнера из недоверенного источника» отслеживает вызовы `docker run` без указания адреса нашего Registry. Правило «Заблокирован запуск неподписанного контейнера» перехватывает запуск утилиты `logger` с тегом `falco-enforcement`, что позволяет фиксировать события блокировки, инициированные скриптом-оберткой.

### Этап 15: Проверка взаимодействия Falco и скрипта-перехватчика

**Зачем:** Убедиться, что при блокировке запуска недоверенного контейнера скриптом-оберткой, событие корректно фиксируется в логах Falco.

**Где:** VM `docker`.

**Команды:**

```bash
sudo systemctl restart falco
docker run --rm alpine echo "test"
sudo tail -n 1 /var/log/falco/events.log
```

<img width="2514" height="545" alt="image" src="https://github.com/user-attachments/assets/7a0169c9-8a01-456a-9914-e57bc1fd516a" />

Скрипт-перехватчик успешно заблокировал запуск неподписанного образа `alpine`. В логах Falco появилось событие с правилом «Запуск контейнера из недоверенного источника». Лог выведен в формате JSON (одна строка), что является стандартом для машинной обработки и интеграции с SIEM-системами.


### Этап 16: Автоматическое логирование блокировок через Falco

**Зачем:** Обеспечить автоматическую запись события в лог Falco при каждой блокировке неподписанного контейнера из Registry. Это позволяет связать Prevention (блокировку) и Detection (логирование).

**Где:** VM `docker`.

**Команды:**

```bash
sudo nano /usr/local/bin/docker
docker run --rm docker-registry:5000/my-signed-app@sha256:c2545a1e4506e0ef3dd1c31151a44044a610862eed38d1683cef83c555b946725
sudo tail -n 1 /var/log/falco/events.log
```

<img src="https://github.com/user-attachments/assets/40d80eb7-a936-4b01-9544-13610eab67a1" alt="image" />

В скрипт-перехватчике есть вызов `logger`, который генерирует системное событие при блокировке. При попытке запустить образ с неверным хэшем (`...946725`) скрипт блокирует запуск, а Falco автоматически фиксирует это событие с приоритетом `CRITICAL` и правилом «Заблокирован запуск неподписанного контейнера из Registry».


## Управление политиками безопасности (OPA)


### Этап 1: Установка Open Policy Agent (OPA)

**Зачем:** Скачать бинарник OPA для последующего развертывания сервиса проверки политик безопасности (Policy as Code) и интеграции с системой контроля доступа.

**Где:** VM `docker`.

**Команды:**

```bash
curl -L -o opa https://openpolicyagent.org/downloads/latest/opa_linux_amd64_static
chmod +x opa
sudo mv opa /usr/local/bin/
opa version
```

<img src="https://github.com/user-attachments/assets/1627807f-de8f-4690-aca7-50241e67799e" alt="image" />

Утилита OPA (версия 1.21.3) успешно скачана с официального сайта, получила права на исполнение и перемещена в системную директорию `/usr/local/bin/`. Команда `opa version` подтверждает, что бинарник работает корректно в среде `linux/amd64`.







