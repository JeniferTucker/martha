# Details #

These instructions will ensure:

  * Empty database created.
  * DNS aliases set up (if required).
  * Virtual hosts set up.

```
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

# DRUPAL 7 SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

pushd /var/local/donkeys-git-dev4/drupal-7/sites

    ln -s italy www.example.com

popd

----

# DRUPAL 7 FILES DIRECTORY.

# Create files directory.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7/sites/italy

        # Create files directory.
        mkdir files

        # Set file permissions.
    	id apache
	    # uid=48(apache) gid=48(apache) groups=48(apache)

    	chgrp -R apache files
    	chmod -R g+w    files

    	pushd files

        	find . -type d -exec chmod g+ws '{}' \;

    	popd

	popd

----

# DRUPAL 7 MODULES.

# Add new contributed modules.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7

        # Download contributed modules.
        drush -l italy dl module_filter
        drush -l italy dl robotstxt
        drush -l italy dl admin_language
        drush -l italy dl redirect

        # Download Calendar module (and dependencies).
        drush -l italy dl date
        drush -l italy dl calendar

        # Download Internatinalization module (and dependencies).
        drush -l italy dl i18n
        drush -l italy dl i18nviews
        drush -l italy dl l10n_update
        drush -l italy dl ctools
        drush -l italy dl pathauto
        drush -l italy dl token
        drush -l italy dl transliteration
        drush -l italy dl variable
        drush -l italy dl views

        # Download image handling modules.
        drush -l sanctuary dl filefield_paths
        drush -l sanctuary dl filefield_sources
        drush -l sanctuary dl imagecache_actions

        # Download Lightbox2 modules.
        drush -l italy dl lightbox2

        # Download Panel modules.
        drush -l italy dl panels

	# Enable core modules.       
    drush -l italy --yes pm-enable contact

	# Enable contributed modules.       
    drush -l italy --yes pm-enable module_filter
    drush -l italy --yes pm-enable robotstxt
    drush -l italy --yes pm-enable admin_language
    drush -l italy --yes pm-enable redirect

	# Enable Calendar modules.       
    drush -l italy --yes pm-enable date
    drush -l italy --yes pm-enable date_api
    drush -l italy --yes pm-enable calendar

	# Enable Internationalization modules.       
    drush -l italy --yes pm-enable i18n
    drush -l italy --yes pm-enable i18nviews
    drush -l italy --yes pm-enable l10n_update
    drush -l italy --yes pm-enable ctools
    drush -l italy --yes pm-enable pathauto
    drush -l italy --yes pm-enable token
    drush -l italy --yes pm-enable transliteration
    drush -l italy --yes pm-enable variable
    drush -l italy --yes pm-enable views
    drush -l italy --yes pm-enable views_ui

	# Enable image handlng modules.       
    drush -l italy --yes pm-enable filefield_paths
    drush -l italy --yes pm-enable filefield_sources
    drush -l italy --yes pm-enable imagecache_actions

	# Enable Lightbox2 modules.       
    drush -l italy --yes pm-enable lightbox2

	# Enable Panels modules.
    drush -l italy --yes pm-enable panels
    drush -l italy --yes pm-enable stylizer
    drush -l italy --yes pm-enable page_manager

    # Clear all caches.
    drush -l italy cc all

	# Enabled manually.
	# http://www.example.com/admin/modules#multilingual__internationalization
    Field translation
    Content translation

    Block languages
    Field translation
    Contact translation
    Menu translation
    Multilingual content
    Multilingual select
    String translation
    Synchronize translations
    Taxonomy translation
    Translation sets
    Views translation


----

# DRUPAL 7 SCHEDULER.

    # Patch fix for ctools module.
    # Required if Scheduler module required (requested by Italy).
    # http://code.google.com/p/martha/wiki/DrupalScheduler
    # drush -l italy dl ctools-7.x-1.x-dev

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
        cp -r subtheme ../pufulet
        cp -r css      ../pufulet

    popd

# Rename the subtheme.info.txt file to match the new theme name.
[admin@server~]

    pushd /var/local/donkeys-git-dev4/drupal-7/sites/all/themes/pufulet

        mv subtheme.info.txt pufulet.info

        vi pufulet.info

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

----

# DRUPAL 7 CUSTOM THEMING.

[admin@server~]

    # Add view image style. 
    pushd /var/local/donkeys-git-dev4/drupal-7/sites/all/themes/pufulet/css/reset

        vi reset.css

        G (end of file command)

/* BEGIN float images right */


.field-type-image {
  float:       right;
  margin-left: 1em;
  clear:       both;
}

/* END float images right */

    popd

----

# DRUPAL 7 OVERRIDES.

# Remove robots.txt file (using robotsTxt module).
[admin@server~]

	pushd /var/local/donkeys-git-dev4/drupal-7

        rm robots.txt

    popd

----

# DRUPAL 7 MULTILANGUAGE CONFIGURATION.

# Regional and local settings.
http://www.example.com/admin/config/regional/settings

    Default country:    United Kingdom
    Timezone new users: Default time zone

# Add a language.
http://www.example.com/admin/config/regional/language

	Predefined language:	Italian (Italiano)

# Multilingual settings.
http://www.example.com/admin/config/regional/i18n

	Enabled languages only

# User interface text language detection.
http://www.example.com/admin/config/regional/language

	Detection method:	URL
----

# DRUPAL 7 ROBOTS CONFIGURATION.

# Disallow 'all' during development.
http://www.example.com/admin/config/search/robotstxt

	User-agent: *
	Crawl-delay: 10
	# Directories
  + Disallow: /

	# Ethics
	# http://www.wired.com/epicenter/2010/08/robot-laws/
	Disallow: /harming/humans
	Disallow: /ignoring/human/orders
	Disallow: /harm/to/self

----

# DRUPAL 7 SITE CONFIGURATION.

# Login as admin user.
http://www.example.com/user

# Clear cache in Drupal 7 site (disable caching while developing).
http://www.example.com/admin/config/development/performance

# Replace spaces with a dash for contact forms.
# http://www.example.com/admin/structure/contact/settings

# Check status report (run cron manually to check available updates).
http://www.example.com/admin/reports/status

# TODO... set up 403 and 404 pages.
http://www.example.com/admin/config/search/redirect

# TODO... Fix 404 errors.
http://www.example.com/admin/config/search/redirect/404

# Configure link checker.
# http://www.example.com/admin/config/content/linkchecker

----

# DRUPAL 7 CONTENT TYPES.

# Add new content types.
http://www.example.com/admin/structure/types

	Event
	Press release
	Web form


# DRUPAL 7 TEASER DISPLAY MANAGEMENT.

# For each content type, set the image format for the teasers.
# http://www.example.com/admin/structure/types/manage/page/display/teaser

# For each content type, set the image format for the default displays.
# http://www.example.com/admin/structure/types/manage/page/display

# For each content type, hide all taxonomy fields.
# http://www.example.com/admin/structure/types/manage/page/display

----

# DRUPAL 7 CONTENT PATHS.

# Configure content paths.
http://www.example.com/admin/config/search/path/patterns

    Default         ->  content/[node:nid]
    Content         ->  content/[node:nid]
    Event           ->  event/[node:nid]
    Page            ->  page/[node:nid]
    Story           ->  story/[node:nid]
    User path       ->  users/[user:name]
    Webform         ->  webform/[node:nid]

# TODO...
# Get Italian version of English content paths.

----

# DRUPAL 7 FILE FIELD PATHS.

# Content type photos.
http://www.example.com/page/4#overlay=admin/structure/types/manage/event/fields/field_image

# Configure file (field) path settings for each content type.

        File name:      [node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]

        # File name options.
        Enable:         Transliterate

        ----
        
        File name:      [node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]

        # File name options.
        Enable:         Transliterate

        ----
        
        File name:      [node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]

        # File name options.
        Enable:         Transliterate

        ----
        
        File name:      [node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]

        # File name options.
        Enable:         Transliterate

        ----

        File name:      [node:nid]-[current-date:raw]-[file:ffp-name-only-original].[file:ffp-extension-original]

        # File name options.
        Enable:         Transliterate

----

# DRUPAL 7 CONFIGURE IMAGE CACHE ACTIONS.

# Enable 'Imagecache Canvas Actions', 'Imagecache Actions' and 'Image Effects Test' modules.
http://www.example.com/admin/modules

# Overlay logo on 'large' sized images.
http://www.example.com/admin/config/media/image-styles/edit/large

    Effect:     Overlay (watermark)
    File path:  sites/italy/files/donkeysanctuarymastertablogo-sanctuary-20120807.png
    
# Enable 'Image Effects Text' module.
http://www.example.com/admin/modules

# Clear performance cache.
http://www.example.com/admin/config/development/performance

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
   
# TODO... fix.
# Duplicating images on the server (in files and styles directories).

----

# DRUPAL 7 PANEL PAGES.

# Enable 'Node template', 'Node add/edit form' and 'Site contact page'.
http://www.example.com/admin/structure/pages

----

# DRUPAL 7 FILE PATHS.

# File system configuration.
http://www.example.com/admin/config/media/file-system

    Public file system path:
    sites/italy/files

    Temporary directory:
    sites/italy/files/tmp

----

# DRUPAL 7 CALENDAR.

    # Check site timezone and first day of week settings.
    http://www.example.com/admin/config/regional/settings
    # Check date format settings.
    http://www.example.com/admin/config/regional/date-time

# Prepare an Event-Calendar for Drupal 7.
https://drupal.org/node/1477602


--------------------

# User documentation.
# Follow to set up.
https://drupal.org/node/1268692

# THE END.

```