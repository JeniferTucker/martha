# Details #

Upgrading a Drupal site from D6 to D7.

These instructions assume:

  * Drupal 6 core is up to date.
  * Contributed modules/themes are up to date.
  * Database backed up.
  * DNS aliases and virtual host set up.


---

```
# Get a working copy of all the Sanctuary websites from Ixis server.
[ixis@server]

    # Git.
    git add .
    git commit -a -m "Snapshot of live sites"
    git push
    
# Get a working copy of all the Sanctuary websites from Ixis server.
[admin@server]

    pushd /var/local/

        # Create our 'master' copy (only need to do this once)
    	git clone git@codebasehq.com:ixis/donkey-sanctuary/ds.git donkeys-git-april

        # Create a new local copy (clone) of the GIT codebase.
        git clone donkeys-git-april donkeys-git-dev1

    popd

    # Update our master copy from the codebasehq repository.

    pushd /var/local/donkeys-git-april

        git pull

    popd

    # Update our 'dev' working copy from our 'april' copy.
    pushd /var/local/donkeys-git-dev1

        git pull

    popd

# Fix file permissions after downloading from git repository.
[admin@server~]

pushd /var/local/donkeys-git-dev1/drupal-6

    id apache
    # uid=48(apache) gid=48(apache) groups=48(apache)

    chgrp -R apache files
    chmod -R g+w    files

    pushd files

        find . -type d -exec chmod g+ws '{}' \;

    popd

popd

----

***********************************************************************
** This doesn't work on admin@server                                 **
** Run on gamma and then copy/paste SQL file to admin@server for now **
** gunzip file                                                       **
***********************************************************************


# Backup mySQL databases on Ixis server.
[admin@server~]

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

    # Add 'dev1' to settings.
	vi ~/local-settings.txt

        #[dev1]
        dev1_mysqldrupaluser=*****
        dev1_mysqldrupalpass=*****
        dev1_mysqldrupaldata=*****

   # Load settings and functions.
	source ~/local-settings.txt

----

# Add the DNS aliases

# Go to easyDNS.

----

# Create virtual host files.
[admin@server~]

    pushd /etc/httpd/conf.d

cat > six.example.com.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@thedonkeysanctuary.org.uk
  ServerName  six.example.com

  LogLevel warn
  ErrorLog  /var/log/httpd/six.example.com.error.log
  CustomLog /var/log/httpd/six.example.com.access.log combined

  DocumentRoot /var/local/donkeys-git-dev1/drupal-6
  <Directory /var/local/donkeys-git-dev1/drupal-6>

  Options Indexes FollowSymLinks MultiViews
  AllowOverride All
  Order allow,deny
  allow from all

  </Directory>

</VirtualHost>
EOF

cat > dev1.example.com.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@thedonkeysanctuary.org.uk
  ServerName  dev1.example.com

  LogLevel warn
  ErrorLog  /var/log/httpd/dev1.example.com.error.log
  CustomLog /var/log/httpd/dev1.example.com.access.log combined

  DocumentRoot /var/local/donkeys-git-dev1/drupal-7
  <Directory /var/local/donkeys-git-dev1/drupal-7>

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

# Create 'sanctuary' database.
[admin@server~]

	# Check mySQL is running.
	/etc/init.d/mysqld status

	# Load local settings.
	source ~/local-settings.txt

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
            < /var/local/backups/ixis/sanctuary-ixis.stanleywindrush.org.uk-20130428034739.sql

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

    # Delete forum content.
    DELETE FROM node WHERE type='forum';
    quit

----

# DRUPAL 7 DATABASE.

# Create 'dev1' database.
[admin@server~]

    # Drop existing database (and other databases not being developed - est, ireland, spain, italy, cyprus, research, welfare).
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="DROP DATABASE ${dev1_mysqldrupaldata}"

    # Create database.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="CREATE DATABASE ${dev1_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

	# Grant permissions.
   	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="GRANT ALL ON ${dev1_mysqldrupaldata}.* TO '${dev1_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${dev1_mysqldrupalpass}'"

    # Show existing databases.
    mysql --user=${mysqladminuser} --password=${mysqladminpass} \
      --execute="SHOW DATABASES"

    # Check database connection.
    mysql --user=${dev1_mysqldrupaluser} --password=${dev1_mysqldrupalpass} ${dev1_mysqldrupaldata} \
      --execute="SELECT VERSION()"

----

# DRUPAL 6 CORE AND MODULES.

# Ensure all contributed modules are up to date.
[admin@server~]

	pushd /var/local/donkeys-git-dev1/drupal-6

        # Clear drush cache.
        drush cc drush

        # Check available updates for enabled modules (automatically runs database updates if required).
        drush -l sanctuary pm-update

	popd

# Remove uninstalled modules from the modules directory.
[admin@server~]

	pushd /var/local/donkeys-git-dev1/drupal-6/sites/all/modules

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

# Disable all themes no longer required.
# http://six.example.com/admin/build/themes

    # Adaptivetheme
    # Blogbuzz

# Remove old themes from the themes directory.
[admin@server~]

	pushd /var/local/donkeys-git-dev1/drupal-6/sites/all/themes

        rm -rf adaptivetheme
        rm -rf blogbuzz

	popd

# Move old imported files from images directory.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-6/files/donkeys/images/import

        # Move images to images directory.
        mv filey* /var/local/donkeys-git-dev1/drupal-6/files/donkeys/images

    popd

# Remove legacy image content.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-6/files/donkeys/imagecache

        rm -rf cart
        rm -rf product
        rm -rf product_list
        rm -rf uc_thumbnail

    popd

# Remove legacy image thumbnails.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-6/files/donkeys/imagefield_thumbs

        rm -rf hannah-sponsoradonkey.png
        rm -rf "Kennel Barn. Photo copyright of The Donkey Sanctuary.jpg"
        rm -rf Slideshow-Egypt.jpg

    popd

----

# DRUSH UPGRADE.

# Install Drupal 7 modules required to run Drush site upgrade (if not already on VM).
[admin@server~]

        drush dl drush_sup

----

# DRUPAL 6 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

pushd /var/local/donkeys-git-dev1/drupal-6/sites

    ln -s sanctuary six.example.com

popd

----

# DRUPAL 6 FILES DIRECTORY.

# Create symbolic links.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-6/files

        # Create symbolic linkS.
        ln -s donkeys sanctuary
        ln -s donkeys six.example.com

    popd

----

# DRUPAL 6 IMAGE ATTACH CONVERTER.

# http://drupal.org/node/201983

    # Login as admin user.
    http://six.example.com/user

    # Clear cached data.
    http://six.example.com/admin/settings/performance

    # Create prerequisite imagefield type.
    # The imagefield field that you have already created and configured as per Prerequisites.
    # $field_name = 'field_image_field'
    # The content type that you have already created as per Prerequisites.
    # $type_name = 'imagefield';
    http://six.example.com/admin/content/types/add

        Name:           Photo
        Type:           imagefield
        Description:    Prerequisite imagefield type for image converter.
    
    http://six.example.com/admin/content/node-type/imagefield/fields
    
        New field:      imagefield
        Field_          image_field
        Data type:      File
        Form element:   Image
        
        Number of values:   Unlimited
        Description field:  Enabled
    
    # Add new 'image_field' to content types (where attached images are enabled).
    http://six.example.com/admin/content/node-type/blog/fields
    http://six.example.com/admin/content/node-type/corporate-partnership/fields
    http://six.example.com/admin/content/node-type/event/fields
    http://six.example.com/admin/content/node-type/fundraising/fields
    http://six.example.com/admin/content/node-type/page/fields
    http://six.example.com/admin/content/node-type/pressrelease/fields
    http://six.example.com/admin/content/node-type/rehome/fields
    http://six.example.com/admin/content/node-type/story/fields
    http://six.example.com/admin/content/node-type/videoclip/fields
    http://six.example.com/admin/content/node-type/webform/fields

    # Remove redundant 'copyright' field.
    http://six.example.com/admin/content/node-type/image/fields/field_image_copyright/remove

    # Migrate all image module nodes to imagefields.
    pushd /var/local/donkeys-git-dev1/drupal-6

        # Copy migration script to Drupal 6 core.
        cp /var/local/scripts/imagefield_migrate.php .

        # Run script.
        http://six.example.com/imagefield_migrate.php

            # - content_type_imagefield populated.
            # - content_field_image_field populated.
            # - files table updated

JLT todo... check as think these are nodes with more than one image.

            update content_type_imagefield failed for 6452
            update content_field_image_field failed for 6452
            update content_type_imagefield failed for 6735
            update content_field_image_field failed for 6735
            update content_type_imagefield failed for 6736
            update content_field_image_field failed for 6736
            update content_type_imagefield failed for 6752
            update content_field_image_field failed for 6752
            update content_type_imagefield failed for 6782
            update content_field_image_field failed for 6782
            update content_type_imagefield failed for 6783
            update content_field_image_field failed for 6783
            update content_type_imagefield failed for 6785
            update content_field_image_field failed for 6785
            update content_type_imagefield failed for 6786
            update content_field_image_field failed for 6786
            update content_type_imagefield failed for 6787
            update content_field_image_field failed for 6787
            update content_type_imagefield failed for 6802
            update content_field_image_field failed for 6802
            update content_type_imagefield failed for 6803
            update content_field_image_field failed for 6803
            update content_type_imagefield failed for 6805
            update content_field_image_field failed for 6805
            update content_type_imagefield failed for 6806
            update content_field_image_field failed for 6806
            update content_type_imagefield failed for 6813
            update content_field_image_field failed for 6813
            update content_type_imagefield failed for 6815
            update content_field_image_field failed for 6815
            update content_type_imagefield failed for 6816
            update content_field_image_field failed for 6816
            update content_type_imagefield failed for 6843
            update content_field_image_field failed for 6843
            update content_type_imagefield failed for 6844
            update content_field_image_field failed for 6844
            update content_type_imagefield failed for 6845
            update content_field_image_field failed for 6845
            update content_type_imagefield failed for 6846
            update content_field_image_field failed for 6846
            update content_type_imagefield failed for 6848
            update content_field_image_field failed for 6848
            update content_type_imagefield failed for 6903
            update content_field_image_field failed for 6903
            update content_type_imagefield failed for 6904
            update content_field_image_field failed for 6904
            update content_type_imagefield failed for 6947
            update content_field_image_field failed for 6947
            update content_type_imagefield failed for 6948
            update content_field_image_field failed for 6948
            update content_type_imagefield failed for 6949
            update content_field_image_field failed for 6949
            update content_type_imagefield failed for 6950
            update content_field_image_field failed for 6950
            update content_type_imagefield failed for 6951
            update content_field_image_field failed for 6951
            update content_type_imagefield failed for 6954
            update content_field_image_field failed for 6954
            update content_type_imagefield failed for 6955
            update content_field_image_field failed for 6955
            update content_type_imagefield failed for 6956
            update content_field_image_field failed for 6956
            update content_type_imagefield failed for 6964
            update content_field_image_field failed for 6964
            update content_type_imagefield failed for 6966
            update content_field_image_field failed for 6966
            update content_type_imagefield failed for 6975
            update content_field_image_field failed for 6975
            update content_type_imagefield failed for 6976
            update content_field_image_field failed for 6976
            update content_type_imagefield failed for 6977
            update content_field_image_field failed for 6977
            update content_type_imagefield failed for 6979
            update content_field_image_field failed for 6979
            update content_type_imagefield failed for 6980
            update content_field_image_field failed for 6980
            update content_type_imagefield failed for 6981
            update content_field_image_field failed for 6981
            # - 1647 image_attach relationships were migrated.

        # Remove script to prevent re-running.
        rm -rf imagefield_migrate.php

   popd

----

# DRUPAL 6 IMAGE HANDLING.

# Disable image module (image, image attach, image gallery).
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-6

        # Disable image module and extensions.
        drush -l sanctuary --yes pm-disable image

        # Download new image handling modules.
        drush -l sanctuary dl imagecache 
        drush -l sanctuary dl imageapi

        # Clear drush cache.
        drush cache-clear drush

        # Enable modules (ImageAPI GD2, ImageAPI, ImageCache UI, ImageCache).
        drush -l sanctuary --yes pm-enable imageapi
        drush -l sanctuary --yes pm-enable imagecache


      # JLT TODO... User permissions (if required).
      # http://six.example.com/admin/user/permissions

    popd

# Remove image module.

    pushd /var/local/donkeys-git-dev1/drupal-6/sites/all/modules

         rm -rf image

	popd

# Enable ImageCacheUI and ImageAPI GD2 modules.
http://six.example.com/admin/build/modules

# Enable Dashboard and Toolkit modules.
http://six.example.com/admin/build/modules

# Delete 'donkidz' taxonomy.
http://six.example.com/admin/content/taxonomy/edit/vocabulary/37

# Disable 'rebrand' theme, set 'garland' as default theme.
http://six.example.com/admin/build/themes

# Clear cached data.
http://six.example.com/admin/settings/performance

# Remove 'amadou' theme and 'rebrand' sub-theme from the themes directory.
# JLT TODO... remove 'pippa' theme.
[admin@server~]

	pushd /var/local/donkeys-git-dev1/drupal-6/sites/all/themes

        rm -rf amadou
        rm -rf rebrand

	popd

----

# DRUPAL 6 UPGRADE TO DRUPAL 7.

# Create alias file.
[admin@server~]

	pushd /var/local/donkeys-git-dev1/drupal-6/sites/sanctuary

        # Create alias file.
cat > /var/local/donkeys-git-dev1/drupal-6/sites/sanctuary/dev1.alias.drushrc.php << EOF
<?php
  \$aliases['dev1'] = array(
    'root'   => '/var/local/donkeys-git-dev1/drupal-7',
    'uri'    => 'sanctuary',
    'db-url' => 'mysql://${dev1_mysqldrupaluser}:${dev1_mysqldrupalpass}@localhost/${dev1_mysqldrupaldata}',
  );
EOF

        # Copy default settings file.
        # cp /var/local/donkeys-git-dev1/drupal-7/sites/default/default.settings.php /var/local/donkeys-git-dev1/drupal-7/sites/sanctuary

    popd

# Run site-upgrade command.

[admin@server~]

	pushd /var/local/donkeys-git-dev1/drupal-6

        # Run site upgrade.
        drush -l sanctuary site-upgrade @dev1

----

# DRUPAL 7 OVERRIDES.

# Remove robots.txt file (using robotsTxt module).
[admin@server~]

	pushd /var/local/donkeys-git-dev1/drupal-7

        mv robots.txt robots.txt.20130501

    popd

----

# DRUPAL 7 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

pushd /var/local/donkeys-git-dev1/drupal-7/sites

    ln -s sanctuary dev1.thedonkeysanctuary.org.uk
    ln -s sanctuary dev1.example.com

popd

----

# DRUPAL 7 FILES DIRECTORY.

[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-7/files

        # Create symbolic links.
        ln -s donkeys sanctuary
        ln -s donkeys dev1.example.com

    popd

    # Set file permissions.
    pushd /var/local/donkeys-git-dev1/drupal-7

        chgrp -R apache files
        chmod -R g+w    files

    popd

    pushd /var/local/donkeys-git-dev1/drupal-7/files
        
        chmod o-w       donkeys
        find . -type d -exec chmod g+ws '{}' \;

    popd

----

# DRUPAL 7 PRIVATE FILES DIRECTORY.

# Create private file system path.
[admin@server~]

pushd /var/local/donkeys-git-dev1

    # Create files directory.
    mkdir files-7

    chgrp -R apache files-7
    chmod -R g+w    files-7

    pushd files-7

        find . -type d -exec chmod g+ws '{}' \;

    popd

popd

----

# DRUPAL 7 MODULES.

# Add new contributed modules.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-7

        # Patch fix for ctools module.
        drush -l sanctuary dl ctools-7.x-1.x-dev

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

        # Menu import module.
        drush -l sanctuary dl menu_import

        # Video embed field module.
        drush -l sanctuary dl video_embed_field

        # Media Flickr module.
        drush -l sanctuary dl media_flickr

        drush -l sanctuary dl libraries
        drush -l sanctuary dl views_infinite_scroll

        pushd sites/all

             mkdir libraries
             mkdir autopager

        popd

        pushd /var/local/donkeys-git-dev1/drupal-7/sites/all/autopager

             wget http://jquery-autopager.googlecode.com/files/jquery.autopager-1.0.0.js

        popd

        # Enable existing modules (disabled during site upgrade).
        drush -l sanctuary --yes pm-enable aggregator
        drush -l sanctuary --yes pm-enable blog
        drush -l sanctuary --yes pm-enable image
        drush -l sanctuary --yes pm-enable profile
        drush -l sanctuary --yes pm-enable search
        drush -l sanctuary --yes pm-enable taxonomy
       
JLT to check...

            The following extensions will be enabled: taxonomy
            Do you really want to continue? (y/n): y
            Invalid argument supplied for foreach() taxonomy.views.inc:429                                                                                         [warning]
            Invalid argument supplied for foreach() taxonomy.views.inc:429   

        drush -l sanctuary --yes pm-enable trigger

        # Enable new modules.
        drush -l sanctuary --yes pm-enable contact_forms      
        drush -l sanctuary --yes pm-enable ecard
        drush -l sanctuary --yes pm-enable filefield_paths
        drush -l sanctuary --yes pm-enable filefield_sources
        drush -l sanctuary --yes pm-enable imagecache_actions
        drush -l sanctuary --yes pm-enable linkchecker
        drush -l sanctuary --yes pm-enable term_reference_tree
        drush -l sanctuary --yes pm-enable transliteration

        drush -l sanctuary --yes pm-enable menu_import

        drush -l sanctuary --yes pm-enable video_embed_field
        drush -l sanctuary --yes pm-enable media_flickr
        
        drush -l sanctuary --yes pm-enable libraries
        drush -l sanctuary --yes pm-enable views_infinite_scroll

        # Clear all caches.
        drush -l sanctuary cc all



----

# Note: Media_Flickr
#
# To use it, enable the Media module. You'll also need to apply for a Flickr API
# key, from http://www.flickr.com/services/api/keys, which you will paste into
# the settings page at /admin/configure/media/media_flickr.
# http://dev1.example.com/admin/config/media/media_flickr
# Set up tutorial.
# http://drupal.org/node/1406068#comment-5630428

----

# DRUPAL 7 THEMES.

# Add new contributed themes.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-7

        drush dl marinelli

    popd

# Create new subtheme.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-7/sites/all/themes/marinelli

        # Copy 'marinelli' theme.
        cp -r subtheme ../avocet
        cp -r css      ../avocet

        # Copy 'marinelli' theme.
        cp -r subtheme ../curlew
        cp -r css      ../curlew

        # Copy 'marinelli' theme.
        cp -r subtheme ../heron
        cp -r css      ../heron

    popd

# Rename the subtheme.info.txt file to match the new theme name.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-7/sites/all/themes/avocet

        mv subtheme.info.txt avocet.info

        vi avocet.info

        [find] subtheme [replace] avocet

        # Uncomment stylesheets to override 'marinelli' theme.
        stylesheets[all][] = css/reset/reset.css
        stylesheets[all][] = css/common.css
        stylesheets[all][] = css/links.css
        stylesheets[all][] = css/typography.css
        stylesheets[all][] = css/forms.css
        stylesheets[all][] = css/drupal.css
        stylesheets[all][] = css/layout.css
        stylesheets[all][] = css/primary-links.css
        stylesheets[all][] = css/slideshow.css
        stylesheets[all][] = css/secondary-links.css
        stylesheets[all][] = css/blocks.css
        stylesheets[all][] = css/node.css
        stylesheets[all][] = css/comments.css

    popd

    pushd /var/local/donkeys-git-dev1/drupal-7/sites/all/themes/curlew

        mv subtheme.info.txt curlew.info

        vi curlew.info

        [find] subtheme [replace] curlew

        # Uncomment stylesheets to override 'marinelli' theme (as above).

    popd

    pushd /var/local/donkeys-git-dev1/drupal-7/sites/all/themes/heron

        mv subtheme.info.txt heron.info

        vi heron.info

        [find] subtheme [replace] heron

        # Uncomment stylesheets to override 'marinelli' theme (as above).

    popd

----

# DRUPAL 7 SITE CONFIGURATION.

# Login as admin user.
http://dev1.example.com/user

# Clear cache in Drupal 7 site (disable caching while developing).
http://dev1.example.com/admin/config/development/performance

# Enable update manager module.
http://dev1.example.com/admin/modules

# Check status report (run cron manually to check available updates).
http://dev1.example.com/admin/reports/status

# Check date and time configuration.
http://dev1.example.com/admin/config/regional/date-time

# Migrate content type fields.
http://dev1.example.com/admin/structure/content_migrate

# Enable and set as default the 'avocet' sub-theme.
http://dev1.example.com/admin/appearance

# Disable site name and site slogan.
# Disable default logo (files/donkeys/images/DonkeySanctuaryMasterTabLogo-Sanctuary-20120807.png)
# Disable default favicon (files/donkeys/images/rebrand_favicon.png).
# Disable CSS3 settings.
# Add custom path to custom logo and icon.
http://dev1.example.com/admin/appearance/settings/avocet

# Delete the URL redirect from 404 to node/1535.
http://dev1.example.com/admin/config/search/redirect

# Set URL path setting for 'page not found' (404).
http://dev1.example.com/node/1535/edit

# Set default 404 (not found) page.
http://dev1.example.com/admin/config/system/site-information

# Override default front page.
http://dev1.example.com/admin/config/system/site-information

# Enable 'insert block' and 'insert views' filters.
http://dev1.example.com/admin/config/content/formats/3

# Enable 'search block'.
# Block title <none>
# Add to search region in 'avocet' (default) theme
# Show block on all pages
http://dev1.example.com/admin/structure/block/manage/search/form/configure

# Configure link checker.
http://dev1.example.com/admin/config/content/linkchecker

# Configure private file system path.
http://dev1.example.com/admin/config/media/file-system
    
    Public file system path:    /files/donkeys
    Private file system path    /var/local/donkeys-git-dev1/files-7
    Temporary directory :       /files/donkeys/tmp

# Convert uppercase strings, to lowercase (*are we sure*).
# http://dev1.example.com/admin/config/media/file-system/transliteration


----

# DRUPAL 7 CONTENT TYPES.

# Import content types from 'seven'.
http://dev1.example.com/admin/structure/types/import


----

#  DEPRECATED MENUS (to do before importing views).

# Remove old menus.
http://dev1.example.com/admin/structure/menu/manage/menu-stay-informed/delete
http://dev1.example.com/admin/structure/menu/manage/menu-what-we-do---/delete
http://dev1.example.com/admin/structure/menu/manage/menu-what-you-can-do/delete
http://dev1.example.com/admin/structure/menu/manage/menu-who-we-are---/delete

# Broken admin pages/links after upgrade if "Administer" was not in the "Navigation" menu.
# Workaround http://drupal.org/node/865702#comment-4015758

[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-7

	    # Load local settings.
	    source ~/local-settings.txt

        # Connect to database.
        mysql --user=${mysqladminuser} --password=${mysqladminpass}

        # Connect to database. 
        USE sanctuary ;

        DELETE FROM menu_links WHERE module = 'system';
        
        # Exit.
        quit

    popd
    
# Clear cache.
http://dev1.example.com/admin/config/development/performance

# Find D6 'Administer' menu link in 'Navigation' menu (upgrade messes up admin menu).
# Move to 'Management' menu (weight -50) and nest menu items below.
http://dev1.stanleywindrush.org.uk/admin/structure/menu/manage/management


----


# Configure LightBox2.
http://dev1.example.com/admin/config/user-interface/lightbox2

    General > Lightbox2 lite > Show image caption

# Configure URL aliases.
# http://drupal.org/node/1266222
http://dev1.example.com/admin/config/search/path/patterns

    # Replace Drupal 6 pattern paths with Drupal 7 replacement patterns.

    [title-raw]     ->  [node:title]
    [nid]           ->  [node:nid]
    [vocab-raw]     ->  [term:name]
    [catpath-raw]   ->  [term:url]
    [user-raw]      ->  [user:name]
    [blog-raw]      ->  [user:name]

# Check the roles assigned to text formats (check alongside Drupal 6 /admin/settings/filters). 
http://dev1.example.com/admin/config/content/formats

----

DRUPAL 7 PANEL PAGES.

# Import stylizer settings.
http://dev1.example.com/admin/structure/stylizer/import

# Import mini-panels (override existing).
http://dev1.example.com/admin/structure/mini-panels/import

# Enable node add/edit form.
# Import node add/edit form.
http://dev1.example.com/admin/structure/pages/edit/node_edit

# Import node view variants (delete existing variants).
http://dev1.example.com/admin/structure/pages/edit/node_view

# Delete 'allotment' home page (delete existing landing page variant).
http://dev1.example.com/admin/structure/pages/edit/page-allotment_home_page

# Configure contact form.
# Create custom content forms.
# Adds specific titles for 'contact' pages (for example: Paccombe Training Centre).
http://dev1.example.com/admin/structure/contact/settings

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

# Replace instances of old legacy href code (/trainingcourses).
http://dev1.example.com/trainingcourses

  - If you are interested in attending one of the courses, please 
    <a href="?q=contact&category=Paccombe%20Training%20Centre&subject=Training Centre courses">contact us</a> or telephone 01395 597644.

  + If you are interested in attending one of the courses, please <a title="Contact us" href="/contact/Paccombe Training Centre">contact us</a>.

# Example of custom content form (eg telephone number - 01395 597644 -replaces switchboard number).
http://dev1.example.com/admin/structure/contact/edit/27

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

----

# DRUPAL 7 USER PROFILE DISPLAYS.
# Profile module has been deprecated as of Drupal 7 but still accessible.
# http://drupal.org/node/1008870
# However views using profile fields are broken.
# Panel for node view (blog) broken.

# Delete all views and import from 'seven'.
http://dev1.example.com/admin/structure/views

# Add new fields to 'people' account settings.
http://dev1.example.com/admin/config/people/accounts/fields

    Label:          Photo	        (Use existing field)
    Machine name:   image_field
    Field type:     image

    Label:          About my Blog	
    Machine name:   my_blog
    Field type:     long text

    Label:          Blog URL	
    Machine name:   blog_url
    Field type:     text

# Manage display fields.
http://dev1.example.com/admin/config/people/accounts/display

# Add new field data to user accounts.
http://dev1.example.com/user/3/edit

    Photo:          JLT... todo
    Blogger name:   Jenifer Tucker
    About by blog:  Jenifer works at the Sanctuary as our Web Developer. She has loved donkeys ever since she can remember as well 
                    as being a donkey owner. One of her donkeys, Little Pippa, is here at the Sanctuary so if you're visiting, drop by to say hello and give Little Pippa a cuddle!
    Blog URL:       http://www.thedonkeysanctuary.org.uk/blogs/jenifer-tuck
    
# Check blog display.
http://dev1.example.com/blog/5689

----

# DRUPAL 7 FILEFIELD PATHS.



# Create filefield path directories.
[admin@server~]

    pushd /var/local/donkeys-git-dev1/drupal-7/files/donkeys

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

    popd

    # Set file permissions.
    pushd /var/local/donkeys-git-dev1/drupal-7

        chgrp -R apache files
        chmod -R g+w    files

    popd

    pushd /var/local/donkeys-git-dev1/drupal-7/files
        
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
http://dev1.stanleywindrush.org.uk/admin/appearance/settings


# Content type documents.
http://dev1.example.com/admin/structure/types/manage/corporate-partnership/fields/field_document

# Content type photos.
http://dev1.example.com/admin/structure/types/manage/corporate-partnership/fields/field_image_field

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
http://dev1.example.com/node/6402

{{{
[admin@server~]#

	# Check the files are formatted.
	pushd /var/local/donkeys-git-dev1/drupal-7/files/donkeys/vacancies

		ls -al

		-rw-rw-r--  1 apache apache 339016 Jan 16 12:14 6402-1358338481-application_form_ds_60.pdf
		-rw-rw-r--  1 apache apache 210944 Jan 16 12:14 6402-1358338481-application_form_word_2.doc
		-rw-rw-r--  1 apache apache 236756 Jan 16 12:14 6402-1358338481-application_pack_-_centre_assistant_groom.pdf

	popd
}}}

# Required for each content type using the 'document' field.
http://dev1.example.com/admin/structure/types/manage/event/fields/field_document
http://dev1.example.com/admin/structure/types/manage/page/fields/field_document
http://dev1.example.com/admin/structure/types/manage/vacancy/fields/field_document
etc


JLT ... Testing 20130102 - got this error on dev1 (not on seven).

    Warning: Invalid argument supplied for foreach() in filefield_paths_entity_update() 
    (line 248 of /var/local/donkeys-git-dev1/drupal-7/sites/all/modules/filefield_paths/filefield_paths.module).


----

# DRUPAL 7 IMAGAGECACHE ACTIONS CONFIGURATION.

# Enable Imagecache Canvas Actions.
http://dev1.example.com/admin/modules

# JLT TODO... Configure

# Add 'copyright' notice to images.

----

# DRUPAL 7 CALENDAR.

# TODO... Try one calendar to display different categories (dropdown list of categories).
# Delete separate event views.
# What's on at Sidmouth
# What's on at each of the riding therapy centres
# Training courses
http://dev1.example.com/calendar

----

# DRUPAL 7 CORE AND CONTRIBUTED MODULE UPDATES.

# Ensure all contributed modules are up to date.
[admin@server~]

	pushd /var/local/donkeys-git-dev1/drupal-7

        # Clear drush cache.
        drush cc drush

        # Check available updates for enabled modules (automatically runs database updates if required).
        drush -l sanctuary pm-update


        # Remove robots.txt file if core updated.    

        mv robots.txt robots.txt.20121220

	popd


----

# DRUPAL 7 FULL LIST OF ADMINISTRATIVE TASKS FOR EACH MODULE.
http://dev1.example.com/admin/index

----

# DRUPAL 7 CHECKLIST.

TODO...

1.  Admin > taxonomy
        Seems to have migrated from Drupal 6 to Drupal 7
        Tidy up URLs ---> /taxonomy/vocalulary > /taxonomy/human_resources
        Similar to CIL style.
        http://dev1.example.com/admin/structure/taxonomy
        Go to the Manage Fields tab of any fieldable entity (such as a content type, taxonomy term, or user).
        http://drupal.org/project/term_reference_tree

        http://dev1.example.com/admin/structure/types/manage/blog/fields

        New field:  Term Reference
        Field type: Term Reference
        Widget:     Term reference tree

        Test.
        http://dev1.example.com/node/add/blog
2.  Check all views/configure.
3.  Copy personal profile fields of bloggers into new content fields in user account settings.
        http://dev1.example.com/user/3/edit
 
----

TODO... REQUIRES CONFIGURING

Some user time zones have been emptied and need to be set to the correct values. Use the new time zone options to choose whether to remind  [warning]
users at login to set the correct time zone.

The default time zone has been set to Europe/London. Check the date and time configuration page to configure it correctly.                  [warning]

The contact form information setting was migrated to a custom block and set up to only show on the site-wide contact page. The block was set[status]
to use the default text format, which might differ from the HTML based format used before. Check the block and ensure that the output is
right.

The user registration guidelines were migrated to a custom block and set up to only show on the user registration page. The block was set to[status]
use the default text format, which might differ from the HTML based format used before. Check the block and ensure that the output is right.

The site mission was migrated to a custom block and set up to only show on the front page in the highlighted content region. The block was  [status]
set to use the default text format, which might differ from the HTML based format used before. Check the block and ensure that the output is
right. If your theme does not have a highlighted content region, you might need to relocate the block.

The footer message was migrated to a custom block and set up to appear in the footer. The block was set to use the default text format,     [status]
which might differ from the HTML based format used before. Check the block and ensure that the output is right. If your theme does not have
a footer region, you might need to relocate the block.

A new Plain text format has been created which will be available to all users. You can configure this text format on the text format        [status]
configuration page.

The content type imagefield had uploads disabled but contained uploaded file data. Uploads have been re-enabled to migrate the existing     [status]
data. You may delete the "File attachments" field in the imagefield type if this data is not necessary.

The new "View the administration theme" permission is required in order to view your site's administration theme. This permission has been  [status]
automatically granted to the following roles: chief editor, deputy admin, developer, ixis-admin. You can grant this permission to other
roles on the permissions page.

The widgets for repeating dates have changed. Please check the Display Fields page for each content type that has repeating date fields and [warning]
confirm that the right widget has been selected.

----

Special notes about some of the projects you are using:

cck   :   This project requires data migration or other special processing.  
          Please see http://drupal.org/project/cck and http://drupal.org/node/895314 for   
          more information on how to do this.                                                                        
date  :   This project requires data migration or other special processing.  
          The d6 version of the date_api module in the date project defines a table called 
          date_formats, which is also defined by system/system.install schema in d7.  
          If this problem has not been fixed yet, then the updatedb function will 
          fail, and it will not be possible to upgrade to d7.  If this happens, disable and 
          uninstall the date_api module before running site-upgrade 
          (i.e. add '--uninstall=date_api' to site-upgrade call).  See http://drupal.org/node/1013034.
token :   This project requires data migration or other special processing.  
          In Drupal 7, the contrib token module handles UI, as well as field and profile   
          tokens; all other functionality has been migrated to core. You may encounter problems 
          when enabling the token module during a major upgrade; see http://drupal.org/node/1477932.

----

# THE END.
```