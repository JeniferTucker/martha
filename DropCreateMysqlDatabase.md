
```
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="DROP DATABASE ${seven_mysqldrupaldata}"

    # Create database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="CREATE DATABASE ${seven_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

        # Grant permissions.
        mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="GRANT ALL ON ${seven_mysqldrupaldata}.* TO '${seven_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${seven_mysqldrupalpass}'"

    # Load latest backup into empty database.
        mysql \
            --user=${seven_mysqldrupaluser} \
            --password=${seven_mysqldrupalpass} \
            --database=${seven_mysqldrupaldata} \
            < /var/local/backups/linode/seven-20130122203111.sql
```