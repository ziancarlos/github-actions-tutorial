# VPS MySQL Database Backup & Restore

## 1. Login to Old VPS

SSH into the **old VPS**:

```bash
ssh <<OLD_VPS_USER>>@<<OLD_VPS_IP>>
```

---

## 2. Export the MySQL Database

Run `mysqldump` inside the MySQL Docker container:

```bash
docker exec -i <<OLD_MYSQL_CONTAINER>> mysqldump \
  -u <<OLD_DB_USERNAME>> \
  -p'<<OLD_DB_PASSWORD>>' \
  --no-tablespaces \
  --routines \
  --triggers \
  <<OLD_DATABASE_NAME>> \
  > <<BACKUP_FILE_PATH>>
```

Example:

```bash
docker exec -i shoeboxx-mysql mysqldump \
  -u shoeboxx_user \
  -p'<<OLD_DB_PASSWORD>>' \
  --no-tablespaces \
  --routines \
  --triggers \
  <<OLD_DATABASE_NAME>> \
  > /backup/database.sql
```

The resulting `.sql` file is created **on the VPS host**, not inside the Docker container.

```text
Old VPS
│
├── MySQL Docker Container
│   └── Database
│
└── /backup/database.sql
```

---

## 3. Exit the Old VPS

```bash
exit
```

---

## 4. Login to the New VPS

```bash
ssh <<NEW_VPS_USER>>@<<NEW_VPS_IP>>
```

---

## 5. Copy the Backup from the Old VPS

From the **new VPS**, run:

```bash
scp <<OLD_VPS_USER>>@<<OLD_VPS_IP>>:<<BACKUP_FILE_PATH>> .
```

For example:

```bash
scp root@72.62.125.203:/backup/database.sql .
```

The `.` means:

> Copy the file into my current directory.

You should then have:

```text
New VPS
│
├── database.sql
└── ...
```

---

## 6. Create the Database

If the database doesn't exist, create it:

```bash
docker exec -i <<MYSQL_CONTAINER>> mysql \
  -u <<DB_USERNAME>> \
  -p'<<DB_PASSWORD>>' \
  -e "CREATE DATABASE IF NOT EXISTS <<DATABASE_NAME>>;"
```

Example:

```bash
docker exec -i shoeboxx-mysql mysql \
  -u shoeboxx_user \
  -p'<<DB_PASSWORD>>' \
  -e "CREATE DATABASE IF NOT EXISTS <<DATABASE_NAME>>;"
```

The `IF NOT EXISTS` means this command won't fail if the database already exists.

---

## 7. Restore the Backup

Run:

```bash
docker exec -i <<MYSQL_CONTAINER>> mysql \
  -u <<DB_USERNAME>> \
  -p'<<DB_PASSWORD>>' \
  <<DATABASE_NAME>> < <<BACKUP_FILE_NAME>>
```

Example:

```bash
docker exec -i shoeboxx-mysql mysql \
  -u shoeboxx_user \
  -p'<<DB_PASSWORD>>' \
  <<DATABASE_NAME>> < database.sql
```

The flow is:

```text
OLD VPS
│
│ mysqldump
↓
database.sql
│
│ scp
↓
NEW VPS
│
│ mysql < database.sql
↓
NEW MYSQL
│
└── Restored Database
```

### Required inputs

You need to provide:

```text
<<OLD_VPS_USER>>
<<OLD_VPS_IP>>
<<OLD_MYSQL_CONTAINER>>
<<OLD_DB_USERNAME>>
<<OLD_DB_PASSWORD>>
<<OLD_DATABASE_NAME>>
<<BACKUP_FILE_PATH>>

<<NEW_VPS_USER>>
<<NEW_VPS_IP>>
<<MYSQL_CONTAINER>>
<<DB_USERNAME>>
<<DB_PASSWORD>>
<<DATABASE_NAME>>
<<BACKUP_FILE_NAME>>
```

**Small correction:** you don't need to `exit` the old VPS before running `scp` if you open a second terminal, but your original sequence is perfectly fine.
