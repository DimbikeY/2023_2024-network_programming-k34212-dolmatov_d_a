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
* TODO: Парсеры для Ethernet и IPv4, которые заполняют поля ethernet_t и ipv4_t.  
* Действие для сброса пакета, использующее функцию mark_to_drop().  
* TODO: Действие (называемое ipv4_forward), которое:  
* Устанавливает порт выхода для следующего хопа.  
* Обновляет адрес назначения ethernet адресом следующего хопа.  
* Обновляет адрес источника ethernet адресом коммутатора.  
* Уменьшает TTL.  
* TODO: Элемент управления, который:  
* Определяет таблицу, которая будет считывать адрес назначения IPv4 и вызывать либо drop, либо ipv4_forward.  
* Блок применения, который применяет таблицу.  
* TODO: Депарсер, который выбирает порядок вставки полей в исходящий пакет.  

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

В результате написания программы теперь возможно обращение к соседним сетевым устройствам:  
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20120808.png)  

Схема задания изображена ниже:
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/16.png)  

Полный листинг представлен ниже:  
```bash
/* -*- P4_16 -*- */
#include <core.p4>
#include <v1model.p4>

const bit<16> TYPE_IPV4 = 0x800;

/*************************************************************************
*********************** H E A D E R S  ***********************************
*************************************************************************/

typedef bit<9>  egressSpec_t;
typedef bit<48> macAddr_t;
typedef bit<32> ip4Addr_t;

header ethernet_t {
    macAddr_t dstAddr;
    macAddr_t srcAddr;
    bit<16>   etherType;
}

header ipv4_t {
    bit<4>    version;
    bit<4>    ihl;
    bit<8>    diffserv;
    bit<16>   totalLen;
    bit<16>   identification;
    bit<3>    flags;
    bit<13>   fragOffset;
    bit<8>    ttl;
    bit<8>    protocol;
    bit<16>   hdrChecksum;
    ip4Addr_t srcAddr;
    ip4Addr_t dstAddr;
}

struct metadata {
    /* empty */
}

struct headers {
    ethernet_t   ethernet;
    ipv4_t       ipv4;
}

/*************************************************************************
*********************** P A R S E R  ***********************************
*************************************************************************/

parser MyParser(packet_in packet,
                out headers hdr,
                inout metadata meta,
                inout standard_metadata_t standard_metadata) {

    state start { transition parse; }

    state parse{
      packet.extract(hdr.ethernet);
      transition select(hdr.ethernet.etherType){
        TYPE_IPV4: parse_ipv4;
        default: accept;
        }
    }

    state parse_ipv4{
      packet.extract(hdr.ipv4);
      transition accept;
    }
}


/*************************************************************************
************   C H E C K S U M    V E R I F I C A T I O N   *************
*************************************************************************/

control MyVerifyChecksum(inout headers hdr, inout metadata meta) {
    apply {  }
}


/*************************************************************************
**************  I N G R E S S   P R O C E S S I N G   *******************
*************************************************************************/

control MyIngress(inout headers hdr,
                  inout metadata meta,
                  inout standard_metadata_t standard_metadata) {
    action drop() {
        mark_to_drop(standard_metadata);
    }

    action ipv4_forward(macAddr_t dstAddr, egressSpec_t port) {
        standard_metadata.egress_spec= port;
        hdr.ethernet.srcAddr = hdr.ethernet.dstAddr;
        hdr.ethernet.dstAddr = dstAddr;
        hdr.ipv4.ttl = hdr.ipv4.ttl - 1;
    }

    table ipv4_lpm {
        key = {
            hdr.ipv4.dstAddr: lpm;
        }
        actions = {
            ipv4_forward;
            drop;
            NoAction;
        }
        size = 1024;
        default_action = NoAction();
    }

    apply {
        if(hdr.ipv4.isValid()){
          ipv4_lpm.apply();
        }
    }
}

/*************************************************************************
****************  E G R E S S   P R O C E S S I N G   *******************
*************************************************************************/

control MyEgress(inout headers hdr,
                 inout metadata meta,
                 inout standard_metadata_t standard_metadata) {
    apply {  }
}

/*************************************************************************
*************   C H E C K S U M    C O M P U T A T I O N   **************
*************************************************************************/

control MyComputeChecksum(inout headers hdr, inout metadata meta) {
     apply {
        update_checksum(
            hdr.ipv4.isValid(),
            { hdr.ipv4.version,
              hdr.ipv4.ihl,
              hdr.ipv4.diffserv,
              hdr.ipv4.totalLen,
              hdr.ipv4.identification,
              hdr.ipv4.flags,
              hdr.ipv4.fragOffset,
              hdr.ipv4.ttl,
              hdr.ipv4.protocol,
              hdr.ipv4.srcAddr,
              hdr.ipv4.dstAddr },
            hdr.ipv4.hdrChecksum,
            HashAlgorithm.csum16);
    }
}


/*************************************************************************
***********************  D E P A R S E R  *******************************
*************************************************************************/

control MyDeparser(packet_out packet, in headers hdr) {
    apply {
        packet.emit(hdr.ethernet);
        packet.emit(hdr.ipv4);
    }
}

/*************************************************************************
***********************  S W I T C H  *******************************
*************************************************************************/

V1Switch(
MyParser(),
MyVerifyChecksum(),
MyIngress(),
MyEgress(),
MyComputeChecksum(),
MyDeparser()
) main;
```  

### Basic Tunneling  
Далее, сделаем второе задание  
* Добавлен новый тип заголовка myTunnel_t, который содержит два 16-битных поля: proto_id и dst_id.  
* Заголовок myTunnel_t был добавлен в структуру headers.  
* Обновлён парсер для извлечения заголовка myTunnel или заголовка ipv4 на основе поля etherType в заголовке Ethernet. etherType, соответствующий заголовку myTunnel, равен 0x1212. Парсер также должен извлечь заголовок ipv4 после заголовка myTunnel, если proto_id == TYPE_IPV4 (т.е. 0x0800).  
* Определено новое действие под названием myTunnel_forward, которое просто устанавливает порт выхода (т.е. поле egress_spec шины standard_metadata) на номер порта, предоставленный плоскостью управления.  
* Определена новая таблицу myTunnel_exact, которая выполняет точное совпадение по полю dst_id заголовка myTunnel. Эта таблица должна вызывать действие myTunnel_forward, если в таблице есть совпадение, и должна вызывать действие drop в противном случае.  
* Обновлен оператор apply в блоке управления MyIngress, чтобы применить новую определенную таблицу myTunnel_exact, если заголовок myTunnel действителен. В противном случае вызвана таблица ipv4_lpm, если заголовок ipv4 действителен.  
* Обновлен депарсер, чтобы он выдавал заголовки ethernet, затем myTunnel, затем ipv4. Депарсер будет выдавать заголовок, только если он действителен. Неявный бит валидности заголовка устанавливается парсером при извлечении. Поэтому проверять валидность заголовка здесь не нужно.  
* Добавлены статические правила для новой таблицы, чтобы коммутаторы правильно пересылали для каждого возможного значения dst_id. На схеме ниже показана конфигурация портов топологии, а также то, как мы будем присваивать ID хостам. Для этого шага нужно будет добавить правила пересылки в файлы sX-runtime.json.  

Был добавлен парсер parse_myTunnel, который извлекает заголовок myTunnel. Если значение поля proto_id равно 0x800, то следует перейти на парсер parse_ipv4.  
Далее парсер parse_ethernet был изменен, чтобы извлечь либо ipv4 заголовок, либо myTunnel заголовок в зависимости от значения поля etherType. Значение etherType, которое соответствует заголовку myTunnel, равно 0x1212. Это значение определено в самом начале файла.  

![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20124152.png)  

Далее была определена таблица маршрутизации
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20124228.png) 

и  

![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20124237.png)  

Депарсер представлен ниже:  

![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/Снимок%20экрана%202023-12-11%20124245.png)  

Результаты выполнения данного задания приведены:  
Первый пакет был отправлен без данных:
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/1_final.png)  

Второй пакет был отправлен с данными для туннелирования:  
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/2_final.png)  

Третий пакет был отправлен по IP адресу третьего с dst_id второго. Приоритет идёт по dst_id => пакет ушёл на второй коммутатор, проигнорировав третий:  
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/3_final.png)  

Схема представлена ниже:  
![](https://github.com/DimbikeY/2023_2024-network_programming-k34212-dolmatov_d_a/blob/main/lab4/resources/17.png)  

Полный листинг для второго задания представлен ниже:  
```bash
/* -*- P4_16 -*- */
#include <core.p4>
#include <v1model.p4>

// NOTE: new type added here
const bit<16> TYPE_MYTUNNEL = 0x1212;
const bit<16> TYPE_IPV4 = 0x800;

/*************************************************************************
*********************** H E A D E R S  ***********************************
*************************************************************************/

typedef bit<9>  egressSpec_t;
typedef bit<48> macAddr_t;
typedef bit<32> ip4Addr_t;

header ethernet_t {
    macAddr_t dstAddr;
    macAddr_t srcAddr;
    bit<16>   etherType;
}

// NOTE: added new header type
header myTunnel_t {
    bit<16> proto_id;
    bit<16> dst_id;
}

header ipv4_t {
    bit<4>    version;
    bit<4>    ihl;
    bit<8>    diffserv;
    bit<16>   totalLen;
    bit<16>   identification;
    bit<3>    flags;
    bit<13>   fragOffset;
    bit<8>    ttl;
    bit<8>    protocol;
    bit<16>   hdrChecksum;
    ip4Addr_t srcAddr;
    ip4Addr_t dstAddr;
}

struct metadata {
    /* empty */
}

// NOTE: Added new header type to headers struct
struct headers {
    ethernet_t   ethernet;
    myTunnel_t   myTunnel;
    ipv4_t       ipv4;
}

/*************************************************************************
*********************** P A R S E R  ***********************************
*************************************************************************/

// TODO: Update the parser to parse the myTunnel header as well
parser MyParser(packet_in packet,
                out headers hdr,
                inout metadata meta,
                inout standard_metadata_t standard_metadata) {

    state start {
        transition parse_ethernet;
    }

    state parse_ethernet {
        packet.extract(hdr.ethernet);
        transition select(hdr.ethernet.etherType) {
            TYPE_MYTUNNEL: parse_tunnel;
            TYPE_IPV4 : parse_ipv4;
            default : accept;
        }
    }

    state parse_tunnel {
        packet.extract(hdr.myTunnel);
        transition select(hdr.myTunnel.proto_id){
            TYPE_IPV4: parse_ipv4;
            default: accept;
        }
    }

    state parse_ipv4 {
        packet.extract(hdr.ipv4);
        transition accept;
    }


}

/*************************************************************************
************   C H E C K S U M    V E R I F I C A T I O N   *************
*************************************************************************/

control MyVerifyChecksum(inout headers hdr, inout metadata meta) {
    apply {  }
}


/*************************************************************************
**************  I N G R E S S   P R O C E S S I N G   *******************
*************************************************************************/

control MyIngress(inout headers hdr,
                  inout metadata meta,
                  inout standard_metadata_t standard_metadata) {
    action drop() {
        mark_to_drop(standard_metadata);
    }

    action ipv4_forward(macAddr_t dstAddr, egressSpec_t port) {
        standard_metadata.egress_spec = port;
        hdr.ethernet.srcAddr = hdr.ethernet.dstAddr;
        hdr.ethernet.dstAddr = dstAddr;
        hdr.ipv4.ttl = hdr.ipv4.ttl - 1;
    }

    action myTunnel_forward(egressSpec_t port){
        standard_metadata.egress_spec = port;
    }

    table ipv4_lpm {
        key = {
            hdr.ipv4.dstAddr: lpm;
        }
        actions = {
            ipv4_forward;
            drop;
            NoAction;
        }
        size = 1024;
        default_action = drop();
    }

    table myTunnel_exact {
        key = {
            hdr.myTunnel.dst_id: exact;
        }
        actions = {
            myTunnel_forward;
            drop;
        }
        size = 1024;
        default_action = drop();
    }


    apply {
        if (hdr.ipv4.isValid() && !hdr.myTunnel.isValid()) {
            ipv4_lpm.apply();
        }

        if(hdr.myTunnel.isValid()){
            myTunnel_exact.apply();
        }
    }
}

/*************************************************************************
****************  E G R E S S   P R O C E S S I N G   *******************
*************************************************************************/

control MyEgress(inout headers hdr,
                 inout metadata meta,
                 inout standard_metadata_t standard_metadata) {
    apply {  }
}

/*************************************************************************
*************   C H E C K S U M    C O M P U T A T I O N   **************
*************************************************************************/

control MyComputeChecksum(inout headers  hdr, inout metadata meta) {
     apply {
        update_checksum(
            hdr.ipv4.isValid(),
            { hdr.ipv4.version,
              hdr.ipv4.ihl,
              hdr.ipv4.diffserv,
              hdr.ipv4.totalLen,
              hdr.ipv4.identification,
              hdr.ipv4.flags,
              hdr.ipv4.fragOffset,
              hdr.ipv4.ttl,
              hdr.ipv4.protocol,
              hdr.ipv4.srcAddr,
              hdr.ipv4.dstAddr },
            hdr.ipv4.hdrChecksum,
            HashAlgorithm.csum16);
    }
}

/*************************************************************************
***********************  D E P A R S E R  *******************************
*************************************************************************/

control MyDeparser(packet_out packet, in headers hdr) {
    apply {
        packet.emit(hdr.ethernet);
        packet.emit(hdr.myTunnel);
        packet.emit(hdr.ipv4);
    }
}

/*************************************************************************
***********************  S W I T C H  *******************************
*************************************************************************/

V1Switch(
MyParser(),
MyVerifyChecksum(),
MyIngress(),
MyEgress(),
MyComputeChecksum(),
MyDeparser()
) main;
```
### Выводы:  
В результате выполнения данной лабораторной работы были изучены основы P4 технологии, позволяющей программно конфигурировать правила парсинга IP-пакета на сетевых устройствах, что позволяет настраивать свою конфигурацию forwarding и tunneling.  
