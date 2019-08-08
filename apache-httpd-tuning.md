### Apache Httpd 2.4 Performance Optimizations for Angular ######
We use [Apache Httpd 2.4](https://httpd.apache.org/docs/2.4/) to run our [Angular](https://angular.io/) application in production. Below are few of common Httpd tuning options which we had done for production.

1. **Modules:** Remove unused modules and load only the required minimum modules
```
LoadModule lbmethod_bytraffic_module modules/mod_lbmethod_bytraffic.so
LoadModule lbmethod_heartbeat_module modules/mod_lbmethod_heartbeat.so
LoadModule lbmethod_bybusyness_module modules/mod_lbmethod_bybusyness.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so

LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_module modules/mod_proxy.so

LoadModule reqtimeout_module modules/mod_reqtimeout.so

LoadModule rewrite_module modules/mod_rewrite.so
LoadModule headers_module modules/mod_headers.so

LoadModule slotmem_plain_module modules/mod_slotmem_plain.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
```
2. **Compression:** Setup appropriate Expires, Etag, and Cache-Control Headers
```
SetOutputFilter DEFLATE
SetEnvIfNoCase Request_URI \.(?:exe|t?gz|zip|iso|tar|bz2|sit|rar|png|jpg|gif|jpeg|flv|swf|mp3)$ no-gzip dont-vary
DeflateCompressionLevel 9

TraceEnable Off
Header unset ETag
FileETag None
ServerTokens Prod
ServerSignature Off
```
3. **Caching:** Our application is light weight with very neglible media/content for caching and Single Page Application (SPA) so we dont use caching. But for content heavy application it is recommended to use Content Delivery Network (CDN).
4. We use ELK to store all our logs, we have created rolling log files to log errors and access. We also keep logging to bare minimum, only certain headers for debugging is logged. We log only last 5 digits of our JWT for debugging purpose.
```
LogLevel warn
#purge the logs in 7 days = 60*60*24*7
ErrorLog "|/usr/sbin/rotatelogs -t /app-name/logs/error-log 604800"
#purge the logs in 7 days = 60*60*24*7
SetEnvIf Cookie "(^|;\ *)jwt=([^;\ ]+)" jwt-value=$2
SetEnvIf jwt-value ".{5}$" jwt-last-5=$0
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{user-agent}i\" \"user-id:\" \"%{user-id}i\" \"message-id:\" \"%{co-rel-id}i\" \"browser-id:\" \"%{browser-id}i\" \"jwt:\" \"%{jwt-last-5}e\" %T %D" app-log-format
CustomLog "|/usr/sbin/rotatelogs -t /app-name/logs/access-log 604800" app-log-format
```
5. **Multi Processing Module:** As recommended by Apache we use The **Event** Multi-Processing Module [MPM](https://httpd.apache.org/docs/2.4/mod/event.html). This is default processing module and it allows the requests to be passed off to listener threads freeing up the worker threads to server the request. Event MPM keepsalive connection and keeps some connections to idle state to server requests faster.
We use below configuration. 
```
 <IfModule mpm_event_module>
    ServerLimit         16
    StartServers         2
    MaxRequestWorkers  200
    MinSpareThreads     25
    MaxSpareThreads     75
    ThreadsPerChild     25
 </IfModule>

```
Apache Benchmark(ab) tool that comes with Apache Httpd can be used to simulate the requests and get the metrics.
```
ab -k -c 1000 -n 8000 https://<server>:<port>
```
-k keeps alive the connection similar to browser
-c concurrent connection
-n max# of connections
6. **Reverse proxy load balancer:** Algorithm - by least busy connection
7. **Linux OS Optimizations:** We use RHEL 7.5 and below are few optimizations we did to further improve performance.
 * Increase - ulimit size 4096 - I guess default is 10000 on RHEL 7.5 at CVS, this can be increased by talking to Unix Admins
 * Increase write buffer size (not sure how much) - /proc/sys/net/core/wmem_max
 * Increase swapiness (not sure how much) - /proc/sys/vm/swappiness


### Security Optimizations ######
1. Below are some of http security headers which we have tuned. For more information [check this link](https://nullsweep.com/http-security-headers-a-complete-guide/)
```
<IfModule mod_headers.c>
Header set X-XSS-Protection "1; mode=block"
Header always append X-Frame-Options SAMEORIGIN
Header set X-Content-Type-Options: nosniff
Header set X-Content-Security-Policy "script-src 'self'; object-src 'self'"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
</IfModule>

```
2. Only specific file types served
3. TLS 1.2

