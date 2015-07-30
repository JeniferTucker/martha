# Details #

These instructions require:

  * Empty database created.
  * DNS aliases set up (if required).
  * Virtual hosts set up.
----

# Add new site details to local settings and functions.
[admin@server~]

    # Add 'spain' to settings.
	vi ~/local-settings.txt

    #[spain]
    spain_mysqldrupaluser=*****
    spain_mysqldrupalpass=*****
    spain_mysqldrupaldata=*****

   # Load settings and functions.
   source ~/local-settings.txt

----

# Add the DNS aliases
# Go to easyDNS.

----

# Create virtual host file.
[admin@server~]

    pushd /etc/httpd/conf.d

cat > www.example.com.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@thedonkeysanctuary.org.uk
  ServerName  www.example.com

  LogLevel warn
  ErrorLog  /var/log/httpd/www.example.com.error.log
  CustomLog /var/log/httpd/www.example.com.access.log combined

  DocumentRoot /var/local/donkeys-git-dev4/drupal-7
  <Directory /var/local/donkeys-git-dev4/drupal-7>

  Options Indexes FollowSymLinks MultiViews
  AllowOverride All
  Order allow,deny
  allow from all

  </Directory>

</VirtualHost>
EOF

        # Reload Apache service.
        service httpd reload

    popd

----

# DRUPAL 7 DATABASE.

# Create 'spain' empty database.
[admin@server~]

    # Drop existing database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="DROP DATABASE ${spain_mysqldrupaldata}"

    # Create database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="CREATE DATABASE ${spain_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

	# Grant permissions.
   	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="GRANT ALL ON ${spain_mysqldrupaldata}.* TO '${spain_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${spain_mysqldrupalpass}'"

    # Show existing databases.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="SHOW DATABASES"

    # Check database connection.
    mysql --user=${spain_mysqldrupaluser} --password=${spain_mysqldrupalpass} ${spain_mysqldrupaldata} \
      --execute="SELECT VERSION()"

----

# DRUPAL 7 FILES DIRECTORY.

# Create files directory.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7/sites/
    
    	mkdir spain

        # Create files directory.
        mkdir spain/files

        # Set file permissions.
    	id apache
	    # uid=48(apache) gid=48(apache) groups=48(apache)

    	chgrp -R apache spain/files
    	chmod -R g+w    spain/files

    	pushd spain/files

        	find . -type d -exec chmod g+ws '{}' \;

    	popd

	popd

----

# DRUPAL 7 SETTINGS.


# Copy the default settings file.
[admin@server~]


	pushd /var/local/donkeys-git-dev4/drupal-7/sites/spain

        cp /var/local/donkeys-git-dev4/drupal-7/sites/default/default.settings.php settings.php

		# Make the settings file writeable.
        chgrp apache settings.php
        chmod a+w    settings.php
        
        # After installation script, change back to read only.
        italy7.stanleywindrush.org.uk
        			
	popd

----

# DRUPAL 7 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-7/sites

	    ln -s spain www.example.com

	popd

----

# DRUPAL 7 SITE INSTALL.

# Run install from browser.
http://www.example.com/install.php

----```