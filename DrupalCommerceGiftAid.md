# Introduction #

This module adds support for the UK gift aid process to Drupal commerce.

A product can be defined as being eligible for gift aid. If an order contains any gift aid products then the declaration message is shown on checkout for the whole order. Each line item that is eligible for gift aid then gets its gift aid field ticked.

There is a simple view included which adds a link under the store -> orders menu that lists all line items which have the gift aid flag set. That is, every line item referencing an eligible product which the user clicked yes to.

```
[admin@server~]

    pushd /var/local/donkeys-git-local/drupal-7

        # Download module.
        drush -l shop dl commerce_giftaid

        # Enable module.
        drush -l shop pm-enable commerce_giftaid

    popd

```

## Bug ##

```
[admin@server~]

Notice: Undefined index: commerce_giftaid_pane in commerce_giftaid_pane_checkout_form_validate() (line 45 of /var/local/donkeys-git-local/drupal-7/sites/all/modules/commerce_giftaid/commerce_giftaid.checkout_pane.inc).

    pushd /var/local/donkeys-git-local/drupal-7/sites/all/modules/commerce_giftaid

        // Download the patch.
        wget http://drupal.org/files/commerce_giftaid-hook_order_presave-1792744-1.patch

        // Apply the patch.
        git apply commerce_giftaid-hook_order_presave-1792744-1.patch

        // Tidy up.
        rm commerce_giftaid-hook_presave-1792744-1.patch

    popd

```

## Gift Aid declaration ##

There should be tab under Store > Orders to view for "Gift aid report".
http://example.com/admin/commerce/orders/gift-aid-report