# Evaluating Drupal Commerce module #

```
---------------------------------

# LOCAL SETTINGS.

# Add new site details to local settings and functions.
[admin@server~]

	# Add 'shop' to settings.

	vi ~/local-settings.txt

		#[shop]
		shop_mysqldrupaluser=*****
		shop_mysqldrupalpass=*****
		shop_mysqldrupaldata=*****

	# Load settings and functions.
	source ~/local-settings.txt

---------------------------------

# DNS RECORD.

# Add the DNS alias to EasyDNS.

# Go to easyDNS website.

---------------------------------

# VIRTUAL HOST.

# Create virtual host files.
[admin@server~]

	pushd /etc/httpd/conf.d

cat > shop.example.com.conf << EOF
<VirtualHost *:80>
  ServerAdmin jenifer.tucker@example.com
  ServerName  shop.example.com

  LogLevel warn
  ErrorLog  /var/log/httpd/shop.example.com.error.log
  CustomLog /var/log/httpd/shop.example.com.access.log combined

  DocumentRoot /var/local/donkeys-git-local/drupal-7
  <Directory /var/local/donkeys-git-local/drupal-7>

  Options Indexes FollowSymLinks MultiViews
  AllowOverride All
  Order allow,deny
  allow from all

  </Directory>

</VirtualHost>
EOF

	# Reload Apache service.
	service httpd reload

    popd

---------------------------------

# DATABASE.

# Create 'shop' database.
[admin@server~]

	# Drop existing database.
	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
	  --execute="DROP DATABASE ${shop_mysqldrupaldata}"

	# Create database.
	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
	  --execute="CREATE DATABASE ${shop_mysqldrupaldata} DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci"

	# Grant permissions.
	mysql --user=${mysqladminuser} --password=${mysqladminpass} \
	  --execute="GRANT ALL ON ${shop_mysqldrupaldata}.* TO '${shop_mysqldrupaluser}'@'localhost' IDENTIFIED BY '${shop_mysqldrupalpass}'"

	# Connect to mySQL database.
	mysql --user=${mysqladminuser} --password=${mysqladminpass}

	# Test connect to database. 
	USE shop ;

	# Exit database.
	quit

---------------------------------

# MODULES.

# Download (and enable) modules.
[admin@server~]


	pushd /var/local/donkeys-git-local/drupal-7

		drush -l shop dl commerce
		drush -l shop dl commerce_option
		drush -l shop dl commerce_pricing_attributes
		drush -l shop dl commerce commerce_product_attributes
		drush -l shop dl commerce commerce_saleprice

		drush -l shop dl select_or_other
		drush -l shop dl commerce_donate

		drush -l shop pm-enable commerce
		drush -l shop pm-enable commerce_ui
		drush -l shop pm-enable commerce_product
		drush -l shop pm-enable commerce_cart

		drush -l shop pm-enable commerce_option
		drush -l shop pm-enable commerce_pricing_attributes
		drush -l shop pm-enable commerce commerce_product_attributes
		drush -l shop pm-enable commerce commerce_saleprice

		drush -l shop pm-enable select_or_other
		drush -l shop pm-enable commerce_donate

Bug fix...
https://drupal.org/node/1986592

	popd

---------------------------------

# SYMBOLIC LINKS.

# Create symbolic links.
[admin@server~]

	pushd /var/local/donkeys-git-local/drupal-7/sites

		ln -s shop shop.example.com

	popd

---------------------------------
```

## Site installation ##

  * User interface install new site.
  * http://shop.example.com/install.php

## Related links ##

  * Commerce radioactivity.
  * DrupalCommerceRadioactivity