---
created: 2024-02-22T09:20
updated: 2024-03-01T14:21
tags:
  - GLPI
  - WebHosting
  - MariaDB
---
https://www.bytebang.at/Blog/Reset+GLPI+passwords+in+the+database

```sql
Enter password: ************  
Welcome to the MySQL monitor.  Commands end with ; or \g.  
Your MySQL connection id is 1739  
Server version: 5.1.42-community MySQL Community Server (GPL)  
  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
mysql> use glpi;  
Database changed  
  
mysql> select id,name,password from glpi_users;  
+----+-----------+----------------------------------+  
| id | name      | password                         |  
+----+-----------+----------------------------------+  
|  2 | glpi      | c37093b421c8131b8999d75fd73c55fd |  
|  3 | post-only | c37093b421c8131b8999d75fd73c55fd |  
|  4 | tech      | c37093b421c8131b8999d75fd73c55fd |  
|  5 | normal    | c37093b421c8131b8999d75fd73c55fd |  
+----+-----------+----------------------------------+  
4 rows in set (0.00 sec)  
  
mysql> update glpi_users set  password= MD5('newPassword') where name='glpi';  
Query OK, 1 row affected (0.00 sec)  
Rows matched: 1  Changed: 1  Warnings: 0  
  
mysql> select id,name,password from glpi_users;  
+----+-----------+----------------------------------+  
| id | name      | password                         |  
+----+-----------+----------------------------------+  
|  2 | glpi      | 14a88b9d2f52c55b5fbcf9c5d9c11875 |  
|  3 | post-only | c37093b421c8131b8999d75fd73c55fd |  
|  4 | tech      | c37093b421c8131b8999d75fd73c55fd |  
|  5 | normal    | c37093b421c8131b8999d75fd73c55fd |  
+----+-----------+----------------------------------+  
4 rows in set (0.00 sec)  
  
mysql> exit  
Bye.
```