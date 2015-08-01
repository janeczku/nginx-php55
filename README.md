# Nginx + PHP-FPM 5.5 + NodeJS + Ruby docker container

This is a fork of [million12:nginx-php](https://github.com/million12/docker-nginx-php).

**Things included:**

#### Nginx

This image is based on [million12/nginx](https://github.com/million12/docker-nginx) - go there for more details.  
**Default vhost** is configured and served from `/data/www/default`. Add .php file to that location to have it executed with PHP.

#### - PHP-FPM

**PHP 5.5** is up & running for default vhost. As soon as .php file is requested, the request will be redirected to PHP upstream. See [/etc/nginx/conf.d/php-location.conf](container-files/etc/nginx/conf.d/php-location.conf).

File [/etc/nginx/fastcgi_params](container-files/etc/nginx/fastcgi_params) has improved configuration to avoid repeating same config options for each vhost. This config works well with most PHP applications (e.g. Symfony2, TYPO3, Wordpress, Drupal).

Custom PHP.ini directives are inside [/etc/php.d/zz-php.ini](container-files/etc/php.d/zz-php.ini).

#### Directory structure
```
/data/www # meant to contain web content
/data/www/default # default vhost directory
/data/logs/ # Nginx, PHP logs
```

#### Error logging

PHP errors are forwarded to stderr (by leaving empty value for INI error_log setting) and captured by supervisor. You can see them easily via `docker logs [container]`. In addition, they are captured by parent Nginx worker and logged to `/data/logs/nginx-error.log'. PHP-FPM logs are available in `/data/logs/php-fpm*.log` files. 

##### - pre-defined FastCGI cache for PHP backend

It's not used until specified in location {} context. In your vhost config you can add something like this:  
```
location ~ \.php$ {
    # Your standard directives...
    include               fastcgi_params;
    fastcgi_pass          php-upstream;
    
    # Use the configured cache (adjust fastcgi_cache_valid to your needs):
    fastcgi_cache         APPCACHE;
    fastcgi_cache_valid   60m;
}
```  
As you can see in [fastcgi-cache.conf](container-files/etc/nginx/addon.d/fastcgi-cache.conf), the cache is stored in `/run/user/nginx-cache` directory. To achieve best performance, bind it to host directory on tmpfs volume. Often, the `/run` directory is on tmpfs volume, so your docker run command could look like:  
```
docker run ... -v /run/user/my-container:/run/user ...
```

#### Common dev tools for web app development

* Ruby 2.0, Bundler
* NodeJS and NPM
* NPM packages like gulp, grunt, bower, browser-sync

## Usage

```
docker run -d -v /data --name=web-data busybox
docker run -d --volumes-from=web-data -p=80:80 --name=web million12/nginx-php
```

After that you can see the default vhost content (something like: '*default vhost created on [timestamp]*') when you open http://CONTAINER_IP:PORT/ in the browser.

You can replace `/data/www/default/index.html` with `index.php` and, for instance, phpinfo() to inspect installed PHP setup. You can do that using separate container which mounts /data volume (`docker run -ti --volumes-from=web-data --rm busybox`) and adding the file to the above location.


## Customise

There are several ways to customise this container, both in a runtime or when building new image on top of it:

* See [million12:nginx](https://github.com/million12/docker-nginx) for info regarding Nginx customisation, adding new vhosts etc.
* Override `/etc/nginx/fastcgi_params` if needed.
* Add custom PHP `*.ini` files to `/etc/php.d/`.
* Add own PHP-FPM .conf files to `/data/conf/php-fpm-www-*.conf` to modify PHP-FPM www pool.

## ENV variables

**STATUS_PAGE_ALLOWED_IP**  
Default: `STATUS_PAGE_ALLOWED_IP=127.0.0.1`  
Example: `STATUS_PAGE_ALLOWED_IP=10.1.1.0/16`  
Configure ip address that would be allowed to see PHP-FPM status page on `/fpm_status` URL.
