# Мосты, туннели и VPN 
**Домашнее задание**  
- Между двумя виртуалками поднять vpn в режимах **tun** и **tap**. Почувствовать разницу
- Поднять RAS на базе OpenVPN с клиентскими сертификатами (подключиться с локальной машины на виртуалку)
- Самостоятельно изучить и поднять ocserv. Подключиться с хоста к виртуалке

---
### Поднять vpn в режимах *tun* и *tap*  

1. Из Vagrantfile поднимаем две машины, **'server'** и **'client'**:  
2. Выполняем следующие команды для обеих машин (действуем от рута):  
- устанавливаем epel репозиторий:  
`yum install -y epel-release`  
- устанавливаем пакет openvpn и iperf3  
`yum install -y openvpn iperf3`  
- Отключаем SELinux  
`setenforce 0` (работает до ребута)
3. Настраиваем openvpn на машине **server**:  
- создаём файл-ключ
`openvpn --genkey --secret /etc/openvpn/static.key`
- создаём конфигурационнýй файл vpn-сервера
`vi /etc/openvpn/server.conf`
- Файл server.conf должен содержать следующий текст:  
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
- создаём конфигурационнýй файл клиента  
`vi /etc/openvpn/server.conf`  
- Файл должен содержать следующий конфиг  
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

- На машине **client** в директорию /etc/openvpn/ копируем файл-ключ **static.key**, который был создан на **server**  

6. Запускаем openvpn клиент и добавляем в автозагрузку:  
`systemctl start openvpn@server`  
`systemctl enable openvpn@server`  
7. Далее замеряем скорость в туннеле
- на openvpn **server**  запускаем iperf3 в режиме сервера:  
`iperf3 -s &`  
- на openvpn **client** запускаем iperf3 в режиме клиента и замеряем скорость в туннеле:  
`iperf3 -c 10.10.10.1 -t 40 -i 5`  
8. Для режима работы **tun** повторяем пункты с 1 по 5, но в конфигурационных файлах сервера и клиента изменяем дерективу dev с **tap** на **tun**.  
9. Делаем выводы о режимах, их достоинствах и недостатках.  
