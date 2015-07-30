
```
# Create a general store for backups.
# Only required once.

pushd /var/local

  mkdir /var/local/backups/linode
	
popd


----------------------------

# Load local settings.
[admin@server ~]# 

    source ~/local-settings.txt 

# Create back up directory.
backupdir=/var/local/backups/linode

backupdb() {

	echo "Backing up database..." 

        timestamp=$(date +%Y%m%d%H%M%S)
        filename=${seven_mysqldrupaldata}-${timestamp}.sql
        filepath=${backupdir}/${filename}

        mysqldump --user=${seven_mysqldrupaluser} \
                  --password=${seven_mysqldrupalpass} \
                  --complete-insert \
                  --create-options \
                  ${seven_mysqldrupaldata} > ${filepath}
        }

backupdb

# Create a zipfile of latest backup.
gzip ${filepath}

```