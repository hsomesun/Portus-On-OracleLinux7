# Configuration and examples to run Portus on Oracle Linux 7 (or CentOS 7)

This repository contains sample configuration files for Docker Distribution
(Registry) V2 and Portus to run as Docker containers.

This configuration uses an NGINX proxy so that both Portus and the Registry
appear on the same host. This means that there are no ports specified for both
the UI or the Docker commands.

## Assumptions:

1. Registry images will be stored in `/var/lib/registry/data` on the host
1. Certificates will be stored in `/var/lib/registry/certs` on the host
1. Registry configuration will be stored in `/var/lib/registry` on the host
1. Portus configuration will be stored in `/var/lib/registry/portus` on the host
1. MySQL data will be stored in `/var/lib/registry/mysql` on the host
1. NGINX configuration will be stored in `/var/lib/registry/nginx` on the host
1. You have a valid 3rd-party CA signed SSL certificate and corresponding private
key for the host

> These directories need to be created manually before starting any of the containers below.

## Step 1: Base Configuration Replacements

1. Checkout this repository to your local machine
1. Replace `your.registry.fqdn` in `registry/config.yml` with the FQDN of your host.
1. Replace `your.registry.fqdn` in `portus/config-local.yml` with the FQDN of your host.
1. Replace `registry.fqdn` in `portus/config-local.yml` with _just_ your domain name.
1. Replace `your.registry.fqdn` in `nginx/ssl.conf` with the FQDN of your host.
1. Place your SSL certificate chain _including intermediate CA certificates and
private key_ at `/var/lib/registry/certs/server.crt`
1. Place your SSL private key at `/var/lib/registry/certs/server.key`
1. Copy `registry/config.yml` to `/var/lib/registry/config.yml`
1. Copy `portus/config-local.yml` to `/var/lib/registry/portus/config-local.yml`
1. Copy `nginx/*.conf` to `/var/lib/registry/nginx/`

> **Note:** To successfully configure the Docker Registry to use the webhook to
notify Portus, the full SSL chain including private key is required.

## Step 2: Start the MySQL Container

Use the following command to start a MySQL container to store the Portus data:

```
docker run -d --restart=always --name mysql \
 -e MYSQL_DATABASE=portus \
 -e MYSQL_USER=portus \
 -e MYSQL_PASSWORD=portus \
 -e MYSQL_RANDOM_ROOT_PASSWORD=yes \
 -v /var/lib/registry/mysql:/var/lib/mysql \
 mysql:latest
```

This will initialize a MySQL database and generate a random root password. You
will need to check the output of `docker logs mysql` to find the root password
generated.

It will also create a database for Portus and a user named `portus` with the
password `portus`. For security purposes, it is strongly recommended that you
change this command-line to use a different password and then change the
`docker run` command that starts portus to use the same password.

You could also use MariaDB by replacing the `--name` and switching to the
`mariadb:latest` image.

## Step 3: Start the Registry Container

Use the following command to start the Docker Registry:

```
docker run -d --restart=always --name registry \
  -v /var/lib/registry/config.yml:/etc/docker/registry/config.yml \
  -v /var/lib/registry/data:/var/lib/registry \
  -v /var/lib/registry/certs:/certs \
  -v /var/lib/registry/auth:/auth \
registry:2
```

Remember that all the required directories must be created prior to starting the
container.

## Step 4: Start the Portus Container

Use the following command to start Portus:

```
docker run -d --restart=always --name portus \
 -v /var/lib/registry/certs/server.crt:/certificates/portus-ca.crt \
 -v /var/lib/registry/certs/server.key:/secrets/portus.key \
 -v /var/lib/registry/portus/config-local.yml:/srv/Portus/config/config-local.yml \
 --link mysql:mysql \
  -e MARIADB_SERVICE_HOST=mysql \
 -e MARIADB_USER=portus \
 -e MARIADB_PASSWORD=portus \
 -e MARIADB_DATABASE=portus \
 -e PORTUS_SECRET_KEY_BASE=$(openssl rand -hex 64) \
 -e PORTUS_PORTUS_PASSWORD=$(openssl rand -hex 64) \
opensuse/portus:head
```
*Note:* if you used a MariaDB database or provided a custom username/password,
you need to replace the default values in this command.

> You should use `docker inspect portus` to find the `PORTUS_SECRET_KEY_BASE`
and `PORTUS_PORTUS_PASSWORD` values used when the container was created. This
will allow you to recreate the container using the existing database and maintain
the correct `portus` user password (which is also stored in the database).


## Step 5: Start the NGINX container

Use the follwoing command to start NGINX:

```bash
docker run -d --name nginx --restart=always \
--link portus:portus \
--link registry:registry \
 -v /var/lib/registry/nginx:/etc/nginx/conf.d \
 -v /var/lib/registry/certs/server.crt:/certs/server.crt \
 -v /var/lib/registry/certs/server.key:/certs/server.key \
 -p 80:80 \
 -p 443:443 \
 nginx
 ```

## Step 6: Configure Portus

The first time you access the Portus web interface at `https://your.registry.fqdn` you will be prompted to create the initial admin user.

After the initial admin user is created, you will need to configure access to the
Registry. You can use any value for the name. Use `your.registry.fqdn`for the URL
(do not specify a port) and check the _Enable SSL_ box. 
