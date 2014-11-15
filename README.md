nginx-proxy sets up a container running nginx and [docker-gen](https://github.com/jwilder/docker-gen).  docker-gen generate reverse proxy configs for nginx and reloads nginx when containers they are started and stopped.

See [Automated Nginx Reverse Proxy for Docker](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/) for why you might want to use this.

This fork is from [jwilder](https://github.com/jwilder).

## About this fork

This fork is adding the following:

 - Hidding the Nginx version number (server_tokens)
 - Adding the possibility to define change the default size of uploads (client\_max\_body\_size)
 - Allow to mount a volume to `/etc/nginx/sites-enabled/` in order to check the genenrated Nginx configuration file
 - Adding TLS (SSL) support

You can fetch the image from https://registry.hub.docker.com/u/zedtux/nginx-proxy/.

## The original features

### Usage

To run it:

    $ docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock zedtux/nginx-proxy

Then start any containers you want proxied with an env var `VIRTUAL_HOST=subdomain.youdomain.com`

    $ docker run -e VIRTUAL_HOST=foo.bar.com  ...

Provided your DNS is setup to forward foo.bar.com to the a host running nginx-proxy, the request will be routed to a container with the VIRTUAL_HOST env var set.

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80.  If you need to specify a different port, you can set a VIRTUAL_PORT env var to select a different one.  If your container only exposes one port and it has a VIRTUAL_HOST env var set, that port will be selected.

### Multiple Hosts

If you need to support multipe virtual hosts for a container, you can separate each enty with commas.  For example, `foo.bar.com,baz.bar.com,bar.com` and each host will be setup the same.

### Separate Containers

nginx-proxy can also be run as two separate containers using the [jwilder/docker-gen](https://index.docker.io/u/jwilder/docker-gen/)
image and the official [nginx](https://registry.hub.docker.com/_/nginx/) image.

You may want to do this to prevent having the docker socket bound to a publicly exposed container service.

To run nginx proxy as a separate container you'll need to have [nginx.tmpl](https://github.com/jwilder/nginx-proxy/blob/master/nginx.tmpl) on your host system.

First start nginx with a volume:


    $ docker run -d -p 80:80 --name nginx -v /tmp/nginx:/etc/nginx/conf.d -t nginx

Then start the docker-gen container with the shared volume and template:

```
$ docker run --volumes-from nginx \
    -v /var/run/docker.sock:/tmp/docker.sock \
    -v $(pwd):/etc/docker-gen/templates \
    -t docker-gen -notify-sighup nginx -watch --only-published /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
```

Finally, start your containers with `VIRTUAL_HOST` environment variables.

    $ docker run -e VIRTUAL_HOST=foo.bar.com  ...


### TLS (SSL) Support

In the case you have a TLS (or the SSL) certificate you can easily use it with nginx.

Create a directory on your server where you will upload the `.crt` and `.key` files to be used:

    $ mkdir /etc/docker/my-app/ssl/
    $ cp ~/my-app.* /etc/docker/my-app/ssl/
    $ ls -al /etc/docker/my-app/ssl/
    total 4
    -rw-r--r-- 4 root root 4096 Nov 15 18:01 my-app.crt
    -rw-r--r-- 4 root root 4096 Nov 15 18:01 my-app.key

Run the nginx image with a volume to this folder and with the port 443:

    $ docker run -d -p 80:80 -p 443:443 -v /etc/docker/my-app/ssl/:/etc/nginx/ssl/ -v /var/run/docker.sock:/tmp/docker.sock zedtux/nginx-proxy

Now when you will start you application, just set the variable `SSL_FILENAME`:

    $ docker run -e VIRTUAL_HOST=my-app.domain.tld -e SSL_FILENAME=my-app username/imagename

You should then be able to access https://my-app.domain.tld/.
