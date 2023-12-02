University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network-programming](https://itmo-ict-faculty.github.io/network-programming/)   
Year: 2023/2024  
Group: K34212  
Author: Dolmatov Dmitrii Alexeevich  
Lab: Lab3  
Date of create: 02.12.2023  
Date of finished: --.12.2023  

# Лабораторная работ 3 "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"  
## Описание  
В данной лабораторной работе вы ознакомитесь с интеграцией Ansible и Netbox и изучите методы сбора информации с помощью данной интеграции.  
## Цель работы  
С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.  
### Ход работы  
#### Установка Netbox и окружения
Сначала настроим окружение: PostgreSQL (облегченнуб libpq-dev дистрибутив), создадим БД, к которой добавим суперпользователя (имеющего все привилегии)

Далее, скачаем Redis. Он нам нужен как вариант хранилища значений ключей. NetBox подкачивает из него файлы для кеширования для использования в очереди.

Создание суперпользователя по отношению к БД для NetBox:

![Создание пользователя](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20193427.png)

Подключение к netbox:
![Подключение к NetBox](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20194211.png)

Обращение к Redis:

![Подключение к Redis](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20194806.png)  


#### Настройка NetBox
Для начала создадим секретный ключ и добавим его в итоговый файл конфигурации, содержащим ALLOWED_HOST, определенный под localhost.  

Генерация секретного ключа:

![Генерация секретного ключа](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20202111.png)

Итоговый файл конфигурации:

![Итоговый файл конфигурации](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20202617.png)

Далее были выполнены миграции, внутри виртуального окружения создан супербзер и собрана статика

Выполнение миграций:

![Миграции](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20202958.png)

Создание суперюзера:

![Создание суперюзера](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20203905.png)


Процесс сбора статики:

![Сбор статики](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20203950.png)

## Вывод
В результате выполнения данной лабораторной работы были изучены основы работы с Ansible, который упрощает конфигурацию большого количества устройств одновременно через создания файла-инвентаря hosts и playbook, в котором находится основная конфигурационная последовательность команд
