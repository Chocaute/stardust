---
created: 2024-02-29T09:24
updated: 2024-03-01T14:22
tags:
  - Linux
  - Chat
  - Slack
  - Mattermost
related link: https://docs.mattermost.com/install/prepare-mattermost-database.html
---
```shell
apt install sudo postgresql -y
```

```shell
sudo -u postgres psql
```

```sql
CREATE DATABASE mattermost;
```

```sql
CREATE USER mmuser WITH PASSWORD 'mmuser-password';
```

```sql
GRANT ALL PRIVILEGES ON DATABASE mattermost to mmuser;
```

```sql
ALTER DATABASE mattermost OWNER TO mmuser;
```

```sql
GRANT USAGE, CREATE ON SCHEMA PUBLIC TO mmuser;
```

```sql
\q
```

```shell
sudo systemctl restart postgresql
```

```shell
nano /etc/postgresql/{version}/main/pg_hba.conf
```

```
Find the following lines:

local   all             all                        peer
 
host    all             all         ::1/128        ident

3. Change `peer` and `ident` to `trust`:

local   all             all                        trust
 
host    all             all         ::1/128        trust
```

```shell
sudo systemctl reload postgresql
```

```shell
psql --dbname=mattermost --username=mmuser --password
```