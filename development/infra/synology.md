---
title: Synology
badge: infra
description: NAS configuration and tricks
---

## Local Let's Encrypt + ACME certificates

It's possible to generate an SSL certificate for non-exposed services as explained [here](https://dr-b.io/post/Synology-DSM-7-with-Lets-Encrypt-and-DNS-Challenge). On this guide we will create a certificate for the <code><smart-variable variable="domain" default-value="local.yourdomain.com"></smart-variable></code> domain. It's also possible to generate certs for wildcard domains which might be of your interest.


### 1. Create a user to renew the cert

The first requirement is creating a user with enough privileges to create and update the certs. I created the following username: <code><smart-variable variable="synology-username" default-value="cert_adm"></smart-variable></code>. It requires the following:

* Belonging to the `http` group
* Read / Write access to the homes folder. 
* *(optional)* belong to the `administrators` group. Only required for SSH access. A different user can be used for that.
* *(recommended)* Deny access to all applications.


> Cloudflare will be used in this example, but the same applies to other providers such as Godaddy, Azure, Namescheap, Ionos... Check [the acme.sh](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) documentation to review the differences in configuration.


### 2. Set-Up

SSH into your Synology NAS using an administrator account, and prepare the [acme.sh](https://github.com/acmesh-official/acme.sh) tool to issue and renew the certificates.

``` shell
wget -O /tmp/acme.sh.zip https://github.com/acmesh-official/acme.sh/archive/master.zip
sudo 7z x -o/usr/local/share /tmp/acme.sh.zip
sudo mv /usr/local/share/acme.sh-master/ {{ acmesh-dir /usr/local/share/acme.sh }}
sudo chown -R {{ synology-username cert_adm }} {{ acmesh-dir /usr/local/share/acme.sh }}/

cd {{ acmesh-dir /usr/local/share/acme.sh }}
```

To issue the first certificate, the script will take the following environment variables:

``` shell
export CF_Token="{{ cf-token 4t0ken }}"
export CF_Zone_ID="{{ cf-zone-id id}}"
export SYNO_USERNAME="{{ synology-username cert_adm }}"
export SYNO_PASSWORD="{{ synology-password ch4ng3m3! }}"
export SYNO_CREATE=1
export SYNO_CERTIFICATE="{{ synology-certificate Let's Encrypt & Cloudflare }}"
```

After that, all that is needed is to issue de certificate:

``` shell
./acme.sh --server letsencrypt --issue -d "{{ domain local.yourdomain.com }}" --dns dns_gd --home $PWD
```

> Acording to the [referenced blogpost](https://dr-b.io/post/Synology-DSM-7-with-Lets-Encrypt-and-DNS-Challenge): *If you get `Error 5598` it may be a problem with the certificate that is returned. Users have reported that adding the argument `--keylength 2048` to the command will resolve it*.

Once the process is finished, the ceriticate can be deployed to DSM using the following command:

``` shell
./acme.sh -d "{{ domain local.yourdomain.com }}" --deploy --deploy-hook synology_dsm --home $PWD
```

### 3. Scheduled task

The certicficate should be valid for 90 days, so, a scheduled task will required to renew the certificate; which you can add directly through DSM using the same configuration as above:

``` shell
export CF_Token="{{ cf-token 4t0ken }}"
export CF_Zone_ID="{{ cf-zone-id id}}"
export SYNO_USERNAME="{{ synology-username cert_adm }}"
export SYNO_PASSWORD="{{ synology-password ch4ng3m3! }}"
export SYNO_CREATE=1
export SYNO_CERTIFICATE="{{ synology-certificate Let's Encrypt & Cloudflare }}"

{{ acmesh-dir /usr/local/share/acme.sh }}/acme.sh --renew -d "{{ domain local.yourdomain.com }}" --home {{ acmesh-dir /usr/local/share/acme.sh }} --server letsencrypt
```