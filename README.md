1. Update System Dependencies
Buka command prompt atau terminal dan jalankan perintah berikut pada prompt perintah untuk memperbarui paket ke versi terbaru yang tersedia:
anda dapat menambahkan sudo  jika anda login bukan sebagai root

sudo -s
anda sekarang sudah login sebagai root , ditandai dengan tanda # diawal command prompt

apt update
apt upgrade
Tunggu sampai proses selesai

2. Install Apache
Instal apache di sistem Debian 11, jadi jalankan perintah berikut pada command prompt untuk menginstal apache di sistem Debian 11:

apt install apache2
cek instalasi apache dengan cara ketik IP address dari browser



jika index apache belum bisa terbuka dari browser, kemungkinan masih ada firewall http yang belum di allow, maka perlu dikonfigurasi Firewallnya

3. Setup Firewall
Setelah penginstalan apache selesai, kita perlu menyiapkan Uncomplicated Firewall (UFW) dengan Apache untuk mengizinkan akses publik pada port web default untuk HTTP dan HTTPS

ufw app list
Akan tampil semua aplikasi yang terdaftar.

Output
Available applications:
   Apache
   Apache Full
   Apache Secure
   OpenSSH
Apache: ini membuka port 80 (lalu lintas web normal dan tidak terenkripsi)
Apache Full: ini membuka port 80 (lalu lintas web normal dan tidak terenkripsi) dan port 443 (lalu lintas terenkripsi TLS/SSL)
Apache Secure: ini hanya membuka port 443 (lalu lintas terenkripsi TLS/SSL)
OpenSSH: ini membuka port 22 untuk akses SSH
Jika kita tidak akan menggunakan SSL, kita hanya perlu mengaktifkan profil Apache.

Kemudian aktifkan apache full dengan menggunakan perintah berikut; adalah sebagai berikut:

ufw allow 'Apache Full'
Dengan perintah ini kita bisa melihat status UFW.

sudo ufw status
Akan tampil output

Output
Status: active
 To                         Action      From
 --                         ------      ----
 Apache Full                ALLOW       Anywhere                  
 OpenSSH                    ALLOW       Anywhere                  
 Apache Full (v6)           ALLOW       Anywhere (v6)             
 OpenSSH (v6)               ALLOW       Anywhere (v6)
4. Periksa Instalasi Apache
Setelah Apache terinstal dan konfigurasi firewall sudah selesai, kita bisa mengecek versi Apache menggunakan perintah berikut: adalah sebagai berikut:

apachectl -v


Setiap proses di Apache dikelola dengan perintah systemctl. Periksa status Apache dengan perintah berikut.

systemctl status apache2


5. Instal MariaDB
Instal dan konfigurasikan mysql di Debian 11 dengan menggunakan perintah berikut:

apt-get install mariadb-server -y
Jika instalasi sudah selesai, silahkan start mariadb

systemctl enable mariadb
systemctl start mariadb
Setelah instalasi selesai. Kami dapat memverifikasi bahwa status server MySQL sedang berjalan, ketik:

systemctl status mariadb
Output harus menunjukkan bahwa mysql sudah aktif dan berjalan:



Untuk memeriksa versi mariadb menggunakan perintah berikut:

mysql -V


6. Secure MariaDB
Instalasi MariaDB dilengkapi dengan skrip bernama mysql_secure_installation yang memungkinkan kita meningkatkan keamanan server MySQL dengan mudah.

mysql_secure_installation
Akan diminta untuk mengonfigurasi PLUGIN VALIDATE PASSWORD yang digunakan untuk menguji kekuatan kata sandi pengguna MySQL dan meningkatkan keamanan.

Tekan y jika kami ingin kata sandi validasi atau tombol lain untuk pindah ke langkah berikutnya.

Ada tiga tingkat validasi kata sandi, rendah, sedang, dan kuat. Masukkan 2 untuk validasi kata sandi yang kuat.

Pada prompt berikutnya, akan diminta untuk mengatur kata sandi untuk pengguna root MySQL.

Jika kita menggunakan validasi kata sandi, skrip akan menunjukkan kepada kita kekuatan kata sandi baru kita. Ketik y untuk mengonfirmasi kata sandi.

Selanjutnya, akan diminta untuk menghapus pengguna anonim, membatasi akses pengguna root ke mesin lokal, menghapus database test, dan reload privilege tables.ketik y untuk semua pertanyaan.

7. Install PHP
Instal PHP menggunakan perintah berikut:

apt install php php-cli php-mysql libapache2-mod-php php-gd php-xml php-curl php-common -y
Setelah PHP diinstal selesai, kita dapat menggunakan perintah berikut untuk memeriksa versi php yang diinstal:

php -v


Test instalasi PHPINFO di browser dengan cara ketik:

echo "" > /var/www/html/info.php
Buka dengan Browser dengan cara ketik : IPADDRESS/info.php



8. Konfigurasi PHP
Untuk mengonfigurasi PHP dengan mengubah beberapa variabel di file php.ini

buka file php.ini dengan menggunakan perintah berikut pada command prompt:

nano /etc/php/8.1/apache2/php.ini
Tekan F6 untuk mencari di dalam editor dan perbarui nilai berikut untuk kinerja yang lebih baik.

upload_max_filesize = 32M 
post_max_size = 48M 
memory_limit = 256M 
max_execution_time = 600 
max_input_vars = 3000 
max_input_time = 1000
Setelah kita memodifikasi pengaturan PHP, kita perlu me-restart Apache agar perubahan dapat bekerja.

9.  Konfigurasikan Apache
Nonaktifkan konfigurasi Apache default.

a2dissite 000-default
Buat direktori web.

mkdir -p /var/www/html/domainname/public
Berikan hak akses.

chmod -R 755 /var/www/html/domainname
chown -R www-data:www-data /var/www/html/domainname
Buat konfigurasi host virtual baru.

sudo nano /etc/apache2/sites-available/domainname.conf
paste ini di dalam file konfigurasi

     ServerAdmin admin@domainname.com
     ServerName domainname.com
     ServerAlias www.domainname.com

     DocumentRoot /var/www/html/domainname/public

     
         Options Indexes FollowSymLinks
         AllowOverride All
         Require all granted
     

     ErrorLog ${APACHE_LOG_DIR}/error.log 
     CustomLog ${APACHE_LOG_DIR}/access.log combined 
 
Enable Konfigurasi virtualhost

sudo a2ensite domainname.conf
10. Install PhpMyAdmin
Gunakan perintah berikut untuk menginstal PHPMyAdmin:

sudo apt install phpmyadmin
Konfigurasi phpmyadmin untuk Apache.

sudo cp /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
Enable Konfigurasi virtualhost

sudo a2enconf phpmyadmin.conf
Kemudian jalankan perintah berikut pada command prompt untuk restart apache web server:

sudo service apache2 restart
Proses Instalasi PHPMyadmin selesai, silahkan akses dengan url htt://yourdomain.com/phpmyadmin.

11. Install Let’s Encrypt SSL
HTTPS adalah protokol untuk komunikasi aman antara server (instance) dan klien (browser web). Let’s Encrypt, menyediakan sertifikat SSL gratis, HTTPS diadopsi oleh semua orang dan juga memberikan kepercayaan kepada pengunjung website anda.

sudo apt install python3-certbot-apache
Tunggu proses instalasi Certbot by Let's Encrypt sampai proses selesai. lalu, jalankan perintah ini untuk konfigurasi dan request sertifikat SSL.

sudo certbot --apache --agree-tos --redirect -m youremail@email.com -d domainname.com -d www.domainname.com
Perintah ini akan menginstal SSL Gratis, konfigurasi redirect HTTP ke HTTPS, dan restart web server Apache.

12. Renewing SSL Certificate
Sertifikat yang disediakan oleh Let’s Encrypt hanya berlaku selama 90 hari, jadi Anda harus memperbaruinya setiap 3 bulan sekali. Jadi, mari kita coba fitur pembaruan menggunakan perintah berikut.

sudo certbot renew --dry-run
