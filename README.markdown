# Heroku buildpack

### This is a Heroku buildpack for running [Wordpress](http://wordpress.org) on [Heroku](http://heroku.com)

It uses this [Wordpress](http://github.com/eccentricpixel/heroku-wordpress-project-template) project template to bootstrap a highly tuned Wordpress site built on the following stack:

* `nginx-1.3.11` - Nginx for serving web content.  Built specifically for Heroku.  [See compile options](https://github.com/mchung/heroku-buildpack-wordpress/blob/master/support/package_nginx).
* `php-5.4.11` - PHP-FPM for process management and APC for caching opcodes.  [See compile options](https://github.com/mchung/heroku-buildpack-wordpress/blob/master/support/package_php).
* `wordpress-3.5.1` - Downloaded directly [from wordpress.org](http://wordpress.org/download/release-archive/).
* `MySQL` - ClearDB for the MySQL backend.
* `Sendgrid` - Sendgrid for the email backend.
* `MemCachier` - MemCachier for the memcached backend.

This project template includes base parent and child themes built off of Foundation 4.

## Getting started in 60 seconds

Fork and rename or download the [Wordpress project template](http://github.com/eccentricpixel/heroku-wordpress-project-template).

Let's clone the repository for a new blog, SomeArrestedDevelopmentTributeSite.com
```bash
$ git clone git://github.com/yourGitHubName/heroku-wordpress-project-template.git SomeArrestedDevelopmentTributeSite.com
```

Create Wordpress on Heroku.
```bash
$ cd SomeArrestedDevelopmentTributeSite.com
$ heroku create -s cedar
$ heroku config:add BUILDPACK_URL=https://github.com/eccentricpixel/heroku-wordpress-buildpack.git
```

> Don't have the Heroku Toolbelt installed? Follow these [quickstart instructions](https://devcenter.heroku.com/articles/quickstart). Takes about 2 minutes.

Deploy your Wordpress site to Heroku.
```bash
$ git push heroku master
...
-----> Heroku receiving push
-----> Fetching custom git buildpack... done
-----> Wordpress app detected
.
[...]
.
-----> Discovering process types
       Procfile declares types     -> (none)
       Default types for Wordpress -> web
-----> Compiled slug size: 33.7MB
-----> Launching... done, v7
```

Open your new Wordpress site in a web browser.
```bash
$ heroku apps:open
```


If you want to add a staging site, run:

$ heroku create --remote staging
$ heroku config:add BUILDPACK_URL=https://github.com/eccentricpixel/heroku-wordpress-buildpack.git
$ git push staging master


If you want to access your Heroku Database in Sequel Pro, you can get your clouddb url by typing:

    heroku config --app
    
in Terminal.



> Ready to go.  Start coding!




## Overview

The buildpack bootstraps a Wordpress site using the [mchung/wordpress-on-heroku](http://github.com/mchung/wordpress-on-heroku) project template.  That repo contains everything required to run your own Wordpress site on Heroku.

There are several files available in `config` for configuring your new Wordpress site.

```
└── config                # Your config files goes here.
    ├── public            # The public directory
    │   └── wp-content    # Wordpress themes and plugins
    │       ├── plugins
    │       └── themes
    └── vendor            # Config files for vendored apps
        ├── nginx
        │   └── conf      # nginx.conf + your site.conf
        └── php
            └── etc       # php.ini & php-fpm.conf
```

When you deploy Wordpress to Heroku, everything in `config` is copied over to Heroku.  You can configure your blog by making changes to these files.

A few Wordpress environment variables can be controlled via Heroku using `heroku config:set`:

* `FORCE_SSL_LOGIN`
* `FORCE_SSL_ADMIN`
* `WP_CACHE`
* `DISABLE_WP_CRON`

> To add a Heroku environment variable: `heroku config:set GOOG_UA_ID=UA=1234777-9`

See `wp-config.php` and documentation from Wordpress for details.

Enabling and configuring the following Wordpress plugins will also speed up Wordpress on Heroku significantly.

* `heroku-sendgrid` - Configures phpmailer to send SMTP email with Sendgrid.
* `heroku-google-analytics` - Configures Google Analytics to display on your Wordpress site.
  * GOOG_UA_ID=UA-9999999
* `wpro` - Configures Wordpress to upload everything to S3.
* `batcache` - Configures Wordpress to use memcached for caching.
* `memcachier` - Use a modern memcached plugin.
* `cloudflare` - OPTIONAL, but very awesome.
 * If Cloudflare is installed, the plugin configures Wordpress to play nicely with CloudFlare.  It sets the correct IP addresses from visitors and comments, and also protects Wordpress from spammers.
 * Keep in mind that the free version doesn't support SSL, so you'll need to set both `FORCE_SSL_ADMIN` and `FORCE_SSL_LOGIN` to false in order to login.

## Usage


### Adding a custom domain name
```bash
$ heroku domains:add yourWebsite.come
$ heroku domains:add www.yourWebsite.com
```

Note: Adding a domain still requires some DNS setup work. Basically you'll want to do something like this:
```bash
ALIAS yourWebsite.come -> proxy.herokuapp.com
CNAME www.yourWebsite.com -> proxy.herokuapp.com
```


### Configuring cron
By default, wp-cron is fired on every page load and scheduled to run jobs like future posts or backups.  This buildpack disables wp-cron so that visitors don't have to wait to see the site.

Heroku allows you to trigger wp-cron from their scheduler.
```bash
$ heroku addons:add scheduler:standard

# Visit the Heroku scheduler dashboard and add a new task:
./cron.sh
```

Alternatively, you may also re-enable wp-cron.
```bash
$ heroku config:set DISABLE_WP_CRON=false
```

### Enable system access
The alternative PHP cache and a generic PHPINFO page is available at /apc.php and /phpinfo.php.
```bash
$ heroku config:set ENABLE_SYSTEM_DEBUG=true
$ heroku config:set SYSTEM_USERNAME=admin
$ heroku config:set SYSTEM_PASSWORD=secret123
# Visit /apc.php or /phpinfo.php
```

### Remove the PHP-FPM status page `/status.html`
Delete the directive from `nginx.conf.erb`.

### Add a `favicon.ico` drop one into `public`


System setup
* Single Heroku dyno
* Default Wordpress installation
* Default twentytwelve theme
* Caching turned up
* Cron disabled
* Memcachier + ClearDB





## Constraints with Heroku

The [ephemeral filesystem](http://devcenter.heroku.com/articles/dyno-isolation)

* End-users cannot upload media assets to Heroku. WORKAROUND: Enable `wpro` and use that plugin to upload media assets to S3 instead.
* End-users cannot update themes or plugins from the admin page. WORKAROUND: Add them to `config/public/wp-content/themes` or `config/public/wp-content/plugins` then push to Heroku.

## Security disclosure

Each time Wordpress is deployed, Heroku will fetch the latest buildpack from GitHub and execute the instructions in `compile` and `deploy`.  This buildpack will download the latest precompiled versions of Nginx, PHP, and Wordpress from my personal [S3 bucket](http://heroku-buildpack-wordpress.s3.amazonaws.com) then add in config files from the [`config`](https://github.com/mchung/wordpress-on-heroku/tree/master/config) directory.

## Hacking and Contributing

Not comfortable downloading and running a copy of someone else's PHP or Nginx executables? Not a problem!  The `support` directory also contains a handful of compilation and deployment scripts to automate several processes, which are currently used for maintenance and repo management.

* `package_nginx` - Used to compile and upload the latest version of Nginx to S3.
* `package_php` - Used to compile and upload the latest version of PHP to S3.
* `wordup` - Really useful helper script for creating and destroying Wordpress sites.


## Authors and Contributors

* Marc Chung - Follow [@mchung](https://github.com/mchung) on GitHub and also [@heisenthought](https://twitter.com/heisenthought) on Twitter
* Elijah Shepard

## License

The MIT License - Copyright (c) 2013 Marc Chung
