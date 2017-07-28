## Install migration modules
The following modules will help perform migrations using Drush.
Migrate_plus
Provides extensions to the core migration framework
Migrate_tools
Provides drush commands for managing and running migrations
Migrate_upgrade
Provides drush support for upgrading from Drupal 7 or 6 to Drupal 8
This is optional and used to export your Drupal 7 config that can be imported into Drupal 8. If you are updating field names, changing/manipulating data between sites or only migrating data these config files will need heavy modifications/updates.
The most helpful feature of this module is creating your migration_group yml file.
Migrate_devel
Provides debug information about fields during migration imports
This is an incredibly helpful module for devs creating migrations
Tells devs what fields are being mapped from and where they are mapping to, as well as the field’s data structure. It’s also a great way to see programmatically if your field is mapping at all or if it is skipping the field all together.
------

## Create custom migration module
First, you will need to create a custom module that will hold your custom migration configuration that will be used to map the fields from your D6/D7 site to your D8 site. There are a couple of options here. You can either export and modify your D6/D7 config using migrate_upgrade or you can write the config files from scratch.
All config should be put in custom_migrate_module/config/install
------

##Export your Drupal 7 Config
drush migrate-upgrade --legacy-db-url=mysql://user:pass@12.34.56.78/d6db --legacy-root=http://myd7site.com --configure-only
drush config-export --destination=/path/to/custom_migrate_module/config/install
Remove all non-migration config files, as well as any migration config files you do not need to use. If you are only migrating data from D6/D7 content types into a new D8 content type, all you need are the group, nodes and file yml files. You can ignore the field, taxonomy and revisions config.
Remember to remove the uuid key and value from the yml files you are going to use.
Add your custom module to the list of enforced dependencies to remove the config on uninstall. (This will be necessary when testing your config.)
dependencies:
 enforced:
   module:
     - custom_migrate_module
------

## Modify the exported Drupal 7 config
This is the most labor intensive part of the migration process. If you are mapping the content to new fields you will need to have a deep understanding of the migration plugin system.
When simply importing the data from one field to another with the same structure, you can use field_used_in_drupal8: field_from_drupal7
A few plugins to be aware of are:
default_value
static_map
format_date
extract
Iterator
migration_lookup
When migrating files, the basic structure you will need to use is:

```yml
field_used_in_drupal8:
   plugin: iterator
   source: field_from_drupal7
   process:
     target_id:
       plugin: migration_lookup
       migration: upgrade_d7_file
       source: fid
     alt: alt
     title: title
     height: height
     width: width
```

Migrating taxonomies can be done using the static_map plugin
You can perform several actions using multiple plugins on a single field by using the following format:

```yml
field_article_summary:
 -
   plugin: extract
   source: body
   default: ''
   index:
     - 0
     - summary
 -
   plugin: callback
   callable: strip_tags
```

