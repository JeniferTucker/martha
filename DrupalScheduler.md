# Introduction #

This module allows nodes to be published and unpublished on specified dates.

Dates can be entered either as plain text or with Javascript calendar popups (JSCalendar in Drupal 5, Date Popup in Drupal 6).

```
[admin@server~]

    pushd /var/local/donkeys-git-local/drupal-7

        # Download module.
        drush -l sanctuary dl scheduler

        # Enable module.
        drush -l sanctuary pm-enable scheduler

    popd

```

## Patch fix to work in Panels ##

Right now there's nothing to tell ctools about the scheduler form element - meaning that (for example) if you have a custom node edit page layout using Page Manager, the scheduler form element doesn't appear as an available content type, forcing you to choose between your custom layout and Scheduler.

Source:
http://drupal.org/node/1334780

```
[admin@server~]

    // Install patch.
    yum install patch

    pushd /var/local/donkeys-git-local/drupal-7/sites/all/modules/ctools/plugins/content_types

        // Download the patch.
        wget http://drupal.org/files/scheduler-ctools_info-1334780-8.patch

        // Apply the patch.
        patch scheduler_form_pane.inc < scheduler-ctools_info-1334780-8.patch

        // List directory content.
        drwxr-xr-x  2 6226 6226 4096 Mar  3 13:52 block
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 contact
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 custom
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 entity_context
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 form
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 node
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 node_context
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 node_form
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 page
        -rw-r--r--  1 root root 3029 Jan 10 17:37 scheduler-ctools_info-1334780-8.patch
        -rw-r--r--  1 root root 2499 Apr  3 11:06 scheduler_form_pane.inc
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 search
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 term_context
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 token
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 user_context
        drwxr-xr-x  2 6226 6226 4096 Aug 18  2012 vocabulary_context

        // Tidy up.
        rm scheduler-ctools_info-1334780-8.patch

    popd

```

## Enable Schedule for content types ##

Edit each content type you want to schedule a date/time to publish.

Administration > Structure > Content types > Press release

Scheduler settings > Publishing settings > Enable scheduled publishing
Scheduler settings > Publishing settings > Publishing date/time is required

Publishing options > Published (unchecked)

## Adding form element to Node add/edit page ##

Administration > Structure > Pages > Node add/edit form

Add contents > Node form scheduler

## Schedule content publication ##

Allow users to set a start and end time for content publication

Administration > People > Permissions > Scheduler


---


> pushd /var/local/donkeys-git-local/drupal-7/sites/all/modules/scheduler

> // Download the patch.
> wget http://drupal.org/files/scheduler-ctools_info-1334780-8.patch

> // Apply the patch.
> patch -p1 < scheduler-ctools\_info-1334780-8.patch

> popd