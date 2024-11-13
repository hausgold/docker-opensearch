![mDNS enabled opensearchproject/opensearch](https://raw.githubusercontent.com/hausgold/docker-opensearch/master/docs/assets/project.png)

[![Continuous Integration](https://github.com/hausgold/docker-opensearch/actions/workflows/package.yml/badge.svg?branch=master)](https://github.com/hausgold/docker-opensearch/actions/workflows/package.yml)
[![Source Code](https://img.shields.io/badge/source-on%20github-blue.svg)](https://github.com/hausgold/docker-opensearch)
[![Docker Image](https://img.shields.io/badge/image-on%20docker%20hub-blue.svg)](https://hub.docker.com/r/hausgold/opensearch/)

This Docker images provides the [opensearchproject/opensearch](https://hub.docker.com/r/opensearchproject/opensearch) image as base
with the mDNS/ZeroConf stack on top. So you can enjoy [OpenSearch](https://opensearch.org/) while
it is accessible by default as *opensearch.local*. (Port 9200, 9600, 80 via proxy)

- [Requirements](#requirements)
- [Getting starting](#getting-starting)
- [docker-compose usage example](#docker-compose-usage-example)
- [Host configs](#host-configs)
- [Configure a different mDNS hostname](#configure-a-different-mdns-hostname)
- [Other top level domains](#other-top-level-domains)
- [Further reading](#further-reading)

## Requirements

* Host enabled Avahi daemon
* Host enabled mDNS NSS lookup

## Getting starting

You just need to run it like that, to get a working opensearch:

```bash
$ docker run --rm hausgold/opensearch
```

The port 9200 is proxied by haproxy to port 80 to make *opensearch.local*
directly accessible. The port 9200 and 9600 are untouched.

Checkout the [working with plugins
documentation](https://opensearch.org/docs/latest/opensearch/install/docker/#working-with-plugins)
if you have to deal with custom plugins.

In order to enable the [Elasticsearch version
compatibility](https://opensearch.org/blog/technical-posts/2021/10/moving-from-opensource-elasticsearch-to-opensearch/)
add this environment variable:

```
compatibility.override_main_response_version='true'
```

## docker-compose usage example

```yaml
services:
  opensearch:
    image: hausgold/opensearch
    environment:
      # Mind the .local suffix
      MDNS_HOSTNAME: opensearch.test.local
      OPENSEARCH_JAVA_OPTS: -Xms128m -Xmx128m
      discovery.type: single-node
      compatibility.override_main_response_version: false
    ulimits:
      # Due to systemd/pam RLIMIT_NOFILE settings (max int inside the
      # container), the Java process seams to allocate huge limits which result
      # in a +unable to allocate file descriptor table - out of memory+ error.
      # Lowering this value fixes the issue for now.
      #
      # See: http://bit.ly/2U62A80
      # See: http://bit.ly/2T2Izit
      nofile:
        soft: 100000
        hard: 100000
```

## Host configs

Install the nss-mdns package, enable and start the avahi-daemon.service. Then,
edit the file /etc/nsswitch.conf and change the hosts line like this:

```bash
hosts: ... mdns4_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] dns ...
```

## Configure a different mDNS hostname

The magic environment variable is *MDNS_HOSTNAME*. Just pass it like that to
your docker run command:

```bash
$ docker run --rm -e MDNS_HOSTNAME=something.else.local hausgold/opensearch
```

This will result in *something.else.local*.

You can also configure multiple aliases (CNAME's) for your container by
passing the *MDNS_CNAMES* environment variable. It will register all the comma
separated domains as aliases for the container, next to the regular mDNS
hostname.

```bash
$ docker run --rm \
  -e MDNS_HOSTNAME=something.else.local \
  -e MDNS_CNAMES=nothing.else.local,special.local \
  hausgold/opensearch
```

This will result in *something.else.local*, *nothing.else.local* and
*special.local*.

## Other top level domains

By default *.local* is the default mDNS top level domain. This images does not
force you to use it. But if you do not use the default *.local* top level
domain, you need to [configure your host avahi][custom_mdns] to accept it.

## Further reading

* Docker/mDNS demo: https://github.com/Jack12816/docker-mdns
* Archlinux howto: https://wiki.archlinux.org/index.php/avahi
* Ubuntu/Debian howto: https://wiki.ubuntuusers.de/Avahi/

[custom_mdns]: https://wiki.archlinux.org/index.php/avahi#Configuring_mDNS_for_custom_TLD
