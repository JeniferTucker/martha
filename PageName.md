Incorporate this into install notes

```
s####################################################################
# The Donkey Sanctuary Drupal sites.                               #
# Online gift shop                                                 #
####################################################################

# Get a working copy of all the Sanctuary websites from Ixis server.
[jenifer@jenifer]

    ssh-add ~/.ssh/jenifer.thedonkeysanctuary.com.private
    ssh-add ~/.ssh/jenifer.codon.demon.co.uk.private

# All latest changes to codebasehq repository.
[jenifer@jenifer]

    ssh jenifer@ixis.stanleywindrush.org.uk

    pushd /var/local/donkeys-git

        git add .
        git commit -a -m "Snapshot of live sites"
        git push

    popd

// Connect to virtual machine.
[jenifer@jenifer]

    ssh root@pippa.virtual.metagrid.co.uk
    [root@pippa ~]# 

    pushd /var/local/

        git config --global user.name "Jenifer Tucker"
        git config --global user.email jenifer.tucker@thedonkeysanctuary.org.uk

        # Create our 'master' copy (ony need to do this once).
    	git clone git@codebasehq.com:ixis/donkey-sanctuary/ds.git donkeys-git-master

        # Create a new local copy (clone) of the GIT codebase.
        git clone donkeys-git-master donkeys-git-local

    popd

    # Update our 'master' copy from the 'codebasehq' repository.
    pushd /var/local/donkeys-git-master

        git pull

    popd

    # Update our 'local' working copy from our 'master' copy.
    pushd /var/local/donkeys-git-local

        git pull

    popd

    # Update our 'master' copy from our 'local' working copy.
    git pull /var/local/donkeys-git-local master


# Fix Drupal 6 file permissions after downloading from git repository for public files.
[root@pippa]

pushd /var/local/donkeys-git-local/drupal-7

    chgrp -R apache files
    chmod -R g+w    files

    pushd files

        find . -type d -exec chmod g+ws '{}' \;

    popd

popd

------------------------------------------
------------------------------------------

# LOCAL SETTINGS.
[root@pippa~]

    # Add 'shop' to settings.
	vi ~/local-settings.txt

        #[shop]
        shop_mysqldrupaluser=george
        shop_mysqldrupalpass=***************
        shop_mysqldrupaldata=shop

    # Load settings and functions.
	source ~/local-settings.txt

------------------------------------------
------------------------------------------

# SERVICES CONFIGURATION.

# Set our services to start on boot.    
[root@pippa]

    chkconfig   httpd      on
    chkconfig   mysqld     on

    # Start services.
    service httpd start
    service mysqld start

------------------------------------------
------------------------------------------

# DRUPAL 7 DATABASE.

# Create 'shop' database.
[root@pippa]

    # Drop existing database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="DROP DATABASE ${shop_mysqldrupaldata}"

    # Create database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="CREATE DATABASE ${shop_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

	# Grant permissions.
   	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="GRANT ALL ON ${shop_mysqldrupaldata}.* TO '${shop_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${shop_mysqldrupalpass}'"

    # Connect to database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass}

    # Check we can connect to database. 
    USE shop ;

    # Exit mySQL.
    quit 

------------------------------------------
------------------------------------------

# DRUPAL 7 NEW SITE.

# Create new site sub-directory.
[root@pippa]

	pushd /var/local/donkeys-git-local/drupal-7/sites

        # Create 'shop' site sub-directory.
        mkdir shop

        # Create settings.php file.
    	pushd /var/local/donkeys-git-local/drupal-7/sites/shop

            # Copy alias file.
            cp /var/local/donkeys-git-local/drupal-7/sites/sanctuary/settings.php settings.php

            vi settings.php

               - $db_url = 'mysql://jubilee:***************@localhost/sanctuary';
               + $db_url = 'mysql://nelly:***************@localhost/shop';

        popd

    popd

------------------------------------------
------------------------------------------

# DRUPAL 6 SYMBOLIC LINKS.

# Create symbolic links.
[root@pippa]

pushd /var/local/donkeys-git-local/drupal-7/sites

    ln -s shop shop.stanleywindrush.org.uk
    ln -s shop shop.thedonkeysanctuary.org.uk

popd

------------------------------------------
------------------------------------------

# Create 'files' directory.
[root@pippa]

pushd /var/local/donkeys-git-local/drupal-7/files

    # Create 'shop' files directory.
    mkdir shop
    mkdir shop/tmp

popd

pushd /var/local/donkeys-git-local/drupal-7/sites/shop

    # Create symbolic link.
    ln -s ../../files/shop/ files

popd

# Fix Drupal 7 file permissions.
[root@pippa]

pushd /var/local/donkeys-git-local/drupal-7

    chgrp -R apache files
    chmod -R g+w    files

    pushd files

        find . -type d -exec chmod g+ws '{}' \;

    popd

popd

------------------------------------------
------------------------------------------

# Configure new website.

http://shop.stanleywindrush.org.uk/install.php

------------------------------------------
------------------------------------------

# Install modules.

drush -l shop dl commerce
drush -l shop dl commerce_donate

# Enable modules (and dependencies).

drush -l shop pm-enable commerce commerce_ui
drush -l shop pm-enable commerce_customer commerce_customer_ui
drush -l shop pm-enable commerce_price
drush -l shop pm-enable commerce_line_item commerce_line_item_ui
drush -l shop pm-enable commerce_order commerce_order_ui
drush -l shop pm-enable commerce_checkout commerce_payment commerce_product
drush -l shop pm-enable commerce_product_reference
drush -l shop pm-enable commerce_cart commerce_product_pricing
drush -l shop pm-enable commerce_product_ui
drush -l shop pm-enable commerce_tax_ui

# Additional commerce modules.
drush -l shop pm-enable commerce_donate



------------------------------------------

** TODO **

# File system (tell Drupal where to store the files)
# http://shop.stanleywindrush.org.uk/#overlay=admin/config/media/file-system

	# Public file system path.
	files/shop
	
	# Temporary directory.
	files/shop/tmp
```