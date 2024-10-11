---
created: 2024-05-17T18:43
updated: 2024-05-17T18:52
tags:
  - Docker
  - Dockerfile
related link: https://www.cloudbees.com/blog/what-is-a-dockerfile
written by: Tim Butler
---
## What Is a Dockerfile?

When you run the Docker run command and specify WordPress, Docker uses this file to build the image itself. The Dockerfile is essentially the build instructions to build the image.

The advantage of a Dockerfile over just storing the binary image (or a snapshot/template in other virtualization systems) is that the automatic builds will ensure you have the latest version available. This is a good thing from a security perspective, as you want to ensure you’re not installing any vulnerable software.

```dockerfile
FROM php:5.6-apache
RUN a2enmod rewrite

# install the PHP extensions we need
RUN apt-get update &amp;&amp; apt-get install -y libpng12-dev libjpeg-dev &amp;&amp; rm -rf /var/lib/apt/lists/* \
  &amp;&amp; docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
  &amp;&amp; docker-php-ext-install gd RUN docker-php-ext-install mysqli
VOLUME /var/www/html
ENV WORDPRESS_VERSION 4.2.2
ENV WORDPRESS_UPSTREAM_VERSION 4.2.2
ENV WORDPRESS_SHA1 d3a70d0f116e6afea5b850f793a81a97d2115039

# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
RUN curl -o wordpress.tar.gz -SL https://wordpress.org/wordpress-${WORDPRESS_UPSTREAM_VERSION}.tar.gz \
  &amp;&amp; echo "$WORDPRESS_SHA1 *wordpress.tar.gz" | sha1sum -c - \
  &amp;&amp; tar -xzf wordpress.tar.gz -C /usr/src/ \
  &amp;&amp; rm wordpress.tar.gz \
  &amp;&amp; chown -R www-data:www-data /usr/src/wordpress
COPY docker-entrypoint.sh /entrypoint.sh

# grr, ENTRYPOINT resets CMD now
ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
```

Let's pull this apart piece by piece.
### FROM

```dockerfile
FROM php:5.6-apache
```

The first part is the FROM command, which tells us what image to base this off of. This is the multi-layered approach that makes Docker so efficient and powerful. In this instance, it’s using the php:5.6-apache Docker image, which again references a Dockerfile to automate the build process.

If you look at the PHP Dockerfile, you’ll see that it does all of the work of building the PHP environment with Apache for you. You can use this to start any PHP-based project too, so you’ll see it referenced by other systems like Joomla, Magento, and similar.

### RUN

```dockerfile
RUN a2enmod rewrite

# install the PHP extensions we need
RUN apt-get update &amp;&amp; apt-get install -y libpng12-dev libjpeg-dev &amp;&amp; rm -rf /var/lib/apt/lists/* \
&amp;&amp; docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
&amp;&amp; docker-php-ext-install gd
RUN docker-php-ext-install mysqli
```

The next set of calls are the RUN commands. This is what runs within the container at build time. From the calls, you can see that ModRewrite is enabled (RUN a2enmod rewrite), and it then installs the php-gd and libraries required to run it. This isn’t standard in the default PHP 5.6 build, as it’s designed to be as small as possible. There’s also the mysqli PHP extension, which again is another dependency for WordPress.

### VOLUME

```dockerfile
VOLUME /var/www/html
```

This is the default volume we overrode in the last article. Specifying it in the Dockerfile allows it to be externally mounted via the host itself or a Docker data container. If it’s not defined here, then it’s not possible to access outside of the container.

### ENV

```dockerfile
ENV WORDPRESS_VERSION 4.2.2
ENV WORDPRESS_UPSTREAM_VERSION 4.2.2
ENV WORDPRESS_SHA1 d3a70d0f116e6afea5b850f793a81a97d2115039
# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
RUN curl -o wordpress.tar.gz -SL https://wordpress.org/wordpress-${WORDPRESS_UPSTREAM_VERSION}.tar.gz \
  &amp;&amp; echo "$WORDPRESS_SHA1 *wordpress.tar.gz" | sha1sum -c - \
  &amp;&amp; tar -xzf wordpress.tar.gz -C /usr/src/ \
  &amp;&amp; rm wordpress.tar.gz \
  &amp;&amp; chown -R www-data:www-data /usr/src/wordpress
```

This sets the environment variables, which can be used in the Dockerfile and any scripts that it calls. These are persistent with the container too, so they can be referenced at any time. In the case of the WordPress Dockerfile, we can see that it sets the version number, the upstream version number, and then the SHA1 hash of the download.

Then, with the environment variables set, there’s another RUN command to download the corresponding WordPress tarball, check it against the hash (to ensure it correctly downloaded), untar the files to /usr/src, and set the ownership of the files so that Apache can access them.
### COPY

```dockerfile
COPY docker-entrypoint.sh /entrypoint.sh
```

The COPY command is simply as it sounds. It can copy a file (in the same directory as the Dockerfile) to the container. You can do this for things like custom configuration files or like in this instance, a file to run commands after the container has been set up.

### ENTRYPOINT

```dockerfile
ENTRYPOINT ["/entrypoint.sh"]
```

ENTRYPOINT and CMD are probably the two which will confuse you the most, especially if you’re not a heavy Linux user. In fact, with a casual read of the documentation, they very much seem like the same thing! However, there are some subtle differences, and they can both be used at the same time.

If the ENTRYPOINT isn’t specified, Docker will use /bin/sh -c as the default. However, if you want to override some of the system defaults, you can specify your own entrypoint and therefore manipulate the environment. The ENTRYPOINT command also allows arguments to be set from the Docker run command as well, whereas CMD can’t.

### CMD

```dockerfile
CMD ["apache2-foreground"]
```

Hopefully, if the ENTRYPOINT part makes sense CMD will now fall into place. CMD is the default argument passed to the ENTRYPOINT. In the case of the WordPress Dockerfile, it seems a bit counter-intuitive. If you look at the bottom of the [entrypoint.sh](https://github.com/docker-library/wordpress/blob/master/apache/docker-entrypoint.sh) file, you’ll see exec "$@". This means that after the BASH script has run, the final command is to execute the arguments passed to the script. In this case, CMD is set to apache2-foreground, which therefore starts WordPress.

Don’t worry too much if ENTRYPOINT and CMD still seem very confusing. In most instances, you can simply use the existing configurations (as you’ll see below). If you’re at the point where you need to write your own CMD and ENTRYPOINT commands, CenturyLink Labs has a great blog article which covers [the two commands in depth](http://www.centurylinklabs.com/dockerfile-entrypoint-vs-cmd/). I highly recommend reading it if you’re after more information.

### Further commands

As this was a specific example, not all of the commands available were used. Have a read through the [Dockerfile reference](https://docs.docker.com/reference/builder/) in the official Docker documentation to go over all of the available commands.

## Modifying a Dockerfile

Now that you have a basic understanding, we can now modify the files to suit our own needs. Let's get a copy of the official WordPress Dockerfile and make a few modifications. If you’ve got [Git](https://www.conetix.com.au/blog/getting-started-git) installed, clone the existing repo so that we can modify it:

```bash
git clone https://github.com/docker-library/wordpress.git
```

In the “apache” directory, you’ll see the Dockerfile for WordPress. Since [WP-CLI](http://www.wp-cli.org/) is a popular plugin which makes manipulating WordPress from the command line easy, let's add it to the Dockerfile. Here’s what we need to add:

```dockerfile
RUN curl -o /bin/wp https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
RUN chmod +x /bin/wp
```

The wrapper around the wp-cli is to ensure it runs as the www-data user. Otherwise it will run as the root user. For this, we also need to install the sudo command available as well. Here’s the code for this:

```dockerfile
RUN apt-get update &amp;&amp; apt-get install -y sudo less
```

We also want to quickly delete any of the files regarding apt in order to keep the container as lean as possible, which can be done with the following:

```dockerfile
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

We now need to build the image. This can be done from the command line, and it will create a local Docker image for us. Since it’s not from a repo, we tag the build so that we can easily reference it. Here’s the build command we’ll use:

```
docker build -t wordpress-with-cli .
```

The period at the end of the command specifies the directory to build, which in this case is the directory we’re in. With the freshly built image, we can now fire up an instance of this and give our WP-CLI a go. All we need to do is reference the tag name we used (wordpress-with-cli) when creating the container. Here’s the command:

```bash
docker run --link db-server:mysql -d -e -p 8080:80 --name wpcli-demo WORDPRESS_DB_NAME=wordpressdemo -e WORDPRESS_DB_USER=conetixdemo -e WORDPRESS_DB_PASSWORD=Xumkos26 wordpress-with-cli
```

> **Note:** This assumes you have the database container configured as per [our previous article](http://blog.codeship.com/docker-basics-linking-and-volumes/). If you’re copying the command verbosely, you’ll need to ensure the database container has been created and started. We can now call this from the command line just as if it were a local install. Here’s an example to install a new theme and activate it:

```bash
docker exec cli-demo wp theme install quark
docker exec cli-demo wp theme activate quark
```

The output from the WP-CLI should be displayed, and you should see it going through the process of downloading the theme and then activating it. If you point your browser at the site ([http://:8080](http://:8080)), you should see the lovely [quark theme](http://quarktheme.com/) installed and running for you.

## Creating a New Dockerfile

While modifying an existing Dockerfile works, it also means you need to keep updating it every time the original changes. Because of the layered nature, we can do this quite easily. The changes we put in the file above can now be converted into a standalone Dockerfile and simply reference the parent WordPress layer. Here’s what the Dockerfile would look like:

```dockerfile
FROM wordpress:latest

# Add WP-CLI
RUN apt-get update &amp;&amp; apt-get install -y sudo less
RUN curl -o /bin/wp-cli.phar https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
RUN curl -o /bin/wp https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
COPY wp-su.sh /bin/wp
RUN chmod +x /bin/wp-cli.phar
RUN apt-get clean &amp;&amp; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

As you can see, we simply referenced the WordPress image with the FROM command and then added our modifications. Anytime we build a new container, it’ll always use the latest version of WordPress. You could create builds tied to the latest subversion (_e.g._, 4.2) as well, so that you can still have version stability.

I’ve also uploaded the Dockerfile for the wp-cli to [GitHub](https://github.com/conetix/docker-wordpress-wp-cli) and to the Docker Hub Repository with the package name [wordpress-with-wp-cli](https://registry.hub.docker.com/u/conetix/wordpress-with-wp-cli/). This means that anyone can now use this image without having to first create the Dockerfile, just like you would with any of the other images available. You can also upload images to the public repository for free, or Docker allows private repos if you buy a subscription through them.

Docker also provides the source code to run your own private repository. It comes as the core system only, meaning that there’s no authentication or web interface available by default. These items need to be added on, since you would be customising them for your own environment. We’ll be covering this in a future article as well, so that you have all the pieces to the Docker ecosystem.