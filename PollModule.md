# Poll module #

A poll is a question with a set of possible responses. A poll, once created, automatically provides a simple running count of the number of votes received for each response.

## Configuration ##

  * Enable the core Poll module.
  * /admin/modules

This will enable a Poll content type.

  * Add a 'Title' field to the content type.
  * /admin/structure/types/manage/poll/fields

  * Hide the poll results.
  * /admin/structure/types/manage/poll/display

### Add a poll ###

  * Create a new Poll content type.
  * node/add/poll

### Admin Menu link ###

If you don't see a link to "Poll" under Create content > Poll, check the bottom on the Navigation menu and move it into place.

/admin/structure/menu/manage/navigation

## Problems ##

This is far from easy to do!

With the popup, how do you redirect people if user clicks "X" ?

Useful links:

  * http://drupal.org/node/252260
  * http://drupal.org/project/overlay_paths
  * http://drupal.stackexchange.com/questions/3138/how-do-i-show-content-in-an-
overlay