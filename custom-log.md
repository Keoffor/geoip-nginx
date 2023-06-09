
# Custom Logging Settings

**Source:** [Digital Ocean](https://www.digitalocean.com/community/tutorials/nginx-access-logs-error-logs)

Apply below log configuration in http block - `/etc/nginx/nginx.conf`

```
   http{
   
      log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer"
                      "$http_user_agent" $geoip2_data_country_code $geoip2_data_city_name $geoip2_data_state_name 
                       $geoip2_data_postal_code
                       $geoip2_data_country_iso_code';
   }
```


Add configuration inside server block - ` /etc/nginx/sites-available/default`

 ```
  location / {
                 gzip on;
                 access_log /var/log/nginx/domain1.access.log main;
  }
  ```
 Check log with command `nano access_log /var/log/nginx/domain1.access.log`

 **output:** 
 
`34.174.141.251 - - [07/Mar/2023:16:22:40 +0000] "GET / HTTP/1.1" 200 51 "-" "curl/7.68.0" US Dallas Texas 75270 -`
