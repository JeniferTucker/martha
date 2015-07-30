
```
[drupalroot]

// Add patch file to Marinelli theme directory.
// (sites/all/themes/marinelli)

// Copy over banner.inc (just in case).
pushd sites/all/themes/marinelli/logics

    cp banner.inc banner.inc.original

popd

// Run patch.
pushd sites/all/themes/marinelli

    patch -p1 < filename.patch 

popd


-------------------

You can now add tranlations to Marinelli banners.

Select theme settings.
www.example.com/admin/appearance/settings/[sub-theme]

Banner management.
Upload a new banner (if no banners images in place).
Save configuration.

Switch language to generate the strings to be translated.

Translate the text in the "Translate interface".
www.example.com/admin/config/regional/translate/translate

Use the 'Filter translatable strings' to find the text you want to translate.

Click on 'edit' under 'operations'.
Add translated text.
Click on 'Save translations'.
```

Source:
https://drupal.org/node/1094284#comment-6845938