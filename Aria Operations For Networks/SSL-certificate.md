## Замена сертификата Aria Operations for Networks

1. Необходимо подключиться c помощью SSH к `aon.example.com` под пользователем `support`.
2. Далее создать конфигурационный файл `vrni.cfg` со следующим содержанием:

! `aon.example.com` в ~subjectAltName и ~commonName меняем на fqdn вашего сервера Aria Operations for Networks.     
! Заменяем все значения в [ req_distinguished_name ] на необходимые вам.
```
[ req ]
default_md = sha512
default_bits = 2048
default_keyfile = rui.key
distinguished_name = req_distinguished_name
encrypt_key = no
prompt = no
string_mask = nombstr
req_extensions = v3_req

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = DNS: aon.example.com

[ req_distinguished_name ]
countryName = BY
stateOrProvinceName = Minsk
localityName = Minsk
0.organizationName = LinOps
organizationalUnitName = Individual
commonName = aon.example.com
```
3. Генерируем ключ и соответствующий запрос сертификата.
```
openssl genrsa -out vrni.key 2048
openssl req -new -key vrni.key -out vrni.csr -config vrni.cfg
```
4. На `dc.example.com/certsrv` проводим следующие манипуляции (забираем сертификаты в формате **Base64**):

![изображение](https://github.com/linaduko/mgmt/assets/101510056/596571da-808f-4229-a867-3a3becdba00b)

![изображение](https://github.com/linaduko/mgmt/assets/101510056/3471a6bb-e92e-479e-a126-c7a4d91ce33a)

![изображение](https://github.com/linaduko/mgmt/assets/101510056/d0ad1aaf-d8e4-47b0-9084-8f2601c3a834)

Если у вас нет шаблона VMware SSL:
- [Официальный источник](https://kb.vmware.com/s/article/2112009)
- [Наглядный источник 1](https://www.vexpert.cloud/creating-certificate-template-for-vsphere-vcenter-server/)
- [Наглядный источник 2](https://www.derekseaman.com/2012/09/create-vmware-windows-ca-certificate.html)
- 
![изображение](https://github.com/linaduko/mgmt/assets/101510056/68f0994d-1322-48ba-895b-5bf2ecc6667c)

Вернемся на `dc.example.com/certsrv`

![изображение](https://github.com/linaduko/mgmt/assets/101510056/9920a8f8-8630-44eb-a802-ebfacb44cfce)

![изображение](https://github.com/linaduko/mgmt/assets/101510056/4e68db87-1045-4472-aa64-84f1a2e0d422)


4. Возвращаемся к подключению к aon.example.com под пользователем `support`, куда мы сперва загружаем файлы сертификатов.
Файлы которые у нас в папке:
- vrni.key
- vrni.csr
- vrni.cfg
- vrni.cer (индивидуальный сертификат в base64 полученный от запроса)
- root.cer (сертификат CA в base64)
  
5. Выполняем слудующие команды:
```
cat vrni.cer root.cer > finish.cer
sed -i 's/-----BEGIN PRIVATE KEY-----/-----BEGIN RSA PRIVATE KEY-----/g' vrni.key
sed -i 's/-----END PRIVATE KEY-----/-----END RSA PRIVATE KEY-----/g' vrni.key
```
6. Далее необходимо подключиться c помощью SSH к `aon.example.com` под пользователем `consoleuser`.
```
custom-cert remove
custom-cert copy --host aon.example.com --user support --port 22 --path /home/support/finish.cer
custom-cert copy --host aon.example.com --user suopport --port 22 --path /home/support/vrni.key
custom-cert apply
```
## Проверка соответствия ключа и сертификата:

1.Вычисление хэша модуля SSL-сертификата: 
```
openssl x509 -noout -modulus -in vrni.cer | openssl sha256
```
2.Вычисление хэша приватного ключа:
```
openssl rsa -noout -modulus -in vrni.key | openssl sha256
```
3.Расчет хэша модуля CSR: 
```
openssl req -noout -modulus -in vrni.csr | openssl sha256
```
Вывод каждой команды должен быть одинаков и представлять собой строку аналогичную следующей:           
(stdin)= 342bd7490c3b79c83afbb90b3e78dd67
