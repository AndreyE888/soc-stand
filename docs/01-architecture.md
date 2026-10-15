# Архитектура стенда

## Роли VM

| VM | IP | Роль |
|---|---|---|
| docker | 192.168.56.10 | запуск контейнеров |
| docker_registry | 192.168.56.11 | хранение образов |
| storage | 192.168.56.12 | хранение логов |

## Стек технологий

| Технология | Назначение |
|---|---|
| Vagrant | управление VM |
| VirtualBox | провайдер |
| Docker | контейнеры |
| Docker Registry | реестр образов |
| NFS | общая папка |
| SSL/TLS | шифрование |

## Сеть

Все VM в одной подсети `192.168.56.0/24`.

- Host-Only Adapter — `192.168.56.1`
- SSH — через проброс портов
- Взаимодействие — через `/etc/hosts`

<img width="1097" height="119" alt="image" src="https://github.com/user-attachments/assets/d3dcd8fb-8187-47be-b1e1-7ba17ff21e85" />

<img width="1348" height="171" alt="image" src="https://github.com/user-attachments/assets/8c9b6f8d-293a-47c4-97f8-b91a7f3e2c74" />

<img width="1336" height="413" alt="image" src="https://github.com/user-attachments/assets/778cb626-6528-4b45-aacc-13b750eb6591" />

<img width="1503" height="798" alt="image" src="https://github.com/user-attachments/assets/2df1a6f1-016d-46ad-968d-d010042a61df" />


<img width="1124" height="567" alt="image" src="https://github.com/user-attachments/assets/8decc3bb-d226-4976-a823-bef7bf5e8cb0" />

<img width="1518" height="389" alt="image" src="https://github.com/user-attachments/assets/e3dc8dfb-70e7-4462-b756-9836b7206a6a" />

<img width="1053" height="135" alt="image" src="https://github.com/user-attachments/assets/2705640b-37c8-49ab-9fe3-2174f2f67115" />

<img width="1034" height="117" alt="image" src="https://github.com/user-attachments/assets/74d4e65d-5229-455d-b112-1381e91f2b83" />

<img width="977" height="109" alt="image" src="https://github.com/user-attachments/assets/c583a7a3-dee5-450f-89e8-f18680276f4a" />


<img width="2539" height="898" alt="image" src="https://github.com/user-attachments/assets/77d550a3-f688-456c-b877-6bec68546596" />

<img width="1356" height="125" alt="image" src="https://github.com/user-attachments/assets/8ddab2b8-cae4-4327-96a3-03fb0bc719f2" />


<img width="2538" height="176" alt="image" src="https://github.com/user-attachments/assets/0c98b805-14c0-4973-8333-4c2605f3f315" />


<img width="2497" height="243" alt="image" src="https://github.com/user-attachments/assets/d5211db5-650e-4b8c-aa02-cf8fd8e360c6" />

<img width="1305" height="124" alt="image" src="https://github.com/user-attachments/assets/2180d35c-fc68-429f-92c6-da67a37755f0" />


<img width="1015" height="230" alt="image" src="https://github.com/user-attachments/assets/da452819-9414-4578-948a-d8c1ff289378" />


<img width="1534" height="259" alt="image" src="https://github.com/user-attachments/assets/cf44700a-3211-4875-a59f-adb30c0ad613" />


<img width="1227" height="85" alt="image" src="https://github.com/user-attachments/assets/1038040f-cdd2-459e-a43c-8b9bf02b146e" />


<img width="2512" height="381" alt="image" src="https://github.com/user-attachments/assets/245d1ac7-20fa-4a39-b4f7-32742c014056" />


<img width="1226" height="115" alt="image" src="https://github.com/user-attachments/assets/eb055e82-9667-428e-92fa-36fcf11f90d3" />



<img width="1529" height="263" alt="image" src="https://github.com/user-attachments/assets/66dbf972-abf5-4e98-9cb6-43d5ad92fdaf" />



<img width="2501" height="229" alt="image" src="https://github.com/user-attachments/assets/d710ddf9-ea5c-4892-8ade-f1ad235c172f" />



