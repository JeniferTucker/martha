
```

# Create multi-language home pages.
# http://bengoodyear.com/blog/drupal-7-cracking-the-multilingual-front-page-nut/

# Install and enable modules.

Variable
Variable translations

# Configure 'English' regional variables.
http://www.example.com/admin/config/regional/i18n/variable

# Select 'Site information'.
# Enable 'Default front page'.

# Configure 'Italian' regional variables.
# Add 'it' prefix to URL to configure 'Italian' regional variables.
http://www.example.com/it/admin/config/regional/i18n/variable

# Select 'Site information'.
# Enable 'Default front page'.

# Enable content translation links.
http://www.example.com/admin/config/regional/i18n/node

-------------------------------

# Create 'slide' taxonomy terms.

English homepage
Italian homepage

# Create slide.
http://italy7.thedonkeysanctuary.org.uk/#overlay=node/add/slide

    Notice: Undefined property: stdClass::$vid in i18n_taxonomy_term_name() (line 472 of /var/local/donkeys7-git/drupal-7/sites/all/modules/i18n/i18n_taxonomy/i18n_taxonomy.module).
    Warning: array_flip() [function.array-flip]: Can only flip STRING and INTEGER values! in DrupalDefaultEntityController->load() (line 173 of /var/local/donkeys7-git/drupal-7/includes/entity.inc).
    Warning: array_flip() [function.array-flip]: Can only flip STRING and INTEGER values! in DrupalDefaultEntityController->cacheGet() (line 350 of /var/local/donkeys7-git/drupal-7/includes/entity.inc).
    Notice: Undefined property: stdClass::$vid in i18n_taxonomy_term_name() (line 472 of /var/local/donkeys7-git/drupal-7/sites/all/modules/i18n/i18n_taxonomy/i18n_taxonomy.module).

-------------------------------

# Export/import view of slides from uk7 site.


-------------------------------

# TB Menu translation.


... To figure out ....


-------------------------------

# Checking fields for slide content type.
# http://www.example.com/admin/structure/types/manage/slide/fields/field_slide_image

# Click save (without making any changes) caused these errors to display.

    Notice: Undefined index: title_value in link_field_update_instance() (line 1305 of /var/local/donkeys7-git/drupal-7/sites/all/modules/link/link.module).
    Notice: Undefined index: title_value in link_field_update_instance() (line 1305 of /var/local/donkeys7-git/drupal-7/sites/all/modules/link/link.module).

# Enable active updating for image field.

# Click save (errors displayed again) but enabled field in place.


```


Interesting module?
https://drupal.org/project/rabbit_hole

Rabbit Hole is a module that adds the ability to control what should happen when an entity is being viewed at its own page.

Perhaps you have a content type that never should be displayed on its own page, like an image content type that's displayed in a carousel. Rabbit Hole can prevent this node from being accessible on its own page, through node/xxx.