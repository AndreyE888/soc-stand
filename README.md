# SOC Stand

Стенд из 3 виртуальных машин для развёртывания SOC-инструментов.

## Архитектура

| VM | IP | Роль |
|---|---|---|
| docker | 192.168.56.10 | запуск контейнеров |
| docker_registry | 192.168.56.11 | хранение образов |
| storage | 192.168.56.12 | хранение логов |

## Стек

- **Vagrant** — управление VM
- **VirtualBox** — провайдер
- **Docker** — контейнеры
- **Docker Registry** — хранение образов
- **NFS** — общая папка
- **SSL/TLS** — шифрование

## Быстрый старт

```bash
git clone https://github.com/AndreyE888/soc-stand.git
cd soc-stand
vagrant up
