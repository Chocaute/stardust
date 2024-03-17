---
created: 2023-12-21T16:32
updated: 2024-03-11T19:26
tags:
  - GLPI
  - Linux
  - SQL
  - MariaDB
  - WebHosting
---
In Mariadb :
```sql
show databases;  
use glpi;  
show tables;
```

Here is where you want to look. Your glpi_users table's where your users_id is, mine happened to be 42; 60 was the next number available, 42 was my id number, and 4 is super-admin rights.  
```sql
insert into glpi_profiles_users
(id,users_id,profiles_id,entities_id,is_recursive,is_dynamic) values(60,42,4,0,0,1);
```

To double check it worked run :  
```sql
select * from glpi_profiles_users;
```

