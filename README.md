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
Установка `EcoRouter`
Имеем образ EcoRouter в qcow2 формате.

Шаг 1. Конвертируем qcow2 в vmdk

Используем qumu-img скачать можно [здесь](https://cloudbase.it/downloads/qemu-img-win-x64-2_3_0.zip)

ipv6.qcow2 - имя файла образа EcoRouter в формате qcow2

ipv6.vmdk - имя файла результата конвертации образа EcoRouter (может быть любым) 

```
qemu-img.exe convert -f qcow2 ipv6.qcow2 -O vmdk ipv6.vmdk
```
### vESR
Установка `vESR`
Предвариетльно нужно получить `ISO` образ `vESR`. Это можно сделать на оффсайте `ELTEX` по запросу или просто загуглить.
У vESR есть FREE лицензия. Ограничения только пропускной способности в 1Мб/с
1. Нажимаем `Create/Register VM`.
2. Выбераем пункт `Create a new virtual machine`.
3. Нажимаем `Next` для перехода к следующему шагу.
4. Установите имя виртуальной машины.
5. Поле `Compatibillity` заполняется автоматически. Оставляем как есть.
6. Поле `Guest OS family` выберите операционную систему `Linux`.
7. Поле "Guest OS version" выберите любую 64-битную версию, например "Other Linux (64-bit)".
8. Нажимаем `Next` для перехода к следующему шагу.
9. Выбираем где будет храниться виртуальная машина.
10. Нажимаем `Next` для перехода к следующему шагу.
11. Установите `Memory` минимум 4 Гб.
12. Изменита настройка контроллера HDD на `IDE.
13. Удаляем `SCSI Controller`.
14. Удаляем `USB Controller`.
15. Изменяем `CD/DVD Drive` на `Datastore ISO file`
16. ВЫбираем предварительно загруженный ISO образ vESR
17. Нажимаем `SELECT`.
18. Выделяем созданную виртуальную машину.
19. Включаем ее.
После загрузки появиться меню выбора действий.
20. Вибираем `vESR installation`.
21. Нажимаем `OK`.
22. При помощи пробела выделите диск куда будет проходить установка.
23. Нажимаем `ОК`.
24. Соглашаемся с продолжением установки `Yes`.
25. После завершения установки перезагружаем виртуальную машину. Введите комманду `reboot`.
После переустановки нужно ввести дефолтные учетные данные. После этого нужно сразу сменить парооль администратора
26. Логин `admin`.
27. Пароль `password`.
28. Введите новый пароль. Пароль должен отличаться от дефолтного.
29. Зафиксируйте изменения введите `commit`.
30. Подтвердите изменения `confirm`.

Установка завершена.


###2. Создаем виртуальные сети.
1. Переходим в раздел `Networking` затем во вкладку `Virtual switchs`
2. Добавляем новый виртуальный коммутатор `add standart virtual switch`
3. Задаем имя новому виртуальному коммутатору `HQ-SW`
4. Раскрываем пункт `Security` изменяем все пункты на `Accept`
5. Нажимаем кнопку `ADD`
6. Переходим в раздел `Networking`
7. Переходим во вкладку `Port groups`
8. Добавляем новую порт группу `add port group`
9. Задаем имя порт группе в соответствие с топологией. В этом примере создаем сеть для `HQ-SRV`
10. Задаем номер `VLAN` в соответствие с заданием. В этом примере это номер 100
11. Подключаем эту порт группу в коммутатору, который создали на предыдущем этапе
12. Нажиамем кнопку `ADD` 
13. Проделываем этапы с 6 по 12 для `CLI-net`. Для `HQ-net` нужно сделать `trunk` режим. Для этого в поле `VLAN ID` задаем значение `4095`
В результате должно получиться `четыре` виртуальныйх коммутатора и `шесть` порт групп
###3. Делаем внеполосное управление.
### Включаем службу в firewall на ESXi
1. Переходим в раздел `Networking`.
2. Открываем вкладку `Firewall rules`.
3. Ищем правило `remoteSerialPort`. Нажимаем ПКМ
4. Активируем это правило.
### Настраиваем виртуальные машины
1. Выключаем виртуальную машину нажав кнопку `Power off`
2. Переходим в редактирование виртуальноый машины нажав кнопку `Edit`
3. Добавляем новое устройство нажав на `Add other device`
4. Выбираем пункт `Serial port`
5. Изменяем тип порта на `Use network`
6. В строку `Port URL` добавляем значение в формате `telnet://0.0.0.0:<port>`. Номер порта для каждой виртуальной машины должен быть уникальным
7. Сохраняем изменения нажав кнопку `SAVE`
8. Включаем виртуальную машину.
Проделываем эту процедуру для всех виртуальных машин.
В виртуальных машинах с `Linux` нужно включить службу, добавить ее в автозагрузку и проверить статус:
```
systemctl start serial-getty@ttyS0.service
systemctl enable serial-getty@ttyS0.service
systemctl status serial-getty@ttyS0.service
```
Открывам любую консольку и подключаемся к `ESXi` с ее помощью по протоколу `telnet`
В этом примере используется `PUTTY`
1. Вписываем ip адрес вашего стенда.
2. Указываем порт, который указывали в настройуках виртуальной машины.
6. Настройка консоли `vESR`.
Для того, чтобы логи выводились на удаленную консоль делаем следующие настройки.
```
config
syslog console
virtual-serial
```
Перезагружаем устройство.
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
## HQ-SRV

```
echo "TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes" > /etc/net/ifaces/ens192/options
```

```
echo 192.168.0.2/26 > /etc/net/ifaces/ens192/ipv4address
```

```
echo default via 192.168.0.1 > /etc/net/ifaces/ens192/ipv4route
```

```
systemctl restart network
```

## HQ-CLI

Предварительно нужно настроить `DHCP-сервер` на `HQ-RTR` [->](../dhcp/README.md)

![image](https://github.com/user-attachments/assets/f34b9e68-c1a9-4f28-93ef-d588b3eac9b7)
![image](https://github.com/user-attachments/assets/019b130b-2251-4719-a461-b46ae3ea22f3)
![image](https://github.com/user-attachments/assets/3865e576-e667-4a92-a6a2-f5581e3dda88)
![image](https://github.com/user-attachments/assets/ecc16e26-8cf4-4c6a-8185-d4492dad9bd6)
Если все сработало, то в выводе `ip a` увидим следующее
![image](https://github.com/user-attachments/assets/8656855f-7c88-4055-b111-72dba4e8c575)

## BR-RTR

```
configure

interface gigabitethernet 1/0/1
  ip firewall disable
  ip address 172.16.5.2/28
  no shutdown
exit
interface gigabitethernet 1/0/2
  ip firewall disable
  ip address 192.168.1.1/27
  no shutdown
exit
tunnel gre 1
  ttl 16
  mtu 1400
  ip firewall disable
  local address 172.16.5.2
  remote address 172.16.4.2
  ip address 172.16.1.2/30
  enable
exit
ip route 0.0.0.0/0 172.16.5.1

commit
confirm
```
## BR-SRV

```
echo "TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes" > /etc/net/ifaces/ens192/options
```

```
echo 192.168.1.2/26 > /etc/net/ifaces/ens192/ipv4address
```

```
echo default via 192.168.1.1 > /etc/net/ifaces/ens192/ipv4route
```

```
echo nameserver 192.168.0.2 > /etc/net/ifaces/ens192/resolv.conf
```

```
systemctl restart network
```

```
ip address
```
