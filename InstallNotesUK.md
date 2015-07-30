# Details #

Upgrading a Drupal site from D6 to D7.

These instructions will ensure:

  * Drupal 6 core is up to date.
  * Contributed modules/themes are up to date.
  * Database backed up.
  * DNS aliases and virtual hosts set up.


---

```

# Check disk space on server.
[admin@server]# 

    df -h

        Filesystem            Size  Used Avail Use% Mounted on
        /dev/xvda              24G   23G     0 100% /
        tmpfs                 243M  124K  243M   1% /dev/shm

        # Delete old Drush files (if disk space low).
        pushd /root/drush-backups
        
            rm -rf [backup-directory]

        popd
    
----

# Push updates from Ixis server to codebasehq repository.
[ixis@server]

    # Git.
    git add .
    git commit -a -m "Snapshot of live sites"
    git push
    
# Get a working copy of all the Sanctuary websites from codebasehq repository.
[admin@server]

    pushd /var/local/

        # Create our 'master' copy (only need to do this once)
    	# git clone git@codebasehq.com:ixis/donkey-sanctuary/ds.git donkeys-git-master

        # Create a new 'donkeys7' copy (clone) of the GIT codebase.
        git clone donkeys-git-master donkeys-git-donkeys7

    popd

# Update our master copy from the codebasehq repository.
[admin@server]

    pushd /var/local/donkeys-git-master

       git pull

    popd

# Update our 'donkeys7' working copy from our 'master' copy.
[admin@server]

    pushd /var/local/donkeys-git-donkeys7

        git pull

    popd

# Fix file permissions after downloading from git repository.
[admin@server~]

pushd /var/local/donkeys-git-donkeys7/drupal-6

    id apache
    # uid=48(apache) gid=48(apache) groups=48(apache)

    chgrp -R apache files
    chmod -R g+w    files

    pushd files

        find . -type d -exec chmod g+ws '{}' \;

    popd

popd

----

# Backup mySQL databases on Ixis server (gamma).
[admin@server~]

    ssh root@gamma.virtual.metagrid.co.uk

    # Load settings and functions.
    source ~/local-settings.txt

    # Create backup directories.
    mkdir /var/local/backups
    mkdir /var/local/backups/ixis

# Create a list of database names.	
cat > /var/local/backups/ixis/backup-databases.txt << EOF
cyprus
ireland
italy
research
sanctuary
spain
welfare
EOF

# Create a database dump for each mySQL database.
for database in $(cat /var/local/backups/ixis/backup-databases.txt)
do
    echo "Database is : ${database}"

    backupsite=${database}

    backupuser=jenifer
    backuphost=ixis.stanleywindrush.org.uk
    remotepath=/var/local/donkeys-git/drupal-6

    backupdate=$(date '+%Y%m%d%H%M%S')
    backupfile=${backupsite}-${backuphost}-${backupdate}.sql.gz
    backuppath=/var/local/backups/ixis/${backupfile}

    echo "Running database dump of : ${database}"

    # Run database dump and compress on remote server, store on local disk.
    ssh ${backupuser}@${backuphost} "cd ${remotepath} ; drush -l ${backupsite} sql-dump | gzip" > "${backuppath}"

    echo "Dump location : ${backuppath}"

done

----

# Add new site details to local settings and functions.
[admin@server~]

    # Add 'uk7' to settings.
	vi ~/local-settings.txt

        #[uk7]
        uk7_mysqldrupaluser=*****
        uk7_mysqldrupalpass=*****
        uk7_mysqldrupaldata=*****

   # Load settings and functions.
	source ~/local-settings.txt

----

# Add the DNS aliases
# Go to easyDNS.

----

# Create virtual host files.
[admin@server~]

    pushd /etc/httpd/conf.d

cat > six.stanleywindrush.org.uk.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@thedonkeysanctuary.org.uk
  ServerName  six.stanleywindrush.org.uk

  LogLevel warn
  ErrorLog  /var/log/httpd/six.stanleywindrush.org.uk.error.log
  CustomLog /var/log/httpd/six.stanleywindrush.org.uk.access.log combined

  DocumentRoot /var/local/donkeys-git-donkeys7/drupal-6
  <Directory /var/local/donkeys-git-donkeys7/drupal-6>

  Options Indexes FollowSymLinks MultiViews
  AllowOverride All
  Order allow,deny
  allow from all

  </Directory>

</VirtualHost>
EOF

cat > uk7.stanleywindrush.org.uk.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@thedonkeysanctuary.org.uk
  ServerName  uk7.stanleywindrush.org.uk

  LogLevel warn
  ErrorLog  /var/log/httpd/uk7.stanleywindrush.org.uk.error.log
  CustomLog /var/log/httpd/uk7.stanleywindrush.org.uk.access.log combined

  DocumentRoot /var/local/donkeys-git-donkeys7/drupal-7
  <Directory /var/local/donkeys-git-donkeys7/drupal-7>

  Options Indexes FollowSymLinks MultiViews
  AllowOverride All
  Order allow,deny
  allow from all

  </Directory>

</VirtualHost>
EOF

    popd

# Reload Apache service.
service httpd reload

----

# DRUPAL 6 DATABASE.

# Create 'sanctuary' database.
[admin@server~]

	# Check mySQL is running.
	/etc/init.d/mysqld status

    # Drag/drop SFTP network file from gamma to pippa.

    # Unzip SQL file.
    gunzip /var/local/backups/ixis/sanctuary-ixis.stanleywindrush.org.uk-20130625050426.sql.gz
        
	# Load local settings.
	source ~/local-settings.txt
	backup=sanctuary-ixis.stanleywindrush.org.uk-20130625050426.sql

    # Drop existing database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="DROP DATABASE ${sanctuary_mysqldrupaldata}"

    # Create database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="CREATE DATABASE ${sanctuary_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

	# Grant permissions.
   	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="GRANT ALL ON ${sanctuary_mysqldrupaldata}.* TO '${sanctuary_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${sanctuary_mysqldrupalpass}'"

    # Load latest backup into empty database.
        mysql \
            --user=${sanctuary_mysqldrupaluser} \
            --password=${sanctuary_mysqldrupalpass} \
            --database=${sanctuary_mysqldrupaldata} \
            < /var/local/backups/ixis/${backup}

    # Connect to database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass}

    # Connect to database. 
    USE sanctuary ;

    # Delete duplicate entries.
    CREATE TEMPORARY TABLE myfids1 SELECT min(fid) AS toad FROM files GROUP BY filepath HAVING count(fid) > 1 ;
    DELETE FROM files WHERE fid in (SELECT * FROM myfids1) ;

    # Run command again to catch >2 duplicate entries.
    CREATE TEMPORARY TABLE myfids4 SELECT min(fid) AS toad FROM files GROUP BY filepath HAVING count(fid) > 1 ;
    DELETE FROM files WHERE fid in (SELECT * FROM myfids4) ;

    # Delete 'thumbnail' and '' files.
    # SELECT fid,filename,filepath FROM files WHERE filename='thumbnail';
    # SELECT fid,filename,filepath FROM files WHERE filename='node';
    DELETE FROM files WHERE filename='thumbnail';
    DELETE FROM files WHERE filename='node';

    # Delete forum content.
    DELETE FROM node WHERE type='forum';
    quit

----

# DRUPAL 7 DATABASE.

# Create 'donkeys7' empty database.
[admin@server~]

    # Drop existing database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="DROP DATABASE ${uk7_mysqldrupaldata}"

    # Create database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="CREATE DATABASE ${uk7_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

	# Grant permissions.
   	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="GRANT ALL ON ${uk7_mysqldrupaldata}.* TO '${uk7_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${uk7_mysqldrupalpass}'"

    # Show existing databases.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="SHOW DATABASES"

    # Check database connection.
    mysql --user=${uk7_mysqldrupaluser} --password=${uk7_mysqldrupalpass} ${uk7_mysqldrupaldata} \
      --execute="SELECT VERSION()"

----

# DRUPAL 6 CORE AND MODULES.

# Ensure all contributed modules are up to date.
[admin@server~]

	pushd /var/local/donkeys-git-donkeys7/drupal-6

        # Clear drush cache.
        drush cc drush

        # Check available updates for enabled modules (automatically runs database updates if required).
        drush -l sanctuary pm-update

	popd

# Remove uninstalled modules from the modules directory.
[admin@server~]

	pushd /var/local/donkeys-git-donkeys7/drupal-6/sites/all/modules

        rm -rf custom_breadcrumbs
        rm -rf ecard
        rm -rf elf
        rm -rf event
        rm -rf event_views

        rm -rf flickrapi
        rm -rf googtube

        rm -rf imagefield_crop
        rm -rf mimedetect
        rm -rf mimemail
        rm -rf node_expire
        rm -rf radioactivity

        rm -rf rules
        rm -rf simplenews
        rm -rf site_verify
        rm -rf views_slideshow

	popd

# Remove old themes from the themes directory.
[admin@server~]

	pushd /var/local/donkeys-git-donkeys7/drupal-6/sites/all/themes

        rm -rf adaptivetheme
        rm -rf blogbuzz

	popd

# Move old imported files from images directory.
[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-6/files/donkeys/images/import

        # Move images to images directory.
        mv filey* /var/local/donkeys-git-donkeys7/drupal-6/files/donkeys/images

    popd

# Remove legacy image content.
[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-6/files/donkeys/imagecache

        rm -rf cart
        rm -rf product
        rm -rf product_list
        rm -rf uc_thumbnail

    popd

# Remove legacy image thumbnails.
[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-6/files/donkeys/imagefield_thumbs

        rm -rf hannah-sponsoradonkey.png
        rm -rf "Kennel Barn. Photo copyright of The Donkey Sanctuary.jpg"
        rm -rf Slideshow-Egypt.jpg

    popd

----

# DRUSH UPGRADE.

# Install Drupal 7 modules required to run Drush site upgrade (if not already on VM).
# [admin@server~]

#       drush dl drush_sup

----

# DRUPAL 6 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

pushd /var/local/donkeys-git-donkeys7/drupal-6/sites

    ln -s sanctuary six.stanleywindrush.org.uk

popd

----

# DRUPAL 6 FILES DIRECTORY.

# Create symbolic links.
[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-6/files

        # Create symbolic links.
        ln -s donkeys sanctuary
        ln -s donkeys six.stanleywindrush.org.uk

    popd

----

# DRUPAL 6 IMAGE ATTACH CONVERTER.

# Source:
# http://drupal.org/node/201983

    # Login as admin user.
    http://six.stanleywindrush.org.uk/user


# TODO...
# Look at /admin/blocks - see if we can remove old blocks for different themes.
# May save tryng to tidy up when migrated to D7.

    # Clear cached data.
    http://six.stanleywindrush.org.uk/admin/settings/performance

    # Create prerequisite imagefield type.
    // Table prefix ... in case you use it - otherwise leave blank
    // $table_pfx = '';
    // The imagefield field that you have already created and configured as per Prerequisites.
    // $field_name = 'field_imagefield';
    // The content type that you have already created as per Prerequisites.
    // $type_name = 'imagepage';
    http://six.stanleywindrush.org.uk/admin/content/types/add

        Name:           imagepage
        Type:           imagepage
        Description:    Prerequisite imagefield type for image converter.
    
    http://six.stanleywindrush.org.uk/admin/content/node-type/imagepage/fields
    
        New field:      Photo
        Field_          imagefield
        Data type:      File
        Form element:   Image
        
        Number of values:   		Unlimited

    # Add these after migration - not important at migration stage.
    #		List field:					
    #		Files listed by default:	Enabled
    #       Description field:  		Disabled

    # Add new 'image_field' to content types (where attached images are enabled).
    http://six.stanleywindrush.org.uk/admin/content/node-type/blog/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/corporate-partnership/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/event/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/fundraising/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/page/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/pressrelease/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/rehome/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/story/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/videoclip/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/volunteer/fields
    http://six.stanleywindrush.org.uk/admin/content/node-type/webform/fields

    # Remove redundant 'copyright' field.
    http://six.stanleywindrush.org.uk/admin/content/node-type/image/fields/field_image_copyright/remove

    # Migrate all image module nodes to imagefields.
    pushd /var/local/donkeys-git-donkeys7/drupal-6

        # Copy migration script to Drupal 6 core.
        cp /var/local/scripts/imagefield_migrate.php .

        # Run script.
        http://six.stanleywindrush.org.uk/imagefield_migrate.php

        - content_type_imagefield populated.
        - content_field_image_field populated.
        - files table updated
        update content_type_imagepage failed for 6452
        update content_field_imagefield failed for 6452
        update content_type_imagepage failed for 7022
        update content_field_imagefield failed for 7022
        update content_type_imagepage failed for 6735
        update content_field_imagefield failed for 6735
        update content_type_imagepage failed for 6736
        update content_field_imagefield failed for 6736
        update content_type_imagepage failed for 6752
        update content_field_imagefield failed for 6752
        update content_type_imagepage failed for 6782
        update content_field_imagefield failed for 6782
        update content_type_imagepage failed for 6783
        update content_field_imagefield failed for 6783
        update content_type_imagepage failed for 6785
        update content_field_imagefield failed for 6785
        update content_type_imagepage failed for 6786
        update content_field_imagefield failed for 6786
        update content_type_imagepage failed for 6787
        update content_field_imagefield failed for 6787
        update content_type_imagepage failed for 6802
        update content_field_imagefield failed for 6802
        update content_type_imagepage failed for 6803
        update content_field_imagefield failed for 6803
        update content_type_imagepage failed for 6805
        update content_field_imagefield failed for 6805
        update content_type_imagepage failed for 6806
        update content_field_imagefield failed for 6806
        update content_type_imagepage failed for 6813
        update content_field_imagefield failed for 6813
        update content_type_imagepage failed for 6815
        update content_field_imagefield failed for 6815
        update content_type_imagepage failed for 6816
        update content_field_imagefield failed for 6816
        update content_type_imagepage failed for 6843
        update content_field_imagefield failed for 6843
        update content_type_imagepage failed for 6844
        update content_field_imagefield failed for 6844
        update content_type_imagepage failed for 6845
        update content_field_imagefield failed for 6845
        update content_type_imagepage failed for 6846
        update content_field_imagefield failed for 6846
        update content_type_imagepage failed for 6848
        update content_field_imagefield failed for 6848
        update content_type_imagepage failed for 6903
        update content_field_imagefield failed for 6903
        update content_type_imagepage failed for 6904
        update content_field_imagefield failed for 6904
        update content_type_imagepage failed for 6947
        update content_field_imagefield failed for 6947
        update content_type_imagepage failed for 6948
        update content_field_imagefield failed for 6948
        update content_type_imagepage failed for 6949
        update content_field_imagefield failed for 6949
        update content_type_imagepage failed for 6950
        update content_field_imagefield failed for 6950
        update content_type_imagepage failed for 6951
        update content_field_imagefield failed for 6951
        update content_type_imagepage failed for 6954
        update content_field_imagefield failed for 6954
        update content_type_imagepage failed for 6955
        update content_field_imagefield failed for 6955
        update content_type_imagepage failed for 6956
        update content_field_imagefield failed for 6956
        update content_type_imagepage failed for 6964
        update content_field_imagefield failed for 6964
        update content_type_imagepage failed for 6966
        update content_field_imagefield failed for 6966
        update content_type_imagepage failed for 6975
        update content_field_imagefield failed for 6975
        update content_type_imagepage failed for 6976
        update content_field_imagefield failed for 6976
        update content_type_imagepage failed for 6977
        update content_field_imagefield failed for 6977
        update content_type_imagepage failed for 6979
        update content_field_imagefield failed for 6979
        update content_type_imagepage failed for 6980
        update content_field_imagefield failed for 6980
        update content_type_imagepage failed for 6981
        update content_field_imagefield failed for 6981
        update content_type_imagepage failed for 7012
        update content_field_imagefield failed for 7012
        update content_type_imagepage failed for 7056
        update content_field_imagefield failed for 7056
        update content_type_imagepage failed for 7057
        update content_field_imagefield failed for 7057
        update content_type_imagepage failed for 7058
        update content_field_imagefield failed for 7058
        update content_type_imagepage failed for 7066
        update content_field_imagefield failed for 7066
        update content_type_imagepage failed for 7219
        update content_field_imagefield failed for 7219
        update content_type_imagepage failed for 7241
        update content_field_imagefield failed for 7241
        - 1700 image_attach relationships were migrated.

        # Remove script to prevent re-running.
        rm -rf imagefield_migrate.php

   popd

----

# DRUPAL 6 IMAGE HANDLING.

# Disable image module (image, image attach, image gallery).
[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-6

        # Disable image module and extensions.
        drush -l sanctuary --yes pm-disable image

        # Download new image handling modules.
        drush -l sanctuary dl imagecache 
        drush -l sanctuary dl imageapi

        # Enable modules.
        drush -l sanctuary --yes pm-enable imageapi
        drush -l sanctuary --yes pm-enable imagecache
        drush -l sanctuary --yes pm-enable imagecache_ui
        drush -l sanctuary --yes pm-enable imageapi_gd

        # Clear drush cache.
        drush cache-clear drush

    popd

# Remove image module.

    pushd /var/local/donkeys-git-donkeys7/drupal-6/sites/all/modules

         rm -rf image

	popd

# Disable 'rebrand' theme, set 'garland' as default theme.
http://six.stanleywindrush.org.uk/admin/build/themes

# Clear cached data.
# Disable all caching during upgrade.
http://six.stanleywindrush.org.uk/admin/settings/performance

# Remove 'amadou' theme and 'rebrand' sub-theme from the themes directory.
[admin@server~]

	pushd /var/local/donkeys-git-donkeys7/drupal-6/sites/all/themes

        rm -rf amadou
        rm -rf rebrand

	popd

----

# DRUPAL 6 UPGRADE TO DRUPAL 7.

# Create alias file.
[admin@server~]

	pushd /var/local/donkeys-git-donkeys7/drupal-6/sites/sanctuary

        # Create alias file.
cat > /var/local/donkeys-git-donkeys7/drupal-6/sites/sanctuary/uk7.alias.drushrc.php << EOF
<?php
  \$aliases['uk7'] = array(
    'root'   => '/var/local/donkeys-git-donkeys7/drupal-7',
    'uri'    => 'sanctuary',
    'db-url' => 'mysql://${uk7_mysqldrupaluser}:${uk7_mysqldrupalpass}@localhost/${uk7_mysqldrupaldata}',
  );
EOF

    popd

----

# Run site-upgrade command.
[admin@server~]

	pushd /var/local/donkeys-git-donkeys7/drupal-6

        # Run site upgrade.
        drush -l sanctuary site-upgrade @uk7

    popd

----

# DRUPAL 7 OVERRIDES.

# Remove robots.txt file (using robotsTxt module).
[admin@server~]

	pushd /var/local/donkeys-git-donkeys7/drupal-7

        rm -f robots.txt

    popd

----

# DRUPAL 7 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-7/sites

        ln -s sanctuary donkeys7.thedonkeysanctuary.org.uk
        ln -s sanctuary uk7.stanleywindrush.org.uk

    popd

----

# DRUPAL 7 FILES DIRECTORY.

[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-7/files

        # Create symbolic links.
        ln -s donkeys sanctuary
        ln -s donkeys uk7.stanleywindrush.org.uk

    popd

    # Set file permissions.
    pushd /var/local/donkeys-git-donkeys7/drupal-7

        chgrp -R apache files
        chmod -R g+w    files

    popd

    pushd /var/local/donkeys-git-donkeys7/drupal-7/files
        
        chmod o-w       donkeys
        find . -type d -exec chmod g+ws '{}' \;

    popd

----

# DRUPAL 7 MODULES.

# Add new contributed modules.
[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-7

        # Patch fix for ctools module.
        # Required if Scheduler module required (requested by Italy).
        # http://code.google.com/p/martha/wiki/DrupalScheduler
        # drush -l sanctuary dl ctools-7.x-1.x-dev

        # Download replacement modules.
        drush -l sanctuary dl contact_forms
        drush -l sanctuary dl ecard
        drush -l sanctuary dl transliteration

        # Download new modules.
        drush -l sanctuary dl filefield_paths
        drush -l sanctuary dl filefield_sources
        drush -l sanctuary dl imagecache_actions
        drush -l sanctuary dl linkchecker
        drush -l sanctuary dl term_reference_tree

        # Infinite page scroller.
        # http://drupal.org/project/views_infinite_scroll

        # Import modules.
        drush -l sanctuary dl menu_import

        # Video embed field module.
        drush -l sanctuary dl video_embed_field

        # Media Flickr module.
        drush -l sanctuary dl media_flickr

        # drush -l sanctuary dl libraries
        # drush -l sanctuary dl views_infinite_scroll

        # pushd sites/all

         #    mkdir libraries
         #    mkdir autopager

        # popd

        # pushd /var/local/donkeys-git-donkeys7/drupal-7/sites/all/autopager

         #    wget http://jquery-autopager.googlecode.com/files/jquery.autopager-1.0.0.js

        # popd

        # Enable existing modules (disabled during site upgrade).
        drush -l sanctuary --yes pm-enable aggregator
        drush -l sanctuary --yes pm-enable blog
        drush -l sanctuary --yes pm-enable image
        drush -l sanctuary --yes pm-enable profile
        drush -l sanctuary --yes pm-enable search
        drush -l sanctuary --yes pm-enable taxonomy
       
# JLT to check...
# https://drupal.org/node/1043234

            The following extensions will be enabled: taxonomy
            Do you really want to continue? (y/n): y
            Invalid argument supplied for foreach() taxonomy.views.inc:429      [warning]
            Invalid argument supplied for foreach() taxonomy.views.inc:429      [warning]   

        drush -l sanctuary --yes pm-enable trigger

        # (D7) Enable Dashboard and Toolkit modules.
        drush -l sanctuary --yes pm-enable dashboard
        drush -l sanctuary --yes pm-enable shortcut
        drush -l sanctuary --yes pm-enable toolbar
        drush -l sanctuary --yes pm-enable update

        # Enable new modules.
        drush -l sanctuary --yes pm-enable contact_forms      
        drush -l sanctuary --yes pm-enable ecard
        drush -l sanctuary --yes pm-enable filefield_paths
        drush -l sanctuary --yes pm-enable filefield_sources
        drush -l sanctuary --yes pm-enable imagecache_actions
        drush -l sanctuary --yes pm-enable image_effects_text
        drush -l sanctuary --yes pm-enable linkchecker
        drush -l sanctuary --yes pm-enable term_reference_tree
        drush -l sanctuary --yes pm-enable transliteration
        drush -l sanctuary --yes pm-enable menu_import
        drush -l sanctuary --yes pm-enable video_embed_field
        drush -l sanctuary --yes pm-enable media_flickr
        
        # drush -l sanctuary --yes pm-enable libraries
        # drush -l sanctuary --yes pm-enable views_infinite_scroll

        # Clear all caches.
        drush -l sanctuary cc all

----

# DRUPAL 7 SITE CONFIGURATION.

# Login as admin user.
http://uk7.stanleywindrush.org.uk/user

# Complete Media module install.
http://uk7.stanleywindrush.org.uk/admin/config/media/rebuild_types

# Clear cache in Drupal 7 site (disable caching while developing).
http://uk7.stanleywindrush.org.uk/admin/config/development/performance

# Check regional settings.
http://uk7.stanleywindrush.org.uk/admin/config/regional/settings

    Default country:    United Kingdom
	Disable:			Remind users at login if their time zone is not set
	Disable: 			Users may set their own time zone
    
# Check date and time configuration.
http://uk7.stanleywindrush.org.uk/admin/config/regional/date-time

# Replace spaces with a dash for contact forms.
http://uk7.stanleywindrush.org.uk/admin/structure/contact/settings

# Check status report (run cron manually to check available updates).
http://uk7.stanleywindrush.org.uk/admin/reports/status

# Migrate content type fields.
http://uk7.stanleywindrush.org.uk/admin/structure/content_migrate

# Disable display of author and date information.
# http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/pressrelease

# Enable and set default theme (Bartik).
http://uk7.stanleywindrush.org.uk/admin/appearance

# Disable 'Garland' theme.
http://uk7.stanleywindrush.org.uk/admin/appearance

# Set the administration theme when editing or creating content.
http://uk7.stanleywindrush.org.uk/admin/appearance

# Configure default theme (Bartik).
# Disable site name and site slogan.
# Disable default logo.
# Disable default favicon.
http://uk7.stanleywindrush.org.uk/admin/appearance/settings/bartik

# Delete the URL redirect from 404 to node/1535.
# Set 'URL path settings' for 'page not found' ('URL Alias' 404).
http://uk7.stanleywindrush.org.uk/node/1535/edit

# Check default 403 and 404 error pages.
http://uk7.stanleywindrush.org.uk/admin/config/system/site-information

# Override default front page.
http://uk7.stanleywindrush.org.uk/admin/config/system/site-information

    - http://uk7.stanleywindrush.org.uk/home
    + http://uk7.stanleywindrush.org.uk/
    
# Enable 'Insert views' and 'Insert block' filters.
http://uk7.stanleywindrush.org.uk/admin/config/content/formats/3

# Enable 'search block'.
#   Block title:                <none>
#   Region settings:            Header
#   Show block on all pages:    All pages except those listed
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/search/form/configure

# Configure link checker.
# http://uk7.stanleywindrush.org.uk/admin/config/content/linkchecker

----

# DRUPAL 7 DISPLAY MANAGEMENT.

# For each content type, set the image format for the default display.
# For each content type, set the image format for the teasers.
# For each content type, hide all taxonomy fields.
# For each content type, remove 'ecard' from 'Custom display settings'.
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/blog/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/corporate-partnership/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/vacancy/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/event/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/fundraising/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/pressrelease/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/rehome/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/story/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/videoclip/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/volunteer/display
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/webform/display

# No teaser required.
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/page/display

# Delete 'imagepage' content type?
# http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/imagefield

--------
JLT 20130626 up to here...
--------

# Configure each Image field settings.
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/blog/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/corporate-partnership/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/event/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/fundraising/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/page/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/pressrelease/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/rehome/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/story/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/videoclip/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/volunteer/fields/field_imagefield
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/webform/fields/field_imagefield

{{{
	Label:		Photo

	# File (field) path settings.
	Enable:		Alt field
	Enable:		Title field

	# File name.
	[node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]

	# File sources.
	Enable:		Upload (default)
	Enable:		Remote URL textfield

	# File name options.
	Enable:		Transliterate

	# Photo field settings.
	Enabled:	Unlimited
}}}


# Add new 'Document' field to 'Current vacancy' content type.
http://uk7.stanleywindrush.org.uk/vacancy/7204

{{{
	Label:			Document
	Machine name:	field_document
	Field type:		File
	Widget:			File

	# File field (path) settings.
	File path:		vacancy
		
	# Field settings.
	Enable:			Display field
	Enable:			Files displayed by default
	
	# File name.	
	[node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]
	
	# File name options.
	Enable:		Transliterate

	# Allowed file extensions.
	File extensions:	doc, pdf
	Enable:				Description field
	
	# File sources.
	Enable:		Upload (default)
	Enable:		Remote URL textfield
	
	# Document field settings.
	Enable:		Unlimited
}}}

# Add new 'Document' field to 'Page' content type.
http://uk7.stanleywindrush.org.uk/vacancy/7204

{{{
	Label:			Document
	Machine name:	field_document
	Field type:		File
	Widget:			File

	# File field (path) settings.
	File path:		page
		
	# Field settings.
	Enable:			Display field
	Enable:			Files displayed by default
	
	# File name.	
	[node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]
	
	# File name options.
	Enable:		Transliterate

	# Allowed file extensions.
	File extensions:	doc, pdf
	Enable:				Description field
	
	# File sources.
	Enable:		Upload (default)
	Enable:		Remote URL textfield
	
	# Document field settings.
	Enable:		Unlimited
}}}
	
# Delete 'File attachments' field from all content types.
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/
			
{{{
[admin@server~]#

	# Check the files are formatted after uploading.
	pushd /var/local/donkeys-git-donkeys7/drupal-7/files/donkeys/vacancies

		ls -al

		-rw-rw-r--  1 apache apache 339016 Jun 19 12:39 7205-1371645556-application-form-brookfield.pdf

	popd
}}}

----

# DRUPAL 7 DEPRECATED MENUS (to do before importing views).

# Fix side effect of D6 to D7 upgrade to display 'Main menu' correctly.
# Nest all menu items below 'Administration' menu item under 'Management'.
# Move 'Administration' menu item to 'Main menu'.
# Reset all 'Main menu' menu items to put them into default settings.
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/management

# Reset menu items.
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/navigation
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/user-menu
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/management
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-stay-informed

# Delete all views.
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/all_aboard/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/big_bray_2012/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/calendar_est/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/calendar_training/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/carousel_india/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/carousel_project_south_africa/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/carousel_rug_appeal/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/carousel_south_africa/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/copyright/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/events_pageviews/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/homepage/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/jenifer/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/jenifertest/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/news_feed_dat_centres/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/news_feed_est_belfast/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/news_feed_est_birmingham/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/news_feed_est_ivybridge/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/news_feed_est_leeds/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/news_feed_est_manchester/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/news_feed_est_sidmouth/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/press_release_pageviews/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/south_africa/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/tangible/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/training_course_dates/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/training_courses_pagecount/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/training_courses_pageviews/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/veterinarycare/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Appeals/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/AshleyUpdate/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/BeachDonkeys/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Blogger/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Bonaire/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/BonaireGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/BrookfieldFarm/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/BuxtonArrival/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/BuxtonBlogs/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/BuxtonGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/BuxtonNews/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/BuxtonResidents/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Campaigns/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Cameroon/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/CentreNews/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/China/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/CinnamonFoalGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/CurrentVacancies/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Cyprus/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/CyprusGallery/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/DonkeyHealthRelatedLinks/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/DonkeyHistory/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/DonkeyTourWorkshops/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/DonkeyWeek/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/DonkeyWelfare/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Dreams/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EastAxnollerFarm/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Egypt/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EgyptGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/ElisabethSvendsen/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EnvironmentFriendly/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/ESTNews/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Ethiopia/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EthiopiaCommunity/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EthiopiaDirectHelp/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EthiopiaGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventDerbyshire/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventEST/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventSidmouth/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventSponsorDonkey/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventStage1/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventStage2/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventStage4/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventStage5/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/EventStage6/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/FieldOfDreams/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/FosteredDonkeys/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Fundraise/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/FundraisersGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/FundraisingEvents/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Greece/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/India/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/IndiaDelhiGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/IndiaGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Isolation/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Italy/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/ItalyGallery/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/JubileeGallery/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Kenya/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/KenyaGallery/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Lamu/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/LamuGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/LastingGift/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Legacies/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Links/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/LoanSchemeCaseStudies/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/LoanSchemeFosteringForum/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/LoanSchemeGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/LoanSchemeTestimonials/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/LoanSchemeWaitingGallery/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/MeetDonkeys/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Memorials/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Mexico/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/MexicoGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Morocco/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Mules/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Nepal/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/OldVacancies/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/OurFarms/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/PaccombeFarm/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Peru/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/PoisonousPlants/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/PoitouGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/PressRoom/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/ProjectGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/ProjectsDevelopment/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Romania/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/RomaniaGallery/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/SanctuaryWalks/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/SladeHouseFarm/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Spain/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/SpainGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/SupportUs/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Tanzania/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/TownBartonFarm/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/TownBartonFarmGallery/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/TrowFarm/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/UnpublishedPressReleases/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/UnpublishedVacancies/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/VideoClips/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/VideoClipsWelfare/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Webcams/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/WhatsOn/delete
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/WoodsFarm/delete

http://uk7.stanleywindrush.org.uk/admin/structure/views/view/Zambia/delete

# Disable 'frontpage'.
http://uk7.stanleywindrush.org.uk/admin/structure/views/view/frontpage

# Delete all remaining menu items where 'delete' link exist.
# ** not Menu items in 'management' menu **
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/main-menu
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/navigation

http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-stay-informed
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-what-we-do---
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-what-you-can-do
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-who-we-are---

# Remove deprecated menus.
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-stay-informed/delete
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-what-we-do---/delete
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-what-you-can-do/delete
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/menu-who-we-are---/delete

# Recheck...
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/navigation

# Clear cache
http://uk7.stanleywindrush.org.uk/admin/config/development/performance

# Delete old blocks left after deleting views.
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/12/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/14/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/15/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/16/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/17/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/18/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/19/delete
# http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/20/delete (cookie control)
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/21/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/22/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/23/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/25/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/26/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/27/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/29/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/30/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/31/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/32/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/35/delete
http://uk7.stanleywindrush.org.uk/admin/structure/block/manage/block/36/delete

# Clear cache
http://uk7.stanleywindrush.org.uk/admin/config/development/performance

# Delete redundant menu items from 'navigation' from database.
[admin@server]

    mysql --user=${uk7_mysqldrupaluser} --password=${uk7_mysqldrupalpass} ${uk7_mysqldrupaldata}
    # SELECT mlid, menu_name, link_path, link_title FROM menu_links WHERE menu_name='navigation' ORDER BY mlid DESC;
    SELECT mlid, menu_name, link_path, link_title FROM menu_links WHERE link_path LIKE 'view/%' ORDER BY mlid ;

    DELETE FROM menu_links WHERE link_path='view/news';
    DELETE FROM menu_links WHERE link_path='view/newsletters';
    DELETE FROM menu_links WHERE link_path='view/lastinggift';
    DELETE FROM menu_links WHERE link_path='view/memorials';
    DELETE FROM menu_links WHERE link_path='view/lastinggifts';
    DELETE FROM menu_links WHERE link_path='view/china';
    DELETE FROM menu_links WHERE link_path='view/pressroom';
    DELETE FROM menu_links WHERE link_path='view/ireland';
    DELETE FROM menu_links WHERE link_path='view/fieldofdreams';
    DELETE FROM menu_links WHERE link_path='view/podcasts';
    DELETE FROM menu_links WHERE link_path='view/fundraisersnewsletter';
    DELETE FROM menu_links WHERE link_path='view/fieldofdreamsnewsletter';
    DELETE FROM menu_links WHERE link_path='view/donkeysanctuarynewsletter';
    DELETE FROM menu_links WHERE link_path='view/legacies';
    DELETE FROM menu_links WHERE link_path='view/donkidz';
    DELETE FROM menu_links WHERE link_path='view/leavinggift';
    DELETE FROM menu_links WHERE link_path='view/oldvacancies';
    DELETE FROM menu_links WHERE link_path='view/greece';
    DELETE FROM menu_links WHERE link_path='view/fundraise';
    DELETE FROM menu_links WHERE link_path='view/support';
    DELETE FROM menu_links WHERE link_path='view/fundraising';
    DELETE FROM menu_links WHERE link_path='view/farms';
    DELETE FROM menu_links WHERE link_path='view/meetdonkeys';
    DELETE FROM menu_links WHERE link_path='view/poisonousplants';
    DELETE FROM menu_links WHERE link_path='view/environment';
    DELETE FROM menu_links WHERE link_path='view/isolation';
    DELETE FROM menu_links WHERE link_path='view/projects';
    DELETE FROM menu_links WHERE link_path='view/veterinarycare';
    DELETE FROM menu_links WHERE link_path='view/spain';
    DELETE FROM menu_links WHERE link_path='view/italy';
    DELETE FROM menu_links WHERE link_path='view/cyprus';
    DELETE FROM menu_links WHERE link_path='view/beachdonkeys';
    DELETE FROM menu_links WHERE link_path='view/walks';
    DELETE FROM menu_links WHERE link_path='view/fundraisersgallery';
    
    # Remove duplicate 'content' menu item after D6 to D7 upgrade.
    DELETE FROM menu_links WHERE mlid=897 AND menu_name='management';

    # Delete old block themes.
    DELETE FROM block WHERE theme='aberdeen';
    DELETE FROM block WHERE theme='aberdeen-liquid';
    DELETE FROM block WHERE theme='adaptivetheme';
    DELETE FROM block WHERE theme='amadou';
    DELETE FROM block WHERE theme='pippa';
    DELETE FROM block WHERE theme='pushbutton';
    DELETE FROM block WHERE theme='rebrand';
    DELETE FROM block WHERE theme='bluemarine';

    # Delete old block views.
    DELETE FROM block WHERE module='views';
    DELETE FROM block WHERE module='tagadelic';

    # Clear old blocks roles.
    DELETE FROM block_role WHERE module='tagadelic';
    DELETE FROM block_role WHERE module='views';

# Clear cache
http://uk7.stanleywindrush.org.uk/admin/config/development/performance

----

# Populate missing database column with original file names.
[admin@server]

    mysql --user=${uk7_mysqldrupaluser} --password=${uk7_mysqldrupalpass} ${uk7_mysqldrupaldata}

    UPDATE file_managed SET origname = filename WHERE  origname = '' ;
    Query OK, 2297 rows affected (0.29 sec)
    Rows matched: 2297  Changed: 2297  Warnings: 0

    UPDATE file_managed SET type='image' WHERE type='undefined';

# TODO...
Check module status (uninstall until required).
# Uninstall Tracker
# Uninstall Database logging
# Uninstall Help
# Uninstall PHP filter
# Uninstall Statistics
http://uk7.stanleywindrush.org.uk/admin/modules/uninstall

----

# TODO...
# Add content needs to be moved.
http://uk7.stanleywindrush.org.uk/admin/structure/menu/manage/navigation

# Remove duplicate 'Content' menu item.
[admin@server]

    mysql --user=${uk7_mysqldrupaluser} --password=${uk7_mysqldrupalpass} ${uk7_mysqldrupaldata}
    SELECT mlid, menu_name, link_path, link_title FROM menu_links where link_title='Add content';
    +------+----------------+-----------+-------------+
    | mlid | menu_name      | link_path | link_title  |
    +------+----------------+-----------+-------------+
    |   17 | navigation     | node/add  | Add content |
    |  194 | navigation     | node/add  | Add content |
    | 1474 | shortcut-set-1 | node/add  | Add content |
    +------+----------------+-----------+-------------+

    DELETE FROM menu_links WHERE mlid=194;

# Clear cache
http://uk7.stanleywindrush.org.uk/admin/config/development/performance

----

# Configure LightBox2.
# http://uk7.stanleywindrush.org.uk/admin/config/user-interface/lightbox2
# General > Lightbox2 lite > Show image caption

# Configure URL aliases.
# http://drupal.org/node/1266222
http://uk7.stanleywindrush.org.uk/admin/config/search/path/patterns

    # Replace Drupal 6 pattern paths with Drupal 7 replacement patterns.

    [title-raw]     ->  [node:title]
    [nid]           ->  [node:nid]
    [vocab-raw]     ->  [term:name]
    [catpath-raw]   ->  [term:url]
    [user-raw]      ->  [user:name]
    [blog-raw]      ->  [user:name]

----

# DRUPAL 7 PANEL PAGES.

# Enable node add/edit form.
http://uk7.stanleywindrush.org.uk/admin/structure/pages

# Import node add/edit form.
# *** Disable Drupal blocks/regions ***
# *** Check flexipanels are consistent widths ***
# *** Need to include new 'volunteer' variant.
http://uk7.stanleywindrush.org.uk/admin/structure/pages/edit/node_edit

# Import node view variants (override existing variants).
# http://uk7.stanleywindrush.org.uk/admin/structure/pages/edit/node_view

# Import stylizer settings (override existing variants).
# http://uk7.stanleywindrush.org.uk/admin/structure/stylizer/import

# Import mini-panels (override existing).
# *** on hold until theming restarted ***
http://uk7.stanleywindrush.org.uk/admin/structure/mini-panels/import

# JLT TODO...
# Remove 'UK news' and 'International news' from blog types.
# Add 'UK news' and 'International news' to press release types.

----

# DRUPAL 7 USER PROFILE DISPLAYS.
# Profile module has been deprecated as of Drupal 7 but still accessible.
# http://drupal.org/node/1008870
# However views using profile fields are broken.
# Panel for node view (blog) broken.

# Disable the personal contact form by default for new users.
http://uk7.stanleywindrush.org.uk/admin/config/people/accounts

# Hide user history.
# Disable 'Ecard' from Custom display settings.
http://uk7.stanleywindrush.org.uk/admin/config/people/accounts/display

# Add new fields to 'people' account settings.
http://uk7.stanleywindrush.org.uk/admin/config/people/accounts/fields

    Label:          Photo	        (Use existing 'imagefield' renamed to Photo)
    Machine name:   image_field
    Field type:     image
    # of values:    1

    Label:          Blogger name
    Machine name:   blogger_name
    Field type:     text

    Label:          About my Blog	
    Machine name:   my_blog
    Field type:     long text

    Label:          Blog URL	
    Machine name:   blog_url
    Field type:     text

# Manage display fields.
# Add new field data to user accounts.
http://uk7.stanleywindrush.org.uk/user/3/edit

    Photo:          JLT... todo
    Blogger name:   Jenifer Tucker
    About by blog:  Jenifer works at the Sanctuary as our Web Developer. She has loved donkeys ever since she can remember as well as being a donkey owner. One of her donkeys, Little Pippa, is here at the Sanctuary so if you're visiting, drop by to say hello and give Little Pippa a cuddle!
    Blog URL:       http://www.thedonkeysanctuary.org.uk/blogs/jenifer-tucker

----

# DRUPAL 7 IMPORT VIEWS.

# Import views (as and when required).
http://uk7.stanleywindrush.org.uk/admin/structure/views/import
# Blogger
# Recent blog posts

# Check blog display.
http://uk7.stanleywindrush.org.uk/blog/5689

----

# DRUPAL 7 USER PROFILES.

# Remove 'Personal information' once copied over.
http://uk7.stanleywindrush.org.uk/user/3/edit/Personal%20Information

# Remove deprecated custom profile fields.
http://uk7.stanleywindrush.org.uk/admin/config/people/profile/delete/2
http://uk7.stanleywindrush.org.uk/admin/config/people/profile/delete/3
http://uk7.stanleywindrush.org.uk/admin/config/people/profile/delete/4
http://uk7.stanleywindrush.org.uk/admin/config/people/profile/delete/5
http://uk7.stanleywindrush.org.uk/admin/config/people/profile/delete/6
http://uk7.stanleywindrush.org.uk/admin/config/people/profile/delete/8

# Remove 'Profile' module.
[admin@server~]

    pushd /var/local/donkeys-git-donkeys7/drupal-7
    
        drush -l sanctuary --yes pm-disable profile

    popd

# Uninstall 'Profile' module.
http://uk7.stanleywindrush.org.uk/admin/modules/uninstall

----

# DRUPAL 7 CONFIGURE IMAGE CACHE ACTIONS.

# Enable 'Imagecache Canvas Actions'.
http://uk7.stanleywindrush.org.uk/admin/modules

# Overlay logo on 'large' sized images.
http://uk7.stanleywindrush.org.uk/admin/config/media/image-styles/edit/large

	# Override defaults.
    Effect:     Overlay (watermark)
    File path:  /files/donkeys/donkeysanctuarymastertablogo-sanctuary-20120807.png
    
# Add 'copyright' notice to 'large' images.
http://www.example.com/admin/config/media/image-styles/edit/large

    Effect:             Text
    Font size:          10
    x offset            0
    y offset            Bottom
    Horizontal align    Left
    Vertical align      Bottom
    Hex                 000000
    Opacity             100
    Angle               0
    Font filename:      sites/all/modules/imagecache_actions/image_effects_text/Komika_display.ttf
    Text source:        Static text
    Text:               Copyright. The Donkey Sanctuary.
    
----

# DRUPAL 7 CONTACT FORM.

# Configure contact form.
# Create custom content forms.
# Adds specific titles for 'contact' pages (for example: Paccombe Training Centre).
http://uk7.stanleywindrush.org.uk/admin/structure/contact/settings

<strong>By post</strong>

<p>The Donkey Sanctuary
<br/>
Sidmouth
<br/>
Devon
<br/>
EX10 0NU</p>

<strong>By telephone</strong>

<p>+44 (0) 1395 578222 (8.30 am to 4.30 pm, Monday - Friday)</p>

<strong>By fax</strong>

<p>+44 (0) 1395 579266</p>

<strong>By email</strong>

<p>If you would like to send <strong>@category</strong> a message, please use the contact form below.</p>

----

# DRUPAL 7 INDIVIDUAL CONTACT FORMS.

# Replace instances of old legacy href code (/trainingcourses).
http://uk7.stanleywindrush.org.uk/trainingcourses

  - If you are interested in attending one of the courses, please 
    <a href="?q=contact&category=Paccombe%20Training%20Centre&subject=Training Centre courses">contact us</a> or telephone 01395 597644.

  + If you are interested in attending one of the courses, please <a title="Contact us" href="/contact/Paccombe Training Centre">contact us</a>.

# Example of custom content form (eg telephone number - 01395 597644 -replaces switchboard number).
http://uk7.stanleywindrush.org.uk/admin/structure/contact/edit/27

    <p><strong>By post</strong></p>

    <p>The Donkey Sanctuary
    <br/>Sidmouth
    <br/>Devon
    <br/>EX10 0NU</p>

    <p><strong>By telephone</strong></p>

    <p>+44 (0) 1395 597644 (8.30 am to 4.30 pm, Monday - Friday)</p>

    <p><strong>By fax</strong></p>

    <p>+44 (0) 1395 579266</p>

    <p><strong>By email</strong></p>

    <p>If you would like to send <strong>@category</strong> a message, please use the contact form below.</p>

----------------------------
JLT up to here...
----------------------------

# Note: Media_Flickr
#
# To use it, enable the Media module. You'll also need to apply for a Flickr API
# key, from http://www.flickr.com/services/api/keys, which you will paste into
# the settings page at /admin/configure/media/media_flickr.
# http://uk7.stanleywindrush.org.uk/admin/config/media/media_flickr
# Set up tutorial.
# http://drupal.org/node/1406068#comment-5630428

# Add flickr api key.
www.example.com/admin/config/media/media_flickr

# Configure 'video' file types - manage file display.
www.example.com/admin/config/media/file-types/manage/video/file-display

    # Enabled displays.
    Enable:     Flickr photoset

    # Display settings, configure the width, height.
    Width:     560
    Height:    340

# Configure content types to add flickr photosets.
http://uk7.stanleywindrush.org.uk/admin/structure/types/manage/blog/fields

    # Add new field.
    Type:     File
    Widget:   Embedded file selector

    # Field settings.
    Upload destination:     Public files

    # Custom display settings.
    Disable:  Teaser

    # Manage display.
    Format:     Rendered File
    View mode:  Default

# Clear cache.
www.example.com/admin/config/development/performance

----

# DRUPAL 7 CALENDAR.

# TODO... Try one calendar to display different categories (dropdown list of categories).
# Delete separate event views.
# What's on at Sidmouth
# What's on at each of the riding therapy centres
# Training courses
http://uk7.stanleywindrush.org.uk/calendar

----

# DRUPAL 7 USERS.

# Remove 'Donkidz club' role.
# Disable user 'Marian Gumbrell'
# Disable user 'DonkDave'
# Possibly remove all content associated with these two users?

----

# JLT TODO...
# Check the roles assigned to text formats (check alongside Drupal 6 /admin/settings/filters). 
http://uk7.stanleywindrush.org.uk/admin/config/content/formats

----
   
# TODO... fix.
# Duplicating images on the server (in files and styles directories).

----

# Enable users to add pictures.
http://drupal.org/node/22271
# Modify 'bloggers' view to use User: Picture
# Remove 'Photo' imagefield.
# Use pictures in post
http://uk7.stanleywindrush.org.uk/admin/appearance/settings

----

# DRUPAL 7 FULL LIST OF ADMINISTRATIVE TASKS FOR EACH MODULE.
http://uk7.stanleywindrush.org.uk/admin/index

# RE-ENABLE SITE CACHING

# UPDATE ROBOTSTXT.

----

# THE END.
```