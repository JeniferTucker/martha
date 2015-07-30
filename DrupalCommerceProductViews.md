
```

== Acknowledgement ==

This took a while to solve how to create a view of images that link through to the content for each product.

I eventually found the solution posted by neardark on 8 June 8, 2011.

  * Product image linking to product, not product admin
  * http://www.drupalcommerce.org/comment/1291#comment-1291

== Views ==

This view assumes that you have already created a content type for adding your 'shop' products
(which added the "Product reference" field as "Autocomplete text field widget"
with "Product reference field settings" set to "Unlimited").

# Add a view.
http://example.com/tutorial2#overlay=admin/structure/views/add

	View name:	New products

	Show:		Content > of type > Shop

	Create a page:

		Page title:	New products
		Path:		new-products

	Display format:		Grid of Fields

	Items to display:	8

	... continue and edit

	Advanced > Relationships > Add

		Enable:		Content: Referenced product

		... Apply (all settings)

		Enable:		Require this relationship

		... Apply (all settings)

	Fields > Add

		Content:	Path > Exclude from display
                                *** this field must be above the image field ***

		    Ignore:
			Display "Master" uses fields but there are none defined for it or all are excluded.
		    	Display "Page" uses fields but there are none defined for it or all are excluded.
		    (this will go away when fields are added)

		Commerce:	Product image

				Relationship:	Shop
				Enable:		Exclude from Display

				Image style:	Thumbnail
				Link image to:	Nothing
				
				Rewrite results:
				  Enable:	Output this field as a link
				  Link path:	[path]
				  Enable:	Use absolute path

		Content:	Title

		Commerce Product: Price
				    Formatter:	Formatted amount

		Commerce product: Add to Cart form

	Save

== Permissions ==

View any Product product
Warning: Give to trusted roles only; this permission has security implications.
http://example.com/admin/people/permissions#module-views


```