### **Установка Nginx**

Учитывая, что ранее мы установили LAMP, 80-ый порт, на котором работает Nginx, уже занят веб-сервером Apache, поэтому нужно сперва его остановить и отключить автозапуск командой:  
**Ubuntu и Debian:**  

```
systemctl stop apache2 && systemctl disable apache2
```

**CentOS:**

```
systemctl stop httpd && systemctl disable httpd
```

Затем устанавливаем веб-сервер Nginx:

-   **Ubuntu и Debian:**
    
    ```
    apt update && apt -y install nginx
    ```
    
-   **CentOS:**
    
    ```
    yum -y install nginx
    ```
    

После установки проверим, что веб-сервер запустился. 

```
systemctl status nginx
```

На Ubuntu и Debian службы после установки запускаются и добавляются в автозапуск автоматически, на Centos это нужно сделать вручную:  

```
systemctl enable nginx
```

Чтобы проверить, что веб-сервер заработал и может обрабатывать запросы к серверу, введите IP-адрес вашего сервера в адресной строке браузера. Откроется приветственная страница Apache, и это нормально, так как до этого у нас был установлен Apache  
  
**Установка менеджера процессов PHP-fpm**  
  
Установка производится командой:

-   **Ubuntu и Debian:**
    
    ```
    apt -y install php-fpm
    ```
    
-   **CentOS:**
    
    ```
    yum -y install php-fpm
    ```
    

После установки запускаем менеджер процессов **php-fpm** и добавляем его в автозагрузку:

-   **Ubuntu и Debian (данный пример на Debian 10):**
    
    ```
    systemctl start php7.3-fpm && systemctl enable php7.3-fpm
    ```
    
-   **CentOS:**
    
    ```
    systemctl start php-fpm && systemctl enable php-fpm
    ```
    

После установки нужно отредактировать настройки **php-fpm** по умолчанию, предназначенные для веб-сервера Apache:

-   **Ubuntu и Debian (данный пример на Debian 10):**
    
    ```
    nano /etc/php/7.3/fpm/pool.d/www.conf
    ```
    
-   **CentOS:**
    
    ```
    vim /etc/php-fpm.d/www.conf
    ```
    

В файле ищем блок кода **Unix user/group of processes** и меняем **apache** на **www-data** для Debian и Ubuntu и **nginx** для CentOS.

Открываем для редактирования конфигурационный файл PHP:

-   **Ubuntu и Debian (данный пример на Debian 10):**
    
    ```
    nano /etc/php/7.3/fpm/php.ini
    ```
    
-   **CentOS:**
    
    ```
    vim /etc/php.ini
    ```
    

В файле ищем раздел **Paths and Directories** (он почти в самом конце файла), внутри находим параметр **cgi.fix_pathinfo**. Нужно раскомментировать его (удалить «;» в начале строки) и изменить значение с «**1**» на «**0**».

После этого сохраняем файл и перезапускаем веб-сервер, чтобы новые настройки применились:

-   **Ubuntu и Debian (данный пример на Debian 10):**
    
    ```
    systemctl reload nginx && systemctl reload php7.3-fpm
    ```
    
-   **CentOS:**
    
    ```
    systemctl reload nginx && systemctl reload php-fpm
    ```
    

### **Настройка базового конфигурационного файла для сайта**

1.  Удаляем конфигурационный файл **default**, использующийся по умолчанию, и создаём его замену, **default.conf**:

**Ubuntu и Debian:**

```
unlink /etc/nginx/sites-enabled/default
touch /etc/nginx/sites-available/default.conf
ln /etc/nginx/sites-available/default.conf /etc/nginx/sites-enabled/default.conf
```

**CentOS:**

```
touch /etc/nginx/conf.d/default.conf
```

1.  Откроем новый файл в консольном текстовом редакторе и добавим туда содержимое:

**Ubuntu и Debian:**

```
nano /etc/nginx/sites-available/default.conf
```

**CentOS:**

```
vim /etc/nginx/conf.d/default.conf
```

1.  Скопируйте в файл следующий блок настроек:

**Ubuntu и Debian:**

```
server {
  listen 80;
  server_name _;
  root /var/www/html/;
  index index.php index.html index.htm;

  location / {
    try_files $uri $uri/ =404;
  }

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php7.3-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
  }

  location ~ /\.ht {
      access_log off;
      log_not_found off;
      deny all;
  }
}
```

**CentOS:**

```
server {
  listen 80;
  server_name _;
  root /var/www/html/;
  index index.php index.html index.htm;

  location / {
    try_files $uri $uri/ =404;
  }

  location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

  location ~ /\.ht {
      access_log off;
      log_not_found off;
      deny all;
  }
}
```

Осталось проверить, что в конфигурационном файле отсутствуют ошибки, и перезапустить nginx для применения настроек:

```
nginx -t
systemctl reload nginx
```

Проверим работу веб-сервера и php-обработчика, создав файл с php info.

Для начала перейдите в корневую директорию вашего сайта. Если сайта ещё нет, по умолчанию это каталог **/var/www/html**. Создайте файл с именем **info.php** и откройте его в любом консольном текстовом редакторе:

```
touch /var/www/html/info.php
nano /var/www/html/info.php
```

  
В файл вставьте следующую конструкцию:

```
<?php
phpinfo();
?>
```

Теперь при переходе по ссылке типа **http://IP-адрес-сервера/info.php** вы увидите полный список параметров PHP на вашем сервере.

Не забудьте удалить этот файл после того, как полноценно развернете сайт и убедитесь, что сайт работает. Если оставить его в открытом доступе, информация из него может быть использована злоумышленниками для атак на ваш сайт.

```
rm /var/www/html/info.php
```