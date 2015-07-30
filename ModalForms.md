# Modal Forms module #

Modal forms make use of the modal feature in the ctools module to open some common forms in a modal window.

## Requirements ##

  * ctools (already installed)

```
# Add Modal Forms module.
# http://drupal.org/project/superfish
[root@pippa~]

    pushd /var/local/donkeys-git-local/drupal-7

        # Download and enable Modal Forms module.
        drush -l sanctuary dl modal_forms
        drush -l sanctuary --yes pm-enable modal_forms

    popd

# Configure the Modal Forms module.
/admin/config/development/modal_forms

=== Webforms ===

There is support to open webforms in a modal by constructing special links.

A webform link should look like this one:

<a class="ctools-use-modal ctools-modal-modal-popup-large" href="/modal_forms/nojs/webform/[nid]">Link to click</a>

Replace [nid] with the node id of your webform.

Second class is optional, you can use one of this:

ctools-modal-modal-popup-small (300x300);
ctools-modal-modal-popup-medium (550x450);
ctools-modal-modal-popup-large (80%x80%).

```

## Verdict ##

This possibly isn't the solution for a pop-up style survey question as it suggests that the user has to click on a link first, then the popup will be displayed.

... Look at PollModule.

```
# Disable Modal Forms module.
# http://drupal.org/project/superfish
[root@pippa~]

    pushd /var/local/donkeys-git-local/drupal-7

        # Disable Modal Forms module.
        drush -l sanctuary --yes pm-disable modal_forms

        pushd /var/local/donkeys-git-local/drupal-7/sites/all/modules

            # Tidy up modules directory.
            rm -rf modal_forms

        popd

    popd

```