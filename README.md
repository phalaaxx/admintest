1. Хардуер
==========

Проблеми с хардуера / извършени действия:

	* 12V захранване за процесора не е включено в дъното / 12V захранване включено в дъното
	* SATA захранване за SSD диска не е включено / SATA захранване включено в SSD
	* IDE кабел за DVD ROM не е включен / IDE кабел включен в DVD ROM


2. Софтуер
==========

Стартиране на инсталирания Debian
---------------------------------

Проблеми с bootloader:

	* При стартиране машината стига до конзола в initramfs

Извършени действия:

	* Машината се стартира със boot CD (USB Flash) във rescue mode
	* От rescue mode се стартира shell в root дяла (/dev/sda1) на твърдия диск
	* В конфигурацията на lilo се вижда следния проблем: root = /dev/sda2
	* Проблема се отстранява (root = /dev/sda1)
	* Стартира се lilo за да се запазят промените
	* Сървъра се рестартира


Липса на парола за root:
------------------------

При стартиране по време на менюто на Lilo на командния ред се изписва: Linux init=/bin/bash. Това отваря bash шел, но root дяла е монтиран read-only. За да се смени паролата се изпълняват следните команди:

	mount -o remount,rw /
	passwd
	mount -o remount,ro

След което машината може да се рестартира. Новата парола е *i6ChboiL5kgJPyp*.


Проблеми с мрежовите настройки
------------------------------

Инсталацията имаше следните проблеми с мрежовите настройки:

	* /etc/resolv.conf е с грешно съдържание и защитен от презаписване
	* Мрежовата маска в /etc/network/interfaces е неправилна за посочените IP address и default gateway

За да бъдат отстранени проблемите, извършените действия са:

	* Позволено е презаписване на /etc/resolv.conf (chattr -i /etc/resolv.conf)
	* Отстранена е грешката в /etc/resolv.conf (name server -> nameserver)
	* Променeна е мрежовата маска (netmask) от 255.255.255.128 на 255.255.255.0 във /etc/network/interfaces.



4. Инсталиране на ядро 3.11.5 с grsecurity
==========================================

Подготовка за инсталация
------------------------

Извършени действия:

	* Тегли се [linux-3.11.5.tar.xz](http://ftp.kernel.org/pub/linux/kernel/v3.x/linux-3.11.5.tar.xz)
	* Тегли се [grsecurity-2.9.1-3.11.5-201310162041.patch](http://grsecurity.net/test/grsecurity-2.9.1-3.11.5-201310162041.patch)
	* Със следните команди се прилагат grsecurity пачовете:
		* cd /usr/src
		* wget http://ftp.kernel.org/pub/linux/kernel/v3.x/linux-3.11.5.tar.xz
		* wget http://grsecurity.net/test/grsecurity-2.9.1-3.11.5-201310162041.patch
		* tar Jxf linux-3.11.5.tar.xz
		* cd linux-3.11.5
		* patch -p1 < ../grsecurity-2.9.1-3.11.5-201310162041.patch


За да се компилира ядрото е инсталиран следния допълнителен софтуер:

	apt-get install libncurses5-dev gcc-4.7-plugin-dev


Конфигуриране на ядрото
-----------------------

Поради големия брой настрой, необходими за конфигуриране на ядрото, и липсата на достатъчно време при този тест,
конфигурацията няма да бъде описана в този документ. За справка може да се провери резултата във /usr/src/linux-3.11.5/.config или /proc/config.gz.

Компилиране на ядрото
---------------------

За компилиране се използва следната команда:

	make bzImage -j3

Броя нишки, с които се компилира е избран да бъде N+1 (N-брой ядра на процесора)


Инсталиране на ядрото:
----------------------

Във файла /etc/lilo.conf се добавя конфигурация за ядрото:

	image=/boot/linux-3.11.5-grsec..wo.ms
		label=L3.11.5
		read-only
		append="root=/dev/sda1 quiet"

Копира се ядрото в /boot:

	cp /usr/src/linux-3.11.5/arch/x86/boot/bzImage /boot/linux-3.11.5-grsec..wo.ms
	lilo

След което може да се рестартира машината за да се зареди новото ядро.


5. Инсталиране на apache 2.4.6
==============================

За инсталиране на apache web server от sources е необходимо да направим следното:

	* да изтеглим source кода на apache
	* да го конфигурираме
	* да го компилираме
	* да го инсталираме


Първо трябва да се инсталират dev библиотеки, необходими за компилирането на apache:

	apt-get install libpcre3-dev


Командите, за инсталиране на apache са следните:


	groupadd -r -g 500 apache
	useradd -r -u 500 -g 500 -d /usr/local/apache2.4.6 apache

	mkdir /usr/src/apache
	cd /usr/src/apache
	wget http://apache.cbox.biz/httpd/httpd-2.4.6.tar.bz2
	wget http://apache.cbox.biz/apr/apr-1.4.8.tar.bz2
	wget http://apache.cbox.biz/apr/apr-util-1.5.2.tar.bz2
	tar jxf httpd-2.4.6.tar.bz2
	cd httpd-2.4.6/srclib
	tar jxf ../../apr-1.4.8.tar.bz2
	tar jxf ../../apr-util-1.5.2.tar.bz2
	ln -s apr-1.4.8 apr
	ln -s apr-util-1.5.2 apr-util
	cd ..
	./configure \
		 --prefix=/usr/local/apache2.4.6 \
		 --enable-mods-shared=all \
		 --with-mpm=prefork \
		 --with-included-apr \
		 --enable-cgi \
		 --enable-suexec \
		 --with-suexec-userdir=public_html \
		 --with-suexec-docroot=/home \
		 --with-suexec-uidmin=1000 \
		 --with-suexec-gidmin=100 \
		 --with-suexec-caller=apache
	make -j3
	make install

	mkdir /usr/local/apache2.4.6/conf/vhosts
	mkdir -p /var/log/apache2.4.6/users


Добавя се следната конфигурация в /usr/local/apache2.4.6/conf/httpd.conf.

	Include conf/vhosts/*.conf
	#AddType application/x-httpd-php .php
	LoadModule suexec_module modules/mod_suexec.so
	LoadModule cgi_module modules/mod_cgi.so
	AddHandler cgi-script .cgi
	AddHandler cgi-script .php .php3 .php4 .php5 .phtml
	<Directory /home>
		Order allow,deny
		Allow from all
		DirectoryIndex index.html index.htm index.php
		Options +Indexes +ExecCGI
		Require all granted
	</Directory>




6. Инсталиране на MySQL 5.6.14
==============================

Инсталират се необходимите за компилиране пакети:

	apt-get install cmake


Командите за инсталиране на MySQL от sources са следните:

	mkdir /usr/src/mysql
	cd /usr/src/mysql
	wget -O mysql-5.6.14.tar.gz http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.14.tar.gz/from/http://cdn.mysql.com/
	tar zxf mysql-5.6.14.tar.gz
	cd mysql-5.6.14
	cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql5.6.14 .
	make -j3
	make install

	groupadd -r -g 501 mysql
	useradd -r -u 501 -g 501 -d /usr/local/mysql5.6.14 mysql

	cd /usr/local/mysql5.6.14
	./scripts/mysql_install_db

	chown -R mysql: /usr/local/mysql5.6.14

	cat << EOF > /root/.my.cnf
	[mysql]
	user=root
	password=B8Rcx3970lNDEqx
	EOF

Забележка: Паролата за потребител root в MySQL ще бъде сменена след като се стартира MySQL.


7. Инсталиране на PHP 5.5.4
===========================

Инсталират се необходимите за компилиране пакети:

	apt-get install libxml2-dev


Командите за инсталиране на PHP са следните:

	mkdir /usr/src/php
	cd /usr/src/php
	wget -O php-5.5.4.tar.bz2 http://bg2.php.net/get/php-5.5.4.tar.bz2/from/this/mirror
	tar jxf php-5.5.4.tar.bz2
	cd php-5.5.4
	./configure \
		 --prefix=/usr/local/php5.5.4 \
		 --with-apxs2=/usr/local/apache2.4.6/bin/apxs \
		 --with-mysql=/usr/local/mysql5.6.14

	make -j3
	make install

	echo "cgi.force_redirect=0" > /usr/local/php5.5.4/lib/php.ini


При тази конфигурация (без suphp) е необходимо .php файловете да бъдат изпълними (chmod 705) за да се обработват правилно от apache, suexec и php.
Необходимо е 

Следните редове е необходимо да се изпълняват след boot на машината, например в /etc/rc.local:

	mount binfmt_misc -t binfmt_misc /proc/sys/fs/binfmt_misc
	echo ':PHP:E::php::/usr/local/php5.5.4/bin/php-cgi:' > /proc/sys/fs/binfmt_misc/register


8. Инсталиране на допълнителен софтуер
======================================

daemontools
-----------

Инсталация на daemontools:

	apt-get install daemontools daemontools-run

За стартиране на apache и mysql в случая се използва daemontools.
Apache:

	mkdir -p /usr/local/apache2.4.6/sv/httpd/log/main
	chown nobody /usr/local/apache2.4.6/sv/httpd/log/main

	cat << EOF > /usr/local/apache2.4.6/sv/httpd/run
	#!/bin/sh
	exec 2>&1
	exec /usr/local/apache2.4.6/bin/httpd -D FOREGROUND
	EOF

	cat << EOF > /usr/local/apache2.4.6/sv/httpd/log/run
	#!/bin/sh
	exec setuidgid nobody multilog t ./main
	EOF

	chmod 700 /usr/local/apache2.4.6/sv/httpd/{,log/}run
	ln -s /usr/local/apache2.4.6/sv/httpd /etc/service

MySQL:

	mkdir -p /usr/local/mysql5.6.14/sv/mysql/log/main
	chown nobody /usr/local/mysql5.6.14/sv/mysql/log/main
	
	cat << EOF > /usr/local/mysql5.6.14/sv/mysql/run
	#!/bin/sh
	exec 2&>1
	exec /usr/local/mysql5.6.14/bin/mysqld_safe
	EOF

	cat << EOF > /usr/local/mysql5.6.14/sv/mysql/log/run
	#!/bin/sh
	exec setuidgid nobody multilog t ./main
	EOF

	chmod 700 /usr/local/mysql5.6.14/sv/mysql/{,log/}run
	ln -s /usr/local/mysql5.6.14/sv/mysql /etc/service
	/usr/local/mysql5.6.14/bin/mysqladmin -u root password B8Rcx3970lNDEqx


vsftpd
------

Инсталация:

	apt-get install vsftpd

Конфигурация (показани са само променени/добавени редове) във /etc/vsftpd.conf:

	* anonymous_enable=NO
	* local_enable=YES
	* write_enable=YES
	* local_umask=072
	* chroot_local_user=YES


9. Скрипт за създаване на потребители
=====================================

	#!/bin/bash

	# $1 - username
	# $2 - password
	# $3 - vhost

	USER=$1
	PASS=$2
	VHOST=$3

	useradd -m -g users -s /bin/bash $USER
	echo -n "$USER:$PASS" | chpasswd -c SHA512

	# create apache configuration for user
	cat << EOF > /usr/local/apache2.4.6/conf/vhosts/$USER.conf
	<VirtualHost *:80>
		ServerName $VHOST
		DocumentRoot /home/$USER/public_html
		SuexecUserGroup "$USER" "users"
		CustomLog /var/log/apache2.4.6/users/$USER-access.log combined
		ErrorLog /var/log/apache2.4.6/users/$USER-error.log
	</VirtualHost>
	EOF

	# reload apache configuration
	/usr/local/apache2.4.6/bin/apachectl graceful

	echo -en "create database $USER;" | /usr/local/mysql5.6.14/bin/mysql
	echo -en "create user '$USER'@'localhost' identified by '$PASS';" | /usr/local/mysql5.6.14/bin/mysql
	echo -en "grant all privileges on $USER.* to '$USER'@'localhost';" | /usr/local/mysql5.6.14/bin/mysql




10. Допълнителни настройки
==========================

	/etc/login.defs            UMASK=072
	/etc/adduser.conf          DIR_MODE=0701
	/etc/security/limits.conf  @users hard nproc 25
	                           @users hard cpu 5
	                           @users hard nofile 205
	/etc/ssh/sshd_config       PermitRootLogin without-password


Настройка на iptables:
----------------------

Инсталира се iptables-persistent:

	apt-get install iptables-persistent

Настройват се и се запазват правила за OUTPUT за потребителите в група users:

	iptables -N untrusted
	iptables -A OUTPUT -m owner --gid-owner users -j untrusted
	iptables -A untrusted -p tcp -m multiport --dports 53,80,443 -j ACCEPT
	iptables -A untrusted -p udp --dport 53 -j ACCEPT
	iptables -A untrusted -j REJECT
	/etc/init.d/iptables-persistent save


/etc/skel
---------

Създаваме необходими файлове/директории в /etc/skel:

	mkdir /etc/skel/public_html

	find /etc/skel -type d -exec chmod 705 '{}' \;
	find /etc/skel -type f -exec chmod 604 '{}' \;




A. Troubleshooting
==================

admin-test.pl
-------------

Първо се налага да се инсталира perl за да може да се изпълни скрипта:

	apt-get install perl

Версията на perl, която е инсталирана и версията, посочена в shebang на скрипта са различни; по тази причина трябва да се промени shebang например на "#!/usr/bin/perl" или скрипта да се вика с "perl admin-test.pl".

Следващ проблем е липсата на модули DateTime File::stat за perl:

	apt-get install libdatetime-perl libpath-class-file-stat-perl

Във файла трябва да се добави "use File::stat;", както и да се промени -&gt;mtime на -&gt;[9]; Освен това $m-&gt;[9] трябва да стане на $m{$i}-&gt;[9]. Резултата е:

	#!/usr/bin/perl

	#
	# Sorts files by modification time and prints the filename and the modification time.
	#

	use DateTime;
	use File::stat;

	@files = ("/etc/passwd", "/etc/fstab", "/bin/bash");

	%m = ();

	foreach $file (@files){
		$m{$file}{'s'} = stat($file);
	}

	foreach $i (sort { $m{$a}{'s'}->[9] <=> $m{$b}{'s'}->[9]} keys %m){
		$dt = DateTime->from_epoch( epoch => $m{$i}{'s'}->[9] );
		printf("%s mtime is %s %s\n", $i, $dt->ymd, $dt->hms );
	}



admin-test
----------

При стартираме на програмата без аргументи и strace - резултата е segfault. При пускане на strace откриваме, че програмата се опитва да създаде временен файл в директория /tmp/admin-test, която не съществува.

Създаване на тази директория не променя много крайния резултат.
Програмата променя лимита на максималното количество памет, което може да заема (address space) на soft=1024 и hard=2048, след което се опитва да задели значително повече памет - mmap със 104861696, 104992768, 2097152, 1048576. В резултат получава сигнал SIGSEGV.
