# postgres
# Postgres PITR Backup & Restore Demo Configuration
#### Continuous Archiving and Point-in-Time Recovery (PITR)
[Cick here for Video Demo ](https://youtu.be/WZps_MYYvV8) https://youtu.be/WZps_MYYvV8

## Postgres WAL Archiving 
1. Create the WAL & backup Directory and add necessary Permisson 

```
bash~ mkdir -p  /var/lib/pgsql/wals/
bash~ mkdir -p  /var/lib/pgsql/backups/
bash~ chown postgres:postgres -R /var/lib/pgsql/backups/
bash~ chown postgres:postgres -R /var/lib/pgsql/wals/
```

2. Edit the postgresql.conf with postgres user and add the below changes

```
wal_level=archive
archive_mode=on
archive_command = 'test ! -f /var/lib/pgsql/wals/%f && cp %p /var/lib/pgsql/wals/%f'
```

3. Restart the database

`bash~ service postgresql restart`

4. Prepare for the basebackup 

```
su - postgres
psql -c "SELECT pg_start_backup('label');" postgres 
tar -C /var/lib/pgsql/data/ -czvf /var/lib/pgsql/backups/pg_basebackup_backup.tar.gz .
psql -c "SELECT pg_stop_backup();" postgres
```
## Add Data to PostgreSQL
```
bash~  su - postgres
bash~  createdb mydb
bash~  psql -s mydb
```

## Testing Restore 
1. Stop the postgres 
```
bash~ /etc/init.d/postgresql stop
```
2. Remove the data folder  and Initilize the DB

```
bash~ cd /var/lib/pgsql/
bash~ rm -rf data/
```
3. Initilaize the Database
```
bash~ /etc/init.d/postgresql initdb
bash~ /etc/init.d/postgresql start

```

## Restore from WAL Archiving 
1. Stop the Postgres Server 

```
bash~ /etc/init.d/postgresql stop
```

2. Extract the Basebackup...

```
bash~  tar xvf /var/lib/pgsql/backups/pg_basebackup_backup.tar.gz -C /var/lib/pgsql/data
```

3. Create recovery.conf with the below content and check the permission also 
```
bash~ cd /var/lib/pgsql/data
bash~ cat recovery.conf 
restore_command = 'cp /var/lib/pgsql/wals/%f %p'
```

4. Restart the Postgres DB

```
bash~ /etc/init.d/postgresql start
```
5. Upon completion of the recovery process, the server will rename recovery.conf to recovery.done

```
bash~ cd /var/lib/pgsql/data
bash~ cat recover.done
```


