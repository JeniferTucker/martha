# Investigation #

Download taxonomy term reference tree module.
    # Download module.
    drush -l sanctuary dl term_reference_tree

    # Enable module.
    drush -l sanctuary --yes pm-enable term_reference_tree

    # Go to the Manage Fields tab of any fieldable entity (such as a content type, taxonomy term, or user).

    http://seven.stanleywindrush.org.uk/admin/structure/types/manage/blog/fields

    New field:  Term Reference
    Field type: Term Reference
    Widget:     Term reference tree

    # Test.
http://seven.stanleywindrush.org.uk/node/add/blog

= Source =

http://drupal.org/project/term_reference_tree```