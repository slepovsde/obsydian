https://adw0rd.com/2009/6/15/sphinxsearch/
http://sphinxsearch.com/about/sphinx/

Sphinx — это сервер полнотекстового поиска с открытым исходным кодом, разработанный с нуля с учетом производительности, релевантности (то есть качества поиска) и простоты интеграции. Он написан на C++ и работает в Linux (RedHat, Ubuntu и т. д.), Windows, MacOS, Solaris, FreeBSD и некоторых других системах.

Sphinx позволяет быстро и легко пакетно индексировать и искать данные, хранящиеся в базе данных SQL, хранилище NoSQL или только в файлах, или индексировать и искать данные на лету, работая со Sphinx практически так же, как с сервером базы данных.

Разнообразные функции обработки текста позволяют точно настроить Sphinx для ваших конкретных требований к приложению, а ряд функций релевантности позволяют также настроить качество поиска.

Поиск через SphinxAPI так же прост, как 3 строки кода, а запрос через SphinxQL еще проще, с поисковыми запросами, выраженными в старом добром SQL.

1.  First, add Sphinxsearch repository and update the list of packages:
    
    **`$ sudo add-apt-repository ppa:builds/sphinxsearch-rel22`**
    
    **`$ sudo apt-get update`**
    
2.  Install/update sphinxsearch package:
    
    **`$ sudo apt-get install sphinxsearch`**
    

Sphinx `searchd` daemon can be started/stopped using service command:

**`$ sudo service sphinxsearch start`**


