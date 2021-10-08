# Wordpress + MAMP

This readme will explain how to run a wordpress website on your localhost using MAMP.

# MAMP

## Installation

- Download and install [MAMP](https://www.mamp.info/en/downloads/) (MAMP PRO not required) 


## Database Set-up

- Ensure no other SQL servers are already running on the default port 3306 (this can cause issues)

- Activate the servers within MAMP, then click 'Open web start page' or visit localhost/MAMP
- Note the User and Password for the MYSQL database (both User and Password are 'root')

- Follow the link to the phpMyAdmin page or access [this URL](http://localhost/MAMP/index.php?page=phpmyadmin&language=English)

- From the databases tab, create a new database (use the default utf8_general_ci) and an appropriate name to be used by the site (this will need to be set in the wp-config file so note it for later)

- If importing a site from a backup file, this database will be populated with the previous site contents (its name will not change however)

# Wordpress Installation

## Installation

- Download [Wordpress](https://wordpress.org/download/) (ZIP file). Unzip and place the wordpress folder inside a parent folder in an appropriate location (this parent will be set in MAMP in the next step)

- By default MAMP looks for files to serve in the htdocs folder of its install location. This can be changed by modifying the 'Document Root' in the 'Web Server' preferences tab. 

- Modify the document root to a folder containing the 'wordpress' folder  (or place the wordpress directory in the default htdocs folder)

- Access the following on the localhost to initiate the wordpress installation (linking with database) [localhost/wordpress/wp-admin/install.php](https://localhost/wordpress/wp-admin/install.php)

- Follow the installation steps (using the name of the database previously created)

- Access the wp-admin site to edit the site [localhost/wordpress/wp-admin/admin.php](https://localhost/wordpress/wp-admin/install.php)

## Restoring site from backup

- Install the all-in-one-wp-migration plugin (can be installed from within wordpress on the localhost with [this link](http://localhost/wordpress/wp-admin/plugin-install.php?s=all%20in%20one%20migration&tab=search&type=term))

- Once installed, the plugin will appear in the admin sidebar. Navigate to [the import page](http://localhost/wordpress/wp-admin/admin.php?page=ai1wm_import).

- The file upload size will need to be increased before backups can be uploaded. This can be achieved by adding the following lines to the `.htaccess` file (located in the wordpress folder). After the ```# END WordPress``` line add the following lines (adjust the size accordingly)
```php
php_value upload_max_filesize 256M
php_value post_max_size 256M
php_value memory_limit 256M
php_value max_execution_time 300
php_value max_input_time 300
```

- The backup file can now be uploaded (this will replace any previously added content within this wordpress folder)

## Multiple sites

To deal with multiple wordpress sites: 
- Create a seperate database for each site
- Rename each wordpress folder (within the MAMP document root) to site_1, site_2 etc as appropriate
- Each wordpress installation should point to the appropriate database and can be accessed on the document root as before by replacing 'localhost/wordpress/' with  'localhost/site_name/'
- Ensure the correct database name is set in the `site_name/

# Uni-Soft Site Editing

This section will explain how to edit the current (2021) Uni-Soft website on wordpress, covering the content fields structure and some basic info about how to edit php templates + CSS.

## Pages and Posts

New/existing pages use the following template structure, each new page can be of the following types

### Page: 

- No template structure - write new page in HTML or by defining custom fields (see below)


### Posts:

- For this site there are two main post types; Industry and Service.
- The distinction between a 'Page' and a 'Post' is that a page can have any defined template, whereas all posts follow the same template as defined by the post type (i.e all service posts will have the same template format). 
- Each post will contain a preset structure of 'fields', that can be populated with content such as text, images etc. These are then passed to the php template for rendering. 

## Custom fields

Each post template is built from a set of 'custom fields' (found in the 'Custom Fields' tab on the sidebar). 

- There will be a field group for each page/post type that defines the allowed content on the page 

- Note: New fields will not automatically appear and must be added to the template!


### Industries Post example

- Locate the custom fields for the industries post (under Post Type - Industry). There will be 8 fields divided into four groups (each group has an editor tab separator and subfield group)

- Each group represents a section on the page, for example the page banner or the intro section. This group contains the sub fields that will appear on this part of the page. 

- Each sub field defines an element such as a text box or image, that can be added from the editor. 

- The sub field can specify instructions for what kind of content should be added eg, 'short description 1-2 lines max', 'link to embedded video' etc. (these hints will appear in the editor)

- The `Field Name` will be used to refer to this field within the php template (cannot contain spaces, underscores ok)


# CSS + Template editing

 Templates (php) and CSS files are located in the `site_name/wp-content/themes` folder. 
 - The currently used theme is located in the `dawn` folder, which contains the main `style.css` file and all template files.
 - The `template-pages` folder contains templates for the main site pages (homepage, pages that link to individual post pages) 
- In the root of the `dawn` folder there are the post templates `single-industry.php` and `single-service.php` - these are shared across all posts of that type.

## PHP template examples

- Each template contains HTML (where CSS classes are defined in `/dawn/style.css`) and PHP code for server logic. 

- PHP code within the HTML template must be enclosed with the tags 
```html
<?php CODE; ?>
```

- Content added inside the page fields will be accessed and rendered by the template. For example, the following snippet finds the content within the `intro_section` field group for an industries posts 

```html
<?php $group = get_field( 'intro_section' ); ?>
<?php if ($group['title'] || $group['description'] ) : ?>

<section class="industry-intro-section">
        ...
```
- The variable ```$group``` is a dict containing the subfield groups

- The if statement ensures the template only renders content if the group contains a populated title or description field. 

- The HTML content is then contained within the `<section>` tags, (can also contain PHP logic)

- For example, the following snippet renders an embedded video player specified by the `video_url` subfield (accessed using the subgroup dict as `$group['video_url']`).
```html
<?php if ($group['video_url']) : ?>
    <div class="post-content-card__vid">
        <?php
            $url = $group['video_url'];
            echo wp_oembed_get($url, array( 'width' => 610 ) );
        ?>
    </div>
<?php endif; ?>
```

- The function `wp_oembed_get` is used to embed a video given a url, which is stored in the `$url` variable, obtained from the subfield group dict (the URL was entered in the editor)

- The class `"post-content-card__vid"` defines how the embedded player should appear (classes defined in style.css)











