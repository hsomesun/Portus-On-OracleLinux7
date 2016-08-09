# Configuration and examples to run Portus on Oracle Linux 7 (or CentOS 7)

This repository contains sample configuration files for Docker Distribution (Registry) V2 and Portus to run as Docker containers.

## Assumptions:

1. Registry images will be stored in `/var/lib/registry/data` on the host
2. Certificates will be stored in `/var/lib/registry/certs` on the host
3. Configuration files will be stored in `/var/lib/registry` on the host
4. You have a valid SSL certificate and key for the host

## Base Configuration/Replacements

1. Replace `your.registry.fqdn` with the actual fully qualified domain name of your host. 
2. Place your SSL certificate chain _including intermediate CA certificates and private key_ at `/var/lib/registry/certs/server.crt`
3. Place your SSL private key at `/var/lib/registry/certs/server.key`

*Note:* To successfully configure the Docker Registry to use the webhook to notify Portus, the full SSL chain including private key is required.
