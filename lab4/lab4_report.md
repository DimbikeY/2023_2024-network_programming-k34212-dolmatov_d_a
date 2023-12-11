Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network-programming](https://itmo-ict-faculty.github.io/network-programming/)   
Year: 2023/2024  
Group: K34212  
Author: Dolmatov Dmitrii Alexeevich  
Lab: Lab4    
Date of create: 11.12.2023  
Date of finished: --.12.2023  

# Лабораторная работ 4 "Базовая 'коммутация' и туннелирование используя язык программирования P4"  
## Описание  
В данной лабораторной работе вы познакомитесь на практике с языком программирования P4, разработанный компанией Barefoot (ныне Intel) для организации процесса обработки сетевого трафика на скорости чипа. Barefoot разработал несколько FPGA чипов для обработки трафика которые были встроенны в некоторые модели коммутаторов Arista и Brocade.   
## Цель работы  
Изучить синтаксис языка программирования P4 и выполнить 2 задания обучающих задания от Open network foundation для ознакомления на практике с P4.    
### Ход работы  
Для работы необходимо склонировать скачать образ для ВМ с p4 клиентом и Vagrant.  
Скачанный Vagrant представлен ниже:

![Скачанный Vagrant](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-06%20233809.png)  

Далее, зайдём на ВМ с логиным и паролем: p4/p4.  
Успешный вход представлен ниже:

![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20115613.png)  

Запустим mininet, в которой будем конфигурировать сеть, через make run:  

![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20120731.png)  

Далее, начнем выполнение первого задания

### Basic forwarding  
Полный файл basic.p4 будет содержать следующие компоненты:  

Определения типов заголовков для Ethernet (ethernet_t) и IPv4 (ipv4_t).  
TODO: Парсеры для Ethernet и IPv4, которые заполняют поля ethernet_t и ipv4_t.  
Действие для сброса пакета, использующее функцию mark_to_drop().  
TODO: Действие (называемое ipv4_forward), которое:  
Устанавливает порт выхода для следующего хопа.  
Обновляет адрес назначения ethernet адресом следующего хопа.  
Обновляет адрес источника ethernet адресом коммутатора.  
Уменьшает TTL.  
TODO: Элемент управления, который:  
Определяет таблицу, которая будет считывать адрес назначения IPv4 и вызывать либо drop, либо ipv4_forward.  
Блок применения, который применяет таблицу.  
TODO: Депарсер, который выбирает порядок вставки полей в исходящий пакет.  

Код для заголовка представлен ниже:
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20123726.png)  

Код для парсера представлен ниже:
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20123759.png)  

Код для Ingress (для перессылки IPv4 пакетов) представлен ниже:
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20123851.png)  
и
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20123910.png)  

Код для депарсера (добавление заголовка обратно) представлен ниже:
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20123920.png)  


### Basic Tunneling
