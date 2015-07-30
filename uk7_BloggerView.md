
```
$view = new view();
$view->name = 'blogger';
$view->description = '';
$view->tag = '';
$view->base_table = 'users';
$view->human_name = 'Blogger';
$view->core = 7;
$view->api_version = '3.0';
$view->disabled = FALSE; /* Edit this to true to make a default view disabled initially */

/* Display: Master */
$handler = $view->new_display('default', 'Master', 'default');
$handler->display->display_options['title'] = 'About this blog';
$handler->display->display_options['use_more_always'] = FALSE;
$handler->display->display_options['access']['type'] = 'none';
$handler->display->display_options['cache']['type'] = 'none';
$handler->display->display_options['query']['type'] = 'views_query';
$handler->display->display_options['exposed_form']['type'] = 'basic';
$handler->display->display_options['pager']['type'] = 'some';
$handler->display->display_options['pager']['options']['items_per_page'] = '1';
$handler->display->display_options['style_plugin'] = 'default';
$handler->display->display_options['row_plugin'] = 'panels_fields';
/* Relationship: User: Content authored */
$handler->display->display_options['relationships']['uid']['id'] = 'uid';
$handler->display->display_options['relationships']['uid']['table'] = 'users';
$handler->display->display_options['relationships']['uid']['field'] = 'uid';
$handler->display->display_options['relationships']['uid']['required'] = TRUE;
/* Field: Field: Photo */
$handler->display->display_options['fields']['field_imagefield']['id'] = 'field_imagefield';
$handler->display->display_options['fields']['field_imagefield']['table'] = 'field_data_field_imagefield';
$handler->display->display_options['fields']['field_imagefield']['field'] = 'field_imagefield';
$handler->display->display_options['fields']['field_imagefield']['label'] = '';
$handler->display->display_options['fields']['field_imagefield']['element_label_colon'] = FALSE;
$handler->display->display_options['fields']['field_imagefield']['click_sort_column'] = 'fid';
$handler->display->display_options['fields']['field_imagefield']['settings'] = array(
  'image_style' => 'thumbnail',
  'image_link' => '',
);
/* Field: User: Blogger name */
$handler->display->display_options['fields']['field_blogger_name']['id'] = 'field_blogger_name';
$handler->display->display_options['fields']['field_blogger_name']['table'] = 'field_data_field_blogger_name';
$handler->display->display_options['fields']['field_blogger_name']['field'] = 'field_blogger_name';
$handler->display->display_options['fields']['field_blogger_name']['exclude'] = TRUE;
/* Field: User: About my Blog */
$handler->display->display_options['fields']['field_my_blog']['id'] = 'field_my_blog';
$handler->display->display_options['fields']['field_my_blog']['table'] = 'field_data_field_my_blog';
$handler->display->display_options['fields']['field_my_blog']['field'] = 'field_my_blog';
$handler->display->display_options['fields']['field_my_blog']['label'] = '';
$handler->display->display_options['fields']['field_my_blog']['element_label_colon'] = FALSE;
/* Field: User: Blog URL */
$handler->display->display_options['fields']['field_blog_url']['id'] = 'field_blog_url';
$handler->display->display_options['fields']['field_blog_url']['table'] = 'field_data_field_blog_url';
$handler->display->display_options['fields']['field_blog_url']['field'] = 'field_blog_url';
$handler->display->display_options['fields']['field_blog_url']['label'] = '';
$handler->display->display_options['fields']['field_blog_url']['alter']['alter_text'] = TRUE;
$handler->display->display_options['fields']['field_blog_url']['alter']['text'] = '[field_blogger_name]\'s blog';
$handler->display->display_options['fields']['field_blog_url']['alter']['make_link'] = TRUE;
$handler->display->display_options['fields']['field_blog_url']['alter']['path'] = '[field_blog_url]';
$handler->display->display_options['fields']['field_blog_url']['alter']['absolute'] = TRUE;
$handler->display->display_options['fields']['field_blog_url']['element_label_colon'] = FALSE;
/* Contextual filter: Content: Nid */
$handler->display->display_options['arguments']['nid']['id'] = 'nid';
$handler->display->display_options['arguments']['nid']['table'] = 'node';
$handler->display->display_options['arguments']['nid']['field'] = 'nid';
$handler->display->display_options['arguments']['nid']['relationship'] = 'uid';
$handler->display->display_options['arguments']['nid']['default_argument_type'] = 'fixed';
$handler->display->display_options['arguments']['nid']['summary']['number_of_records'] = '0';
$handler->display->display_options['arguments']['nid']['summary']['format'] = 'default_summary';
$handler->display->display_options['arguments']['nid']['summary_options']['items_per_page'] = '25';

/* Display: Block */
$handler = $view->new_display('block', 'Block', 'block');
$handler->display->display_options['defaults']['hide_admin_links'] = FALSE;

/* Display: Context */
$handler = $view->new_display('ctools_context', 'Context', 'ctools_context_1');
$handler->display->display_options['defaults']['hide_admin_links'] = FALSE;
$handler->display->display_options['style_plugin'] = 'ctools_context';
$handler->display->display_options['row_plugin'] = 'fields';
```