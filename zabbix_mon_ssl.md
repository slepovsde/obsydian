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

Теперь Zabbix будет мониторить срок годности SSL сертификата для выбранного хоста и уведомлять вас в случае необходимости. Убедитесь, что ваш Zabbix-сервер имеет доступ к исполняемому файлу скрипта и что фаервол разрешает подключение к серверам по указанным вами портам.
