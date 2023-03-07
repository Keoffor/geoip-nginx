
# Rate Limiting

#This limits the amount of http requests a user can make in a given period of time.
# Source: https://www.nginx.com/blog/rate-limiting-nginx/
# Source: https://www.youtube.com/watch?v=0fevo6Aa3BA
#Apply the configuration in Nginx directory - /etc/nginx/sites-avaliable/default as demonstrated below:

#The rate is specified to handle 5 requests per second in a given session.
#add config outside server block.
 limit_req_zone $binary_remote_addr zone=mylimit:10m rate=5r/s;
 limit_req_status 429;

#implement limit_req inside the location block:
server {

        listen 80 default_server;
        listen [::]:80 default_server;
        server_name  _;
       location / {

                limit_req zone=mylimit;
       }
 }

#Reason: This will prevent the server from vulnerability and hard for attackers to crack the system within the 5requests
#per second sesssion. it's relatively impossible for malicious users to break through.


##
# Blocking access to all files and directories that start with .(dot) expect .well-knwon
##
#Source: https://stackoverflow.com/questions/34259548/how-to-disallow-access-to-all-dot-directories-except-well-known
#Create location block inside server block in nginx directory - /etc/nginx/sites-avaliable/default:
#add configuration to allow .well-known:
        location /\.well-known {
           allow all;
        }
#add config to deny all access to files or directories.
        location /\. {
            deny all;
            return 404;

        }



















