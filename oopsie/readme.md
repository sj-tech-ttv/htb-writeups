## Oopsie

Scanned this machine with nmap, identified that ports 22 and 80 were open. A website was hosted on port 80 via apache. We could navigate to the website with our browser, and we didn't see anything obvious on it that would lead us to a clue. After using `curl` on the website, one of the last few lines in the document returned looked like this:

```
<script src="/cdn-cgi/login/script.js"></script>
```

I navigated to the /login page with my browser, which gave me a login page. I tried the Administrator username and password from archetype, which worked, logging me in to a page titled "Repair Management System".

We found that the user ID was a simple URL parameter, and the way the system identified which user was logged in was through a plaintext cookie containing a different unique user identifier, which was displayed on the page when I passed in that user's ID as a URL parameter. Rather than try a ton of numbers of user IDs I simply looked at the guide and it told me that the user ID I was looking for was 30. I passed 30 into the URL parameter, which gave me the other user identifier. With Firefox's dev tools, I was able to modify my user cookie to contain the ID I found. I then refreshed, and saw that I now had super-admin privelages. These privelages gave me permission to use a file upload feature on the webpage. I uploaded a PHP reverse shell script, which gave me a shell under the user www-data.

With the reverse shell (and the writeup for the box), I was able to find a file in /var/www/html/cdn-cgi called db.php. This file contained database credentials. The contents of the file are as follows:

```
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

We were then able to log into the machine proper with an SSH connection courtesy of Robert's username and password. In his home directory (/home/robert) we found the user flag in user.txt. After that we enumerated Robert's permissions. We can see that robert is a member of two groups: users and "bugtracker". We can see that bugtracker is a group used to execute a command of the same name, which has a SUID bit set so it always executes as root. It uses `cat` to read bug reports that match an ID provided by the user. To leverage the fact that this tool runs as root, we can create our own `cat` and add it to the beginning of our $PATH variable so the program finds ours and executes it first. For the purposes of this exercise, our `cat` was a file containing `/bin/bash`. We ran `bugtracker`, entered some arbitrary thing, and it dropped us to a shell. With this shell, we were able to find the machine flag in /root/root.txt.

Inside root's folder, we see a .config folder, which contains a FileZilla config file with the credentials ftpuser / mc@F1l3ZilL4

