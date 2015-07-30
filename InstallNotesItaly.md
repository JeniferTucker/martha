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
        /dev/xvda              24G   21G  1.4G  94% /
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
    
# Get a working copy of all the italy websites from codebasehq repository.
[admin@server]

    pushd /var/local/

        # Create our 'master' copy (only need to do this once)
    	# git clone git@codebasehq.com:ixis/donkey-italy/ds.git donkeys-git-master

        # Create a new 'dev4' copy (clone) of the GIT codebase.
        git clone donkeys-git-master donkeys-git-dev4

    popd

# Update our master copy from the codebasehq repository.
[admin@server]

    pushd /var/local/donkeys-git-master

       git pull

    popd

# Update our 'dev4' working copy from our 'master' copy.
[admin@server]

    pushd /var/local/donkeys-git-dev4

        git pull

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
italy
spain
welfare
EOF

# Create a database dump for each mySQL database.
for database in $(cat /var/local/backups/ixis/backup-databases.txt)
do
    echo "Database is : ${database}"

    backupsite=${database}

    backupuser=jenifer
    backuphost=ixis.example.com
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

    # Add 'dev4' to settings.
	vi ~/local-settings.txt

        #[dev4]
        dev4_mysqldrupaluser=*****
        dev4_mysqldrupalpass=*****
        dev4_mysqldrupaldata=*****

   # Load settings and functions.
	source ~/local-settings.txt

   # Update existing $db_url in settings file.
   pushd /var/local/donkeys-git-dev4/drupal-6/sites/italy
    $db_url = 'mysql://dev4_mysqldrupaluser:dev4_mysqldrupalpass@localhost/dev4_mysqldrupaldata';

   popd

----

# Add the DNS aliases
# Go to easyDNS.

----

# Create virtual host files.
[admin@server~]

    pushd /etc/httpd/conf.d

cat > six-italy.example.com.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@thedonkeyitaly.org.uk
  ServerName  six-italy.example.com

  LogLevel warn
  ErrorLog  /var/log/httpd/six-italy.example.com.error.log
  CustomLog /var/log/httpd/six-italy.example.com.access.log combined

  DocumentRoot /var/local/donkeys-git-dev4/drupal-6
  <Directory /var/local/donkeys-git-dev4/drupal-6>

  Options Indexes FollowSymLinks MultiViews
  AllowOverride All
  Order allow,deny
  allow from all

  </Directory>

</VirtualHost>
EOF

cat > dev4.example.com.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@thedonkeyitaly.org.uk
  ServerName  dev4.example.com

  LogLevel warn
  ErrorLog  /var/log/httpd/dev4.example.com.error.log
  CustomLog /var/log/httpd/dev4.example.com.access.log combined

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

# DRUPAL 6 DATABASE.

# Create 'italy' database.
[admin@server~]

	# Check mySQL is running.
	/etc/init.d/mysqld status

    # Drag/drop SFTP network file from gamma to pippa.

    # Unzip file.
    gunzip /var/local/backups/ixis/italy-ixis.example.com-20130527180708.sql.gz
        
	# Load local settings.
	source ~/local-settings.txt
	backup=italy-ixis.example.com-20130527180708.sql

    # Drop existing database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="DROP DATABASE ${italy_mysqldrupaldata}"

    # Create database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="CREATE DATABASE ${italy_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

	# Grant permissions.
   	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="GRANT ALL ON ${italy_mysqldrupaldata}.* TO '${italy_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${italy_mysqldrupalpass}'"

    # Load latest backup into empty database.
        mysql \
            --user=${italy_mysqldrupaluser} \
            --password=${italy_mysqldrupalpass} \
            --database=${italy_mysqldrupaldata} \
            < /var/local/backups/ixis/${backup}

    # Connect to database.
    mysql --user=${italy_mysqldrupaluser} --password=${italy_mysqldrupalpass} ${italy_mysqldrupaldata} \
      --execute="SELECT COUNT(uid) FROM users;"

----

# DRUPAL 7 DATABASE.

# Create 'dev4' empty database.
[admin@server~]

    # Drop existing database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="DROP DATABASE ${dev4_mysqldrupaldata}"

    # Create database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="CREATE DATABASE ${dev4_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

	# Grant permissions.
   	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="GRANT ALL ON ${dev4_mysqldrupaldata}.* TO '${dev4_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${dev4_mysqldrupalpass}'"

    # Show existing databases.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="SHOW DATABASES"

    # Check database connection.
    mysql --user=${dev4_mysqldrupaluser} --password=${dev4_mysqldrupalpass} ${dev4_mysqldrupaldata} \
      --execute="SELECT VERSION()"

----

# DRUPAL 6 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

pushd /var/local/donkeys-git-dev4/drupal-6/sites

    ln -s italy six-italy.example.com

popd

----

# DRUPAL 6 FILES DIRECTORY.

# Create symbolic links.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-6/files

        # Create symbolic links.
        ln -s italy six-italy.example.com

    popd

----

# DRUPAL 6 CORE AND MODULES.

# Ensure all contributed modules are up to date.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-6

        # Clear drush cache.
        drush cc drush

        # Check available updates for enabled modules (automatically runs database updates if required).
        drush -l italy pm-update

	popd

# Remove uninstalled modules from the modules directory.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-6/sites/all/modules

        ???

	popd

# Disable all themes no longer required.
# http://six-italy.example.com/admin/build/themes

    # Adaptivetheme
    # Blogbuzz

# Remove old themes from the themes directory.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-6/sites/all/themes

        rm -rf adaptivetheme
        rm -rf blogbuzz

	popd

----

# DRUSH UPGRADE.

# Install Drupal 7 modules required to run Drush site upgrade (if not already on VM).
[admin@server~]

        drush dl drush_sup

----

# DRUPAL 6 FILE PERMISSIONS.

# Fix file permissions after downloading from git repository.
[admin@server~]

pushd /var/local/donkeys-git-dev4/drupal-6

    id apache
    # uid=48(apache) gid=48(apache) groups=48(apache)

    chgrp -R apache files
    chmod -R g+w    files

    pushd files

        find . -type d -exec chmod g+ws '{}' \;

    popd

popd

----

# DRUPAL 6 IMAGE ATTACH CONVERTER.

# Source:
# http://drupal.org/node/201983

    # Login as admin user.
    http://six-italy.example.com/en/user
    
    # Error messages.         
    Il file selezionato /tmp/filedj9XDN non può essere caricato, perché la destinazione files/italy/languages/it_19c836a2fce7d419cd342227b1bac68f.js non è configurata in modo appropriato.
    Il file selezionato /tmp/fileA3Oci6 non può essere caricato, perché la destinazione files/italy/languages/it_19c836a2fce7d419cd342227b1bac68f.js non è configurata in modo appropriato.

    # Clear cached data.
    http://six-italy.example.com/en/admin/settings/performance

    # Enable 'Filefield' and 'ImageField' module.
    http://six-italy.example.com/en/admin/build/modules

    # Create prerequisite imagefield type.
    # The imagefield field that you have already created and configured as per Prerequisites.
    # $field_name = 'field_image_field'
    # The content type that you have already created as per Prerequisites.
    # $type_name = 'imagefield';
    http://six-italy.example.com/en/admin/content/types/add

        Name:           imagefield
        Type:           imagefield
        Description:    Prerequisite imagefield type for image converter.
    
    http://six-italy.example.com/en/admin/content/node-type/imagefield/fields
    
        New field:      imagefield
        Field_          image_field
        Data type:      File
        Form element:   Image
        
        Number of values:   Unlimited
        Description field:  Disabled

    # Add new 'image_field' to content types (where attached images are enabled).
    http://six-italy.example.com/en/admin/content/node-type/appeal/fields
    http://six-italy.example.com/en/admin/content/node-type/blog/fields
    http://six-italy.example.com/en/admin/content/node-type/campaign/fields
    http://six-italy.example.com/en/admin/content/node-type/event/fields
    http://six-italy.example.com/en/admin/content/node-type/page/fields
    http://six-italy.example.com/en/admin/content/node-type/pressrelease/fields
    http://six-italy.example.com/en/admin/content/node-type/story/fields
    http://six-italy.example.com/en/admin/content/node-type/therapy/fields
    http://six-italy.example.com/en/admin/content/node-type/update/fields
    http://six-italy.example.com/en/admin/content/node-type/webform/fields
    http://six-italy.example.com/en/admin/content/node-type/welcome/fields
    http://six-italy.example.com/en/admin/content/node-type/welfair/fields
    
    # Migrate all image module nodes to imagefields.
    pushd /var/local/donkeys-git-dev4/drupal-6

        # Copy migration script to Drupal 6 core.
        cp /var/local/scripts/imagefield_migrate.php .

        # Run script.
        http://six-italy.example.com/imagefield_migrate.php

            # - content_type_imagefield populated.
            # - content_field_image_field populated.
            # - files table updated
            # - 245 image_attach relationships were migrated.

        # Remove script to prevent re-running.
        rm -rf imagefield_migrate.php

   popd

----

# DRUPAL 6 IMAGE HANDLING.

# Disable image module (image, image attach, image gallery).
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-6

        # Disable image module and extensions.
        drush -l italy --yes pm-disable image

        # Download new image handling modules.
        drush -l italy dl imagecache 
        drush -l italy dl imageapi

        # Clear drush cache.
        drush cache-clear drush

        # Enable modules.
        drush -l italy --yes pm-enable imageapi
        drush -l italy --yes pm-enable imagecache
        drush -l italy --yes pm-enable imagecache_ui
        drush -l italy --yes pm-enable imageapi_gd

    popd

# Remove image module.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-6/sites/all/modules

         rm -rf image

	popd

----

# DRUPAL 6 THEMES.

# Disable 'rebrand' and 'marinelli' themes, set 'garland' as default theme.
http://six-italy.example.com/en/admin/build/themes

# Clear cached data.
http://six-italy.example.com/en/admin/settings/performance

# Remove 'amadou' theme and 'rebrand' sub-theme from the themes directory.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-6/sites/all/themes

        # JLT TODO... remove 'pippa' theme.

        rm -rf amadou
        rm -rf rebrand

	popd

----

# DRUPAL 6 UPGRADE TO DRUPAL 7.

# Create alias file.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-6/sites/italy

        # Create alias file.
cat > /var/local/donkeys-git-dev4/drupal-6/sites/italy/dev4.alias.drushrc.php << EOF
<?php
  \$aliases['dev4'] = array(
    'root'   => '/var/local/donkeys-git-dev4/drupal-7',
    'uri'    => 'italy',
    'db-url' => 'mysql://${dev4_mysqldrupaluser}:${dev4_mysqldrupalpass}@localhost/${dev4_mysqldrupaldata}',
  );
EOF

    popd

----

# Run site-upgrade command.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-6

20
        # Run site upgrade.
        drush -l italy site-upgrade @dev4

             Project               Titolo                     Installed                      Available       Status                
             module_filter         Module filter              6.x-1.7                        7.x-2.x-dev     Recommended           
             favicon               Favicon                    6.x-1.0                        7.x-1.x-dev     Recommended           
             menu_breadcrumb       Menu breadcrumb            6.x-1.3                        7.x-1.3         Recommended           
             pathauto              Pathauto                   6.x-1.6                        7.x-1.2         Recommended           
             robotstxt             robots.txt                 6.x-1.3                        7.x-1.1         Recommended           
             site_map              Site map                   6.x-2.2                        7.x-1.0         Recommended           
             token                 Token                      6.x-1.19                       7.x-1.5         Recommended           
             workspace             Workspace                  6.x-1.3                        7.x-1.x-dev     Recommended           
             varnish               Varnish                    6.x-1.2                        7.x-1.0-beta2   Recommended           
             cck                   Content                    6.x-2.9                        7.x-2.x-dev     Supported,Development 
             calendar              Calendar                   6.x-2.4                        7.x-3.4         Recommended           
             date                  Date Timezone              6.x-2.9                        7.x-2.6         Recommended           
             i18n                  Taxonomy translation       6.x-1.10                       7.x-1.8         Recommended           
             languageicons         Language icons             6.x-2.1                        7.x-1.0         Recommended           
             translation_overview  Translation overview       6.x-2.4                        7.x-2.0-beta1   Recommended           
             translation_table     Translation table          6.x-1.4                        7.x-1.0-beta1   Recommended           
             captcha               CAPTCHA                    6.x-2.4                        7.x-1.0-beta2   Supported             
             google_analytics      Google Analytics           6.x-3.5                        7.x-2.x-dev     Recommended           
             extlink               External Links             6.x-1.11                       7.x-1.12        Recommended           
             insert_view           Insert view                6.x-2.0                        7.x-2.0         Recommended           
             views                 Views UI                   6.x-2.16                       7.x-3.7         Recommended           
             webform               Webform                    6.x-3.19                       7.x-4.0-alpha6  Recommended           
             redirect              Redirect                   (path_redirect)                7.x-1.0-rc1     Recommended           
             bundle_copy           Bundle copy                (content_copy)                 7.x-2.x-dev     Recommended           
             field_permissions     Field Permissions          (content_permissions)          7.x-1.0-beta2   Recommended           
             field_group           Field group                (fieldgroup)                   7.x-2.x-dev     Recommended           
             ctools                Chaos tool suite (ctools)  (views_export)                 7.x-1.3         Recommended           
             references            References                 (nodereference,userreference)  7.x-2.1         Recommended        

            No .info file could be found for i18nblocks                                                                                                        [warning]
            No .info file could be found for i18ncontent                                                                                                       [warning]
            No .info file could be found for i18nmenu                                                                                                          [warning]
            No .info file could be found for i18nstrings                                                                                                       [warning]
            No .info file could be found for i18ntaxonomy                                                                                                      [warning]
         
    popd

----

# DRUPAL 7 OVERRIDES.

# Remove robots.txt file (using robotsTxt module).
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-7

        mv robots.txt robots.txt.20130603

    popd

----

# DRUPAL 7 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

pushd /var/local/donkeys-git-dev4/drupal-7/sites

    ln -s italy dev4.thedonkeyitaly.org.uk
    ln -s italy dev4.example.com

popd

----

# DRUPAL 7 FILES DIRECTORY.

[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7/files

        mkdir files
        mkdir files/italy

        cd files
                
        # Create symbolic links.
        ln -s italy dev4.example.com

    popd

    # Set file permissions.
    pushd /var/local/donkeys-git-dev4/drupal-7

        chgrp -R apache files
        chmod -R g+w    files

    popd

    pushd /var/local/donkeys-git-dev4/drupal-7/files
        
        chmod o-w       italy
        find . -type d -exec chmod g+ws '{}' \;

    popd

----

# DRUPAL 7 MODULES.

# Add new contributed modules.
[admin@server~]

JLT ** not sure what modules I need here **

    pushd /var/local/donkeys-git-dev4/drupal-7

        # Enable existing modules (disabled during site upgrade).
        drush -l italy --yes pm-enable aggregator
        drush -l italy --yes pm-enable blog
        drush -l italy --yes pm-enable image
# ?     drush -l italy --yes pm-enable profile
        drush -l italy --yes pm-enable search
        drush -l italy --yes pm-enable taxonomy

        # Download replacement modules.
        drush -l italy dl contact_forms
        drush -l italy dl transliteration

        # Download new modules.
        drush -l italy dl filefield_paths
        drush -l italy dl filefield_sources
        drush -l italy dl imagecache_actions
        drush -l italy dl linkchecker
        drush -l italy dl term_reference_tree

        # Video embed field module.
        drush -l italy dl video_embed_field

        # Media Flickr module.
        drush -l italy dl media_flickr

        drush -l italy dl libraries
        drush -l italy dl views_infinite_scroll

        pushd sites/all

             mkdir libraries
             mkdir autopager

        popd

        pushd /var/local/donkeys-git-dev4/drupal-7/sites/all/autopager

             wget http://jquery-autopager.googlecode.com/files/jquery.autopager-1.0.0.js

        popd
       
        drush -l italy --yes pm-enable trigger

        # (D7) Enable Dashboard and Toolkit modules.
        drush -l italy --yes pm-enable dashboard
        drush -l italy --yes pm-enable toolbar
        drush -l italy --yes pm-enable update

        # Enable new modules.
        drush -l italy --yes pm-enable contact_forms      

        drush -l italy --yes pm-enable ecard
        drush -l italy --yes pm-enable filefield_paths
        drush -l italy --yes pm-enable filefield_sources
        drush -l italy --yes pm-enable imagecache_actions
        drush -l italy --yes pm-enable linkchecker
        drush -l italy --yes pm-enable term_reference_tree
        drush -l italy --yes pm-enable transliteration

        drush -l italy --yes pm-enable video_embed_field
        drush -l italy --yes pm-enable media_flickr
        
        drush -l italy --yes pm-enable libraries
        drush -l italy --yes pm-enable views_infinite_scroll

        # Clear all caches.
        drush -l italy cc all

----

# Note: Media_Flickr
#
# To use it, enable the Media module. You'll also need to apply for a Flickr API
# key, from http://www.flickr.com/services/api/keys, which you will paste into
# the settings page at /admin/configure/media/media_flickr.
# http://dev4.example.com/admin/config/media/media_flickr
# Set up tutorial.
# http://drupal.org/node/1406068#comment-5630428

----

# DRUPAL 7 THEMES.

# Add new contributed themes.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7

        drush dl marinelli

    popd

----

# DRUPAL 7 SITE CONFIGURATION.

# Login as admin user.
http://dev4.example.com/en/user

# Clear cache in Drupal 7 site (disable caching while developing).
http://dev4.example.com/admin/config/development/performance

# Complete Media module install.
http://dev4.example.com/admin/config/media/rebuild_types

# Check regional settings.
http://dev4.example.com/admin/config/regional/settings

    Default country:    United Kingdom
    Timezone new users: Default time zone
    
# Check date and time configuration.
http://dev4.example.com/admin/config/regional/date-time

# Replace spaces with a dash for contact forms.
http://dev4.example.com/admin/structure/contact/settings

# Check status report (run cron manually to check available updates).
http://dev4.example.com/admin/reports/status

# Migrate content type fields.
http://dev4.example.com/admin/structure/content_migrate

# Disable display of author and date information.
http://dev4.example.com/admin/structure/types/manage/pressrelease

# Enable and set as default the 'avocet' sub-theme.
http://dev4.example.com/admin/appearance

# Disable site name and site slogan.
# Disable default logo (files/donkeys/images/DonkeyitalyMasterTabLogo-italy-20120807.png)
# Disable default favicon (files/donkeys/images/rebrand_favicon.png).
# Disable CSS3 settings.
# Add custom path to custom logo and icon.
http://dev4.example.com/admin/appearance/settings/avocet

# Delete the URL redirect from 404 to node/1535.
http://dev4.example.com/admin/config/search/redirect

# Set 'URL path settings' for 'page not found' ('URL Alias' 404).
http://dev4.example.com/node/1535/edit

# Check default 403 and 404 error pages.
http://dev4.example.com/admin/config/system/site-information

# Override default front page.
http://dev4.example.com/admin/config/system/site-information

# Enable 'insert block' and 'insert views' filters.
http://dev4.example.com/admin/config/content/formats/3

# Enable 'search block'.
# Block title <none>
# Add to search region in 'avocet' (default) theme
# Show block on all pages
http://dev4.example.com/admin/structure/block/manage/search/form/configure

# Configure link checker.
http://dev4.example.com/admin/config/content/linkchecker

# Configure private file system path.
http://dev4.example.com/admin/config/media/file-system
    
    Public file system path:    /files/donkeys
    Private file system path    /var/local/donkeys-git-dev4/files-7
    Temporary directory :       /files/donkeys/tmp

# Convert uppercase strings, to lowercase (*are we sure*).
# http://dev4.example.com/admin/config/media/file-system/transliteration

----

# DRUPAL 7 TEASER DISPLAY MANAGEMENT.

# For each content type, set the image format for the teasers.
http://dev4.example.com/admin/structure/types/manage/blog/display/teaser

# For each content type, set the image format for the default displays.
http://dev4.example.com/admin/structure/types/manage/blog/display

# For each content type, hide all taxonomy fields.
http://dev4.example.com/admin/structure/types/manage/blog/display

# Click on clog (righthand side of format).

    Field:  Image
    Label:  Hidden
    Format:
        Image style:    thumbnail
        Link to:        content

    Field:  Body
    Label:  Hidden
    Format: Trimmed (trim length: 800)

    Hidden: File attachments                ---> Look at replace File upload with CCK where content types use this old method.
    Label:  Hidden
    Format: Hidden

----

# DRUPAL 7 DEPRECATED MENUS (to do before importing views).

# Reset menu items.
http://dev4.example.com/admin/structure/menu/manage/navigation
http://dev4.example.com/admin/structure/menu/manage/user
http://dev4.example.com/admin/structure/menu/manage/management

# Delete all views.
http://dev4.example.com/admin/structure/views/view/all_aboard/delete
http://dev4.example.com/admin/structure/views/view/big_bray_2012/delete
http://dev4.example.com/admin/structure/views/view/calendar_est/delete
http://dev4.example.com/admin/structure/views/view/calendar_training/delete
http://dev4.example.com/admin/structure/views/view/carousel_india/delete
http://dev4.example.com/admin/structure/views/view/carousel_project_south_africa/delete
http://dev4.example.com/admin/structure/views/view/carousel_rug_appeal/delete
http://dev4.example.com/admin/structure/views/view/carousel_south_africa/delete
http://dev4.example.com/admin/structure/views/view/copyright/delete
http://dev4.example.com/admin/structure/views/view/events_pageviews/delete
http://dev4.example.com/admin/structure/views/view/homepage/delete
http://dev4.example.com/admin/structure/views/view/jenifer/delete
http://dev4.example.com/admin/structure/views/view/news_feed_dat_centres/delete
http://dev4.example.com/admin/structure/views/view/news_feed_est_belfast/delete
http://dev4.example.com/admin/structure/views/view/news_feed_est_birmingham/delete
http://dev4.example.com/admin/structure/views/view/news_feed_est_ivybridge/delete
http://dev4.example.com/admin/structure/views/view/news_feed_est_leeds/delete
http://dev4.example.com/admin/structure/views/view/news_feed_est_manchester/delete
http://dev4.example.com/admin/structure/views/view/news_feed_est_sidmouth/delete
http://dev4.example.com/admin/structure/views/view/press_release_pageviews/delete
http://dev4.example.com/admin/structure/views/view/south_africa/delete
http://dev4.example.com/admin/structure/views/view/tangible/delete
http://dev4.example.com/admin/structure/views/view/training_courses_pagecount/delete
http://dev4.example.com/admin/structure/views/view/training_courses_pageviews/delete
http://dev4.example.com/admin/structure/views/view/veterinarycare/delete

http://dev4.example.com/admin/structure/views/view/Appeals/delete
http://dev4.example.com/admin/structure/views/view/AshleyUpdate/delete

http://dev4.example.com/admin/structure/views/view/BeachDonkeys/delete
http://dev4.example.com/admin/structure/views/view/Blogger/delete
http://dev4.example.com/admin/structure/views/view/Bonaire/delete
http://dev4.example.com/admin/structure/views/view/BonaireGallery/delete
http://dev4.example.com/admin/structure/views/view/BrookfieldFarm/delete
http://dev4.example.com/admin/structure/views/view/BuxtonArrival/delete
http://dev4.example.com/admin/structure/views/view/BuxtonBlogs/delete
http://dev4.example.com/admin/structure/views/view/BuxtonGallery/delete
http://dev4.example.com/admin/structure/views/view/BuxtonNews/delete
http://dev4.example.com/admin/structure/views/view/BuxtonResidents/delete

http://dev4.example.com/admin/structure/views/view/Campaigns/delete
http://dev4.example.com/admin/structure/views/view/Cameroon/delete
http://dev4.example.com/admin/structure/views/view/CentreNews/delete
http://dev4.example.com/admin/structure/views/view/China/delete
http://dev4.example.com/admin/structure/views/view/CinnamonFoalGallery/delete
http://dev4.example.com/admin/structure/views/view/CurrentVacancies/delete
http://dev4.example.com/admin/structure/views/view/Cyprus/delete
http://dev4.example.com/admin/structure/views/view/CyprusGallery/delete

http://dev4.example.com/admin/structure/views/view/DonkeyHealthRelatedLinks/delete
http://dev4.example.com/admin/structure/views/view/DonkeyHistory/delete
http://dev4.example.com/admin/structure/views/view/DonkeyTourWorkshops/delete
http://dev4.example.com/admin/structure/views/view/DonkeyWeek/delete
http://dev4.example.com/admin/structure/views/view/DonkeyWelfare/delete
http://dev4.example.com/admin/structure/views/view/Dreams/delete

http://dev4.example.com/admin/structure/views/view/EastAxnollerFarm/delete
http://dev4.example.com/admin/structure/views/view/Egypt/delete
http://dev4.example.com/admin/structure/views/view/EgyptGallery/delete
http://dev4.example.com/admin/structure/views/view/ElisabethSvendsen/delete
http://dev4.example.com/admin/structure/views/view/EnvironmentFriendly/delete
http://dev4.example.com/admin/structure/views/view/ESTNews/delete
http://dev4.example.com/admin/structure/views/view/Ethiopia/delete
http://dev4.example.com/admin/structure/views/view/EthiopiaCommunity/delete
http://dev4.example.com/admin/structure/views/view/EthiopiaDirectHelp/delete
http://dev4.example.com/admin/structure/views/view/EthiopiaGallery/delete
http://dev4.example.com/admin/structure/views/view/EventDerbyshire/delete
http://dev4.example.com/admin/structure/views/view/EventEST/delete
http://dev4.example.com/admin/structure/views/view/EventSidmouth/delete
http://dev4.example.com/admin/structure/views/view/EventSponsorDonkey/delete
http://dev4.example.com/admin/structure/views/view/EventStage1/delete
http://dev4.example.com/admin/structure/views/view/EventStage2/delete
http://dev4.example.com/admin/structure/views/view/EventStage4/delete
http://dev4.example.com/admin/structure/views/view/EventStage5/delete
http://dev4.example.com/admin/structure/views/view/EventStage6/delete

http://dev4.example.com/admin/structure/views/view/FieldOfDreams/delete
http://dev4.example.com/admin/structure/views/view/FosteredDonkeys/delete
http://dev4.example.com/admin/structure/views/view/Fundraise/delete
http://dev4.example.com/admin/structure/views/view/FundraisersGallery/delete
http://dev4.example.com/admin/structure/views/view/FundraisingEvents/delete

http://dev4.example.com/admin/structure/views/view/Greece/delete

http://dev4.example.com/admin/structure/views/view/India/delete
http://dev4.example.com/admin/structure/views/view/IndiaDelhiGallery/delete
http://dev4.example.com/admin/structure/views/view/IndiaGallery/delete
http://dev4.example.com/admin/structure/views/view/Isolation/delete
http://dev4.example.com/admin/structure/views/view/Italy/delete
http://dev4.example.com/admin/structure/views/view/ItalyGallery/delete

http://dev4.example.com/admin/structure/views/view/JubileeGallery/delete

http://dev4.example.com/admin/structure/views/view/Kenya/delete
http://dev4.example.com/admin/structure/views/view/KenyaGallery/delete

http://dev4.example.com/admin/structure/views/view/Lamu/delete
http://dev4.example.com/admin/structure/views/view/LamuGallery/delete
http://dev4.example.com/admin/structure/views/view/LastingGift/delete
http://dev4.example.com/admin/structure/views/view/Legacies/delete
http://dev4.example.com/admin/structure/views/view/Links/delete
http://dev4.example.com/admin/structure/views/view/LoanSchemeCaseStudies/delete
http://dev4.example.com/admin/structure/views/view/LoanSchemeFosteringForum/delete
http://dev4.example.com/admin/structure/views/view/LoanSchemeGallery/delete
http://dev4.example.com/admin/structure/views/view/LoanSchemeTestimonials/delete
http://dev4.example.com/admin/structure/views/view/LoanSchemeWaitingGallery/delete

http://dev4.example.com/admin/structure/views/view/MeetDonkeys/delete
http://dev4.example.com/admin/structure/views/view/Memorials/delete
http://dev4.example.com/admin/structure/views/view/Mexico/delete
http://dev4.example.com/admin/structure/views/view/MexicoGallery/delete
http://dev4.example.com/admin/structure/views/view/Morocco/delete
http://dev4.example.com/admin/structure/views/view/Mules/delete

http://dev4.example.com/admin/structure/views/view/Nepal/delete

http://dev4.example.com/admin/structure/views/view/OldVacancies/delete
http://dev4.example.com/admin/structure/views/view/OurFarms/delete

http://dev4.example.com/admin/structure/views/view/PaccombeFarm/delete
http://dev4.example.com/admin/structure/views/view/Peru/delete
http://dev4.example.com/admin/structure/views/view/PoisonousPlants/delete
http://dev4.example.com/admin/structure/views/view/PoitouGallery/delete
http://dev4.example.com/admin/structure/views/view/PressRoom/delete
http://dev4.example.com/admin/structure/views/view/ProjectGallery/delete
http://dev4.example.com/admin/structure/views/view/ProjectsDevelopment/delete

http://dev4.example.com/admin/structure/views/view/Romania/delete
http://dev4.example.com/admin/structure/views/view/RomaniaGallery/delete

http://dev4.example.com/admin/structure/views/view/italyWalks/delete
http://dev4.example.com/admin/structure/views/view/SladeHouseFarm/delete
http://dev4.example.com/admin/structure/views/view/Spain/delete
http://dev4.example.com/admin/structure/views/view/SpainGallery/delete
http://dev4.example.com/admin/structure/views/view/SupportUs/delete

http://dev4.example.com/admin/structure/views/view/Tanzania/delete
http://dev4.example.com/admin/structure/views/view/TownBartonFarm/delete
http://dev4.example.com/admin/structure/views/view/TownBartonFarmGallery/delete
http://dev4.example.com/admin/structure/views/view/TrowFarm/delete

http://dev4.example.com/admin/structure/views/view/UnpublishedPressReleases/delete
http://dev4.example.com/admin/structure/views/view/UnpublishedVacancies/delete

http://dev4.example.com/admin/structure/views/view/VideoClips/delete
http://dev4.example.com/admin/structure/views/view/VideoClipsWelfare/delete

http://dev4.example.com/admin/structure/views/view/Webcams/delete
http://dev4.example.com/admin/structure/views/view/WhatsOn/delete
http://dev4.example.com/admin/structure/views/view/WoodsFarm/delete

http://dev4.example.com/admin/structure/views/view/Zambia/delete

# Disable 'frontpage'.
http://dev4.example.com/admin/structure/views/view/frontpage/disable

# Delete redundant menu items from 'navigation' from database.
[admin@server]

    mysql --user=${dev4_mysqldrupaluser} --password=${dev4_mysqldrupalpass} ${dev4_mysqldrupaldata}
    SELECT mlid, menu_name, link_path, link_title FROM menu_links WHERE menu_name='navigation' ORDER BY link_path DESC;

    DELETE FROM menu_links WHERE mlid=84;
    DELETE FROM menu_links WHERE mlid=85;
    DELETE FROM menu_links WHERE mlid=86;
    DELETE FROM menu_links WHERE mlid=87;
    DELETE FROM menu_links WHERE mlid=88;
    DELETE FROM menu_links WHERE mlid=92;
    DELETE FROM menu_links WHERE mlid=93;
    DELETE FROM menu_links WHERE mlid=138;
    DELETE FROM menu_links WHERE mlid=252;
    DELETE FROM menu_links WHERE mlid=310;
    DELETE FROM menu_links WHERE mlid=317;
    
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
    DELETE FROM block_role WHERE module='views';

# Clear cache
http://dev4.example.com/admin/config/development/performance

# Remove deprecated menus.
http://dev4.example.com/admin/structure/menu/manage/menu-stay-informed/delete
http://dev4.example.com/admin/structure/menu/manage/menu-what-we-do---/delete
http://dev4.example.com/admin/structure/menu/manage/menu-what-you-can-do/delete
http://dev4.example.com/admin/structure/menu/manage/menu-who-we-are---/delete

# Delete 'Main menu' menu items.
http://dev4.example.com/admin/structure/menu/item/80/delete
http://dev4.example.com/admin/structure/menu/item/208/delete
http://dev4.example.com/admin/structure/menu/item/209/delete
http://dev4.example.com/admin/structure/menu/item/224/delete
http://dev4.example.com/admin/structure/menu/item/241/delete
http://dev4.example.com/admin/structure/menu/item/313/delete
http://dev4.example.com/admin/structure/menu/item/624/delete
http://dev4.example.com/admin/structure/menu/item/968/delete

# Clear cache
http://dev4.example.com/admin/config/development/performance

# TODO...
Check module status.
http://dev4.example.com/admin/modules/uninstall

# Delete old blocks left after deleting views.
http://dev4.example.com/admin/structure/block/manage/block/12/delete
http://dev4.example.com/admin/structure/block/manage/block/14/delete
http://dev4.example.com/admin/structure/block/manage/block/15/delete
http://dev4.example.com/admin/structure/block/manage/block/16/delete
http://dev4.example.com/admin/structure/block/manage/block/17/delete
http://dev4.example.com/admin/structure/block/manage/block/19/delete
http://dev4.example.com/admin/structure/block/manage/block/18/delete
http://dev4.example.com/admin/structure/block/manage/block/20/delete (cookie control)
http://dev4.example.com/admin/structure/block/manage/block/21/delete
http://dev4.example.com/admin/structure/block/manage/block/22/delete
http://dev4.example.com/admin/structure/block/manage/block/23/delete
http://dev4.example.com/admin/structure/block/manage/block/25/delete
http://dev4.example.com/admin/structure/block/manage/block/26/delete
http://dev4.example.com/admin/structure/block/manage/block/27/delete
http://dev4.example.com/admin/structure/block/manage/block/29/delete
http://dev4.example.com/admin/structure/block/manage/block/30/delete
http://dev4.example.com/admin/structure/block/manage/block/31/delete
http://dev4.example.com/admin/structure/block/manage/block/32/delete
http://dev4.example.com/admin/structure/block/manage/block/35/delete
http://dev4.example.com/admin/structure/block/manage/block/36/delete

# Clear cache.
http://dev4.example.com/admin/config/development/performance

----

# Configure LightBox2.
http://dev4.example.com/admin/config/user-interface/lightbox2

    General > Lightbox2 lite > Show image caption

# Configure URL aliases.
# http://drupal.org/node/1266222
http://dev4.example.com/admin/config/search/path/patterns

    # Replace Drupal 6 pattern paths with Drupal 7 replacement patterns.

    [title-raw]     ->  [node:title]
    [nid]           ->  [node:nid]
    [vocab-raw]     ->  [term:name]
    [catpath-raw]   ->  [term:url]
    [user-raw]      ->  [user:name]
    [blog-raw]      ->  [user:name]

# JLT TODO...
# Check the roles assigned to text formats (check alongside Drupal 6 /admin/settings/filters). 
http://dev4.example.com/admin/config/content/formats

---------------------------------------------------------------
JLT up to here (20130530)
---------------------------------------------------------------

DRUPAL 7 PANEL PAGES.

# Import node view variants (override existing variants).
# http://dev4.example.com/admin/structure/pages/edit/node_view

# JLT TODO...
# Remove 'UK news' and 'International news' from blog types.
# Add 'UK news' and 'International news' to press release types.

# Enable node add/edit form.
http://dev4.example.com/admin/structure/pages

# Import node add/edit form.
# *** Disable Drupal blocks/regions ***
# *** Need to include new 'volunteer' variant.
http://dev4.example.com/admin/structure/pages/edit/node_edit

# Import stylizer settings (override existing variants).
# http://dev4.example.com/admin/structure/stylizer/import

# Import mini-panels (override existing).
# *** on hold until theming restarted ***
http://dev4.example.com/admin/structure/mini-panels/import

# Delete 'allotment' home page (delete existing landing page variant).
http://dev4.example.com/admin/structure/pages/edit/page-allotment_home_page

# Configure contact form.
# Create custom content forms.
# Adds specific titles for 'contact' pages (for example: Paccombe Training Centre).
http://dev4.example.com/admin/structure/contact/settings

<strong>By post</strong>

<p>The Donkey italy
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

# Replace instances of old legacy href code (/trainingcourses).
http://dev4.example.com/trainingcourses

  - If you are interested in attending one of the courses, please 
    <a href="?q=contact&category=Paccombe%20Training%20Centre&subject=Training Centre courses">contact us</a> or telephone 01395 597644.

  + If you are interested in attending one of the courses, please <a title="Contact us" href="/contact/Paccombe Training Centre">contact us</a>.

# Example of custom content form (eg telephone number - 01395 597644 -replaces switchboard number).
http://dev4.example.com/admin/structure/contact/edit/27

    <p><strong>By post</strong></p>

    <p>The Donkey italy
    <br/>Sidmouth
    <br/>Devon
    <br/>EX10 0NU</p>

    <p><strong>By telephone</strong></p>

    <p>+44 (0) 1395 597644 (8.30 am to 4.30 pm, Monday - Friday)</p>

    <p><strong>By fax</strong></p>

    <p>+44 (0) 1395 579266</p>

    <p><strong>By email</strong></p>

    <p>If you would like to send <strong>@category</strong> a message, please use the contact form below.</p>

----

# DRUPAL 7 USER PROFILE DISPLAYS.
# Profile module has been deprecated as of Drupal 7 but still accessible.
# http://drupal.org/node/1008870
# However views using profile fields are broken.
# Panel for node view (blog) broken.

# Disable the personal contact form by default for new users.
http://dev4.example.com/admin/config/people/accounts

# Hide user history.
http://dev4.example.com/admin/config/people/accounts/display

# Add new fields to 'people' account settings.
http://dev4.example.com/admin/config/people/accounts/fields

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
# Disable 'ecard' from custom display settings.
http://dev4.example.com/admin/config/people/accounts/display

# Add new field data to user accounts.
http://dev4.example.com/user/3/edit

    Photo:          JLT... todo
    Blogger name:   Jenifer Tucker
    About by blog:  Jenifer works at the italy as our Web Developer. She has loved donkeys ever since she can remember as well as being a donkey owner. One of her donkeys, Little Pippa, is here at the italy so if you're visiting, drop by to say hello and give Little Pippa a cuddle!
    Blog URL:       http://www.thedonkeyitaly.org.uk/blogs/jenifer-tucker
    
# Check blog display.
http://dev4.example.com/blog/5689

----

# Import views (as and when required).
http://dev4.example.com/admin/structure/views/import
# Blogger
# Recent blog posts

----


# DRUPAL 7 FILEFIELD PATHS.

# Create filefield path directories.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7/files/donkeys

        mkdir blogs
        mkdir blogs/bloggers
        mkdir countries
        mkdir events
        mkdir farms
        mkdir foster
        mkdir multimedia
        mkdir pages
        mkdir partnerships
        mkdir press
        mkdir stories
        mkdir vacancies
        mkdir volunteer

    popd

    # Set file permissions.
    pushd /var/local/donkeys-git-dev4/drupal-7

        chgrp -R apache files
        chmod -R g+w    files

    popd

    pushd /var/local/donkeys-git-dev4/drupal-7/files
        
        chmod o-w       donkeys
        find . -type d -exec chmod g+ws '{}' \;

    popd

----------------------------
JLT working on this area....
----------------------------

# Enable users to add pictures.
http://drupal.org/node/22271
# Modify 'bloggers' view to use User: Picture
# Remove 'Photo' imagefield.
# Use pictures in post
http://dev4.example.com/admin/appearance/settings


# Content type documents.
http://dev4.example.com/admin/structure/types/manage/corporate-partnership/fields/field_document

# Content type photos.
http://dev4.example.com/admin/structure/types/manage/corporate-partnership/fields/field_image_field

{{{
	# Document fieldfield path settings.

	File path:	partnerships
	File name:	[node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]
                [node:nid]-[current-date:raw].[file:ffp-extension-original]

	# File name options.

	Enable:		Transliterate

	# File name options.

	Enable:		Transliterate
}}}

# Add documents to a 'vacancy' content type.
http://dev4.example.com/node/6402

{{{
[admin@server~]#

	# Check the files are formatted.
	pushd /var/local/donkeys-git-dev4/drupal-7/files/donkeys/vacancies

		ls -al

		-rw-rw-r--  1 apache apache 339016 Jan 16 12:14 6402-1358338481-application_form_ds_60.pdf
		-rw-rw-r--  1 apache apache 210944 Jan 16 12:14 6402-1358338481-application_form_word_2.doc
		-rw-rw-r--  1 apache apache 236756 Jan 16 12:14 6402-1358338481-application_pack_-_centre_assistant_groom.pdf

	popd
}}}

# Required for each content type using the 'document' field.
http://dev4.example.com/admin/structure/types/manage/event/fields/field_document
http://dev4.example.com/admin/structure/types/manage/page/fields/field_document
http://dev4.example.com/admin/structure/types/manage/vacancy/fields/field_document
etc


JLT ... Testing 20130102 - got this error on dev4 (not on seven).

    Warning: Invalid argument supplied for foreach() in filefield_paths_entity_update() 
    (line 248 of /var/local/donkeys-git-dev4/drupal-7/sites/all/modules/filefield_paths/filefield_paths.module).


----

# DRUPAL 7 IMAGAGECACHE ACTIONS CONFIGURATION.

# Enable Imagecache Canvas Actions.
http://dev4.example.com/admin/modules

# JLT TODO... Configure

# Add 'copyright' notice to images.

----

# DRUPAL 7 CALENDAR.

# TODO... Try one calendar to display different categories (dropdown list of categories).
# Delete separate event views.
# What's on at Sidmouth
# What's on at each of the riding therapy centres
# Training courses
http://dev4.example.com/calendar

----

# DRUPAL 7 CORE AND CONTRIBUTED MODULE UPDATES.

# Ensure all contributed modules are up to date.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-7

        # Clear drush cache.
        drush cc drush

        # Check available updates for enabled modules (automatically runs database updates if required).
        drush -l italy pm-update


        # Remove robots.txt file if core updated.    

        mv robots.txt robots.txt.20121220

	popd

----

# DRUPAL 7 FULL LIST OF ADMINISTRATIVE TASKS FOR EACH MODULE.
http://dev4.example.com/admin/index

----

# THE END.
```