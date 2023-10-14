## Замена сертификата Aria Operations for Networks

1. Необходимо подключиться c помощью SSH к aon.example.com под пользователем `support`.
2. Далее создать конфигурационный файл `vrni.cfg` со следующим содержанием:
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
subjectAltName = DNS: ~current dns-name~

[ req_distinguished_name ]
countryName = CountryName
stateOrProvinceName = StateOrProvinceName
localityName = LocalityName
0.organizationName = OrganizationName
organizationalUnitName = OrganizationUnitName
commonName = ~current dns-name~

```
3. Генерируем ключ и соответствующий запрос сертификата.
```
openssl genrsa -out vrni.key 2048
openssl req -new -key vrni.key -out vrni.csr -config vrni.cfg
```
4. На dc.example.com/certsrv проводим следующие манипуляции (забираем сертификаты в формате **Base64**):

![изображение](https://github.com/linaduko/mgmt/assets/101510056/d0ad1aaf-d8e4-47b0-9084-8f2601c3a834)

![изображение](https://github.com/linaduko/mgmt/assets/101510056/68f0994d-1322-48ba-895b-5bf2ecc6667c)

![изображение](https://github.com/linaduko/mgmt/assets/101510056/c3051701-3653-4634-8a6d-9490ff846e82)

![изображение](https://github.com/linaduko/mgmt/assets/101510056/7179b057-4eaa-44b4-8343-23d7a63211c1)

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
6. Далее необходимо подключиться c помощью SSH к aon.example.com под пользователем `consoleuser`.
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
