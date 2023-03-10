
# Use GeoIP to restricts access against specifc country ips
**Source:** [Maxime Durand blog post](https://medium.com/@maxime.durand.54/add-the-geoip2-module-to-nginx-f0b56e015763), [Tech Expert Tips Blog](https://techexpert.tips/nginx/nginx-blocking-access-from-country/)

Create GeoLite2 account on [Maxmind's database](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data)

Install the following dependencies from your package manager as referred [here]( https://github.com/treebright/kenneth#gravitons-geoip-setup):
```
sudo add-apt-repository ppa:maxmind/ppa
sudo apt update
sudo apt install geoipupdate libmaxminddb0 libmaxminddb-dev mmdb-bin
```
Open file - `/etc/GeoIP.conf` and replace `YOUR_ACCOUNT_ID_HERE` and `YOUR_LICENSE_KEY_HERE`with your own credentials copied from your Maxmind account

Add a cron rule to enable database update `nano /etc/cron.d/geoip2update`
```
# Run GeoIP database update every 5mintues
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

0 */5 * * * root /usr/bin/geoipupdate
```
Check your nginx version with command  `nginx -v`

change directory to your nginx-version directory `cd /etc/nginx/nginx-VERSION`

configure and make your module `./configure --with-compat --add-dynamic-module=../ngx_http_geoip2_module`

install dependencies if not installed `apt-get install libpcre3 libpcre3-dev`

Copy the GeoIP2 module into Nginx directory  using command 
```
mkdir -p /etc/nginx/modules` 
cp -vi objs/ngx_http_geoip2_module.so /etc/nginx/modules/
```

This config will restrict the specified country ips access to our server. Edit file -  `/etc/nginx/nginx.conf:`

```
http {
       geoip2 /etc/nginx/geoip/geodb2/GeoLite2-Country.mmdb {
          $geoip2_data_country_code country iso_code;
          $geoip2_data_country_name country names en;
       }
      geoip2 /etc/nginx/geoip/geodb/GeoLite2-City.mmdb {
         $geoip2_data_city_name   city names en;
         $geoip2_data_postal_code postal code;
         $geoip2_data_state_name  subdivisions 0 names en;
         $geoip2_data_state_code  subdivisions 0 iso_code;
         $geoip2_data_country_iso_code iso_code;
      }

       # This will deny ALL except countries listed below:
       #map $geoip2_data_country_code $allowed_country {
       #default no;
       # FR yes; # France
       # BE yes; # Belgium
       # DE yes; # Germany
       # CH yes; # Switzerland
       # US yes; # United States
       #}

       #This will allow ALL except countries listed below
       map $geoip2_data_country_code $allowed_country {
       default yes;

       # Block forbidden country
        CN no; # China
        RU no; # Russia
        IR no; # Iran
        SY no; # Syria
        KP no; # North Korea

        }
 }
 ```
For GeoIp to block countries ip apply configuration inside server block - `/etc/nginx/sites-available/default`
```
        location / {
           if ($allowed_country = no) {
                 return 403;
        }
```
restart nginx and test configuration `systemctl restart nginx`

**Output**
```
HTTP/1.1 403 Forbidden
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 05 Mar 2023 15:56:47 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
```