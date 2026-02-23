# Домашнее задание к занятию "`Репликация и масштабирование. Часть 1`" - `Гилельс Кирилл`


### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

### Решение 

Master-slave и master-master — это два основных режима репликации баз данных (например, MySQL/MariaDB). В master-slave один сервер (master) принимает все записи, а слейвы (slaves) только читают и копируют изменения. В master-master все серверы равноправны: каждый может принимать записи и реплицировать их на другие. Master-slave проще в настройке, подходит для read-heavy нагрузок, но имеет единую точку отказа(master); master-master повышает доступность и отказоустойчивость, но рискует конфликтами данных.

---

### Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

### Решение

Работа выполнялась c docker, файлы конфигов были взяты из конспекта по лекции, приложенного в личном кабинете.
При первом запуске и проверке статусов на slave выдало ошибку:
Last_IO_Error: Error connecting to source 'repl@mysql_master:3306'. This was attempt 4/86400, with a delay of 60 seconds between attempts. Message: Authentication plugin 'caching_sha2_password' reported error: Authentication requires secure connection.

Решил проблему пересборкой slave с заменой на мастере:
```bash
docker exec -it mysql_master mysql -uroot -prootpassword
```
```sql
ALTER USER 'repl'@'%' IDENTIFIED WITH mysql_native_password BY 'slavepass';
FLUSH PRIVILEGES;
EXIT;
```
Внес модификацию в slave.sql 
```sql
STOP REPLICA;
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST='mysql_master',
    SOURCE_PORT=3306,
    SOURCE_USER='repl',
    SOURCE_PASSWORD='slavepass',
    SOURCE_LOG_FILE='mysql-bin.000003',
    SOURCE_LOG_POS=157,
    SOURCE_SSL=0;
START REPLICA;
```
В дальнейшем работа проводилась без проблем, была создана тестовая БД, таблица с двумя строками, проведено тестирование, скриншоты приложил:
![2.1](/img/2.1.png)
![2.2](/img/2.2.png)
![2.3](/img/2.3.png)
![2.3](/img/2.4.png)
![2.4](/img/2.5.png)



---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

---

### Задание 3* 

Выполните конфигурацию master-master репликации. Произведите проверку.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*
