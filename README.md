# Configuration and examples to run Portus on Oracle Linux 7 (or CentOS 7)

This repository contains sample configuration files for Docker Distribution (Registry) V2 and Portus to run as Docker containers.

## Assumptions:

1. Registry images will be stored in `/var/lib/registry/data` on the host
2. Certificates will be stored in `/var/lib/registry/certs` on the host
3. Configuration files will be stored in `/var/lib/registry` on the host
4. MariaDB data will be stored in `/var/lib/registry/mariadb` on the host
4. You have a valid SSL certificate and key for the host

These directories need to be created manually before starting any of the containers below.

## Step 1: Base Configuration Replacements

1. Checkout this repository to your local machine
2. Replace `your.registry.fqdn` in `registry/config.yml` with the actual fully qualified domain name of your host. 
3. Replace `your.registry.fqdn` in `portus/config-local.yml` with the actual fully qualified domain name of your host. 
4. Replace `registry.fqdn` in `portus/config-local.yml` with your domain name.
2. Place your SSL certificate chain _including intermediate CA certificates and private key_ at `/var/lib/registry/certs/server.crt`
3. Place your SSL private key at `/var/lib/registry/certs/server.key`

*Note:* To successfully configure the Docker Registry to use the webhook to notify Portus, the full SSL chain including private key is required.

## Step 2: Create MariaDB Container

Use the following command to start a MariaDB container to store the Portus data:

```
# docker run -d --restart=always --name mariadb \
 -e MYSQL_ROOT_PASSWORD=portus \
 -v /var/lib/registry/mariadb:/var/lib/mysql \
 mariadb:latest
```

Passing the MariaDB root password on the command line is insecure. You can review the official MariaDB container documentation for alternative methods to create the container with a more secure password option. Note that if you use an alternative password, you will need to change the `MARIADB_PASSWORD` environment variable when starting the Portus container.
