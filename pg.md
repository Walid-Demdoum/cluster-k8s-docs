### Create database backup
```bash
createdb "db_name" -O "owner"
```
### Restore database from backup file
```bash
pg_restore -c -d "db_name" -v "path_to_backup.dump" -p 5432 -h "host" -U "pg_user"
```
