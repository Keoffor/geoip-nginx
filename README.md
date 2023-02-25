
# Navigation Menu

<!-- TOC -->

- [Code Challenge Purpose: NGINX, Bash/Shell Scripting, and SysAdmin](#code-challenge-purpose-nginx-bashshell-scripting-and-sysadmin)
- [Evaluation and Judging Criteria](#evaluation-and-judging-criteria)
- [Project Background](#project-background)
    - [Graviton's Previous Server Build](#gravitons-previous-server-build)
- [Code Challenge: Your Assignment](#code-challenge-your-assignment)
    - [Required](#required)
    - [Due Date](#due-date)
    - [Optional, Not Required](#optional-not-required)
- [Pro-tips: Things to Keep in Mind For this Code Challenge](#pro-tips-things-to-keep-in-mind-for-this-code-challenge)
- [Pro-Tip: How To Clone a Private Repo](#pro-tip-how-to-clone-a-private-repo)

<!-- /TOC -->

# Code Challenge Purpose: NGINX, Bash/Shell Scripting, and SysAdmin

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

Your client is a *fictional* company called ***Graviton Sciences Corp***.  They are a [La Jolla, CA](https://www.google.com/search?q=La+Jolla%2C+CA) based materials science start-up.  Graviton researches, synthesizes, and manufactures novel metallic alloys, thin films, and dielectrics.  These engineered materials are used in the power generation, aerospace, and Satellite Communications (SatCom) industries.

Graviton has 5 customers which include SpaceX, NRG, Inmarsat, Nasa's Jet Propulsion Laboratory (JPL), and Sandia National Labs.  Given the sensitive nature of their work, Graviton no longer maintains a public facing website.

In 2021, their IT guy, Jim Dev, created 5 shell scripts that configured, tested, and deployed a proof-of-concept for Graviton's [extranet](https://www.lumapps.com/modern-intranet/difference-intranet-vs-extranet/).  The extranet was built on the LEMP (Linux, NGINX, MySQL, and PHP) stack. It was hosted in-house, at the company's headquarters in La Jolla. Its purpose was to manage secure communication amongst Graviton's remote employees and customers. 


## Graviton's Previous Server Build
To build the server, Jim ran his scripts manually and sequentially. For example:

* Script 1: Installed Ubuntu 20.04, created admin users, locked down SSH, and configured iptables
    * Test Result: Server is up and running. SSH passwordless login works
    * Ports Open:  random ports for SSH and MySQL 8.0 as well as 80 and 443
* Script 2: Installed NGINX, PHP 7.4, and configured server blocks  
    * Test Result 1: NGINX is up and running. Accessible on port 80
    * Test Result 2: PHP info page successfully renders
* Script 3: Installed + configured MySQL, and created MySQL database with test data  
    * Test Result: MySQL up and Running. Is accessible on a random port via SSH tunneling using MySQL Workbench 
* Script 4: 
    1. Installed and configured Maxmind's GeoIP module for NGINX
    1. Customized `/etc/nginx/nginx.conf` to work with GeoIP
        * Access to extranet only permitted for 5-eyes countries
    1. Customized access log file format within the server block
        * Test Result: Website is only accessible by 5-eyes countries, web logs show detailed geo information on website visitors 
* Script 5: Used Let’s Encrypt certbot to install SSL certs, performed renewal dry-run, and further "tuned" `/etc/nginx/nginx.conf` for security and performance
    * Test Result: SSL is working, site is only accessible via SSL

Unfortunately, Jim only saved the scripts on his personal laptop.  In January 31, 2022 he accidentally deleted the development dfolder on his Windows 10 laptop.  Most of  his scripts were permanently lost. 

By the end of 2022, Jim has since managed to recreate scripts 1 through 3, as highlighted above. These scripts programmatically install and configure Ubuntu **22.04**, SSH, admin users, iptables, MySQL 8, and PHP **8.1**.
 

 This contents of his `/etc/nginx/sites-available/graviton.com` server block file from Script 2:

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

1. Install Maxmind's GeoIp module and database  

    * Programmatically create a cron job to refresh the DB once a month.  
    
    * Whitelist or Blacklist countries:

       * In your bash script, offer the end-user the option to either "Deny ALL Except" and "Allow ALL Except" website visitors based on the following criteria:

	        a. "Deny ALL Except": Use GeoIP to only allow the 5-Eyes countries to access the website

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

1. Modularize (via importable, custom config files) the custom NGINX features and directives you add.  For example, you script can create a separate "zone file" for rate limiting.  Then import this file into the `location` portion of the server block file

1. There are numerous "arm-chair" experts that have their NGINX tuning tips. Many of their suggestions are outdated, questionable, or just outright false because they are based on a single data-point. In your README address the following:
    * Which "tried and true" suggestions would you recommend Graviton use to better tune NGINX for performance, scalability, and reliability?
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

1. Complete the project before your deadline
1. Sending us an [email](java@treebright.com) if you found typos or something that unclear in this README
1. Going beyond the minimum of the [Code Challenge: Your Assignment](#code-challenge-your-assignment) section
1. Providing suggestions on what should be added (or removed) from future iterations of the [Code Challenge: Your Assignment](#code-challenge-your-assignment) section
1. Committing regular Work-In-Progress (WIP) drafts and updates to this repo before the deadline
1. Sending us status emails on how your progress is coming along


# Pro-tips: Things to Keep in Mind For this Code Challenge

1. As a Consultants and Software Engineers, we see ourselves as teachers and mentors to our clients.  This is the perspective you should take while completing this assignment.  Your scripts should be well commented. They should serve as learning aids for Graviton and Jim

1. You are strongly encouraged to use online tutorials and StackOverlow solutions for best practices and guidance

1. We encourage you to email us if you have questions, need clarifications, or spot any typos in this assignment. We will respond within 24 hours. Typically after 5 PM EST.

1. You can find up to 4 online tutorials on each code challenge topic. These tutorials will take about 15 minutes for an experienced NGINX SysAdmin  to review and understand.

1. Jim Dev was able to recover the URLs of the tutorials he used to build his original scripts via his Google "My Activity". Email us at java@treebright.com if you feel this information will be useful to you.  Be sure to pre-pend the subject of any emails with your name (and candidate ID if applicable). For example:  `John Smith - 25: Question on Tuning`

1. If you need more time to complete the project, please reach out to us and request an extension. Our email is java@treebright.com

1. If you feel we are not a good fit for you and prefer to dropout of the interview process, please send an email to java@treebright.com. Feel free to let us know how we should improve the interview process

# Pro-Tip: How To Clone a Private Repo

Although, forking is not permitted on private repos, you have been given write access to this repo.  You may clone this repo locally and push your updates directly. If you use GitHub from the command line this is how you would clone:

**NB:** The repo is automatically tagged as *origin* when you clone it!

**Cloning via HTTPS (entire repo and all branches):**

`git clone https://YOUR_GITHUB_USERNAME:YOUR_GITHUB_TOKEN@github.com/treebright/candidate-id-2.git`

**Cloning  via SSH (entire repo and all branches):**

`git clone git@github.com:treebright/john-smith.git`

**Cloning a specific branch via HTTPS:**

**NB:** The following command should be on a single line. `SPECIFIC_BRANCH_NAME` and `https://...` are only separated by a single space.

`git clone -b SPECIFIC_BRANCH_NAME https://YOUR_GITHUB_USERNAME:YOUR_GITHUB_TOKEN@github.com/treebright/candidate-id-2.git`

<!-- After cloning create a new branch locally, make your edits, stage, and commit locally. Then push your new local branch to the repo. -->


**After cloning:** create a new branch locally (or just overwrite the master branch), make your edits, stage, and commit locally. Then push your changes to this repo. <!-- new local branch to the repo. -->
