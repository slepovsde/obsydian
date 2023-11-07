**Список процессов в долгой обработке по времени:**  

cat access.log | grep 'time' | awk '{print $7 " " $9}'| sort -rnk2 | head -25

**Список процессов в долгой обработке за последние 10 000 запросов:**  

cat access.log | tail -n 10000 | awk '{print $7 " " $9}'| sort -rnk2 | head -25

**Список наиболее активных IP адресов за последнии 10 000 запросов:**  

cat access.log | tail -10000 | awk '{print $1}'| sort -nr | uniq -c | sort -nr | head

**Список активных ботов:**  

cat access.log | tail -10000 |awk '($15 ~ /[Bb]ot/)' | awk '{print $5, $15}' |  sort -nr | uniq -c | sort -nr | head

**Вывести самые тяжелые директории:**  

du -hsx * | sort -rh | head -10
du -h . | sort -n -r | head -n 20
du -k . | sort -n | tail -5

**Вывести IP адреса с соединениями в mysql:**

mysql -BNe "show processlist;"|awk '{print $3}'|awk -F":" '{print $1}'|sort|uniq -c

**Список всех crontab задач пользователей:**  

cat /var/spool/cron/crontabs/ 

**Отсортированный список процессов по использованию ресурсов процессора :**  

ps -eo pid,ppid,%mem,%cpu,command --sort=-%cpu | head

Посмотреть количество процессов:  

**ps -aux | awk '{print $11}'| sort -nr | uniq -c | sort -nr | head**

### Memory

Отсортированный список процессов по использованию ресурсов оперативной памяти:  

ps -eo pid,ppid,%mem,%cpu,command --sort=-%mem | head

Топ жрущих процессов память  

ps axo rss,comm,pid \
| awk '{ proc_list[$2]++; proc_list[$2 "," 1] += $1; } \
END { for (proc in proc_list) { printf("%d\t%s\n", \
proc_list[proc "," 1],proc); }}' | sort -n | tail -n 10 | sort -rn \
| awk '{$1/=1024;printf "%.0fMB\t",$1}{print $2}'

Проверка сертификата SSL

echo | openssl s_client -servername ДОМЕН -connect ДОМЕН:443 2>/dev/null | openssl x509 -noout -dates -subject -issuer

Проверяем есть ли на диске неразмеченное место:

vgdisplay

Изменить раздел добавить дискового пространства:

lvextend -L+20G /dev/mapper/vg0-var+www 
resize2fs /dev/mapper/vg0-var+www

**inodes:**  
1) Чаще всего на проектах установлен крон который очищает неипользованные docker-образы . Необходимо посмотреть время запуска крона/таймера. Если время запуска в скором времени предстоит, то ожидаем активации крона и отписываем клиенту.  
2) Если ситуация критическая то производим самостоятельную очистку неипользованных docker-образов:  

**docker rmi `docker images | awk '{ print $1":"$2; }'` 
docker rmi `docker images --filter "dangling=true" -q`** 

Проверяем, что очистка прошла успешно и количество свободных **inode** стало больше:  

docker system df
df -i /var/lib/docker

Проверяем топ процессов:  

ps aux | awk '{print $11}'| sort -nr | uniq -c | sort -nr | head



for i in {1..15}; do
  filename="CTDA_SBRS_20231101001000-$(printf "%04d" $i).csv"
  touch "$filename"
done

for i in $(seq -w 1 15); do
  filename="CTDA_SBRS_20231101001000-${i}.csv"
  touch "$filename"
done

