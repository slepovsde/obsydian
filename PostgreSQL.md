
# Обновляем репозитории:
<pre>
apt-get update
</pre>
# Проверяем версию postgresql которая будет установлена (должна быть 9.6):
<pre>
apt-cache policy postgresql
</pre>
# Установка происходит командой:
<pre>
apt-get install postgresql postgresql-client
</pre>

h2. Базовая настройка

h3. Настройка верной locale

PostgreSQL работая с локалью ru_RU.UTF-8(которая обычно стоит по умолчанию в ОС) ведет лог-файлы на русском языке, для ведения логов на английском необходимо перевести её работу на локаль en_US.UTF-8(для более удобного чтения логов и ошибок PostgreSQL).

p(important). Внимание! Данную операцию можно производить только сразу после установки PostgreSQL, т.к. при ней будут удалены изменённые уже настройки в PostgreSQL.


* Подключаем необходимую локаль _en_US.UTF-8_, если она ещё не включена.
        
# Редактируем файл _/etc/locale.gen_, добавляя в него, или расскоментируя строку:
 <pre>
 en_US.UTF-8 UTF-8
 </pre>
#  Генерируем локали из файла шаблона(_/etc/locale.gen_)
<pre>
 locale-gen
 </pre>
#  Можем проверить выбранные локали на сервере:
 <pre>
 locale -a
 </pre>
* Останавливаем созданный по умолчанию кластер БД:
<pre>
    pg_dropcluster --stop 9.6 main
</pre>
где (упрощённо, более "подробно":https://wiki.debian.org/ru/PostgreSql):
* 9.6 - это имя каталога /etc/postgresql/9.4
* main - это имя каталога /etc/postgresql/9.4/main

# Создаём новый кластер, с настройками по умолчанию и нужной нам локалью:
<pre>
pg_createcluster --locale en_US.UTF-8 --start 9.6 main
</pre>
# После этого все сообщения от PostgreSQL будут на корректном, английском языке.

h2. Создание root пользователя

Для сохранения общей логики работы с БД, нам было бы желательно работать с PostgreSQL из под пользователя root.
Т.к. он изначально не создаётся при установке PostgreSQL - нам необходимо его создать:

* Создание возможно только из под пользователя postgres(который является суперпользователем по умолчанию)

 <pre>
 su - postgres
 createuser --superuser --pwprompt --encrypted root
 </pre>

 где:
* --pwprompt - назначить пароль новой роли
* --encrypted - зашифровать сохранённый пароль
* --superuser - роль с полномочиями суперпользователя

* Выйдем из под пользователя postgres
<pre>
exit
</pre>

* Далее мы можем проверить корректность подключения под новым пользователем root, полной командой:
<pre>
psql -U root -W -d postgres
</pre>
где:

* -W, --password - запрашивать пароль всегда (обычно не требуется)
* -U, --username=ИМЯ - имя пользователя (по умолчанию: "root")
* -d, --dbname=БД - имя подключаемой базы данных (по умолчанию "root")

* Либо её более простым вариантом:

<pre>
psql -d postgres
</pre>

* Настраиваем автоматическое подключения к БД:
** Создаем файл /root/.pgpass, следующего содержания:
<pre>
IP:PORT:DATABASE:DB_USER:PASSWORD
</pre>
где:

* _IP_ - _IP-адрес_ который слушает _PostgresSQL_
* _PORT_ - порт который слушает _PostgresSQL_ (по умолчанию: 5432)
* _DATABASE_ - имя подключаемой базы данных (по умолчанию "postgres")
* _DB_USER_ - пользователь базы данных (по умолчанию "root")
* _PASSWORD_ - пароль пользователя
    Для всех вариантов возможно использования маски "все" - *.
    _Пример:_
<pre>
*:5432:postgres:root:<PASSWD>
</pre>
** Устанавливаем права на файл /root/.pgpass:
<pre>
chown root: /root/.pgpass
chmod 600 /root/.pgpass
</pre>

*Настройка переменной search_path*

* Добавляем в переменную _search_path_ схему _data_ в файле _postgresql.conf_:

# - Statement Behavior -

%{background:ping}-#search_path = '"$user",public'                     # schema names%
+search_path = '"$user",data,public'            # schema names
 #default_tablespace = ''               # a tablespace name, '' uses the default
 #temp_tablespaces = ''                 # a list of tablespace names, '' uses
                                        # only default tablespace
