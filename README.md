# SistemYoneticiligi_Zabbix
# Zabbix Kurulum Komutları

## 1.	Root kullanıcı olun:
### Root ayrıcalıklarıyla yeni bir kabuk oturumu başlatın
$ sudo -s

## 2.	Zabbix deposunu kurun:

 wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
 dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
 apt update

## 3.	Zabbix sunucusunu, arayüzünü ve aracısını kurun:

 apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

## 4.	İlk veritabanını oluşturun: 
### MySQL'e root olarak bağlanın (kurulum sırasında belirlediğiniz parolayı girmeniz istenecektir)

mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;

### Şimdi Zabbix şemasını ve verilerini içe aktarın

 zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

### Veritabanı şemasını içe aktardıktan sonra log_bin_trust_function_creators seçeneğini devre dışı bırakın

mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;

## 5.	Zabbix sunucusu için veritabanını yapılandırın:
Edit file /etc/zabbix/zabbix_server.conf

DBPassword=password

## 6.	Zabbix sunucusu ve aracısı işlemlerini başlatın: 
### Zabbix sunucusunu, aracısını ve Apache'yi yeniden başlatın ve açılışta otomatik başlamalarını sağlayın

 systemctl restart zabbix-server zabbix-agent apache2
 systemctl enable zabbix-server zabbix-agent apache2

## 7.	Zabbix arayüzünü yapılandırın: 
Bir web tarayıcısı açın ve Zabbix sunucunuzun IP adresini veya alan adını girerek Zabbix arayüzüne gidin: http://host/zabbix[1]

