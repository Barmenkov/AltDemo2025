# AltDemo2025
1. Сначала нужно развернуть все виртуальные машины, которые указаны в таблице.

| Машина | RAM,ГБ | CPU | HDD/SDD, ГБ | Шаблон |
| ------ | ------ | --- | ----------- | ------ |
| ISP | 4 | 1 | 10 | «vESR» |
| HQ-RTR | 4 | 1 | 10 | «EcoRouter» |
| BR-RTR | 4 | 1 | 10 | «vESR» |
| HQ-SRV | 2 | 1 | 10 | «ALT-server» |
| BR-SRV | 2 | 1 | 10 | «ALT-server» |
| HQ-CLI | 3 | 2 | 15 | «ALT-workstatison» |
### EcoRouter
Установка `EcoRouter`[->](./ecorouter_install/README.md)

### vESR
Установка `vESR`[->](./install_vesr/README.md)

2. Создаем виртуальные сети.[->](./create_virtual_switch/README.md)
   
4. Делаем внеполосное управление.[->](./mgmt/README.md)
   
6. Настройка консоли `vESR`.[->](./settings_vesr_console/README.md)
# Распределение IP адресов
 
| Имя устройства | Интерфейс | IP          | Маска           | Шлюз        |
| -------------- | --------- | ----------  | --------------- | ----------- |
| ISP            | gi1/0/1   | DHCP        |                 |             |
|                | gi1/0/2   | 172.16.5.1  | 255.255.255.240 |             |
|                | gi1/0/3   | 172.16.4.1  | 255.255.255.240 |             |
| HQ-RTR         | ge0       | 172.16.4.2  | 255.255.255.240 | 172.16.4.1  |      
|                | ge1.100   | 192.168.0.1 | 255.255.255.192 |             |      
|                | ge1.200   | 192.168.0.65| 255.255.255.240 |             |      
|                | ge1.999   | 192.168.0.81| 255.255.255.248 |             |      
|                | tunnel.1  | 172.16.1.1  | 255.255.255.252 |             |      
| BR-RTR         | gi1/0/1   | 172.16.5.2  | 255.255.255.240 | 172.16.5.1  |      
|                | gi1/0/2   | 192.168.1.1 | 255.255.255.224 |             |
|                | gre1      | 172.16.1.2  | 255.255.255.252 |             |
| HQ-SRV         | ens192    | 192.168.0.2 | 255.255.255.192 | 192.168.0.1 |      
| HQ-CLI         | ens192    | DHCP        | 255.255.255.240 | 192.168.0.65|      
| BR-SRV         | ens192    | 192.168.1.2 | 255.255.255.224 | 192.168.1.1 |      
# Настраиваем IP адреса

## ISP

```
configure

int gi1/0/3
ip address 172.16.4.1/28
ip firewall disable
no shutdown

int gi1/0/2
ip address 172.16.5.1/28
ip firewall disable
no shutdown

commit
confirm
```
## HQ-RTR - EcoRouter

Создаем сущность интерфейса и назначаем IP

```
int TO-ISP
ip address 172.16.4.2/28
no shutdown
```

Привязываем созданный интерфейс к физическому протоколу

1. Заходим в `port ge0`
2. Создаем service-instance `service-instance SI-ISP`
3. Указываем, что кадры на этом интерфейсе будут без тега `encapsulation untagged`
4. Привязываем сущьность интрефейса к порту `connect ip interface TO-ISP`

```
port ge0
service-instance SI-ISP
encapsulation untagged
connect ip interface TO-ISP
```

Создаем интерфейсы, которые будут обрабатывать трафик vlan 100, 200, 999

```
interface HQ-SRV
 ip mtu 1500
 ip address 192.168.0.1/26
!
interface HQ-CLI
 ip mtu 1500
 ip address 192.168.0.65/28
!
interface HQ-MGMT
 ip mtu 1500
 ip address 192.168.0.81/29
!
```

Заходим на порт и создаем для каждой `vlan` свой `service-instance`.

1. Заходим в `port ge1`.
2. Создаем service-instance `service-instance ge1/vlan100`
3. Указываем инкапсуляцию для `100 vlan`
4. Чтобы кадры из этого интерфейса выходили с тегом задаем настройку `rewrite pop 1`
5. Привязываем сущьность интрефейса к порту `connect ip interface HQ-SRV`

Проделываем эти действия для vlan 200 и 999

```
port ge1
 mtu 9234
 service-instance ge1/vlan100
  encapsulation dot1q 100
  rewrite pop 1
  connect ip interface HQ-SRV
 service-instance ge1/vlan200
  encapsulation dot1q 200
  rewrite pop 1
  connect ip interface HQ-CLI
 service-instance ge1/vlan999
  encapsulation dot1q 999
  rewrite pop 1
  connect ip interface HQ-MGMT
```

<img src="02.png" width='600'>

Создаем GRE туннель

```
interface tunnel.1
 ip mtu 1400
 ip address 172.16.1.1/30
 ip tunnel 172.16.4.2 172.16.5.2 mode gre
```

Задаем маршрут по умолчанию в сторону ISP

```
ip route 0.0.0.0/0 172.16.4.1
```
