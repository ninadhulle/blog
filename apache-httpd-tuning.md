### Optimizations done ######
1. Remove unused modules
2. Setup appropriate Expires, Etag, and Cache-Control Headers
3. Put Cache on separate disk - No caching used
4. Use Piped Logging instead of direct logging - log files overwritten after a week due ELK integration
5. Utilize mod_gzip/mod_deflate
6. Default event mutli processing module used - this keepsalive connection and keeps some connections to idle state to server requests faster
7. Increase - ulimit size 4096 - I guess default is 10000 on RHEL 7.5 at CVS, this can be increased by talking to Unix Admins
8. Reverse proxy algorithm - by least busy connection

### Security Optimizations done  ######
1. Mod headers security tuning
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
