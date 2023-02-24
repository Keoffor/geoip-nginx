
# Navigation Menu

<!-- TOC -->

- [Code Challenge Purpose](#code-challenge-purpose)
- [Evaluation and Judging Criteria](#evaluation-and-judging-criteria)
- [Project Background](#project-background)
    - [Graviton's Previous Server Build](#gravitons-previous-server-build)
- [Code Challenge: Your Assignment](#code-challenge-your-assignment)
    - [Required](#required)
    - [Due Date](#due-date)
    - [Optional, Not Required](#optional-not-required)
- [Pro-tips:](#pro-tips)

<!-- /TOC -->

# Code Challenge Purpose

> As a Consultants and Software Engineers, we see ourselves as teachers and mentors to our clients.  This is the perspective you should take while completing this assignment

In your self-assessment you indicated that you have experience with bash scripting and NGINX. Our Java consulting and dev work may get slow at times &mdash; there may be a lot of down time doing nothing. As such, having additional skills are competitive differentiators during the interview process.


The purpose of this code challenge is to assess your skills with respect to bash scripting, researching information, commenting & documenting code as well as email etiquette & communication skills. 

A firm understanding and appreciation of NGINX is required to successfully complete this challenge.

<!-- Being a remote worker has its unique challenger. However, communication as a potential remote employee with our firm. 
 -->

# Evaluation and Judging Criteria

1. Successfully completing all components of the code challenge 
1. Quality and readability of your code (as well as comments)
1. Insights as shared in your project's README.md and within your code's comments
1. Overall attention to detail


# Project Background

Your client is a fictional company called Graviton Sciences.  They are a La Jolla, CA based materials science start-up that researches, synthesizes, and manufactures novel metallic alloys, thin films, and dielectrics.  These engineered materials are used in the power generation, aerospace industries, and SatCom.

Graviton has 5 customers which include SpaceX, NRG, Inmarsat, Nasa's JPL, and Sandia National Labs   

In 2021, their IT guy, Jim Dev, created 5 shell scripts that configured, tested, and deployed a proof-of-concept for Graviton's [extranet](https://www.lumapps.com/modern-intranet/difference-intranet-vs-extranet/).  The extranet was built on the LEMP (Linux, NGINX, MySQL, and PHP) stack. It was hosted in-house, at the company's headquarters in La Jolla. Its purpose was to manage secure communication amongst Graviton's remote employees and customers. 


## Graviton's Previous Server Build
To build the server, Jim ran his scripts manually and sequentially. For example:

* Script 1: installed Ubuntu 20.04, created admin users, locked down SSH, and configured iptables
    * Test Result: Server is up and running. SSH passowordless login works
    * Ports Open:  random ports for SSH and MySQL 8.0 as well as 80 and 443
* Script 2: installed NGINX. PHP 7.4 and configured server blocks  
    * Test Result 1: NGINX is up and running. Accessible on port 80
    * Test Result 2: PHP info page succefully renders
* Script 3: Installed and configured MySQL. Creates dummy database  
    * Test Result: MySQL up and Running. Is accessible on a random port via SSH tunneling using MySQL Workbench 
* Script 4: 
    1. Installs and configure's Maxmind's GeoIP module for NGINX
    1. Customizes `/etc/nginx/nginx.conf` to work with GeoIP
        * Access to intranet only permitted for 5-eyes countries
    1. Customizes access log files within the server block
        * Test Result: Website is only accessible by 5-eyes countries, web logs show detailed geo information on website visitors 
* Script 5: Uses Let’s Encrypt certbot to install SSL certs, performs renewal dry-run, and further "tunes" `/etc/nginx/nginx.conf`for  security and performance
    * Test Result: SSL is working, site is only accesible via SSL

Unfortunately, Jim only saved the scripts on his personal laptop.  In January 31, 2022 he accidentally deleted the development directly on his laptop.  Most of  his scripts were permanently lost. 

By the end of 2022, Jim has since managed to recreate scripts 1 through 3. as highlighted above. These scripts programmatically install and configure Ubuntu **22.04**, SSH, admin users, iptables, MySQL 8, and PHP **8.1**
 

 This contents of his `/etc/nginx/sites-available/graviton.com` server block file:

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

1. Implement an NGINX rate limiter using some or all of these directives: limit_req_zone, limit_req, and limit_req_status

1. Block all access to:
    * "." directories (e.g., .git) except .well-known

    * Hidden files (e.g., any file beginning with ".". Examples include `.htaccess`, `.htpasswd`, `.DS_Store`)

    * A few sensitive extensions specified here: https://docs.nginx.com/nginx-app-protect-waf/configuration-guide/configuration#disallowed-file-types 

1. Install and then use Fail2Ban to block any IP address after it attempts to access an unauthorized file or directory mentioned above 
    * Jail time should be for 48 hours

1. Install Maxmind's GeoIp module and database.  

    * Programmatically create a cron job to refresh the DB once a month.  
    
    * Whitelist or Blacklist countries:

       * In your bash script, offer the end-user the option to either "Deny ALL Except" and "Allow ALL Except" website visitors based on the following criteria:

	        a. "Deny ALL Except": Use GeoIP to only allow the 5-Eyes countries to access the website

	        b. "Allow ALL Except": Use GeoIP to allow all countries except China, Russia, North Korea, and Brazil

	<!-- **NB:** "Deny ALL Except" and "Allow ALL except" or not technical NGINX terms, but convey the needed functionality in "plain English" -->

1. Create custom access logs to access rich data provided by the GeoIP module. Include visitor's City, State, ZipCode, Country, and ISP
    * Make sure to place access log withing the custom server block

1. Create a script to auto install CertBot (LetsEncrypt SSL Certificates). Ask the user to enter their domain name.  Programmatically perform a renewal dry-run to make sure the setup is correct

    * programmatically update Graviton's existing server-block file to support SSL 

1. Update default NGINX config file `/etc/nginx/nginx.conf` to work with the GeoIP module. Also: 

   * [Mozilla](https://ssl-config.mozilla.org) and [DigitalOcean](https://nginxconfig.io) have templates for creating a security-oriented `nginx.conf` files. Choose and/or customize the best one to use. Additionally:
   
   * Be sure to import any custom modules and configurations you created above

   * Feel free to use tips from other online tutorials and references


## Required

1. Within your scripts, use comments to cite the source (e.g., URL of online Tutorial or StackOverflow solution) of the commands and directives you used to solve each of the above challenges.  

1. Your scripts should be human readable and self-documenting
    * Use non-cryptic variable names
    * Use comments liberally, but wisely, to explain what each section of your scripts do.  

1. Be sure to use `echo`s (or `print`s) to show your end-user what is happening while the script is running

1. Modularize (via importable, custom config files) the custom NGINX features and directives you add.  For example, use bash to create a zone file for rate limiting.  Then import this file into the `location` portion of the server block file

1. There are numerous "arm-chair" experts that have their NGINX tuning tips. Many of their suggestions are outdated, questionable, or just outright false because they are based on a single data-point. In your README address the following:
    * Which "tried and true" suggestions would you recommend Graviton use to better tune NGINX for performance, scalability and reliability?
        * Which have you implemented in your modified `nginx.conf` and server block files? 
        * Which tutorials have high quality NGINX performance tuning tips?
    * Which tutorials did you find with questionable or outdated `nginx.conf` performance suggestions?  


<!--
## Optional Bonus Questions:	

 1. This [gentleman](https://haydenjames.io/nginx-tuning-tips-tls-ssl-https-ttfb-latency) makes some assertions on NGINX tuning. Parts of his recommendations are opposite of what the Mozilla and DigitalOcean represent as best practices.  Which of his claims are true and which ones (if any) are assumptions based on a single data-point? -->

## Due Date

Email us within 24 hours of receiving this code challenge with a reasonable due date. If you later find you need more time to complete, you will not be penalized.  However, please let us know at least 48 hours before the deadline that you'll need an extension.

## Optional, Not Required

Although not required, you're welcomed to do any or all of the following:

1. Completing the project early
1. Found typos or something is unclear in this README?  Send us an [email](java@treebright.com) so we may fix it!
1. Going beyond the minimum of the [Code Challenge: Your Assignment](#code-challenge-your-assignment) or providing suggestion of what else should be done
1. Committing regular Work-In-Progress (WIP) drafts and updates to this repo before the deadline
1. Sending us status emails on how your progress is coming along


# Pro-tips:

1. As a consultants and software engineers, we see ourselves as teachers and mentors to our clients.  This is the perspective you should take while completing this assignment.  Your scripts should be well commented. They should serve as learning aids for Graviton and Jim

1. You're encouraged to use online tutorials and StackOverlow solutions for best practices and guidance

1. We encourage you to email us if you have questions, need for clarifications, or spot any typos in this assignment. We will respond within 24 hours after 5 PM EST.

1. You can find up to 4 online tutorials of each code challenge topic. These tutorials will take no more than 15 minutes for an experienced NGINX SysAdmin  to review and understand.

1. Jim Dev was able to recover the URLs of the tutorials he used to build his original scripts via his Google "My Activity". Email us at java@treebright.com if you feel this information will be useful to you.  Be sure to pre-pend the subject of any emails with your name (and candidate ID if applicable). For example:  `John Smith - 25: Question on Tuning`

1. If you need more time to complete the project, please reach out to us and request an extension. Our email is java@treebright.com

1. If you feel we are not a good fit for you and prefer to dropout of the interview process, please send an email to java@treebright.com. Feel free to let us know how we can improve the interview process
