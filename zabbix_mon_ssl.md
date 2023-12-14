Для отслеживания срока годности SSL сертификатов в Zabbix, вы можете использовать утилиту `openssl` в сочетании с пользовательскими скриптами и элементами данных в Zabbix. Вот пример шагов:

1. **Создайте скрипт для проверки срока годности сертификата:**

   Создайте скрипт на вашем сервере, который будет использовать `openssl` для проверки срока годности SSL сертификата. Например, создайте файл `check_ssl_expiry.sh` со следующим содержимым:

   ```bash
   #!/bin/bash

   host="$1"
   port="${2:-443}"

   expiry_date=$(openssl s_client -servername $host -connect $host:$port -showcerts </dev/null 2>/dev/null | openssl x509 -noout -dates | grep notAfter | cut -d= -f2)

   expiry_timestamp=$(date -d "$expiry_date" +%s)
   current_timestamp=$(date +%s)

   days_until_expiry=$(( ($expiry_timestamp - $current_timestamp) / (60 * 60 * 24) ))

   echo $days_until_expiry
   ```

   Убедитесь, что у скрипта есть права на выполнение (`chmod +x check_ssl_expiry.sh`).

2. **Настройте внешний скрипт в Zabbix:**

   В интерфейсе Zabbix перейдите в раздел "Configuration" -> "General" -> "Scripts" и создайте новый скрипт:

   - **Name:** Check_SSL_Expiry
   - **Type:** Script
   - **Script:** Укажите путь к вашему скрипту, например, `/path/to/check_ssl_expiry.sh`

3. **Создайте элемент данных в Zabbix:**

   Перейдите в раздел "Configuration" -> "Hosts" и выберите хост, для которого вы хотите отслеживать SSL сертификат. Далее перейдите на вкладку "Items" и создайте новый элемент данных:

   - **Name:** SSL_Expiry_Days
   - **Type:** External Check
   - **Key:** `script.sh["Check_SSL_Expiry","{HOST.NAME}"]`
   - **Type of Information:** Numeric (unsigned)
   - **Units:** Days
   - Остальные параметры по вашему усмотрению.

4. **Создайте триггер для мониторинга срока годности:**

   Перейдите на вкладку "Triggers" и создайте новый триггер:

   - **Name:** SSL Certificate Expiry
   - **Expression:** `{your_host:script.sh["Check_SSL_Expiry",{HOST.NAME}].last()} < 7` (Это предполагает, что вы хотите получать предупреждение за 7 дней до истечения срока годности. Вы можете настроить это значение в соответствии с вашими предпочтениями.)

   При необходимости, добавьте условия срабатывания триггера.


### 1. Серверные сертификаты на Apache:

Для мониторинга срока годности серверных сертификатов Apache вы можете использовать предложенный ранее скрипт. Однако, вам нужно будет настроить Apache для предоставления информации о сертификате через `mod_status`. Например:

1. Установите и включите `mod_status`:

   ```bash
   sudo a2enmod status
   sudo systemctl restart apache2
   ```

2. Настройте доступ к `mod_status` в вашем конфигурационном файле Apache:

   ```apache
   <Location "/server-status">
       SetHandler server-status
       Require ip 127.0.0.1
   </Location>
   ```

   Это позволит получить информацию о сервере по URL, например, `http://your-server/server-status`.

3. В скрипте проверки срока годности, используйте `openssl` с параметром `-servername` и `SNI`:

   ```bash
   #!/bin/bash

   host="$1"
   port="${2:-443}"

   expiry_date=$(openssl s_client -servername $host -connect $host:$port -showcerts </dev/null 2>/dev/null | openssl x509 -noout -dates | grep notAfter | cut -d= -f2)

   expiry_timestamp=$(date -d "$expiry_date" +%s)
   current_timestamp=$(date +%s)

   days_until_expiry=$(( ($expiry_timestamp - $current_timestamp) / (60 * 60 * 24) ))

   echo $days_until_expiry
   ```

### 2. Клиентские сертификаты в Java:

Для клиентских сертификатов в Java, у вас может быть скрипт, который использует `keytool` для проверки срока годности. Пример:

```bash
#!/bin/bash

keystore_path="/path/to/your/keystore.p12"
keystore_password="your_keystore_password"

expiry_date=$(keytool -list -v -keystore $keystore_path -storetype PKCS12 -storepass $keystore_password | grep 'Valid from' | cut -d' ' -f5-)

expiry_timestamp=$(date -d "$expiry_date" +%s)
current_timestamp=$(date +%s)

days_until_expiry=$(( ($expiry_timestamp - $current_timestamp) / (60 * 60 * 24) ))

echo $days_until_expiry
```

Теперь Zabbix будет мониторить срок годности SSL сертификата для выбранного хоста и уведомлять вас в случае необходимости. Убедитесь, что ваш Zabbix-сервер имеет доступ к исполняемому файлу скрипта и что фаервол разрешает подключение к серверам по указанным вами портам.
