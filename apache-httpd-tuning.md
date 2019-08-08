### Optimizations done ######
1. Remove unused modules and load only the required minimum modules
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
2. Setup appropriate Expires, Etag, and Cache-Control Headers
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
3. Caching, our application is light weight with very neglible media/content for caching and Single Page Application (SPA) so we dont use caching. But for content heavy application it is recommended to use Content Delivery Network (CDN).
4. We use ELK to store all our logs, we have created rolling log files to log errors and access. We also keep logging to bare minimum, only certain headers for debugging is logged. We log only last 5 digits of our JWT for debugging purpose.
```
LogLevel warn
#purge the logs in 7 days = 60*60*24*7
ErrorLog "|/usr/sbin/rotatelogs -t /app-name/logs/error-log 604800"
#purge the logs in 7 days = 60*60*24*7
SetEnvIf Cookie "(^|;\ *)jwt=([^;\ ]+)" jwt-value=$2
SetEnvIf access-token-value ".{5}$" jwt-last-5=$0
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{user-agent}i\" \"user-id:\" \"%{user-id}i\" \"message-id:\" \"%{message-id}i\" \"browser-id:\" \"%{browser-id}i\" \"jwt:\" \"%{jwt-last-5}e\" %T %D" app-log-format
CustomLog "|/usr/sbin/rotatelogs -t /app-name/logs/access-log 604800" app-log-format
```
5. Utilize mod_gzip/mod_deflate
6. Default event mutli processing module used - this keepsalive connection and keeps some connections to idle state to server requests faster
7. Increase - ulimit size 4096 - I guess default is 10000 on RHEL 7.5 at CVS, this can be increased by talking to Unix Admins
8. Reverse proxy algorithm - by least busy connection

### Security Optimizations done  ######
1. Below are some of http security headers which we have tuned. For more information (check this link)[https://nullsweep.com/http-security-headers-a-complete-guide/]
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

### Potential tune ups  ######
1. Tune mpm event module through PT or jmeter or Apache Workbench 
```
<IfModule mpm_event_module>
    StartServers             4
    MinSpareThreads          25
    MaxSpareThreads          75
    ThreadLimit              64
    ThreadsPerChild          25
    MaxRequestWorkers        30
    MaxConnectionsPerChild   1000
</IfModule>    
```
2. Linux - increase write buffer size (not sure how much) - /proc/sys/net/core/wmem_max
3. Linux - increase swapiness (not sure how much) - /proc/sys/vm/swappiness
4. Setup CDN - not needed due to low volumn of multimedia content and SPA
5. No caching needed on server due to SPA as chrome will cache
