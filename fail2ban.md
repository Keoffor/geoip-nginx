
# Using Fail2Ban to block any IP address after it attempts to access any unauthorized file or directory

**Source:** [Maciej Blog Post](https://iceburn.medium.com/how-to-apply-fail2ban-to-nginx-excess-404-and-403-6b601285df02), [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-protect-an-nginx-server-with-fail2ban-on-ubuntu-20-04)
Apply the following commands to install fail2ban in your server
 ```
 sudo apt update
 sudo apt install fail2ban
```

Switch to directory - `/etc/fail2ban` and create *jail.local* file out of *jail.conf* using the command:
```
cd /etc/fail2ban
sudo cp jail.conf jail.local
```

Use vim or nano editor to open the file - `jail.local`

`nano jail.local`

Navigate through the script to locate bantime and set it to 48hrs. save changes and restart:
```
[DEFAULT]
. . .
bantime = 48h
```
restart nginx `systemctl restart nginx`

Add jail configuration in `/etc/fail2ban/jail.local`
```
[nginx]
enabled = true
port = http,https
filter = nginx
logpath = /var/log/nginx*/*access.log
action = iptables-multiport[name=404, port="http,https", protocol=tcp]
maxretry = 3
findtime = 20
```

Create file - `/etc/fail2ban/filter.d/nginx.conf` and add regex configuration
```
#this enables fail2ban to monitor nginx logs and ban any ip that fails to access a resource after third attempts.
[Definition]
failregex =  ^<HOST>.*"(GET|POST).*" (403|404|503|429) .*$
ignoreregex =

```

Use command to check the details/status being enforced by nginx jail
`sudo fail2ban-client status nginx`

**Output**
```
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     3
|  `- File list:        /var/log/nginx/access.log
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   34.174.141.251
```