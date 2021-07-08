## Установите Hashicorp Vault в виртуальной машине Vagrant/VirtualBox

```
$ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -

$ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"

$ sudo apt-get update && sudo apt-get install vault
```

---

## Запустить Vault-сервер в dev-режиме

```
$ vault server -dev -dev-listen-address="0.0.0.0:8200"

# в новом окне, в котором выполнялись остальные команды

$ export VAULT_ADDR='http://127.0.0.1:8200'
```

---

##  Создайте Root CA и Intermediate CA

root CA:
```
$ vault secrets enable pki

$ vault write -field=certificate pki/root/generate/internal common_name="example.com" ttl=87600h > CA_cert.crt

$ vault write pki/config/urls issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"
```

intermediate CA:
```
$ vault secrets enable -path=pki_int pki

$ sudo apt install jq

$ vault write -format=json pki_int/intermediate/generate/internal common_name="example.com Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr

$ vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > intermediate.cert.pem

$ vault write pki_int/intermediate/set-signed certificate=@intermediate.cert.pem
```

---

## Подпишите Intermediate CA csr на сертификат для тестового домена netology.example.com

Сначала создадим роль example-dot-com
```
$ vault write pki_int/roles/example-dot-com allowed_domains="example.com" allow_subdomains=true max_ttl="720h"
```

Потом подпишем сертификат
```
$ vault write pki_int/issue/example-dot-com common_name="netology.example.com" ttl="24h"
```

---

## Поднимите на localhost nginx, сконфигурируйте default vhost для использования подписанного Vault Intermediate CA сертификата и выбранного вами домена.

```
$ sudo apt install nginx -y
```

Результат команды из предыдущего пункта я разнез по файлам `netolog.key` и `netology.crt`, поместив соджержимым соответсвующие секции.

После этого, в дефолтный конфиг nginx добавил следующие секции:
```
listen 443 ssl default_server;
listen [::]:443 ssl default_server;

ssl_certificate /home/vagrant/netology.crt;
ssl_certificate_key /home/vagrant/netology.key;
```

---

## Добейтесь безошибочной с точки зрения HTTPS работы curl на ваш тестовый домен.

```
# Добавил запись в /etc/hosts:
$ echo 127.0.0.1 netology.example.com >> /etc/hosts

# Добавил сертификат в доверенные:
$ ln -s ./netology.crt /usr/local/share/ca-certificates/netology.crt

# Обновил системный trust-store
$ update-ca-certificates

# Вывод сказал нам что все ОК
> Updating certificates in /etc/ssl/certs...
> rehash: warning: skipping netology.pem,it does not contain exactly one certificate or CRL
> 1 added, 0 removed; done.
> Running hooks in /etc/ca-certificates/update.d...
> done.

# Проверяем, что все заработало:
$ curl -I https://netology.example.com | head -n1

# Вывод сменился с curl: (60) SSL certificate problem: unable to get local issuer certificate
# На нормальный пацанский:
> HTTP/1.1 200 OK
```

---

## Ознакомьтесь с протоколом ACME и CA Let's encrypt

Ознакомился)))
К сожалению, у меня сейчас нет зарегестрированных доменных имен. Но, с Let's Encrypt я уже работал, сертификаты наворачивал.
