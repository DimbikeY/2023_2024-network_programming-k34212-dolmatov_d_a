University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network-programming](https://itmo-ict-faculty.github.io/network-programming/)   
Year: 2023/2024  
Group: K34212  
Author: Dolmatov Dmitrii Alexeevich  
Lab: Lab2  
Date of create: 26.11.2023  
Date of finished: --.12.2023  

# Лабораторная работ №2 "Развертывание дополнительного CHR, первый сценарий Ansible"  
## Описание  
В данной лабораторной работе вы на практике ознакомитесь с системой управления конфигурацией Ansible, использующаяся для автоматизации настройки и развертывания программного обеспечения.
## Цель работы  
С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.
### Ход работы  
#### Установка второго CHR
Выполним аналогичные действия по подключению второго локального CHR к удаленному OpenVPN серверу. Скачаем файл пользователя, содержащий сертификат и приватный ключ для подключения
> Стоит добавить, что OpenVPN сервер позволяет бесплатно обеспечить только двух OVPN пользователей персональными данными для подключения
Результат успешного подключения представлен ниже

![Подключения](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab2/resources/2%20vpn%20clients.png)

Пинг с сервера на локальные CHR представлен ниже, что подтверждает установленную связность

![Связность](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab2/resources/ping%20from%20server.png)

#### Работа с Ansible
> Ansible позволяет писать общую настройку конфигурации для нескольких устройств, что ускоряет и облегчает установку одинаковых служб/плагинов (в нашем случае, NTP & OSPF протоколов
Для работы необходимо сделать две вещи: настроить в hosts инвентаризацию (список роутеров, ip, логинов и паролей). К счастью, они у нас одинаковые
Файл hosts представлен ниже:

![hosts](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab2/resources/hosts.png)

Файл с playbook.yml представлен ниже:

![playbook](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab2/resources/playbook.png)

> Стоит дополнить, что необходимо вручную зайти на каждый из устройств, чтобы в списках known_users ввелись данные подключения с сервера. Да, в каком-то моменте это костыль, ведь не получится автоматизировать конфигурацию роутеров при их большом количестве, но исходим из того, что имеем
>> Стоит также скачать pylibssh для возможности подключения по ssh со стороны ansible

После запуска playbook и задании ему файла-инвентаря происходит выполнения настройки конфига роутеров:

![playbook-summary](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab2/resources/summary-playbook.png)

После успешного выполнения скрипта мы можем проверить соседей по OSPF
>> Дополнение: нужно следить за номером интерфейса при настройке playbook и его соотношением с номером интерфейса на virtualbox

![neighbours](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab2/resources/users.png)

Итоговая схема изображена ниже:

![scheme](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab2/resources/Схема_2.jpg)

## Вывод
В результате выполнения данной лабораторной работы были изучены основы работы с Ansible, который упрощает конфигурацию большого количества устройств одновременно через создания файла-инвентаря hosts и playbook, в котором находится основная конфигурационная последовательность команд
