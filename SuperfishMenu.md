# Superfish menu #

This module implements the jQuery plugin called Superfish [1](1.md) on Drupal menus. It provides Drupal users the ability to add some "splash" to Drupal menus with very little effort.

## Requirements ##

  * Superfish library
  * Libraries module (already installed)

```
# Add Superfish module.
# http://drupal.org/project/superfish
[root@pippa~]

    # Download Superfish library from github.
    
    mkdir /var/local/superfish-git

    git-clone https://github.com/mehrpadin/Superfish-for-Drupal.git

    pushd /var/local/donkeys-git-local/drupal-7/sites/all/libraries

        # Create link to Superfish library.
        ln -s /var/local/superfish-git/Superfish-for-Drupal/ superfish

    popd

    pushd /var/local/donkeys-git-local/drupal-7

        # Download and enable superfish module.
        drush -l sanctuary dl superfish
        drush -l sanctuary --yes pm-enable superfish

    popd

# Configure the Superfish module.
/admin/config/user-interface/superfish

  Number of blocks:  4 (default)

  Path to Superfish library: (default configuration)
  sites/all/libraries/superfish/jquery.hoverIntent.minified.js
  sites/all/libraries/superfish/jquery.bgiframe.min.js
  sites/all/libraries/superfish/superfish.js
  sites/all/libraries/superfish/supersubs.js
  sites/all/libraries/superfish/supposition.js
  sites/all/libraries/superfish/sftouchscreen.js

# Configure menu.
/admin/structure/menu

Title:        Main menu
Description:  Primary navigtion.
XML sitemap:  Excluded

# Configure Superfish 1 (Superfish)" block.
/admin/structure/block

  Menu parent:     Main menu
  Menu type:       Horizontal
  Style:           Coffee
  Region settings: Curlew > Topbar
                   Garland > Header
                   Avocet > Topbar
  Auto-arrows:     Enabled

  # Superfish plugins
  Use jQuery BgiFrame plugin for this menu.
  (Helps ease the pain when having to deal with IE z-index issues.)
  Use jQuery Supposition plugin for this menu.

  # sf-Touchscreen (Beta)
  sf-Touchscreen provides touchscreen compatibility for your menus.
  Use jQuery sf-Touchscreen plugin for this menu.
  Use this mode only if below user agents were detected. 
    iPhone*Android*iPad
  (all other settings left as default)


```

# Disable Superfish module #

Informed by Mark Cross today (23/01/13) not to proceed with this drop down style of primary navigation and to carry on with mega menu style.

```
# Disable Superfish module.
[root@pippa~]

    pushd /var/local/donkeys-git-local/drupal-7

        drush -l sanctuary --yes pm-disable superfish

    popd

    # Remove Superfish library.
    
    pushd /var/local/donkeys-git-local/drupal-7/sites/all/libraries

        rm -rf superfish

    popd

    # Tidy up initial download directory.

    pushd /var/local

        rm -rf superfish-git

    popd

    # Tidy up modules directory.

    pushd /var/local/donkeys-git-local/drupal-7/sites/all/modules

        rm -rf superfish

    popd
```