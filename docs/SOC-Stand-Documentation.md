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

Docker настроен доверять Registry по HTTP. Без этой настройки `docker push` завершится ошибкой `http: server gave HTTP response to HTTPS client`, поскольку по умолчанию Docker требует HTTPS. После изменения файла нужно перезапустить Docker: `sudo systemctl restart docker`. **Настройку нужно выполнить на всех VM, которые будут работать с Registry** — то есть на `docker` (и позже на других клиентах).

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



<img width="1095" height="134" alt="image" src="https://github.com/user-attachments/assets/4af1bffb-299b-407a-aacc-76c94b9b2e7f" />


<img width="2515" height="824" alt="image" src="https://github.com/user-attachments/assets/ca26c8a0-9cdd-4961-80da-271d270d0fad" />

<img width="1133" height="294" alt="image" src="https://github.com/user-attachments/assets/b4aeeecf-4b94-4f79-a395-7a547f0e8e1f" />

<img width="2540" height="526" alt="image" src="https://github.com/user-attachments/assets/4ec3e3c2-317d-443d-bfd7-d42d618d6265" />

<img width="1454" height="381" alt="image" src="https://github.com/user-attachments/assets/8ad8b949-3f79-40ed-832e-865898943784" />



<img width="2445" height="380" alt="image" src="https://github.com/user-attachments/assets/25caff9d-c2be-470e-b552-eda778dcfb05" />


<img width="1739" height="96" alt="image" src="https://github.com/user-attachments/assets/2ba29222-d70b-495d-87f6-cf581629be1a" />


<img width="941" height="93" alt="image" src="https://github.com/user-attachments/assets/68d7d170-d104-461b-bdea-cf361a1c38f6" />


<img width="1003" height="164" alt="image" src="https://github.com/user-attachments/assets/fd428812-4100-485e-89bf-f649113ee57b" />

<img width="1462" height="309" alt="image" src="https://github.com/user-attachments/assets/1899b8b8-88b5-47bd-8086-564fb682aa58" />





