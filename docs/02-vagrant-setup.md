
```markdown
# SOC Stand

Стенд из 3 виртуальных машин для развёртывания SOC-инструментов.

## Содержание

- [Цель](#цель)
- [Архитектура](#архитектура)
- [Этапы работы](#этапы-работы)
- [Проблемы и решения](#проблемы-и-решения)
- [Что осталось](#что-осталось)
- [Ссылки](#ссылки)

## Цель

Построить стенд из **3 виртуальных машин** для развёртывания и тестирования **SOC-инструментов** (Falco, OPA, cosign).

## Архитектура

| VM | IP | Роль | Ресурсы |
|---|---|---|---|
| **docker** | 192.168.56.10 | запуск контейнеров | 2 CPU, 2 GB |
| **docker_registry** | 192.168.56.11 | хранение образов | 2 CPU, 2 GB |
| **storage** | 192.168.56.12 | хранение логов | 2 CPU, 2 GB |

## Этапы работы

### Этап 1: Проверка Vagrant

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant --version
```

**Пояснение:** команда выводит установленную версию Vagrant. Если команда не найдена — Vagrant не установлен в PATH.

**Результат:**

```
Vagrant 2.4.9
```

**Скриншот:**

<img width="1097" height="119" alt="image" src="https://github.com/user-attachments/assets/d3dcd8fb-8187-47be-b1e1-7ba17ff21e85" />

### Этап 2: Проверка Vagrantfile

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant init bento/ubuntu-24.04
```

**Пояснение:** команда создаёт базовый `Vagrantfile` в текущей директории. Если файл уже существует — Vagrant откажется перезаписывать и выведет предупреждение.

**Результат:**

```
`Vagrantfile` already exists in this directory. Remove it before running `vagrant init`.
```

**Скриншот:**

<img width="1348" height="171" alt="image" src="https://github.com/user-attachments/assets/8c9b6f8d-293a-47c4-97f8-b91a7f3e2c74" />

### Этап 3: Статус VM

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant status
```

**Пояснение:** команда показывает текущее состояние всех машин, описанных в `Vagrantfile`. Значение `running` означает, что VM запущена и доступна по SSH.

**Результат:**

```
Current machine states:

docker                    running (virtualbox)
docker_registry           running (virtualbox)
storage                   running (virtualbox)
```

**Скриншот:**

<img width="1336" height="413" alt="image" src="https://github.com/user-attachments/assets/778cb626-6528-4b45-aacc-13b750eb6591" />

### Этап 4: Вход в VM docker

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant ssh docker
```

**Пояснение:** команда устанавливает SSH-соединение с VM `docker` по ключу, сгенерированному Vagrant при первом запуске. Пароль не запрашивается.

**Результат:**

```
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-86-generic x86_64)
...
vagrant@docker:~$
```

**Скриншот:**

<img width="1503" height="798" alt="image" src="https://github.com/user-attachments/assets/2df1a6f1-016d-46ad-968d-d010042a61df" />

### Этап 5: Проверка /etc/hosts

**Где:** VM `docker`.

**Команда:**

```bash
cat /etc/hosts
```

**Пояснение:** команда выводит содержимое файла `/etc/hosts` — локального справочника «имя → IP». Записи о трёх VM позволяют обращаться к ним по имени, а не по IP-адресу.

**Результат:**

```
127.0.0.1       localhost
127.0.1.1       vagrant
...
192.168.56.10   docker
192.168.56.11   docker-registry
192.168.56.12   storage
```

**Скриншот:**

<img width="1124" height="567" alt="image" src="https://github.com/user-attachments/assets/8decc3bb-d226-4976-a823-bef7bf5e8cb0" />

### Этап 6: Ping между VM

**Где:** VM `docker`.

**Команда:**

```bash
ping docker-registry
```

**Пояснение:** команда отправляет ICMP-пакеты на VM `docker-registry` по имени. Успешный ответ подтверждает, что сеть между VM работает и `/etc/hosts` настроен корректно.

**Результат:**

```
PING docker-registry (192.168.56.11) 56(84) bytes of data.
64 bytes from 192.168.56.11: icmp_seq=1 ttl=64 time=3.05 ms
64 bytes from 192.168.56.11: icmp_seq=2 ttl=64 time=0.495 ms
64 bytes from 192.168.56.11: icmp_seq=3 ttl=64 time=0.535 ms
--- docker-registry ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
```

**Скриншот:**

<img width="1518" height="389" alt="image" src="https://github.com/user-attachments/assets/e3dc8dfb-70e7-4462-b756-9836b7206a6a" />

### Этап 7: Список боксов

**Где:** хост (PowerShell).

**Команда:**

```powershell
vagrant box list
```

**Пояснение:** команда выводит список образов (box), скачанных в локальный кэш Vagrant. Эти образы используются как основа для создания VM.

**Результат:**

```
bento/ubuntu-24.04 (virtualbox, 0, (amd64))
```

**Скриншот:**

<img width="1053" height="135" alt="image" src="https://github.com/user-attachments/assets/2705640b-37c8-49ab-9fe3-2174f2f67115" />

### Этап 8: Версия Docker

**Где:** VM `docker`.

**Команда:**

```bash
docker --version
```

**Пояснение:** команда выводит версию установленного Docker. Используется для проверки, что установка прошла успешно.

**Результат:**

```
Docker version 29.1.3, build 29.1.3-0ubuntu3~24.04.2
```

**Скриншот:**

<img width="1034" height="117" alt="image" src="https://github.com/user-attachments/assets/74d4e65d-5229-455d-b112-1381e91f2b83" />

### Этап 9: Добавление в группу docker

**Где:** VM `docker`.

**Команда:**

```bash
sudo usermod -aG docker vagrant
```

**Пояснение:** команда добавляет пользователя `vagrant` в группу `docker`. Это даёт право работать с Docker без `sudo`. Флаг `-aG` добавляет в группу, не удаляя из остальных.

**Скриншот:**

<img width="977" height="109" alt="image" src="https://github.com/user-attachments/assets/c583a7a3-dee5-450f-89e8-f18680276f4a" />

### Этап 10: Статус Docker

**Где:** VM `docker`.

**Команда:**

```bash
sudo systemctl status docker
```

**Пояснение:** команда показывает состояние службы Docker в systemd. Значение `active (running)` означает, что демон работает, а `enabled` — что он запустится автоматически при загрузке VM.

**Результат:**

```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled)
     Active: active (running)
```

**Скриншот:**

<img width="2539" height="898" alt="image" src="https://github.com/user-attachments/assets/77d550a3-f688-456c-b877-6bec68546596" />

### Этап 11: Проверка docker ps

**Где:** VM `docker`.

**Команда:**

```bash
docker ps
```

**Пояснение:** команда показывает список запущенных контейнеров. Пустой список означает, что контейнеров нет — это нормально для чистой установки.

**Результат:**

```
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```

**Скриншот:**

<img width="1356" height="125" alt="image" src="https://github.com/user-attachments/assets/8ddab2b8-cae4-4327-96a3-03fb0bc719f2" />

### Этап 12: Запуск Registry

**Где:** VM `docker_registry`.

**Команда:**

```bash
docker run -d --restart=always --name registry -p 5000:5000 -e REGISTRY_STORAGE_DELETE_ENABLED=true registry:2
```

**Пояснение:** команда запускает контейнер `registry:2` в фоне (`-d`). Флаг `--restart=always` обеспечивает автозапуск при перезагрузке. `-p 5000:5000` пробрасывает порт. `-e REGISTRY_STORAGE_DELETE_ENABLED=true` разрешает удаление образов через API.

**Результат:**

```
ce3cadc2bfab1edf30d216bc265ae61d695665a00aba7fbc473e35743ad0bd9b
```

**Скриншот:**

<img width="2538" height="176" alt="image" src="https://github.com/user-attachments/assets/0c98b805-14c0-4973-8333-4c2605f3f315" />

### Этап 13: Проверка Registry

**Где:** VM `docker_registry`.

**Команда:**

```bash
docker ps
```

**Пояснение:** команда проверяет, что контейнер `registry` запущен. В выводе должно быть имя `registry` и проброс порта `0.0.0.0:5000->5000/tcp`.

**Результат:**

```
CONTAINER ID   IMAGE         ...   PORTS                    NAMES
ce3cadc2bfab   registry:2    ...   0.0.0.0:5000->5000/tcp   registry
```

**Скриншот:**

<img width="2497" height="243" alt="image" src="https://github.com/user-attachments/assets/d5211db5-650e-4b8c-aa02-cf8fd8e360c6" />

### Этап 14: Проверка API Registry

**Где:** VM `docker_registry`.

**Команда:**

```bash
curl http://localhost:5000/v2/_catalog
```

**Пояснение:** команда отправляет HTTP-запрос к API Registry. Эндпоинт `/v2/_catalog` возвращает список репозиториев. Пустой массив означает, что Registry работает, но образов пока нет.

**Результат:**

```
{"repositories":[]}
```

**Скриншот:**

<img width="1305" height="124" alt="image" src="https://github.com/user-attachments/assets/2180d35c-fc68-429f-92c6-da67a37755f0" />

### Этап 15: Настройка insecure-registries

**Где:** VM `docker`.

**Команда:**

```bash
cat /etc/docker/daemon.json
```

**Пояснение:** команда выводит содержимое конфигурационного файла Docker. Параметр `insecure-registries` разрешает работу с Registry по HTTP (без HTTPS). Без этой настройки `docker push` завершится ошибкой.

**Результат:**

```json
{
  "insecure-registries": ["docker-registry:5000"]
}
```

**Скриншот:**

<img width="1015" height="230" alt="image" src="https://github.com/user-attachments/assets/da452819-9414-4578-948a-d8c1ff289378" />

### Этап 16: Pull alpine

**Где:** VM `docker`.

**Команда:**

```bash
docker pull alpine
```

**Пояснение:** команда скачивает образ `alpine` из Docker Hub. Образ используется как тестовый для проверки работы Registry.

**Результат:**

```
Using default tag: latest
latest: Pulling from library/alpine
Digest: sha256:294b683cb724975bec92580e1e685676bd4b50bda910ddb8c51d4cabeaec77e6
Status: Image is up to date for alpine:latest
```

**Скриншот:**

<img width="1534" height="259" alt="image" src="https://github.com/user-attachments/assets/cf44700a-3211-4875-a59f-adb30c0ad613" />

### Этап 17: Tag alpine

**Где:** VM `docker`.

**Команда:**

```bash
docker tag alpine docker-registry:5000/alpine
```

**Пояснение:** команда создаёт второй тег для образа `alpine` с указанием адреса Registry. Docker использует имя тега, чтобы определить, куда отправлять образ при `push`.

**Скриншот:**

<img width="1227" height="85" alt="image" src="https://github.com/user-attachments/assets/1038040f-cdd2-459e-a43c-8b9bf02b146e" />

### Этап 18: Push alpine

**Где:** VM `docker`.

**Команда:**

```bash
docker push docker-registry:5000/alpine
```

**Пояснение:** команда отправляет образ `alpine` в Registry. Успешный вывод `Pushed` и `digest` подтверждает, что образ сохранён в Registry.

**Результат:**

```
The push refers to repository [docker-registry:5000/alpine]
e2de96513ba9: Pushed
latest: digest: sha256:d56c381f961d307a21b3ca004cf1e3910f106644aefb1f43e654c8a56c4fd395 size: 1022
```

**Скриншот:**

<img width="2512" height="381" alt="image" src="https://github.com/user-attachments/assets/245d1ac7-20fa-4a39-b4f7-32742c014056" />

### Этап 19: Проверка каталога

**Где:** VM `docker`.

**Команда:**

```bash
curl http://docker-registry:5000/v2/_catalog
```

**Пояснение:** команда проверяет список репозиториев в Registry. Появление `alpine` в списке подтверждает, что push прошёл успешно.

**Результат:**

```
{"repositories":["alpine"]}
```

**Скриншот:**

<img width="1226" height="115" alt="image" src="https://github.com/user-attachments/assets/eb055e82-9667-428e-92fa-36fcf11f90d3" />

### Этап 20: Pull из Registry

**Где:** VM `docker`.

**Команда:**

```bash
docker pull docker-registry:5000/alpine
```

**Пояснение:** команда скачивает образ `alpine` из локального Registry. Успешный вывод `Downloaded` подтверждает полный цикл `push → pull` и работоспособность Registry.

**Результат:**

```
Using default tag: latest
latest: Pulling from alpine
Digest: sha256:d56c381f961d307a21b3ca004cf1e3910f106644aefb1f43e654c8a56c4fd395
Status: Downloaded newer image for docker-registry:5000/alpine:latest
```

**Скриншот:**

<img width="1529" height="263" alt="image" src="https://github.com/user-attachments/assets/66dbf972-abf5-4e98-9cb6-43d5ad92fdaf" />

### Этап 21: Проверка образов

**Где:** VM `docker`.

**Команда:**

```bash
docker images
```

**Пояснение:** команда выводит список локальных образов. Оба тега (`alpine` и `docker-registry:5000/alpine`) ссылаются на один и тот же образ.

**Результат:**

```
IMAGE                           ID             DISK USAGE   CONTENT SIZE
alpine:latest                   294b683cb724   13MB         3.94MB
docker-registry:5000/alpine     d56c381f961d   12.9MB       3.85MB
```

**Скриншот:**

<img width="2501" height="229" alt="image" src="https://github.com/user-attachments/assets/d710ddf9-ea5c-4892-8ade-f1ad235c172f" />

## Проблемы и решения

| Проблема | Решение |
|---|---|
| VPN не работает | зеркало `elab.pro` |
| Драйвер Host-Only сломан | переустановка VirtualBox |
| VMware конфликтовала | отключил |
| SSH не работал | `systemctl enable ssh` |
| Push падал с HTTPS | `insecure-registries` |
| `vagrantfile` с маленькой буквы | переименовал в `Vagrantfile` |

## Что осталось

| № | Что | Приоритет |
|---|---|---|
| 1 | SSL/TLS для Registry | 🔴 высокий |
| 2 | NFS на storage | 🟡 средний |
| 3 | Falco | 🟡 средний |
| 4 | OPA | 🟡 средний |
| 5 | cosign | 🟡 средний |

## Прогресс

```
████████████████████████████████████░░░░░░░░░░░░░░░░░░░░
~65%                                              осталось 35%
```

## Ссылки

| Что | Где |
|---|---|
| **GitHub** | https://github.com/AndreyE888/soc-stand |
| **Папка проекта** | `C:\Users\Andrey\vagrant\soc-stand` |
| **Документация** | `docs/` |

## Выводы

**Сделано:**
- Стенд из 3 VM через Vagrant.
- Сеть между VM + `/etc/hosts`.
- Docker на `docker` и `docker_registry`.
- Registry + push/pull.
- Полный цикл `docker ↔ registry` работает.

**Изучено:**
- Vagrant — multi-machine.
- Docker — `run`, `ps`, `push`, `pull`, `tag`.
- Registry — API.

**Дальше:** SSL/TLS, NFS, SOC-инструменты.
```
