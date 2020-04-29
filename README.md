# MySQL

## Задание

развернуть базу из дампа и настроить репликацию  
В материалах приложены ссылки на вагрант для репликации  
и дамп базы bet.dmp  
базу развернуть на мастере  
и настроить чтобы реплицировались таблицы  
| bookmaker |  
| competition |  
| market |  
| odds |  
| outcome  

* Настроить GTID репликацию  

варианты которые принимаются к сдаче  
- рабочий вагрантафайл  
- скрины или логи SHOW TABLES  
* конфиги  
* пример в логе изменения строки и появления строки на реплике  

## Выполнение

репликация настраивается плэйбуком [replication.yml](replication.yml)  
при провижене машины slave  
* установка сервера percona на обе машины
* копирование конфигов
* установка пароля
* на мастере импорт базы bet, создание пользователя repl с правами для репликации
* на слейве настройка сервера в роль слейва и запуск
* вывод статуса репликации

## Проверка

### MASTER
```bash
mysql> USE bet;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SHOW TABLES;
+------------------+
| Tables_in_bet    |
+------------------+
| bookmaker        |
| competition      |
| events_on_demand |
| market           |
| odds             |
| outcome          |
| v_same_event     |
+------------------+
7 rows in set (0.00 sec)

mysql> INSERT INTO bookmaker (id,bookmaker_name) VALUES(1,'1xbet');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  1 | 1xbet          |
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
5 rows in set (0.00 sec)
```
### SLAVE
```bash
mysql> USE bet;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
4 rows in set (0.00 sec)

mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  1 | 1xbet          |
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
5 rows in set (0.00 sec)
```
### slave status
```bash
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.11.150
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 119757
               Relay_Log_File: slave-relay-bin.000002
                Relay_Log_Pos: 119970
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: bet.events_on_demand,bet.v_same_event

....

                  Master_UUID: 9c732723-8a20-11ea-a263-5254008afee6
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 9c732723-8a20-11ea-a263-5254008afee6:1-40
            Executed_Gtid_Set: 9c6bc4e4-8a20-11ea-a7cd-5254008afee6:1,
9c732723-8a20-11ea-a263-5254008afee6:1-40
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)

```
### binlog
в binlog на slave видно строку с добавлением значения 1xbet в таблицу bookmaker  
```bash
SET @@SESSION.GTID_NEXT= '9c732723-8a20-11ea-a263-5254008afee6:40'/*!*/;
# at 113930
#200429 14:41:21 server id 1  end_log_pos 114003 CRC32 0xe7ea5d3a 	Query	thread_id=8	exec_time=0	error_code=0
SET TIMESTAMP=1588171281/*!*/;
BEGIN
/*!*/;
# at 114003
#200429 14:41:21 server id 1  end_log_pos 114130 CRC32 0x4a5b86da 	Query	thread_id=8	exec_time=0	error_code=0
use `bet`/*!*/;
SET TIMESTAMP=1588171281/*!*/;
INSERT INTO bookmaker (id,bookmaker_name) VALUES(1,'1xbet')
/*!*/;
# at 114130
#200429 14:41:21 server id 1  end_log_pos 114161 CRC32 0xd32b375b 	Xid = 72
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```
