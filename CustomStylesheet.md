# Create custom stylesheet.
[root@pippa ~]

> pushd /var/local/donkeys-git-local/drupal-7/sites/all/themes/avocet/css

> touch custom.css

> popd

# Add custom stylesheet to array list.
[root@pippa ~]

> pushd /var/local/donkeys-git-local/drupal-7/sites/all/themes/avocet

> vi avocet.css

> + stylesheets[all](all.md)[.md](.md) = css/custom.css

> popd

# Add CSS properties to home page panes.
/admin/structure/mini-panels/list/allotment\_boxes/edit/content

```
    # Configure CSS on New custom content

    Column1:    Custom: Donate box
                CSS ID: homepage-donate-box

                Class: pane-plain-box-allotment-secondary-violet

    Column2:    Custom: Adopt box
                CSS ID: homepage-adopt-box

                Class: pane-plain-box-allotment-primary-blue

    Column3:    Custom: Donkey cruelty box
                CSS ID: homepage-donkey-cruelty-box

                Class: pane-plain-box-allotment-secondary-red

    Column4:    Custom: Giftshop box
                CSS ID: homepage-giftshop-box

                Class: pane-plain-box-allotment-primary-brown
```


---


# Add class IDs to custom CSS file.
[root@pippa ~]

> pushd /var/local/donkeys-git-local/drupal-7/sites/all/themes/avocet/css

> vi custom.css

```
        /*
        *
        *
        * Custom-specific layout statements
        *
        */


        /* Panels
        -------------------------------------------------------------- */

        .pane-plain-box-allotment-secondary-violet 
        .pane-plain-box-allotment-primary-blue 
        .pane-plain-box-allotment-secondary-red 
        .pane-plain-box-allotment-primary-brown {
          padding-right: 0.5em;
          padding-left: 0em;
        }

        #homepage-donate-box {
            padding-left: 0;
            padding-right: 0.5em;
        }

        #homepage-adopt-box {
            padding-left: 0.1em;
            padding-right: 0.5em;
        }

        #homepage-donkey-cruelty-box {
            padding-left: 0.2em;
            padding-right: 0.5em;
        }

        #homepage-giftshop-box {
            padding-left: 0.3em;
            padding-right: 0.5em;
        }
```


---


## Bug ##

Required the 'cached' css files in /files/donkeys/ctools/css to be manually tweaked for each of the padding-lefts for pane-content to be changed from 0.5em to 0em

It appears the clearing cache in Drupal does not clear the ctools cached css files.

Monitor and find a way to fix permanently using the panels class/id properties.