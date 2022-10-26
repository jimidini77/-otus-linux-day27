# otus-linux-day27
## *Архитектура сетей*

## **Prerequisite**
- Host OS: Debian 12.0.0
- Guest OS: CentOS 7.8.2003
- VirtualBox: 6.1.40
- Vagrant: 2.3.2

# **Содержание ДЗ**

1. Скачать и развернуть Vagrant-стенд (https://github.com/erlong15/otus-linux/tree/network)
2. Построить следующую сетевую архитектуру:

* Сеть office1
  - 192.168.2.0/26 - dev
  - 192.168.2.64/26 - test servers
  - 192.168.2.128/26 - managers
  - 192.168.2.192/26 - office hardware

* Сеть office2
  - 192.168.1.0/25 - dev
  - 192.168.1.128/26 - test servers
  - 192.168.1.192/26 - office hardware

* Сеть central
  - 192.168.0.0/28 - directors
  - 192.168.0.32/28 - office hardware
  - 192.168.0.64/26 - wifi

Итого должны получиться следующие сервера:
- inetRouter
- centralRouter
- office1Router
- office2Router
- centralServer
- office1Server
- office2Server

Задание состоит из 2-х частей: теоретической и практической.

В теоретической части требуется:
- Найти свободные подсети
- Посчитать количество узлов в каждой подсети, включая свободные
- Указать Broadcast-адрес для каждой подсети
- Проверить, нет ли ошибок при разбиении

В практической части требуется:
- Соединить офисы в сеть согласно логической схеме и настроить роутинг
- Интернет-трафик со всех серверов должен ходить через inetRouter
- Все сервера должны видеть друг друга (должен проходить ping)
- У всех новых серверов отключить дефолт на NAT (eth0), который vagrant поднимает для связи
- Добавить дополнительные сетевые интерфейсы, если потребуется

# **Выполнение**

## **Теория**

В таблице перечислены информация по подсетям: имя, адрес подсети, маска, количество узлов, минимальный/максимальный адрес узла, широковещательный адрес:

| Name | Network | Netmask | N | Hostmin | Hostmax | Broadcast |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
 **Central Network**
| Directors | 192.168.0.0/28 | 255.255.255.240 | 14 | 192.168.0.1 | 192.168.0.14 | 192.168.0.15 |
| Office hardware | 192.168.0.32/28 | 255.255.255.240 | 14 | 192.168.0.33 | 192.168.0.46 | 192.168.0.47 |
| Wifi (mgt network) | 192.168.0.64/26 | 255.255.255.192 | 62 | 192.168.0.65 | 192.168.0.126 | 192.168.0.127 |
 **Office 1 network** 
| Dev | 192.168.2.0/26 | 255.255.255.192 | 62 | 192.168.2.1 | 192.168.2.62 | 192.168.2.63 | 
| Test | 192.168.2.64/26 | 255.255.255.192 | 62 | 192.168.2.65 | 192.168.2.126 | 192.168.2.127 |
|Managers | 192.168.2.128/26 | 255.255.255.192 | 62 | 192.168.2.129 | 192.168.2.190 | 192.168.2.191 |
| Office hardware | 192.168.2.192/26 | 255.255.255.192 |62| 192.168.2.193 |192.168.2.254 |192.168.2.255|
**Office 2 network**
|Dev| 192.168.1.0/25| 255.255.255.128| 126 |192.168.1.1 |192.168.1.126 |192.168.1.127|
|Test| 192.168.1.128/26| 255.255.255.192 |62| 192.168.1.129| 192.168.1.190 |192.168.1.191|
|Office |192.168.1.192/26 |255.255.255.192 |62 |192.168.1.193 |192.168.1.254 |192.168.1.255|
**InetRouter - CentralRouter network**
Inet-central|192.168.255.0/30| 255.255.255.252 |2 |192.168.255.1| 192.168.255.2| 192.168.255.3|

## **Практика**

На всех маршрутизаторах: 
- разрешен форвардинг траффика:
```yml
    - name: Persistent forwarding enable
      lineinfile:
        line: "net.ipv4.ip_forward = 1"
        path: /etc/sysctl.conf
```
- установлены пакеты iptables:
```yml
    - name: SW CONFIG | Install iptables
      yum:
        name:
          - iptables
          - iptables-services
        state: present

    - name: start and enable iptables service
      service:
        name: iptables
        state: restarted
        enabled: true
```
На маршрутизаторе inetRouter настроен маскарадинг и прописаны маршруты до всех подсетей:
```yml
    - name: NAT settings
      iptables:
        table: nat
        chain: POSTROUTING
        destination: "! 192.168.0.0/16"
        out_interface: eth0
        jump: MASQUERADE
      when: (ansible_hostname == "inetRouter")

    - name: inetRouter static routes set
      lineinfile:
        line: '{{ item }}'
        path: /etc/sysconfig/network-scripts/route-eth1
        create: yes
      loop:
        - "192.168.255.4/30 via 192.168.255.2"
        - "192.168.255.8/30 via 192.168.255.2"
        - "192.168.0.0/24 via 192.168.255.2"
        - "192.168.1.0/24 via 192.168.255.2"
        - "192.168.2.0/24 via 192.168.255.2"
      when: (ansible_hostname == "inetRouter")
```

На всех внутренних маршрутизаторах и серверах изменены дефолтные маршруты, на центральном маршрутизаторе кроме того прописаны маршруты к подсетям офисов:
```yml
    - name: centralRouter static routes set
      lineinfile:
        line: '{{ item.line }}'
        path: '{{ item.file }}'
        create: yes
      with_items: 
        - { line: "192.168.2.0/24 via 192.168.255.10", file: "/etc/sysconfig/network-scripts/route-eth5" }
        - { line: "192.168.1.0/24 via 192.168.255.6", file: "/etc/sysconfig/network-scripts/route-eth6" }
        - { line: "DEFROUTE=no", file: "/etc/sysconfig/network-scripts/ifcfg-eth0" }
        - { line: "GATEWAY=192.168.255.1", file: "/etc/sysconfig/network-scripts/ifcfg-eth1"}
      when: (ansible_hostname == "centralRouter")

    - name: Office1Router static routes set
      lineinfile:
        line: '{{ item.line }}'
        path: '{{ item.file }}'
        create: yes
      with_items: 
        - { line: "DEFROUTE=no", file: "/etc/sysconfig/network-scripts/ifcfg-eth0" }
        - { line: "GATEWAY=192.168.255.9", file: "/etc/sysconfig/network-scripts/ifcfg-eth1"}
      when: (ansible_hostname == "Office1Router")

    - name: Office2Router static routes set
      lineinfile:
        line: '{{ item.line }}'
        path: '{{ item.file }}'
        create: yes
      with_items: 
        - { line: "DEFROUTE=no", file: "/etc/sysconfig/network-scripts/ifcfg-eth0" }
        - { line: "GATEWAY=192.168.255.5", file: "/etc/sysconfig/network-scripts/ifcfg-eth1"}
      when: (ansible_hostname == "Office2Router")

    - name: centralServer static routes set
      lineinfile:
        line: '{{ item.line }}'
        path: '{{ item.file }}'
        create: yes
      with_items: 
        - { line: "DEFROUTE=no", file: "/etc/sysconfig/network-scripts/ifcfg-eth0" }
        - { line: "GATEWAY=192.168.0.1", file: "/etc/sysconfig/network-scripts/ifcfg-eth1"}
      when: (ansible_hostname == "centralServer")

    - name: Office1Server static routes set
      lineinfile:
        line: '{{ item.line }}'
        path: '{{ item.file }}'
        create: yes
      with_items: 
        - { line: "DEFROUTE=no", file: "/etc/sysconfig/network-scripts/ifcfg-eth0" }
        - { line: "GATEWAY=192.168.2.1", file: "/etc/sysconfig/network-scripts/ifcfg-eth1"}
      when: (ansible_hostname == "Office1Server")

    - name: Office2Server static routes set
      lineinfile:
        line: '{{ item.line }}'
        path: '{{ item.file }}'
        create: yes
      with_items: 
        - { line: "DEFROUTE=no", file: "/etc/sysconfig/network-scripts/ifcfg-eth0" }
        - { line: "GATEWAY=192.168.1.1", file: "/etc/sysconfig/network-scripts/ifcfg-eth1"}
      when: (ansible_hostname == "Office2Server")

```

Все машины после конфигурирования перегружаются т.к. рестарт network.service в половине случаев не приводил к изменению маршрута по-умолчанию.

После конфигурирования все машины имеют доступ в интернет и к соседним подсетям. Например:
```sh
[vagrant@Office1Server ~]$ ping otus.ru
PING otus.ru (188.114.99.234) 56(84) bytes of data.
64 bytes from 188.114.99.234 (188.114.99.234): icmp_seq=1 ttl=57 time=36.2 ms
64 bytes from 188.114.99.234 (188.114.99.234): icmp_seq=2 ttl=57 time=35.7 ms
^C
--- otus.ru ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 35.748/35.979/36.211/0.299 ms
[vagrant@Office1Server ~]$
[vagrant@Office1Server ~]$
[vagrant@Office1Server ~]$ ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=10.2 ms
^C
--- 192.168.0.2 ping statistics ---
2 packets transmitted, 1 received, 50% packet loss, time 1004ms
rtt min/avg/max/mdev = 10.275/10.275/10.275/0.000 ms
[vagrant@Office1Server ~]$
[vagrant@Office1Server ~]$
[vagrant@Office1Server ~]$ ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=61 time=8.75 ms
^C
--- 192.168.1.2 ping statistics ---
2 packets transmitted, 1 received, 50% packet loss, time 1002ms
rtt min/avg/max/mdev = 8.753/8.753/8.753/0.000 ms
[vagrant@Office1Server ~]$
[vagrant@Office1Server ~]$
[vagrant@Office1Server ~]$ tracepath otus.ru
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.2.129                                         3.045ms
 1:  192.168.2.129                                         2.733ms
 2:  192.168.255.9                                         6.304ms
 3:  192.168.255.1                                        11.070ms
 4:  no reply
^C
[vagrant@Office1Server ~]$
```

# **Результаты**

Полученный в ходе работы `Vagrantfile` и плейбук Ansible помещены в публичный репозиторий:

- **GitHub** - https://github.com/jimidini77/otus-linux-day27
