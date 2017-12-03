### Apache CORS configuration for Angular development

We are currently doing development in Angular and we have lot of services consumed by our Angular application. Our development environment for Angular consists of
* Visual Studio Code editor(vs code)
* node js
* npm

Since vs code is built in Javascript, we were seeing very high CPU consumption around 90% or more. This is because vs code was consuming huge amount of CPU performance during typescript compilation of npm start. Other major contributor to CPU utilization was that we had around 4-6 common rest services which developer would run on machine any time in order for our Angular application to work. This common services for authentication, logging, concurrency, etc were run in either jetty or tomcat maven plugin. This again increased lot of CPU for the developer, essentially slowing their machines so much so that, it started impacting productivity. For simple single line code fix, sometime a developer would take 15-30 mins to test on local machine.

In order our development environment we deploy all our common java rest apis on a JBoss service. And our Angular application is hosted on Apache Httpd web server. We have setup reverse proxies for all our common apis on Apache Httpd service. 

In order for developer to not suffer with heavy CPU utilization we decided to point out Angular application running on local machines to Apache Httpd reverse proxies. This needed workaround on Apache Httpd httpd.conf file to get around [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). Below is the configuration for Apache Httpd's httpd.conf

```
    Header always set Access-Control-Allow-Origin "http://localhost:7000"
    Header always set Access-Control-Allow-Credentials "true"
    Header always set Access-Control-Max-Age "9000"
    Header always set Access-Control-Allow-Headers "X-Requested-With, Content-Type, Origin, Authorization, Accept, Client-Security-Token, Accept-Encoding, some-other-headers-used-in-our-applications"
    Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"
```
Above headers specify below. 
* Apache should allow requests coming in from localhost:7000. We informed all our developers to run Angular on localhost:7000 port.
* Allow authentication cookies. We used jwt based authentication sent in a cookie.
* Max age as 9000, and this is used only in development environment.
* List of headers which are acceptable in our application. This also denies some random headers.
* And lists the Http verbs which are allowed.

*always set* is required, without it some how Apache Httpd was not sending the these headers.

```
    RewriteEngine On
    RewriteCond %{REQUEST_METHOD} OPTIONS
    RewriteRule ^(.*)$ $1 [R=200,L]
```
Above final configuration was hack to let respond to browsers CORS requests. Browsers call the services twice, first to do kind of handshake for CORS where server informs what headers are allowed, etc. Above modification tells that send 200 when browser requests for CORS details from Apache httpd for OPTIONS header.

This small hack helped developers where we no longer needed to run services in jetty or tomcat maven plugins on our local machines and also reduced the CPU consumed by these services, freeing precious CPU cycles.

We could have upgraded the processors for developers but this seemed cheaper. :-)
 
