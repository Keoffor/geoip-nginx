
# Navigation Menu

<!-- TOC -->

- [Code Challenge Purpose: NGINX, Bash/Shell Scripting, and SysAdmin](#code-challenge-purpose-nginx-bashshell-scripting-and-sysadmin)
- [Evaluation and Judging Criteria](#evaluation-and-judging-criteria)
- [Project Background](#project-background)
    - [Graviton's Previous Server Build](#gravitons-previous-server-build)
    - [Status of Graviton's Current Scripts and NGINX Server Block File](#status-of-gravitons-current-scripts-and-nginx-server-block-file)
- [Code Challenge: Your Assignment](#code-challenge-your-assignment)
    - [Required](#required)
    - [Due Date](#due-date)
    - [Optional, Not Required](#optional-not-required)
- [Graviton's GeoIP Setup](#gravitons-geoip-setup)
    - [ngx_http_geoip2_module](#ngx_http_geoip2_module)
    - [Database Updates](#database-updates)
- [Pro Tips 1 - 7: Things to Keep in Mind For this Code Challenge](#pro-tips-1---7-things-to-keep-in-mind-for-this-code-challenge)
- [Pro Tip 8: How To Clone a Private Repo](#pro-tip-8-how-to-clone-a-private-repo)

<!-- /TOC -->

# Code Challenge Purpose: NGINX, Bash/Shell Scripting, and SysAdmin

> As a Consultants and Software Engineers, we see ourselves as teachers and mentors to our clients.  This is the perspective you should take while completing this assignment

In your self-assessment you indicated that you have experience with bash scripting and NGINX. Our Java consulting and dev work may get slow at times &mdash; there may be a lot of down time doing nothing. As such, having additional skills are competitive differentiators during the interview process.

This code challenge is "packaged" as a situational case study.  Its purpose is to assess your skills with respect to bash scripting, configuring NGINX, researching information, commenting & documenting code as well as email etiquette & communication skills. 

A firm understanding and appreciation of NGINX is required to successfully complete this case study and code challenge.

<!-- Being a remote worker has its unique challenger. However, communication as a potential remote employee with our firm. 
 -->

# Evaluation and Judging Criteria

1. Successfully completing all tasks in the [Code Challenge: Your assignment](https://github.com/treebright/susmitha/blob/master/README.md#code-challenge-your-assignment) section 
1. Quality and readability of your code (as well as comments)
1. Insights as shared in your project's README.md and within your code's comments
1. Overall attention to detail


# Project Background

Your client is a *fictional* company called ***Graviton Sciences Corp***.  They are a [La Jolla, CA](https://www.google.com/search?q=La+Jolla%2C+CA) based materials science start-up.  Graviton researches, synthesizes, and manufactures novel metallic alloys, thin films, and dielectrics.  These engineered materials are used in the power generation, aerospace, and Satellite Communications (SatCom) industries.

Graviton has 5 customers which include [SpaceX](https://www.spacex.com/), [NRG](https://www.nrg.com/), [Inmarsat](https://www.inmarsat.com/en/index.html), [Nasa JPL](https://www.jpl.nasa.gov/), and [Sandia National Labs](https://www.sandia.gov/).  Given the sensitive nature of their work, Graviton no longer maintains a public facing website.

In 2021, their IT guy, Jim Dev, created 5 shell scripts that configured, tested, and deployed a proof-of-concept for Graviton's [extranet](https://www.lumapps.com/modern-intranet/difference-intranet-vs-extranet/#what-is-an-extranet).  The extranet was built on the LEMP (Linux, NGINX, MySQL, and PHP) stack. It was hosted in-house, at the company's headquarters in La Jolla. Its purpose was to manage secure communication amongst Graviton's remote employees and customers. 


## Graviton's Previous Server Build
To build the server, Jim ran his scripts manually and sequentially. For example:

* **Script 1:** Installed Ubuntu 20.04, created admin users, locked down SSH, and configured iptables
    * Test Result: Server is up and running. SSH passwordless login works
    * Ports Open:  random ports for SSH and MySQL 8.0 as well as 80 and 443
* **Script 2:** Installed NGINX, PHP 7.4, and generated an NGINX server block file for graviton.com
    * Test Result 1: NGINX is up and running. Accessible on port 80
    * Test Result 2: PHP info page successfully renders
* Script 3: Installed + configured MySQL, and created MySQL database with test data  
    * Test Result: MySQL up and Running. Is accessible on a random port via SSH tunneling using MySQL Workbench 
* **Script 4:**
    1. Installed and configured MaxMind's GeoIP module for NGINX
    1. Customized `/etc/nginx/nginx.conf` to work with GeoIP
        * Access to extranet only permitted for Five Eyes countries
    1. Customized access log file format within the server block
        * Test Result: Website is only accessible by Five Eyes countries, web logs show detailed geo information on website visitors 
* **Script 5:** Used Let’s Encrypt certbot to install SSL certs, performed renewal dry-run, and further "tuned" `/etc/nginx/nginx.conf` for security and performance
    * Test Result: SSL is working, site is only accessible via SSL

Unfortunately, Jim only saved the scripts on his personal laptop.  In January 31, 2022 he accidentally deleted the development folder on his Windows 10 laptop.  Most of  his scripts were permanently lost. 

## Status of Graviton's Current Scripts and NGINX Server Block File

By the end of 2022, Jim has since managed to recreate scripts 1 through 3. These scripts programmatically install and configure Ubuntu **22.04**, SSH, admin users, iptables, MySQL 8, and PHP **8.1**.
 

Here's the contents of the `/etc/nginx/sites-available/graviton.com` server block generated from Script 2:

 ```
server {
    listen      80;
    listen      [::]:80;

    server_name graviton.com www.graviton.com;
    # return         301 https://graviton.com$request_uri;

    }

    root        /home/jim-dev/public_html/graviton.com/public;

    index       index.php index.htm index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

}
 ```

# Code Challenge: Your Assignment

Your job is to help Graviton and Jim get to the last mile by **further securing and locking-down their extranet**.

Specifically, create the needed shell scripts to programmatically do the following:

1. Implement an NGINX rate limiter using some or all of these directives: `limit_req_zone`, `limit_req`, and `limit_req_status`

1. Block all access to:
    * "." directories (e.g., .git) ***except*** .well-known

    * Hidden files (e.g., any file beginning with ".". Examples include `.htaccess`, `.htpasswd`, `.DS_Store`)

    * The sensitive file extensions specified here: https://docs.nginx.com/nginx-app-protect-waf/configuration-guide/configuration#disallowed-file-types 

1. Install and then use Fail2Ban to block any IP address after it attempts to access an unauthorized file or directory mentioned above 
    * Jail time should be for 48 hours

1. Install MaxMind's GeoIP module and database. Refer to [Graviton's GeoIP Setup](#gravitons-geoip-setup) for details on Jim's Graviton setup  

    * Programmatically create a cron job to refresh the DB at a frequency you deem appropriate 
    
    * Whitelist or Blacklist countries:

       * In your bash script, offer the end-user the option to either "Deny ALL Except" or "Allow ALL Except" web server visitors based on the following criteria:

	        a. "Deny ALL Except": Use GeoIP to only allow the Five Eyes countries to access the website

	        b. "Allow ALL Except": Use GeoIP to allow all countries except China, Russia, North Korea, Iran, Syria, and Belarus 

	<!-- **NB:** "Deny ALL Except" and "Allow ALL except" or not technical NGINX terms, but convey the needed functionality in "plain English" -->

1. Create custom access logs that show the rich data provided by the GeoIP module. Include visitor's City, State, ZipCode, Country, and ISP. 
    * Make sure to also place this access log within the custom server block

1. Create a script to auto install CertBot LetsEncrypt SSL Certificates. Ask the user to enter their domain name.  After install, programmatically perform a renewal dry-run to make sure the setup is correct

    * Programmatically update Graviton's existing server-block file to support SSL 

1. Update default NGINX config file `/etc/nginx/nginx.conf` to work with the GeoIP module. Also: 

   * [Mozilla](https://ssl-config.mozilla.org) and [DigitalOcean](https://nginxconfig.io) have templates for creating a security-oriented `nginx.conf` file. Choose and/or customize the best one. Additionally:
   
   * Be sure to import any custom modules and configurations you created above

   * Feel free to use tips from other online tutorials and references to build off Mozilla's or Digital Ocean's template


## Required

1. Within your scripts, use comments to cite the source (e.g., URL of online Tutorial or StackOverflow solution) of the commands and directives you used to solve each of the above challenges.  

1. Your scripts should be human readable and self-documenting
    * Use non-cryptic variable names
    * Use comments liberally, but wisely, to explain what each section of your scripts do

1. Be sure to use `echo` (or similar directives) to show your end-user what is happening while the script is running

1. Modularize (via importable, custom config files) the custom NGINX features and directives you add.  For example, you script can create a separate "zone file" for rate limiting.  Then import it into the `location` portion of the server block file

1. Explain your reasoning for the values you chose in your rate limiter 

1. There are numerous "arm-chair" experts that have their NGINX tuning tips. Many of their suggestions are outdated, questionable, or just outright false because they are based on a single data-point. Address the following:
    * Which "tried and true" suggestions would you recommend Graviton use to better tune NGINX for performance, scalability, and reliability?
        * Which have you implemented in your modified `nginx.conf` and server block files? 
        * Which tutorials have high quality NGINX performance tuning tips?
    * Should `gzip` compression be turned on or off? Why?


<!--
## Optional Bonus Questions:	

 1. This [gentleman](https://haydenjames.io/nginx-tuning-tips-tls-ssl-https-ttfb-latency) makes some assertions on NGINX tuning. Parts of his recommendations are opposite of what the Mozilla and DigitalOcean represent as best practices.  Which of his claims are true and which ones (if any) are assumptions based on a single data-point? -->

## Due Date

Email us within 24 hours of receiving this code challenge with a reasonable due date. If you later find you need more time to complete, you will not be penalized.  However, please let us know at least 48 hours before the deadline that you'll need an extension.

## Optional, Not Required

Although not required, you're welcomed to do any or all of the following:

1. Completing the project before your deadline
1. Sending us an [email](java@treebright.com) if you found typos or something that's unclear in this case study 
1. Going beyond the minimum of the [Code Challenge: Your Assignment](#code-challenge-your-assignment) section
1. Providing suggestions on what should be added (or removed) from future iterations of the [Code Challenge: Your Assignment](#code-challenge-your-assignment) section
1. Committing regular Work-In-Progress (WIP) drafts and updates to this repo before the deadline
1. Sending us status emails on how your progress is coming along

# Graviton's GeoIP Setup

You reached out to Jim by phone and you took the following notes:

Jim registered Graviton to use MaxMind's [GeoLite2 Free Geolocation Data](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data).  Jim read the information in the aforementioned link, **very carefully**. Following those instructions he generated a free [license key](https://www.maxmind.com/en/accounts/current/license-key).

While studying MaxMind's developer docs as well as online tutorials in 2020, Jim had an epiphany. He realized that ***all pre-2019 tutorials and StackOverflow answers might be partially outdated***! In 2019 MaxMind upgraded to GeoIP2 which uses their new open-sourced `mmdb` database format. The "official" module from NGINX could only read the deprecated `dat` format. 

Jim realized he needed to go "beast mode" and head over to GitHub.  In the final analysis, Jim concluded he needed two MaxMind programs and one unofficial NGINX module which had to be built from the source as a dynamic NGINX module:  

1. [libmaxminddb C library](https://github.com/maxmind/libmaxminddb/) for reading MaxMind DB files. 
1. [GeoIP Update program](https://github.com/maxmind/geoipupdate) to update the GeoLite2 binary databases:
1. [ngx_http_geoip2_module](https://github.com/leev/ngx_http_geoip2_module) built from the source as a dynamic NGINX module. Several dependencies (e.g., `libpcre3` `libpcre3-dev`, etc) will be need to installed as well

After reading the README.md files of the above repos, Jim found it easiest to proceed by installing MaxMind's Ubuntu PPA. After installing the PPA, he installed the GeoIP Update tool and databases simultaneously via: 

```
sudo apt update
sudo apt install geoipupdate libmaxminddb0 libmaxminddb-dev mmdb-bin
```

The above should install a file called: `/etc/GeoIP.conf` with missing credentials.

## `ngx_http_geoip2_module`

In 2021, Ubuntu may have started including the updated version of this  module in its official packages. If this is the case, it could be easily installed via `sudo apt install libnginx-mod-http-geoip2`. Dependencies would automatically be installed as well!


## Database Updates

After reading MaxMind's dev docs on the [GeoIP Update](https://dev.maxmind.com/geoip/updating-databases) program, Jim downloaded a copy of his pre-filled `GeoIp.conf` file and saved it to on the server.  Below are the contents of the pre-filled `GeoIp.conf` (license key and account ID have been obfuscated):

```
# GeoIP.conf file for `geoipupdate` program, for versions >= 3.1.1.
# Used to update GeoIP databases from https://www.maxmind.com.
# For more information about this config file, visit the docs at
# https://dev.maxmind.com/geoip/updating-databases?lang=en.

# Numeric `AccountID` is found here: https://www.maxmind.com/en/accounts/current/edit
AccountID #######

# Alphanumeric `LicenseKey` may be regenerated here: https://www.maxmind.com/en/accounts/current/license-key
LicenseKey ######

# `EditionIDs` is from your MaxMind account.
EditionIDs GeoLite2-ASN GeoLite2-City GeoLite2-Country

```

# Pro Tips 1 - 7: Things to Keep in Mind For this Code Challenge

1. As a Consultants and Software Engineers, we see ourselves as teachers and mentors to our clients.  This is the perspective you should take while completing this assignment.  Your scripts should be well commented. They should serve as learning aids for Graviton and Jim

1. You are strongly encouraged to use online tutorials and StackOverlow solutions for best practices and guidance

1. We encourage you to email us if you have questions, need clarifications, or spot any typos in this assignment. We will respond within 24 hours. Typically after 5 PM EST.

1. You can find up to 4 online tutorials on each code challenge topic. These tutorials will take about 15 minutes for an experienced NGINX SysAdmin  to review and understand.

1. Jim Dev was able to recover the URLs of the tutorials he used to build his original scripts via his Google "My Activity". Email us at java@treebright.com if you feel this information will be useful to you.  Be sure to pre-pend the subject of any emails with your name (and candidate ID if applicable). For example:  `John Smith - 25: Question on Tuning`

1. If you need more time to complete the project, please reach out to us and request an extension. Our email is java@treebright.com

1. If you feel we are not a good fit for you and prefer to dropout of the interview process, please send an email to java@treebright.com. Feel free to let us know how we should improve the interview process


# Pro Tip 8: How To Clone a Private Repo

Although, forking is not permitted on private repos, you have been given write access to this repo.  You may clone this repo locally and push your updates directly. If you use GitHub from the command line this is how you would clone:

**NB:** The repo is automatically tagged as *origin* when you clone it!

**Option 1: Cloning via HTTPS (entire repo and all branches):**

`git clone https://YOUR_GITHUB_USERNAME:YOUR_GITHUB_TOKEN@github.com/treebright/REPO_NAME.git`

**Option 2: Cloning  via SSH (entire repo and all branches):**

`git clone git@github.com:treebright/REPO_NAME.git`

**Option 3: Cloning a specific branch via HTTPS:**

**NB:** The below command should be on a single line. `SPECIFIC_BRANCH_NAME` and `https://...` are only separated by a single space.

`git clone -b SPECIFIC_BRANCH_NAME https://YOUR_GITHUB_USERNAME:YOUR_GITHUB_TOKEN@github.com/treebright/REPO_NAME.git`

<!-- After cloning create a new branch locally, make your edits, stage, and commit locally. Then push your new local branch to the repo. -->

**After cloning:** create a new branch locally (or just merge into our master branch), make your edits, stage, and commit locally. Then push your changes to this repo. <!-- new local branch to the repo. -->
