
```
== Module ==

  * Video Embed Field Module
  * http://drupal.org/project/video_embed_field

== Install ==

[admin@server]#

     pushd /var/local/donkeys-git-local/drupal-7
     
     drush -l sanctuary dl video_embed_field
     drush -l sanctuary -y pm-enable video_embed_field

== Settings ==

# Disable related videos.
/admin/config/media/vef_video_styles

# Add video field to content type (video clip).
/admin/structure/types/manage/videoclip/fields

# Add field to 'Node add/edit form' for video.
/admin/structure/pages/nojs/operation/node_edit/handlers/node_edit_panel_context_9/content

```