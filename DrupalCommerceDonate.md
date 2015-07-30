
```
[admin@server]

    pushd /var/local/donkeys-git-local/drupal-7

        # Download 'dev' module.
        # (beta version contains bug)
        drush -l shop dl commerce_donate --select

        # Enable module.
        drush -l shop pm-enable commerce_donate

    popd

== Configuration ==

# Create a donation product at Administration > Store > Products

    SKU:    DONATION
    Title:  General donation
    Amount: 0.00

# Capture donations during the checkout process.

# Ensure the "Donation form" checkout pane is enabled at 
Store > Configuration > Checkout settings

# Configure the "Donation form" checkout pane with the donation product created

== Customisation ==

  * To override the default donation amount options:

  1) Install the "Commerce Customizable Products" module.

  2) Edit the "Amount" field at Administration > Store > Configuration > Line
    iitem types > Donation and alter the default options there.

Read module README.txt file.
```