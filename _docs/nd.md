---
layout: docs
title: Permalinks
permalink: /docs/nd/
---

### Apache

The Apache web server has very extensive support for content negotiation and can
handle extensionless URLs by setting the [multiviews][] option in your
`httpd.conf` or `.htaccess` file:

[multiviews]: https://httpd.apache.org/docs/current/content-negotiation.html#multiviews

{% highlight apache %}
Options +MultiViews
{% endhighlight %}

### Nginx

The [try_files][] directive allows you to specify a list of files to search for
to process a request. The following configuration will instruct nginx to search
for a file with an `.html` extension if an exact match for the requested URI is
not found.

[try_files]: http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files

{% highlight nginx %}
try_files $uri $uri.html $uri/ =404;
{% endhighlight %}