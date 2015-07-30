# Question: How do I change the length of a mini panel? #

The question arose because adding panels/mini panels into my Marinelli theme, used the default settings and gave borders/margins, etc around each of the elements.

What I wanted to achieve was a consistent vertical alignment of logo, banner and panel elements down the page.

# Answer #

After many hours, if not days of looking at this, I figured out a way ... not sure if it's the right way, but time will tell!

Edit mini panel that was previously created with all the default 'builders' settings.

/admin/structure/mini-panels/list/allotment\_boxes/edit/content

Click on 'Show layout designer'

Click on Canvas > Canvas settings

The 'Fixed width' text box is empty.

Added 1080

(If a value is entered, the layout canvas will be fixed to the given pixel width.)

Manually adjusted each of my 4 columns to be 248 pixels wide (allowing for the default 0.5 spacing that can also be changed in the canvas settings).

Save.




See also:

  * MarinelliThemeChanges

