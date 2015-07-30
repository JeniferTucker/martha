Commerce Product Popularity integrates with Radioactivity to provide a
product popularity field. If you're not familiar with Radioactivity, then it's
probably worth familiarizing yourself. This module allows you to configure
individual field instances to be updated on checkout. Every time a customer
pays for an order, any appropriately configured Radioactivity fields in any
products or product displays will have their energy increased (proportional to
the quantity of products sold).

It can be used for creating Views of most popular products and marking
individual products (or product displays) as 'hot sellers'.

```
= Module =

  * Radioactivity
  * http://www.drupal.org/radioactivity

  * Commerce Product Popularity
  * http://drupal.org/project/commerce_productpopularity

== Install ==

[admin@server]#

     pushd /var/local/donkeys-git-local/drupal-7
     
         drush -l shop dl radioactivity
         drush -l shop pm-enable radioactivity

         drush -l shop dl commerce_productpopularity
         drush -l shop  pm-enable commerce_productpopularity

     popd

== README.txt ==

The following are required:

* Drupal Commerce (commerce)[2]
* Entity API (entity)[3]
* Radioactivity (radioactivity)[1]
* Rules (rules)[4]

1.  Ensure you've downloaded the dependencies.
2.  Enable Commerce Product Popularity as normal[5].
3.  Create a Radioactivity profile (see example[6]).
4.  Add a Radioactivity field to a product or a product display, and on the
    field instance settings page, check the checkbox marked 'Update with
    Commerce Product Popularity'.
5.  If you only want purchases to affect the item's popularity (this is the
    anticipated use) then ensure the field's formatter's not configured to add
    energy on view.

Where's example 6??

== Admin settings ==

# Administer Radioactivity decay profiles.
http://example.com/admin/structure/radioactivity

== Tutorials ==

  * Radioactivity 2: basics
  * http://www.wunderkraut.com/blog/radioactivity-2-basics/2011-12-05

```