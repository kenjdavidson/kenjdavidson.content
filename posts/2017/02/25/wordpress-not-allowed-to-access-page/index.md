---
type: Blog
title: Wordpress plugin - Not allowed to access page
summary: How I resolved the "not allowed to access page" error while developing a Wordpress plugin.
categories: [Blog]
tags: [PHP, Wordpress]
---

So I’ve been trying to up my WordPress game by moving from Themes to Plugins.  it’s been going along pretty well until I started to work on a Settings page for my Fading Header Images plugin.  Just a small little plugin that takes all the uploaded header images and rotates through them at a set interval.  The plugin itself was all working and good, so I started working on the Admin panel using the Settings API, and this is where I had the issue.

After a bunch of looking around the web, going through all the “deactivate and activate”, “try manually updating user permissions”, and all the other things; it all came down to me being ridiculous.  I had originally added the action callback using the following:

```php
function initialize_admin() {
  add_action( 'admin_init', 'kd_fhi_admin_init' );
}
```

and in the method kd_fhi_admin_init I performed both the add_submenu_page and the register_setting and add_settings_* methods. This is where the problem was, These have to be split:

```php
function initialize_admin() {
  add_action( 'admin_init', 'kd_fhi_admin_init' );
  add_action( 'admin_menu', 'kd_fhi_init_admin_menu' );
}
```

where the admin_menu was separated from the admin_init. With that said, having them in the same callback worked correctly, admin_init action added my submenu to the settings, but didn’t allow me to view the actual page.