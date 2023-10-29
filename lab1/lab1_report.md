University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network-programming](https://itmo-ict-faculty.github.io/network-programming/)   
Year: 2023/2024  
Group: K34212  
Author: Dolmatov Dmitrii Alexeevich  
Lab: Lab1  
Date of create: 25.10.2023  
Date of finished: --.12.2023  

# Лабораторная работ №1 "Установка CHR и Ansible, настройка VPN"  
## Описание  
Данная работа предусматривает обучение развертыванию виртуальных машин (VM) и системы контроля конфигураций Ansible а также организации собственных VPN серверов
## Цель работы  
Целью данной работы является развертывание виртуальной машины на базе платформы Microsoft Azure с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox
### Ход работы  
#### Подготовка виртуальной машины
Для начала, была создана виртуальная машина на сайте Яндекс. Результат подключения по SSH представлен ниже ![Успешное подключение по SSH](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Заход%20на%20локальную%20виртуалку.png)  
Далее, были скачаны дополнительные плагины и зависимости для дальнейших лабораторных работ  
Например, python & ansible ![Успешная установка Python & Ansible](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Cкачивание%20python%20%26%20ansible.png)  
Настройка адаптера представлена ниже: в режиме Bridge. ![Настройка интерфейса](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Настройки%20адаптера.png)  
#### Работа с OpenVPN server
Сначала установим сервер OpenVPN на удаленную виртуальную машину  
Скачанный сервер представлен далее ![Скачанный сервер](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Скачивание%20openvpn%20сервера.png)  
Отключим tls ![Отключенный tls](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Отключение%20tls.png)  
Поставим режим только в режиме tcp, поскольку WinBox работает только с ним (chr) ![Выбор TCP](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Выбор%20TCP.png)  
#### Подключение к серверу
Подключимся по MAC-адресу к Winbox, чтобы в более удобном варианте настроить сертификаты и установить PPP соединение в режиме OVPN-клиента  

Зайдём на сервер OpenVPN по адресу белого ip удаленной ВМ. Данные логина и пароля получим автоматически при установке OpenVPN-as сервера на удаленной ВМ ![Вход в личный кабинет](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Вход%20в%20личный%20кабинет.png)  
Не забудь поменять ip адрес VPN сервера  
Создадим пользователя с автологином без пароля ![Создание клиента](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Создание%20пользователя%20с%20автологином.png)  

Создав клиента, получим сертификаты, которые вставим в Winbox.  
Подключение по PPP представлено далее: ![Подключение по PPP](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Задание%20PPP%20подключения.png)  

Как можно заметить, было выполнено успешное подключение к удаленному OpenVPN серверу ![Успешное подключение](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Успешный%20успех.png)  

P.S. Была установлена утилита для локального времени, поскольку не всегда сервер работает корректно с локальной машиной в разных часовых зонах ![Утилита для времени](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab1/resources/Утилита%20для%20локального%20времени.png)  
