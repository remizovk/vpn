# Мосты, туннели и VPN 
**Домашнее задание**  
- Между двумя виртуалками поднять vpn в режимах **tun** и **tap**. Почувствовать разницу
- Поднять RAS на базе OpenVPN с клиентскими сертификатами (подключиться с локальной машины на виртуалку)
- Самостоятельно изучить и поднять ocserv. Подключиться с хоста к виртуалке

---
### Поднять vpn в режимах *tap* и *tun*  

1. Из Vagrantfile поднимаем две машины, **'server'** и **'client'**  
2. Следующие команды выполняем для обеих машин (действуем от рута):  
- устанавливаем epel репозиторий:  
`yum install -y epel-release`  
- устанавливаем пакет openvpn и iperf3  
`yum install -y openvpn iperf3`  
- отключаем SELinux  
`setenforce 0` (работает до ребута)
3. Настраиваем openvpn на машине **server**:  
- создаём файл-ключ  
`openvpn --genkey --secret /etc/openvpn/static.key`  
- создаём конфигурационнýй файл vpn-сервера  
`vi /etc/openvpn/server.conf`  

> dev tap  
> ifconfig 10.10.10.1 255.255.255.0  
> topology subnet  
> secret /etc/openvpn/static.key  
> comp-lzo  
> status /var/log/openvpn-status.log  
> log /var/log/openvpn.log  
> verb 3  
4. Запускаем openvpn сервер и добавляем в автозагрузку:  
`systemctl start openvpn@server`  
`systemctl enable openvpn@server`  
`systemctl status openvpn@server`  

![](https://github.com/remizovk/vpn/blob/8ccdde866dca205ae01bf41f4f82852c671885ec/screenshots/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202022-09-19%2015-11-06.png)  
5. Настройка openvpn **client**:  
- создаём конфигурационный файл клиента  
`vi /etc/openvpn/server.conf`  

> dev tap  
> remote 192.168.10.10  
> ifconfig 10.10.10.2 255.255.255.0  
> topology subnet  
> route 192.168.10.0 255.255.255.0  
> secret /etc/openvpn/static.key  
> comp-lzo  
> status /var/log/openvpn-status.log  
> log /var/log/openvpn.log  
> verb 3  

- на машине **client** в директорию /etc/openvpn/ копируем файл-ключ **static.key**, который был создан на **server**  

6. Запускаем openvpn клиент и добавляем в автозагрузку:  
`systemctl start openvpn@server`  
`systemctl enable openvpn@server`  

7. Далее замеряем скорость в туннеле
- на openvpn **server**  запускаем iperf3 в режиме сервера:  
`iperf3 -s &`  
- на openvpn **client** запускаем iperf3 в режиме клиента и замеряем скорость в туннеле:  
`iperf3 -c 10.10.10.1 -t 40 -i 5`  

![](https://github.com/remizovk/vpn/blob/0b6ce0c800019954e7d6a67730328902adf15511/screenshots/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202022-09-19%2015-34-41.png)  

![](https://github.com/remizovk/vpn/blob/0b6ce0c800019954e7d6a67730328902adf15511/screenshots/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202022-09-19%2015-35-15.png)  

8. Для режима работы **tap** повторяем пункты с 1 по 5, но в конфигурационных файлах сервера и клиента изменяем дерективу dev с **tun** на **tap**.  
9. Делаем выводы о режимах, их достоинствах и недостатках.  
> Отличие интерфейсов tun и tap заключается в том, что tap старается больше походить на реальный сетевой интерфейс, а именно он позволяет себе принимать и отправлять ARP запросы, обладает MAC адресом и может являться одним из интерфейсов сетевого моста, так как он обладает полной поддержкой ethernet - протокола канального уровня (уровень 2). Интерфейс tun этой поддержки лишен, поэтому он может принимать и отправлять только IP пакеты и никак не ethernet кадры. Он не обладает MAC-адресом и не может быть добавлен в бридж. Зато он более легкий и быстрый за счет отсутствия дополнительной инкапсуляции и прекрасно подходит для тестирования сетевого стека или построения виртуальных частных сетей (VPN).

### Поднять RAS на базе OpenVPN с клиентскими сертификатами
Для выполнения данного задания можно воспользоваться Vagrantfile из 1 задания, только убрать 1 ВМ.  
После запуска ВМ отключаем SELinux (setenforce 0) или создаём правило для него.  
1. Устанавливаем репозиторий EPEL:  
`yum install -y epel-release`  
2. Устанавливаем необходимые пакеты:  
`yum install -y openvpn easy-rsa`  
3. Переходим в директорию /etc/openvpn/ и инициализируем pki  
`cd /etc/openvpn/`  
`/usr/share/easy-rsa/3.0.8/easyrsa init-pki`  
4. Сгенерируем необходимые ключи и сертификаты для сервера:  
`echo 'rasvpn' | /usr/share/easy-rsa/3.0.3/easyrsa build-ca nopass`  
`echo 'rasvpn' | /usr/share/easy-rsa/3.0.3/easyrsa gen-req server``
`nopass`  
`echo 'yes' | /usr/share/easy-rsa/3.0.3/easyrsa sign-req server server`  
`/usr/share/easy-rsa/3.0.3/easyrsa gen-dh`  
`openvpn --genkey --secret ta.key`  
5. Сгенерируем сертификаты для клиента:
`echo 'client' | /usr/share/easy-rsa/3/easyrsa gen-req client`  
`nopass`  
`echo 'yes' | /usr/share/easy-rsa/3/easyrsa sign-req client`  
`client`  
6. Создадим конфигурационный файл:  
`vi /etc/openvpn/server.conf` 
>port 1207  
>proto udp  
>dev tun  
>ca /etc/openvpn/pki/ca.crt  
>cert /etc/openvpn/pki/issued/server.crt  
>key /etc/openvpn/pki/private/server.key  
>dh /etc/openvpn/pki/dh.pem  
>server 10.10.10.0 255.255.255.0  
>route 192.168.10.0 255.255.255.0  
>push "route 192.168.10.0 255.255.255.0"  
>ifconfig-pool-persist ipp.txt  
>client-to-client  
>client-config-dir /etc/openvpn/client  
>keepalive 10 120  
>comp-lzo  
>persist-key  
>persist-tun  
>status /var/log/openvpn-status.log  
>log /var/log/openvpn.log  
>verb 3  

7. Зададим параметр iroute для клиента:  
`echo 'iroute 192.168.33.0 255.255.255.0' > /etc/openvpn/client/client`  
8. Запускаем openvpn сервер и добавляем в автозагрузку:  
`systemctl start openvpn@server`  
`systemctl enable openvpn@server`
9. Скопируем следующие файлы сертификатов и ключ для клиента на хостмашину:  
`/etc/openvpn/pki/ca.crt`  
`/etc/openvpn/pki/issued/client.crt`  
`/etc/openvpn/pki/private/client.key`  
*(файлы рекомендуется расположить в той же директории, что и client.conf)*   
10. Создадим конфигурационнй файл клиента client.conf на хост-машине  
`client.conf >`  
>dev tun  
>proto udp  
>remote 192.168.10.10 1207  
>client  
>resolv-retry infinite  
>ca ./ca.crt  
>cert ./client.crt  
>key ./client.key  
>route 192.168.10.0 255.255.255.0  
>persist-key  
>persist-tun  
>comp-lzo  
>verb 3  
>
*В этом конфигурационном файле указано, что файлы сертификатов располагаются в директории, где располагается client.conf. Но при желании можно разместить сертификаты в других директориях и в конфиге скорректироватя пути.*   

11. После того, как все готово, подключаемся к openvpn сервер с хост-машины.  
`openvpn --config client.conf`  
12. При успешном подключении проверяем пинг к внутреннему IP-адресу сервера в туннеле.  
`ping -c 4 10.10.10.1`  
13. Также проверяем командой ip r на хостовой машине, что сеть туннеля импортирована в таблицу маршрутизации.  
