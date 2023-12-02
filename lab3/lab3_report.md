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


Также был установлен Nginx, который позволяет создать веб-сервер (обратный прокси, почтовый сервер), Gunicorn - для запуска Web приложений на Python (аналог Tomcat JE)
>sudo su
> apt-get install -y nginx  
> cp /opt/netbox/netbox-3.37v/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox  
> cd /etc/nginx/sites-enabled/  
> rm default  
> ln -s /etc/nginx/sites-available/netbox  
> nginx -t  
> nginx -s reload  
> cp contrib/gunicorn.py /opt/netbox/gunicorn.py  
> cp contrib/*.service /etc/systemd/system/  
> systemctl daemon-reload  
> systemctl start netbox netbox-rq  
> systemctl enable netbox netbox-rq

Переходим в браузер, вводим логин и пароль от учетки суперпользователя:

![Учетная запись](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20204601.png)  

#### Работа с NetBox
Была добавлена базовая информация об двух роутеров на Ether2 интерфейсе:

![Базовая информация](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/Снимок%20экрана%202023-12-02%20204900.png)  

Далее скачаем netbox.csv файл с описанием роутеров

#### Написание сценария для настройки двух роутеров
Предполагается, что NetBox будет считывать csv файл и распределять их в два разных файла yml, при этом скачанный ранее csv перенесем на сервер с ansible, а playbook изменябщий имена устройств и их IP адресов представлен далее:  

Полный inventory файл для подключения к роутерам представлен ниже: [Конфиг](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab3/resources/inventory.yml 

Playbook:  
```yaml  
- name: Setting Routers  
  hosts: ungrouped  
  tasks:  
    - name: Setting name  
      community.routeros.command:  
        commands:  
          - /system identity set name="{{interfaces[0].device.name}}"  
    - name: Seting name  
      community.routeros.command:  
        commands:  
        - /ip address add address="{{interfaces[0].ip_addresses[1].address}}" interface="{{interfaces[0].display}}"  
```  
#### Выгрузка данных из роутера на Netbox
Выполним обратную задачу: данные из chr передаются в файл на виртуальную машину, а с помощью playbook выгружается в Netbox.

```yaml  
- name: Getting number
  hosts: ungrouped
  tasks:
    - name: Getting number
      community.routeros.command:
        commands:
          - /system license print
      register: license_print
    - name: Getting name
      community.routeros.command:
        commands:
          - /system identity print
      register: identity_print
    - name: Add number to Netbox
      netbox_device:
        netbox_url: http://127.0.0.1:8000
        netbox_token: 7a60121059b705c5bd9323fb1f1c40e9dd90124d32fd12
        data:
          name: "{{identity_print.stdout_lines[0][0].split(' ').1}}"
          serial: "{{license_print.stdout_lines[0][0].split(' ').1}}"
```
Схема связи:

![Схема](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab2/resources/Схема_2.jpg)  


## Вывод
В результате выполнения данной лабораторной работы были изучены основы работы с NetBox, взаимодействие со стороны роутеров, а также взаимодействие со стороны самого Netbox. Связка Ansible-NetBox является рациональным способом моделирования и документирования современных сетей (Voice and UC Source of Truth)
