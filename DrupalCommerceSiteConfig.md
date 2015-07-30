
```
------------ Warehouse Store configuration ------------

# Set store currency to GBP.
http://shop.example.com/admin/commerce/config/currency

# Configure product type.
http://shop.example.com/admin/commerce/products/types/product/fields

	# Add new field (use default settings for now).
	Label:		Desription
	Field type:	Long text

	# Add existing field (use default settings for now).
	Label:	Image
	Machine name:	Image: field_image (Image)
	Widget:		Image

# Add store products.
http://shop.example.com/admin/commerce/products

	Product SKU:	254
	Title:		Dave
	Image:		(upload)
	Price:		8.50
	Description:	Dave would just love to be hugged! 
			Approx 20cm (8"). 
			Conforms to BS5665 and EN71. 
			* Not suitable for children under three years.
	
------------ Shop display configuration ------------

# Disable comments.
http://shop.example.com/admin/modules

# Create a new 'product' content type.
http://shop.example.com/admin/structure/types/add

Name:		Shop product
Description:	A new product for sale in our online giftshop.

# Add a reference to the store product type.
http://shop.example.com/admin/structure/types/manage/shop-product/fields

	# Add a new field.
	Label:		Product reference
	Mahine name:	(autocompletes)
	Field type:	Product reference
	Widget:		Autocomplete text field

	Required field:				Enable
	Product types that can be referenced:	Product

# Delete 'body' field (?).


# 'Add to Cart' on Teasers
By neardark on on July 22, 2011

    Choose Structure > Content Types > [ your product display content type > Manage Display
    Click the Teaser option at the top-right of the page
    Drag the Product Reference field above the Hidden header
    Order the fields the in the order you want the fields to be displayed on the teaser
    Click Save


------------ Shop display (add products) ------------

# Permissions.
# Allow anonymous users to see views.

# Add a new product for sale.
http://shop.example.com/node/add/shop-product

	Title:			Dave
	Product reference:	(autocomplete)

# Create a 'new product' view.
http://shop.example.com/admin/structure/views/add


```

## Extending Drupal commerce ##

  * Shop display popularity.
  * Requires DrupalCommerceRadioactivity module enabled.