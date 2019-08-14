### Security Optimizations ######

1. **Headers**
Below are some of http security headers which we have tuned. For more information [check this link](https://nullsweep.com/http-security-headers-a-complete-guide/). 
  * We set our JWT cookie as to work only with https(**Secure**). JWT cookie cannot be access by javascript(**httpOnly**). 
  * **X-Frame-Options** ensures that only iframes from same domain are allowed. 
  * **Content Security Policy** specifies that all content is loaded only from same domain. 
  * **Strict-Transport-Security** ensures that it is https only for all domains including sub domains. 
  * **X-Content-Type-Options** informs browser to respect the allowed mime types set by server.
  * **X-XSS-Protection** informs the browser to stop execution of detected xss attacks.
  * For **[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)** only allowed http methods are POST, GET, OPTIONS, DELETE and     PUT
  * For **[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)** allowed domain, we allow only from our root domain.
  
 ```
 SetEnvIf Origin "^(.*\.my-root\.com)$" root-domain=$1
 <IfModule mod_headers.c>
  Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
  Header always append X-Frame-Options SAMEORIGIN
  Header set X-Content-Security-Policy "script-src 'self'; object-src 'self'"
  Header always set Strict-Transport-Security "max-age=36000; includeSubDomains"
  Header set X-Content-Type-Options: nosniff
  Header set X-XSS-Protection "1; mode=block"
  Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"
  Header set Access-Control-Allow-Origin "%{root-domain}e" env=root-domain
 </IfModule>
 ```
 
2. **Mime Types**
Only specific mime/file types served
```
 <IfModule mime_module>
    TypesConfig /etc/mime.types

    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz

    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
 </IfModule>

```

3. **Https**
We use [TLS 1.2](https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_1.2) as default for all our communication right from browser to database. Below are configuration to turn on TLS 1.2 settings for Apache Httpd.
*StrictRequire* returns forbidden access when someone tries access http site against https.
We enable *SSLProxyEngine* to reverse proxy to our micro-services backend and for browser to send the our authentication cookie(JWT) back to our micro-services backend. We only compare the common-name(CN) of certificate matches the host-name by turning on *SSLProxyCheckPeerCN* and turning off *SSLProxyCheckPeerName*
```
 SSLEngine on
 SSLOptions +StrictRequire
 SSLProxyEngine on
 SSLProxyVerify require
 SSLProxyCheckPeerCN on
 SSLProxyCheckPeerName off
```
Below option specifies that https is required to access for all paths. This option along with *StrictRequire* gives forbidden access error.
```
 <Directory />
   SSLRequireSSL
 </Directory>
```
We enable on TLS 1.2 rejecting any SSL connections lower than TLS 1.2
```
 SSLProtocol -ALL -SSLv2 -SSLv3 -TLSv1 -TLSv1.1 +TLSv1.2
 SSLProxyProtocol -ALL -SSLv2 -SSLv3 -TLSv1 -TLSv1.1 +TLSv1.2
 ```
We also enable only the strongest ciphers.
 ```
 SSLCipherSuite ALL:!aNULL:!ADH:!eNULL:!LOW:!EXP:!RC4+RSA:+HIGH:!MEDIUM:!SSLv2:!SSLv3:!IDEA:!RC2:!DSS:!TLSv1:!TLSv1.1:!DES:!3DES
 SSLProxyCipherSuite ALL:!aNULL:!ADH:!eNULL:!LOW:!EXP:!RC4+RSA:+HIGH:!MEDIUM:!SSLv2:!SSLv3:!IDEA:!RC2:!DSS:!TLSv1:!TLSv1.1:!DES:!3DES
 ```
Below are certificate settings. We use [CRT](https://en.wikipedia.org/wiki/X.509#Certificate_filename_extensions) format for our certificates. We have Global Root certificate of our CA in root-ca.crt and rest of chain including server, intermediate certificate  is kept in app.crt file. The private key is saved in app.key file.
 ```
 SSLCACertificateFile /app-directory/tlscerts/root-ca.crt
 SSLCertificateFile /app-directory/tlscerts/app.crt
 SSLCertificateKeyFile /app-directory/tlscerts/app.key
```

4. **Directory access permissions**

```
 <Directory "/app-directory">
    AllowOverride All
    Options FollowSymLinks
    Options SymLinksIfOwnerMatch
    Require all granted
 </Directory>

```
