# 18F Drupal development container

This project aims to be an easy way to get started running Drupal in a Docker
environment for development at 18F. Importantly, it handles making the container
work with zscaler.
 
The zscaler root certificate is copied into the Drupal container. Then, the
CURL_CA_BUNDLE environment variable is set so cURL will use that root. Finally,
the default `php.ini-proudction` is made the default `php.ini` and is updated to
set the `openssl.cafile` option to point to the zscaler root. All of this is
necessary because zscaler does an entity-in-the-middle style SSL inspection,
which means all SSL connections are signed with its certificates. zscaler's
certificates are not trusted by default by ***anything***, so that trust has to
be configured manually.

## Getting started

Clone this repo and run `docker compose up` from the source root directory. This
is the minimal use case and will install Drupal, create a database, configure
Drupal to use the containerized database, and install drush and the `devel`
module. Once that is complete, you can browse to [localhost:8080](http://localhost:8080)
to create your site.

> [!WARNING]  
> If you stop here, you should delete the contents of the `drupal/confg`
> directory. The files in there represent the configuration of a basic site and
> is used in the next step. However, if you don't use it, you should delete it
> so you don't end up with conflicts later.

But that's the minimal use case. This project can take you a step further. After
`docker compose up` finishes, run `make install-site` to install a
pre-configured site. User 0 (the root user) will have these credentials:

* **username**: `admin`
* **password**: `root`

The installed site will enable the `devel` module, set default themes, etc. It's
not a *lot*, but it's still work you don't have to do manually.

## Keeping going

### Modules and themes

You can put any custom modules or themes in the `drupal/modules` or
`drupal/themes` directories, accordingly. If you're using docker compose, those
directories will be mapped to the correct places in the container for Drupal to
be able to see them. You'll still need to use drush or the Drupal admin UI to
enable and configure them.

The Drupal CLI and drush are both available in the Drupal container. To get a
shell, run `make shell`. Both programs are available on the container's `$PATH`
so you can just run them. For example, `drupal generate-theme` will just...
work.

### Modifying site configuration

As you make changes to the configuration of your site, you'll want to export
those changes so your teammates can stay in sync. No worries! You may notice the
`drupal/config` directory in this project. That's where the base configuration
used by `make install-site` resides. When you make changes, you can export your
changes by running:

```
make export-config
```

This will export your configuration to the `drupal/config` directory where it
can be version controlled, peer-reviewed, and shared within a team. To import
the configuration changes made by a teammate, run:

```
make import-config
```

## Makefile tools

To help manage everything without having to work from the shell of your Drupal
container, this project provides a Makefile to help with some of the more common
things you may run into.

* **`make clear-cache`** - Resets your Drupal cache.

* **`make export-config`** - Export your current site configuration into the
  `drupal/config` directory.

* **`make import-config`** - Import site configuration from the `drupal/config`
  directory.

* **`make install-site`** - Install a site using the configuration in
  `drupal/config`. Note that this command will only work if you have not already
  created a site. User 0 (admin)'s credentials will be username `admin` and
  password `root`.

* **`make own-settings`** - Drupal will make `settings.php` read-only. If you
  need to make changes or if you're pulling changes down from git, you may first
  need to make it writeable. This command will do that for you.

* **`make rebuild`** - Deletes your Drupal container and rebuilds it from
  source. This command leaves your database intact, so your site and all of its
  configuration will survive.

* **`make reinstall-site`** - Deletes your Drupal database but not the Drupal
  container, then creates a new database and installs from the default
  configuration.

* **`make shell`** - Gives you a shell in the Drupal container, with access to
  drush and the Drupal CLI.

* **`make zap`** - Deletes all of the containers for this project and rebuilds
  everything from scratch. Your site and configuration will be lost.

* **`make destroy`** - Deletes all of the containers for this project. Does not
  delete any images that were created.