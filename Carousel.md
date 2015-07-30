Look at this module for rolling jCarousel

This module provides a central function for adding jCarousel jQuery plugin
elements.

http://drupal.org/project/jcarousel

```
[root@pippa]

    pushd /var/local/donkeys-git-local/drupal-7

        drush -l sanctuary dl jcarousel

    popd
```

### Usage ###

  * Add a new view.
/admin/structure/views/add


View name:  Working worldwide

Create a block: (no title)
Display format: jCarousel of titles (linked)
Items per page: 8


  * Click on the "Settings" link next to the jCarousel Format to configure the
options for the carousel such as the animation speed and skin.
/admin/structure/views/nojs/display/working\_worldwide/block/style\_options

Leave as defaults.

  * Add the items you would like to include in the rotator under the "Fields"
section, and build out the rest of the view as you would normally. Note that
the preview of the carousel within Views probably will not appear correctly
because the necessary JavaScript and CSS is not loaded in the Views
interface. Save your view and visit a page URL containing the view to see
how it appears.