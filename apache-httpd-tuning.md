### Apache Httpd 2.4 Performance Optimizations ######
We use [Apache Httpd 2.4](https://httpd.apache.org/docs/2.4/) to run our [Angular](https://angular.io/) application in production. Below are few of common Httpd tuning options which we had done for production.

1. **Multi Processing Module:** As recommended by Apache we use The **Event** Multi-Processing Module [MPM](https://httpd.apache.org/docs/2.4/mod/event.html). This is default processing module and it allows the requests to be passed off to listener threads freeing up the worker threads to server the request. Event MPM keepsalive connection and keeps some connections to idle state to server requests faster.
We have 16 core CPU so we set out *ServerLimit* to 16. The calculation is *MaxRequestWorker*=*ServerLimit* x *ThreadsPerChild*
```
  <IfModule mpm_event_module>
     ServerLimit         16
     StartServers         2
     MaxRequestWorkers  400
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
 
2. **Modules:** Remove unused modules and load only the required minimum modules. We have configured below minimal set of modules to be loaded.
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
 
 LoadModule mpm_event_module modules/mod_mpm_event.so 
```

3. **Compression:** 
 Apache Httpd's [mod_deflate](https://httpd.apache.org/docs/2.4/mod/mod_deflate.html) to zip the data when sent to browser. Below configuration enables compression for file extensions. Also we use compression level(*DeflateCompressionLevel*) as 9 as our web CPU is barely utilized due to Single Page Application(SPA) architecture.
```
 SetOutputFilter DEFLATE
 SetEnvIfNoCase Request_URI \.(?:exe|t?gz|zip|iso|tar|bz2|sit|rar|png|jpg|gif|jpeg|flv|swf|mp3)$ no-gzip dont-vary
 DeflateCompressionLevel 9
```

4. **Caching:** Our application is light weight with very neglible media/content for caching and as we have SPA architecture so we dont use caching or Content Delivery Network (CDN). But for content heavy application it is recommended to use Content Delivery Network (CDN).
```
 <IfModule mod_headers.c>
  Header set Cache-Control no-cache
  Header set Expires: 0
 </IfModule>
```

5. **Logging:** We use ELK to store all our logs, we have created rolling log files to log errors and access. We also keep logging to bare minimum, only certain headers for debugging is logged. We log only last 5 digits of our JWT for debugging purpose.
```
 LogLevel warn
 #purge the logs in 7 days = 60*60*24*7
 ErrorLog "|/usr/sbin/rotatelogs -t /app-name/logs/error-log 604800"
 #purge the logs in 7 days = 60*60*24*7
 SetEnvIf Cookie "(^|;\ *)jwt=([^;\ ]+)" jwt-value=$2
 SetEnvIf jwt-value ".{5}$" jwt-last-5=$0
 LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{user-agent}i\" \"user-id:\" \"%{user-id}i\" \"log-co-relation-id:\" \"%{co-  rel-id}i\" \"browser-id:\" \"%{browser-id}i\" \"jwt:\" \"%{jwt-last-5}e\" %T %D" app-log-format
 CustomLog "|/usr/sbin/rotatelogs -t /app-name/logs/access-log 604800" app-log-format
```
6. **Reverse proxy load balancer:** Our Angular application forwards all http requests to fetch data through API Gateway Proxy. We use *by least busy connection* reverse proxy load balancer algorithm. This always forwards request to least busy server. One drawback of *by least busy connection* is it doesnt ping if server is down, so it still forwards the connection when server is down, but our other reverse proxy infrastructure takes care of high availability.
```
 <Proxy balancer://api-gateway-proxy>
   BalancerMember https://<api-gateway-proxy-1>:<port-1>/
   BalancerMember https://<api-gateway-proxy-2>:<port-2>/
   BalancerMember https://<api-gateway-proxy-3>:<port-3>/
   BalancerMember https://<api-gateway-proxy-4>:<port-4>/
   ProxySet lbmethod=bybusyness
 </Proxy>

  ProxyPass "/api-gateway-proxy" "balancer://api-gateway-proxy" timeout=90
  ProxyPassReverse "/api-gateway-proxy" "balancer://api-gateway-proxy"
```

7. **Linux ulimit:** 
We increased the RHEL 7.5 ulimit - number of process RHEL 7.5 Kernel will allow to be opened to 4096. Default earlier was 1096 but  now it is increased to 4096. This should be changed before starting the server and would need root access.
 


