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

    # Add 'uk' to settings.
	vi ~/local-settings.txt

        #[uk7]
        uk7_mysqldrupaluser=*****
        uk7_mysqldrupalpass=*****
        uk7_mysqldrupaldata=*****

   # Load settings and functions.
	source ~/local-settings.txt

----

# Add the 'uk7' DNS aliases
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

  DocumentRoot /var/local/donkeys-git-dev4/drupal-6
  <Directory /var/local/donkeys-git-dev4/drupal-6>

  Options Indexes FollowSymLinks MultiViews
  AllowOverride All
  Order allow,deny
  allow from all

  </Directory>

</VirtualHost>
EOF

cat > uk7.example.com.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@thedonkeysanctuary.org.uk
  ServerName  uk7.example.com

  LogLevel warn
  ErrorLog  /var/log/httpd/uk7.example.com.error.log
  CustomLog /var/log/httpd/uk7.example.com.access.log combined

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

# Create 'sanctuary' database.
[admin@server~]

	# Check mySQL is running.
	/etc/init.d/mysqld status

    # Drag/drop SFTP network file from gamma to pippa.
    
	# Load local settings.
	source ~/local-settings.txt
	backup=sanctuary-ixis.example.com-20130613202103.sql    
	
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

    # Delete forum content.
    DELETE FROM node WHERE type='forum';
    quit

----

# DRUPAL 7 DATABASE.

# Create 'uk7' empty database.
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

	pushd /var/local/donkeys-git-dev4/drupal-6

        # Clear drush cache.
        drush cc drush

        # Check available updates for enabled modules (automatically runs database updates if required).
        drush -l sanctuary pm-update

	popd

# Remove uninstalled modules from the modules directory.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-6/sites/all/modules

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

	pushd /var/local/donkeys-git-dev4/drupal-6/sites/all/themes

        rm -rf adaptivetheme
        rm -rf blogbuzz

	popd

# Move old imported files from images directory.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-6/files/donkeys/images/import

        # Move images to images directory.
        mv filey* /var/local/donkeys-git-dev4/drupal-6/files/donkeys/images

    popd

# Remove legacy image content.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-6/files/donkeys/imagecache

        rm -rf cart
        rm -rf product
        rm -rf product_list
        rm -rf uc_thumbnail

    popd

# Remove legacy image thumbnails.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-6/files/donkeys/imagefield_thumbs

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

pushd /var/local/donkeys-git-dev4/drupal-6/sites

    ln -s sanctuary six.example.com

    # Removed old symbolic links.
    rm -f est.donkeys.metagrid.co.uk
    rm -f est.example.com
    rm -f est.thedonkeysanctuary.org.uk

popd

----

# DRUPAL 6 FILES DIRECTORY.

# Create symbolic links.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-6/files

        # Create symbolic links.
        ln -s donkeys sanctuary
        ln -s donkeys six.example.com

    popd

----

# DRUPAL 6 IMAGE ATTACH CONVERTER.

# Source:
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

        Name:           imagefield
        Type:           imagefield
        Description:    Prerequisite imagefield type for image converter.
    
    http://six.example.com/admin/content/node-type/imagefield/fields
    
        New field:      imagefield
        Field_          image_field
        Data type:      File
        Form element:   Image
        
        Number of values:   Unlimited
        Description field:  Disabled

    # The content type imagefield had uploads disabled but contained uploaded file data. 
    # Uploads have been re-enabled to migrate the existing data. 
    # You may delete the "File attachments" field in the imagefield type if this data is not necessary.
    
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
    http://six.example.com/admin/content/node-type/volunteer/fields
    http://six.example.com/admin/content/node-type/webform/fields

    # Remove redundant 'copyright' field.
    http://six.example.com/admin/content/node-type/image/fields/field_image_copyright/remove

    # Migrate all image module nodes to imagefields.
    pushd /var/local/donkeys-git-dev4/drupal-6

        # Copy migration script to Drupal 6 core.
        cp /var/local/scripts/imagefield_migrate.php .

        # Run script.
        http://six.example.com/imagefield_migrate.php

            # - content_type_imagefield populated.
            # - content_field_image_field populated.
            # - files table updated

        # Remove script to prevent re-running.
        rm -rf imagefield_migrate.php

   popd

----

# DRUPAL 6 IMAGE HANDLING.

# Disable image module (image, image attach, image gallery).
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-6

        # Disable image module and extensions.
        drush -l sanctuary --yes pm-disable image

        # Download new image handling modules.
        drush -l sanctuary dl imagecache 
        drush -l sanctuary dl imageapi

        # Clear drush cache.
        drush cache-clear drush

        # Enable modules.
        drush -l sanctuary --yes pm-enable imageapi
        drush -l sanctuary --yes pm-enable imagecache
        drush -l sanctuary --yes pm-enable imagecache_ui
        drush -l sanctuary --yes pm-enable imageapi_gd

      # JLT TODO... User permissions (if required).
      # http://six.example.com/admin/user/permissions

    popd

# Remove image module.

    pushd /var/local/donkeys-git-dev4/drupal-6/sites/all/modules

         rm -rf image

	popd

# Delete 'donkidz' taxonomy.
# http://six.example.com/admin/content/taxonomy/edit/vocabulary/37

# Disable 'rebrand' theme, set 'garland' as default theme.
http://six.example.com/admin/build/themes

# Clear cached data.
http://six.example.com/admin/settings/performance

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

	pushd /var/local/donkeys-git-dev4/drupal-6/sites/sanctuary

        # Create alias file.
cat > /var/local/donkeys-git-dev4/drupal-6/sites/sanctuary/uk7.alias.drushrc.php << EOF
<?php
  \$aliases['uk7'] = array(
    'root'   => '/var/local/donkeys-git-dev4/drupal-7',
    'uri'    => 'sanctuary',
    'db-url' => 'mysql://${uk7_mysqldrupaluser}:${uk7_mysqldrupalpass}@localhost/${uk7_mysqldrupaldata}',
  );
EOF

    popd

----

# Run site-upgrade command.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-6

        # Run site upgrade.
        drush -l sanctuary site-upgrade @uk7

    popd

----

# DRUPAL 7 OVERRIDES.

# Remove robots.txt file (using robotsTxt module).
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-7

        rm robots.txt

    popd

----

# DRUPAL 7 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

pushd /var/local/donkeys-git-dev4/drupal-7/sites

    ln -s sanctuary uk7.thedonkeysanctuary.org.uk
    ln -s sanctuary uk7.example.com

popd

----

# DRUPAL 7 FILES DIRECTORY.

[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7/files

        # Create symbolic links.
        ln -s donkeys sanctuary
        ln -s donkeys dev4.example.com

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

----

# DRUPAL 7 PRIVATE FILES DIRECTORY.

# Create private file system path.
[admin@server~]

pushd /var/local/donkeys-git-dev4

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

    pushd /var/local/donkeys-git-dev4/drupal-7

        # Patch fix for ctools module.
        # JLT... ignore as not sure why need to patch fix (monitor).
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

        pushd /var/local/donkeys-git-dev4/drupal-7/sites/all/autopager

             wget http://jquery-autopager.googlecode.com/files/jquery.autopager-1.0.0.js

        popd

        # Enable existing modules (disabled during site upgrade).
        drush -l sanctuary --yes pm-enable aggregator
        drush -l sanctuary --yes pm-enable blog
        drush -l sanctuary --yes pm-enable image
        drush -l sanctuary --yes pm-enable profile
        drush -l sanctuary --yes pm-enable search
        drush -l sanctuary --yes pm-enable taxonomy
       
# JLT to check...

            The following extensions will be enabled: taxonomy
            Do you really want to continue? (y/n): y
            Invalid argument supplied for foreach() taxonomy.views.inc:429      [warning]
            Invalid argument supplied for foreach() taxonomy.views.inc:429      [warning]   

        drush -l sanctuary --yes pm-enable trigger

        # (D7) Enable Dashboard and Toolkit modules.
        drush -l sanctuary --yes pm-enable dashboard
        drush -l sanctuary --yes pm-enable toolbar
        drush -l sanctuary --yes pm-enable update

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
# http://uk7.example.com/admin/config/media/media_flickr
# Set up tutorial.
# http://drupal.org/node/1406068#comment-5630428

----

# DRUPAL 7 THEMES.

# Add new contributed themes.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7

        drush dl marinelli

    popd

# Create new subtheme.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7/sites/all/themes/marinelli

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

    pushd /var/local/donkeys-git-dev4/drupal-7/sites/all/themes/avocet

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

    pushd /var/local/donkeys-git-dev4/drupal-7/sites/all/themes/curlew

        mv subtheme.info.txt curlew.info

        vi curlew.info

        [find] subtheme [replace] curlew

        # Uncomment stylesheets to override 'marinelli' theme (as above).

    popd

    pushd /var/local/donkeys-git-dev4/drupal-7/sites/all/themes/heron

        mv subtheme.info.txt heron.info

        vi heron.info

        [find] subtheme [replace] heron

        # Uncomment stylesheets to override 'marinelli' theme (as above).

    popd

----

# DRUPAL 7 SITE CONFIGURATION.

# Login as admin user.
http://uk7.example.com/user

# Clear cache in Drupal 7 site (disable caching while developing).
http://uk7.example.com/admin/config/development/performance

# Complete Media module install.
http://uk7.example.com/admin/config/media/rebuild_types

# Check regional settings.
http://uk7.example.com/admin/config/regional/settings

    Default country:    United Kingdom
    Timezone new users: Default time zone
    
# Check date and time configuration.
http://uk7.example.com/admin/config/regional/date-time

# Replace spaces with a dash for contact forms.
http://uk7.example.com/admin/structure/contact/settings

# Check status report (run cron manually to check available updates).
http://uk7.example.com/admin/reports/status

# Migrate content type fields.
http://uk7.example.com/admin/structure/content_migrate

# Disable display of author and date information.
http://uk7.example.com/admin/structure/types/manage/pressrelease

# Enable and set as default the 'avocet' sub-theme.
http://uk7.example.com/admin/appearance

# Disable site name and site slogan.
# Disable default logo (files/donkeys/images/DonkeySanctuaryMasterTabLogo-Sanctuary-20120807.png)
# Disable default favicon (files/donkeys/images/rebrand_favicon.png).
# Disable CSS3 settings.
# Add custom path to custom logo and icon.
http://uk7.example.com/admin/appearance/settings/avocet

# Delete the URL redirect from 404 to node/1535.
http://uk7.example.com/admin/config/search/redirect

# Set 'URL path settings' for 'page not found' ('URL Alias' 404).
http://uk7.example.com/node/1535/edit

# Check default 403 and 404 error pages.
http://uk7.example.com/admin/config/system/site-information

# Override default front page.
http://uk7.example.com/admin/config/system/site-information

# Enable 'insert block' and 'insert views' filters.
http://uk7.example.com/admin/config/content/formats/3

# Enable 'search block'.
# Block title <none>
# Add to search region in 'avocet' (default) theme
# Show block on all pages
http://uk7.example.com/admin/structure/block/manage/search/form/configure

# Configure link checker.
http://uk7.example.com/admin/config/content/linkchecker

# Configure private file system path.
http://uk7.example.com/admin/config/media/file-system
    
    Public file system path:    /files/donkeys
    Private file system path    /var/local/donkeys-git-dev4/files-7
    Temporary directory :       /files/donkeys/tmp

# Convert uppercase strings, to lowercase (*are we sure*).
# http://uk7.example.com/admin/config/media/file-system/transliteration

----

# DRUPAL 7 TEASER DISPLAY MANAGEMENT.

# For each content type, set the image format for the teasers.
http://uk7.example.com/admin/structure/types/manage/blog/display/teaser

# For each content type, set the image format for the default displays.
http://uk7.example.com/admin/structure/types/manage/blog/display

# For each content type, hide all taxonomy fields.
http://uk7.example.com/admin/structure/types/manage/blog/display

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
http://uk7.example.com/admin/structure/menu/manage/navigation
http://uk7.example.com/admin/structure/menu/manage/user
http://uk7.example.com/admin/structure/menu/manage/management

# Delete all views.
http://uk7.example.com/admin/structure/views/view/all_aboard/delete
http://uk7.example.com/admin/structure/views/view/big_bray_2012/delete
http://uk7.example.com/admin/structure/views/view/calendar_est/delete
http://uk7.example.com/admin/structure/views/view/calendar_training/delete
http://uk7.example.com/admin/structure/views/view/carousel_india/delete
http://uk7.example.com/admin/structure/views/view/carousel_project_south_africa/delete
http://uk7.example.com/admin/structure/views/view/carousel_rug_appeal/delete
http://uk7.example.com/admin/structure/views/view/carousel_south_africa/delete
http://uk7.example.com/admin/structure/views/view/copyright/delete
http://uk7.example.com/admin/structure/views/view/events_pageviews/delete
http://uk7.example.com/admin/structure/views/view/homepage/delete
http://uk7.example.com/admin/structure/views/view/jenifer/delete
http://uk7.example.com/admin/structure/views/view/news_feed_dat_centres/delete
http://uk7.example.com/admin/structure/views/view/news_feed_est_belfast/delete
http://uk7.example.com/admin/structure/views/view/news_feed_est_birmingham/delete
http://uk7.example.com/admin/structure/views/view/news_feed_est_ivybridge/delete
http://uk7.example.com/admin/structure/views/view/news_feed_est_leeds/delete
http://uk7.example.com/admin/structure/views/view/news_feed_est_manchester/delete
http://uk7.example.com/admin/structure/views/view/news_feed_est_sidmouth/delete
http://uk7.example.com/admin/structure/views/view/press_release_pageviews/delete
http://uk7.example.com/admin/structure/views/view/south_africa/delete
http://uk7.example.com/admin/structure/views/view/tangible/delete
http://uk7.example.com/admin/structure/views/view/training_courses_pagecount/delete
http://uk7.example.com/admin/structure/views/view/training_courses_pageviews/delete
http://uk7.example.com/admin/structure/views/view/veterinarycare/delete

http://uk7.example.com/admin/structure/views/view/Appeals/delete
http://uk7.example.com/admin/structure/views/view/AshleyUpdate/delete

http://uk7.example.com/admin/structure/views/view/BeachDonkeys/delete
http://uk7.example.com/admin/structure/views/view/Blogger/delete
http://uk7.example.com/admin/structure/views/view/Bonaire/delete
http://uk7.example.com/admin/structure/views/view/BonaireGallery/delete
http://uk7.example.com/admin/structure/views/view/BrookfieldFarm/delete
http://uk7.example.com/admin/structure/views/view/BuxtonArrival/delete
http://uk7.example.com/admin/structure/views/view/BuxtonBlogs/delete
http://uk7.example.com/admin/structure/views/view/BuxtonGallery/delete
http://uk7.example.com/admin/structure/views/view/BuxtonNews/delete
http://uk7.example.com/admin/structure/views/view/BuxtonResidents/delete

http://uk7.example.com/admin/structure/views/view/Campaigns/delete
http://uk7.example.com/admin/structure/views/view/Cameroon/delete
http://uk7.example.com/admin/structure/views/view/CentreNews/delete
http://uk7.example.com/admin/structure/views/view/China/delete
http://uk7.example.com/admin/structure/views/view/CinnamonFoalGallery/delete
http://uk7.example.com/admin/structure/views/view/CurrentVacancies/delete
http://uk7.example.com/admin/structure/views/view/Cyprus/delete
http://uk7.example.com/admin/structure/views/view/CyprusGallery/delete

http://uk7.example.com/admin/structure/views/view/DonkeyHealthRelatedLinks/delete
http://uk7.example.com/admin/structure/views/view/DonkeyHistory/delete
http://uk7.example.com/admin/structure/views/view/DonkeyTourWorkshops/delete
http://uk7.example.com/admin/structure/views/view/DonkeyWeek/delete
http://uk7.example.com/admin/structure/views/view/DonkeyWelfare/delete
http://uk7.example.com/admin/structure/views/view/Dreams/delete

http://uk7.example.com/admin/structure/views/view/EastAxnollerFarm/delete
http://uk7.example.com/admin/structure/views/view/Egypt/delete
http://uk7.example.com/admin/structure/views/view/EgyptGallery/delete
http://uk7.example.com/admin/structure/views/view/ElisabethSvendsen/delete
http://uk7.example.com/admin/structure/views/view/EnvironmentFriendly/delete
http://uk7.example.com/admin/structure/views/view/ESTNews/delete
http://uk7.example.com/admin/structure/views/view/Ethiopia/delete
http://uk7.example.com/admin/structure/views/view/EthiopiaCommunity/delete
http://uk7.example.com/admin/structure/views/view/EthiopiaDirectHelp/delete
http://uk7.example.com/admin/structure/views/view/EthiopiaGallery/delete
http://uk7.example.com/admin/structure/views/view/EventDerbyshire/delete
http://uk7.example.com/admin/structure/views/view/EventEST/delete
http://uk7.example.com/admin/structure/views/view/EventSidmouth/delete
http://uk7.example.com/admin/structure/views/view/EventSponsorDonkey/delete
http://uk7.example.com/admin/structure/views/view/EventStage1/delete
http://uk7.example.com/admin/structure/views/view/EventStage2/delete
http://uk7.example.com/admin/structure/views/view/EventStage4/delete
http://uk7.example.com/admin/structure/views/view/EventStage5/delete
http://uk7.example.com/admin/structure/views/view/EventStage6/delete

http://uk7.example.com/admin/structure/views/view/FieldOfDreams/delete
http://uk7.example.com/admin/structure/views/view/FosteredDonkeys/delete
http://uk7.example.com/admin/structure/views/view/Fundraise/delete
http://uk7.example.com/admin/structure/views/view/FundraisersGallery/delete
http://uk7.example.com/admin/structure/views/view/FundraisingEvents/delete

http://uk7.example.com/admin/structure/views/view/Greece/delete

http://uk7.example.com/admin/structure/views/view/India/delete
http://uk7.example.com/admin/structure/views/view/IndiaDelhiGallery/delete
http://uk7.example.com/admin/structure/views/view/IndiaGallery/delete
http://uk7.example.com/admin/structure/views/view/Isolation/delete
http://uk7.example.com/admin/structure/views/view/Italy/delete
http://uk7.example.com/admin/structure/views/view/ItalyGallery/delete

http://uk7.example.com/admin/structure/views/view/JubileeGallery/delete

http://uk7.example.com/admin/structure/views/view/Kenya/delete
http://uk7.example.com/admin/structure/views/view/KenyaGallery/delete

http://uk7.example.com/admin/structure/views/view/Lamu/delete
http://uk7.example.com/admin/structure/views/view/LamuGallery/delete
http://uk7.example.com/admin/structure/views/view/LastingGift/delete
http://uk7.example.com/admin/structure/views/view/Legacies/delete
http://uk7.example.com/admin/structure/views/view/Links/delete
http://uk7.example.com/admin/structure/views/view/LoanSchemeCaseStudies/delete
http://uk7.example.com/admin/structure/views/view/LoanSchemeFosteringForum/delete
http://uk7.example.com/admin/structure/views/view/LoanSchemeGallery/delete
http://uk7.example.com/admin/structure/views/view/LoanSchemeTestimonials/delete
http://uk7.example.com/admin/structure/views/view/LoanSchemeWaitingGallery/delete

http://uk7.example.com/admin/structure/views/view/MeetDonkeys/delete
http://uk7.example.com/admin/structure/views/view/Memorials/delete
http://uk7.example.com/admin/structure/views/view/Mexico/delete
http://uk7.example.com/admin/structure/views/view/MexicoGallery/delete
http://uk7.example.com/admin/structure/views/view/Morocco/delete
http://uk7.example.com/admin/structure/views/view/Mules/delete

http://uk7.example.com/admin/structure/views/view/Nepal/delete

http://uk7.example.com/admin/structure/views/view/OldVacancies/delete
http://uk7.example.com/admin/structure/views/view/OurFarms/delete

http://uk7.example.com/admin/structure/views/view/PaccombeFarm/delete
http://uk7.example.com/admin/structure/views/view/Peru/delete
http://uk7.example.com/admin/structure/views/view/PoisonousPlants/delete
http://uk7.example.com/admin/structure/views/view/PoitouGallery/delete
http://uk7.example.com/admin/structure/views/view/PressRoom/delete
http://uk7.example.com/admin/structure/views/view/ProjectGallery/delete
http://uk7.example.com/admin/structure/views/view/ProjectsDevelopment/delete

http://uk7.example.com/admin/structure/views/view/Romania/delete
http://uk7.example.com/admin/structure/views/view/RomaniaGallery/delete

http://uk7.example.com/admin/structure/views/view/SanctuaryWalks/delete
http://uk7.example.com/admin/structure/views/view/SladeHouseFarm/delete
http://uk7.example.com/admin/structure/views/view/Spain/delete
http://uk7.example.com/admin/structure/views/view/SpainGallery/delete
http://uk7.example.com/admin/structure/views/view/SupportUs/delete

http://uk7.example.com/admin/structure/views/view/Tanzania/delete
http://uk7.example.com/admin/structure/views/view/TownBartonFarm/delete
http://uk7.example.com/admin/structure/views/view/TownBartonFarmGallery/delete
http://uk7.example.com/admin/structure/views/view/TrowFarm/delete

http://uk7.example.com/admin/structure/views/view/UnpublishedPressReleases/delete
http://uk7.example.com/admin/structure/views/view/UnpublishedVacancies/delete

http://uk7.example.com/admin/structure/views/view/VideoClips/delete
http://uk7.example.com/admin/structure/views/view/VideoClipsWelfare/delete

http://uk7.example.com/admin/structure/views/view/Webcams/delete
http://uk7.example.com/admin/structure/views/view/WhatsOn/delete
http://uk7.example.com/admin/structure/views/view/WoodsFarm/delete

http://uk7.example.com/admin/structure/views/view/Zambia/delete

# Disable 'frontpage'.
http://uk7.example.com/admin/structure/views/view/frontpage/disable

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
http://uk7.example.com/admin/config/development/performance

# Remove deprecated menus.
http://uk7.example.com/admin/structure/menu/manage/menu-stay-informed/delete
http://uk7.example.com/admin/structure/menu/manage/menu-what-we-do---/delete
http://uk7.example.com/admin/structure/menu/manage/menu-what-you-can-do/delete
http://uk7.example.com/admin/structure/menu/manage/menu-who-we-are---/delete

# Delete 'Main menu' menu items.
http://uk7.example.com/admin/structure/menu/item/80/delete
http://uk7.example.com/admin/structure/menu/item/208/delete
http://uk7.example.com/admin/structure/menu/item/209/delete
http://uk7.example.com/admin/structure/menu/item/224/delete
http://uk7.example.com/admin/structure/menu/item/241/delete
http://uk7.example.com/admin/structure/menu/item/313/delete
http://uk7.example.com/admin/structure/menu/item/624/delete
http://uk7.example.com/admin/structure/menu/item/968/delete

# Clear cache
http://uk7.example.com/admin/config/development/performance

# TODO...
Check module status.
http://uk7.example.com/admin/modules/uninstall

# Delete old blocks left after deleting views.
http://uk7.example.com/admin/structure/block/manage/block/12/delete
http://uk7.example.com/admin/structure/block/manage/block/14/delete
http://uk7.example.com/admin/structure/block/manage/block/15/delete
http://uk7.example.com/admin/structure/block/manage/block/16/delete
http://uk7.example.com/admin/structure/block/manage/block/17/delete
http://uk7.example.com/admin/structure/block/manage/block/19/delete
http://uk7.example.com/admin/structure/block/manage/block/18/delete
http://uk7.example.com/admin/structure/block/manage/block/20/delete (cookie control)
http://uk7.example.com/admin/structure/block/manage/block/21/delete
http://uk7.example.com/admin/structure/block/manage/block/22/delete
http://uk7.example.com/admin/structure/block/manage/block/23/delete
http://uk7.example.com/admin/structure/block/manage/block/25/delete
http://uk7.example.com/admin/structure/block/manage/block/26/delete
http://uk7.example.com/admin/structure/block/manage/block/27/delete
http://uk7.example.com/admin/structure/block/manage/block/29/delete
http://uk7.example.com/admin/structure/block/manage/block/30/delete
http://uk7.example.com/admin/structure/block/manage/block/31/delete
http://uk7.example.com/admin/structure/block/manage/block/32/delete
http://uk7.example.com/admin/structure/block/manage/block/35/delete
http://uk7.example.com/admin/structure/block/manage/block/36/delete

# Clear cache.
http://uk7.example.com/admin/config/development/performance

----

# Configure LightBox2.
http://uk7.example.com/admin/config/user-interface/lightbox2

    General > Lightbox2 lite > Show image caption

# Configure URL aliases.
# http://drupal.org/node/1266222
http://uk7.example.com/admin/config/search/path/patterns

    # Replace Drupal 6 pattern paths with Drupal 7 replacement patterns.

    [title-raw]     ->  [node:title]
    [nid]           ->  [node:nid]
    [vocab-raw]     ->  [term:name]
    [catpath-raw]   ->  [term:url]
    [user-raw]      ->  [user:name]
    [blog-raw]      ->  [user:name]

# JLT TODO...
# Check the roles assigned to text formats (check alongside Drupal 6 /admin/settings/filters). 
http://uk7.example.com/admin/config/content/formats

---------------------------------------------------------------
JLT up to here (20130530)
---------------------------------------------------------------

DRUPAL 7 PANEL PAGES.

# Import node view variants (override existing variants).
# http://uk7.example.com/admin/structure/pages/edit/node_view

# JLT TODO...
# Remove 'UK news' and 'International news' from blog types.
# Add 'UK news' and 'International news' to press release types.

# Enable node add/edit form.
http://uk7.example.com/admin/structure/pages

# Import node add/edit form.
# *** Disable Drupal blocks/regions ***
# *** Need to include new 'volunteer' variant.
http://uk7.example.com/admin/structure/pages/edit/node_edit

# Import stylizer settings (override existing variants).
# http://uk7.example.com/admin/structure/stylizer/import

# Import mini-panels (override existing).
# *** on hold until theming restarted ***
http://uk7.example.com/admin/structure/mini-panels/import

# Delete 'allotment' home page (delete existing landing page variant).
http://uk7.example.com/admin/structure/pages/edit/page-allotment_home_page

# Configure contact form.
# Create custom content forms.
# Adds specific titles for 'contact' pages (for example: Paccombe Training Centre).
http://uk7.example.com/admin/structure/contact/settings

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
http://uk7.example.com/trainingcourses

  - If you are interested in attending one of the courses, please 
    <a href="?q=contact&category=Paccombe%20Training%20Centre&subject=Training Centre courses">contact us</a> or telephone 01395 597644.

  + If you are interested in attending one of the courses, please <a title="Contact us" href="/contact/Paccombe Training Centre">contact us</a>.

# Example of custom content form (eg telephone number - 01395 597644 -replaces switchboard number).
http://uk7.example.com/admin/structure/contact/edit/27

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

# Disable the personal contact form by default for new users.
http://uk7.example.com/admin/config/people/accounts

# Hide user history.
http://uk7.example.com/admin/config/people/accounts/display

# Add new fields to 'people' account settings.
http://uk7.example.com/admin/config/people/accounts/fields

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
http://uk7.example.com/admin/config/people/accounts/display

# Add new field data to user accounts.
http://uk7.example.com/user/3/edit

    Photo:          JLT... todo
    Blogger name:   Jenifer Tucker
    About by blog:  Jenifer works at the Sanctuary as our Web Developer. She has loved donkeys ever since she can remember as well as being a donkey owner. One of her donkeys, Little Pippa, is here at the Sanctuary so if you're visiting, drop by to say hello and give Little Pippa a cuddle!
    Blog URL:       http://www.thedonkeysanctuary.org.uk/blogs/jenifer-tucker
    
# Check blog display.
http://uk7.example.com/blog/5689

----

# Import views (as and when required).
http://uk7.example.com/admin/structure/views/import
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
http://uk7.example.com/admin/appearance/settings


# Content type documents.
http://uk7.example.com/admin/structure/types/manage/corporate-partnership/fields/field_document

# Content type photos.
http://uk7.example.com/admin/structure/types/manage/corporate-partnership/fields/field_image_field

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
http://uk7.example.com/node/6402

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
http://uk7.example.com/admin/structure/types/manage/event/fields/field_document
http://uk7.example.com/admin/structure/types/manage/page/fields/field_document
http://uk7.example.com/admin/structure/types/manage/vacancy/fields/field_document
etc


JLT ... Testing 20130102 - got this error on dev4 (not on seven).

    Warning: Invalid argument supplied for foreach() in filefield_paths_entity_update() 
    (line 248 of /var/local/donkeys-git-dev4/drupal-7/sites/all/modules/filefield_paths/filefield_paths.module).


----

# DRUPAL 7 IMAGAGECACHE ACTIONS CONFIGURATION.

# Enable Imagecache Canvas Actions.
http://uk7.example.com/admin/modules

# JLT TODO... Configure

# Add 'copyright' notice to images.

----

# DRUPAL 7 CALENDAR.

# TODO... Try one calendar to display different categories (dropdown list of categories).
# Delete separate event views.
# What's on at Sidmouth
# What's on at each of the riding therapy centres
# Training courses
http://uk7.example.com/calendar

----

# DRUPAL 7 CORE AND CONTRIBUTED MODULE UPDATES.

# Ensure all contributed modules are up to date.
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-7

        # Clear drush cache.
        drush cc drush

        # Check available updates for enabled modules (automatically runs database updates if required).
        drush -l sanctuary pm-update


        # Remove robots.txt file if core updated.    

        mv robots.txt robots.txt.20121220

	popd

----

# DRUPAL 7 FULL LIST OF ADMINISTRATIVE TASKS FOR EACH MODULE.
http://uk7.example.com/admin/index

----

# THE END.
```