nginx-proxy sets up a container running nginx and [docker-gen](https://github.com/jwilder/docker-gen).  docker-gen generate reverse proxy configs for nginx and reloads nginx when containers they are started and stopped.

See [Automated Nginx Reverse Proxy for Docker](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/) for why you might want to use this.

This fork is from [jwilder](https://github.com/jwilder).

## About this fork

This fork is adding the following:

 - Hidding the Nginx version number (server_tokens)
 - Adding the possibility to define change the default size of uploads (client\_max\_body\_size)
 - Allow to mount a volume to `/etc/nginx/sites-enabled/` in order to check the genenrated Nginx configuration file

You can fetch the image from https://registry.hub.docker.com/u/zedtux/nginx-proxy/.

## The original features

### Usage

To run it:

    $ docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock zedtux/nginx-proxy

Then start any containers you want proxied with an env var VIRTUAL_HOST=subdomain.youdomain.com

    $ docker run -e VIRTUAL_HOST=foo.bar.com  ...

Provided your DNS is setup to forward foo.bar.com to the a host running nginx-proxy, the request will be routed to a container with the VIRTUAL_HOST env var set.

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80.  If you need to specify a different port, you can set a VIRTUAL_PORT env var to select a different one.  If your container only exposes one port and it has a VIRTUAL_HOST env var set, that port will be selected.

### Multiple Hosts

If you need to support multipe virtual hosts for a container, you can separate each enty with commas.  For example, `foo.bar.com,baz.bar.com,bar.com` and each host will be setup the same.
