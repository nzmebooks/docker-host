# What is this?

This is a docker-compose project to run up a LEMP stack comprising of:

* [Traefik](https://docs.traefik.io/)
* [Portainer](https://portainer.readthedocs.io/en/stable/)
* Nginx
* MySQL
* PHP
* Redis

The intention is to setup a stack that can be used for local development, but that can also be deployed to a production machine.

Note that, unlike [craftcms-docker](https://github.com/nzmebooks/craftcms-docker/blob/craft2), we install Craft 2 locally, and then share this local folder into the web container.

We use basic auth on the traefik and portainer services, to allow a semblance of security in a production setting.


## Setup

Copy `.env.example` to `.env`, and amend accordingly.

The value of `DOMAIN` specifies the domain that the services will be exposed as. Given the default of `local.host`, we'll endup with:

* https://traefik.local.host
* https://portainer.local.host
* https://web.local.host

If you're developing locally, you'll want to ensure DNS is set in `/etc/hosts` for the above routes pointing to 127.0.0.1:

```
# Adds:
# 127.0.0.1	traefik.local.host
# 127.0.0.1	portainer.local.host
# 127.0.0.1	web.local.host

DOMAIN=local.host
sudo -- sh -c -e "echo '127.0.0.1\ttraefik.$DOMAIN' >> /etc/hosts";
sudo -- sh -c -e "echo '127.0.0.1\tportainer.$DOMAIN' >> /etc/hosts";
sudo -- sh -c -e "echo '127.0.0.1\tweb.$DOMAIN' >> /etc/hosts";
```

Alternatively, you could use [Dnsmasq](https://github.com/elalemanyo/docker-localhost#hosts-file---wildcard-dns-domain-on-mac-os-x).


If you want to use a domain other than the default `local.host` domain, you'll want to create a cert -- we suggest using the [wildcard](https://github.com/jcdarwin/wildcard) script, but that's up to you.

Once your cert has been created, you'll want to copy them to `/etc/traefik/`, and amend the following lines in `/etc/traefik/traefik.toml` accordingly:

    certFile = "/etc/traefik/local.host.crt"
    keyFile = "/etc/traefik/local.host.key"

Presuming you're on a Mac, you'll also want to register the cert as trusted so the browser doesn't complain -- this can be done at the command line using a command such as the following:

    sudo security add-trusted-cert -d  -k /Library/Keychains/System.keychain ./etc/traefik/local.host.crt

Note: originally we did try to use `localhost` as our domain, however Chrome doesn't seem to like a domain with only a single step.


## Usage

Download Craft 2 and extract locally:

    ./download.sh

Use `docker-compose` to start, stop and destroy the stack:

    # starting
    docker-compose up -d --build

    # stopping
    docker-compose stop

    # destroy
    docker-compose down

    # view service bindings
    docker-compose ps

    # run a shell in the container
    # note that some images (redis, traefik) use alpine, therefor bin/ash
    docker exec -it web.local.host /bin/bash
    docker exec -it redis.local.host /bin/ash

    # view logs (also available via portainer)
    docker logs web.local.host
    docker exec -it web.local.host ls -la /var/log/

Once the stack is up, you should be able to visit the following in your browser:

* https://traefik.local.host
* https://portainer.local.host
* https://web.local.host/info.php

Navigate to https://<HOSTNAME>/admin to begin installing Craft 2.

    https://web.local.host/admin


## MySQL

To connect to MySQL via, you'll need to do something like the following:

    # for root access, entering <DB_ROOT_PASSWORD> as the password when challenged
    mysql -u root -p -h 0.0.0.0

    # for non-root access, entering <DB_PASSWORD> as the password when challenged
    mysql -u <DB_USERNAME> -p -h 0.0.0.0


# Further reading

* https://github.com/wyveo/craftcms-docker/blob/craft2/docker-compose.yml
* http://tech.osteel.me/posts/2017/01/15/how-to-use-docker-for-local-web-development-an-update.html
* https://deliciousbrains.com/https-locally-without-browser-privacy-errors/
* [Craft CMS docker-compose dev setup](https://gist.github.com/jackmcpickle/59efc98a99c067b08020)
* https://github.com/pnglabz/docker-compose-lamp
* https://github.com/elalemanyo/docker-localhost
