title: Connect ke MySQL melalui SSL/TLS
date: 2015-05-31 15:44:24
tags:
    - ssl
    - php
    - mysql
category: DevOps
permalink: connect-mysql-melalui-ssl-tls.html
summary: >
            Panduan ringkas untuk sambungan ke MySQL melalui SSL/TLS.
---

Mulakan dengan menjana certificate dalam folder `/etc/ssl/mysql`:-

```
# Generate a CA key and certificate with SHA1 digest

openssl genrsa 2048 > ca-key.pem;
openssl req -sha1 -new -x509 -nodes -days 3650 -key ca-key.pem > ca-cert.pem;

# Create server key and certficate with SHA1 digest, sign it and convert
# the RSA key from PKCS #8 (OpenSSL 1.0 and newer) to the old PKCS #1 format

openssl req -sha1 -newkey rsa:2048 -days 3650 -nodes -keyout server-key.pem > server-req.pem;
openssl x509 -sha1 -req -in server-req.pem -days 3650 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 > server-cert.pem;
openssl rsa -in server-key.pem -out server-key.pem;

# Create client key and certificate with SHA digest, sign it and convert
# the RSA key from PKCS #8 (OpenSSL 1.0 and newer) to the old PKCS #1 format

openssl req -sha1 -newkey rsa:2048 -days 3650 -nodes -keyout client-key.pem > client-req.pem;
openssl x509 -sha1 -req -in client-req.pem -days 3650 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 > client-cert.pem;
openssl rsa -in client-key.pem -out client-key.pem;
```

Sebelum kita enable SSL pada server ini, tambah satu user baru yang perlukan ssl untuk berhubung. Jika user tersebut cuba connect tanpa menggunakan ssl, mysql akan *reject* connection tersebut.

```
mysql> create database test_ssl;
Query OK, 1 row affected (0.00 sec)

mysql> grant all privileges on test_ssl.* to 'test_user'@'%' identified by 'testpass' require ssl;
Query OK, 0 rows affected (0.10 sec)
```

Kemudian, tambahkan konfigurasi berikut pada fail konfigurasi mysql (`/etc/mysql/conf.d/ssl.cnf`):-

```
[client]
ssl-cert=/etc/mysql/client-cert.pem
ssl-key=/etc/mysql/client-key.pem

[mysqld]
ssl-ca=/etc/mysql/ca-cert.pem
ssl-cert=/etc/mysql/server-cert.pem
ssl-key=/etc/mysql/server-key.pem
```

Restart MySQL server:-

```
sudo service mysql restart
```

Untuk menguji sambungan SSL ini menggunakan PHP, gunakan kod berikut:-

```php
<?php
ini_set ('error_reporting', E_ALL);
ini_set ('display_errors', '1');
error_reporting (E_ALL|E_STRICT);

$db = mysqli_init();
mysqli_options ($db, MYSQLI_OPT_SSL_VERIFY_SERVER_CERT, true);

$db->ssl_set('/etc/ssl/mysql/client-key.pem', '/etc/ssl/mysql/client-cert.pem', '/etc/ssl/mysql/ca-cert.pem', NULL, NULL);
$link = mysqli_real_connect ($db, '10.0.3.154', 'test_user', 'testpass', 'test_ssl', 3306, NULL, MYSQLI_CLIENT_SSL);
if (!$link)
{
    die ('Connect error (' . mysqli_connect_errno() . '): ' . mysqli_connect_error() . "\n");
} else {
    $res = $db->query('SHOW TABLES;');
    print_r ($res);
    $db->close();
}
```

Anda seterusnya boleh menyalin (copy) fail `client-key.pem`, `client-cert.pem` dan `ca-cert.pem` ke mana-mana host yang perlukan sambungan MySQL melalui SSL.

## Rujukan:-
* http://www.madirish.net/214
* http://serverfault.com/questions/622317/mysql-ssl-bad-other-signature-confirmation
* http://stackoverflow.com/questions/9738712/connect-to-remote-mysql-server-with-ssl-from-php
