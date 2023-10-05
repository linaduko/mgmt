## IPv6

Чтобы выключить IPv6, необходимо открыть файл конфигурации:
```
nano /etc/sysctl.conf
```
Добавить в конец файла следующее:
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
net.ipv6.conf.ens192.disable_ipv6 = 1
```
Перечитать файл конфигурации:
```
/usr/sbin/sysctl -p
```
