# Lab Report: Exploiting Credential Exposure and PATH Hijacking for Full Linux System Compromise.

## Objectives

-   To exploit a target and retrieve sensitive information.

## Background

According to information from our intelligence network, ICA is working
on a secret project. We need to find out what the project is. Once you
have the access information, send them to us. We will place a backdoor
to access the system later. You just focus on what the project is. You
will probably have to go through several layers of security. The Agency
has full confidence that you will successfully complete this mission.
Good Luck, Agent!

## Tools Used

-   Kali Linux
-   Gobuster
-   Hydra
-   Nmap
-   Searchploit
-   `https://www.base64decode.org`

## Methodology

Starting, I run an Nmap scan to determine initial attack vectors. Thus,
opened ports running vulnerable services and you can bet, I found some
just as seen in the image below.

![](media/ICA/image1.png)

I then accessed the web service running on Port 80.

![](media/ICA/image2.png)

The webpage run on qdPM 9.2. I then searched for available exploit
through searchploit![](media/ICA/image3.png)

Making an ExploitDB lookup, I found the perfect exploit to use but I did
not proceed with it.

![](media/ICA/image4.png)

I actually fuzzed with Gobuster and identified the directory /core which
contained some juicy info.

![](media/ICA/image5.png)

In the core directory, I selected config

![](media/ICA/image6.png)

Next, I saw and downloaded the databases.yml file which Exploitdb had
suggested earlier. Well, manual fuzzing still got me to my destination
lol.

![](media/ICA/image7.png)

Opening the databases.yml file got me seeing the MySQL backend database
admin username and password as seen
below.

![](media/ICA/image8.png)

I logged in to MySQL successfully with the identified
credentials.

![](media/ICA/image9.png)

I dumped the entire database and selected from the staff table the users
and their logins.

![](media/ICA/image10.png)
![](media/ICA/image11.png)

The user logins were encoded in base64, hence I used
https://www.base64decode.org/ to decode the passwords. I then created
user and password text files to facilitate dictionaries for ssh
bruteforce. I had two successful
logins.

![](media/ICA/image12.png)

I logged into travis account via ssh and identified the user flag placed
in the /home directory.

![](media/ICA/image13.png)

Having found the user flag, I was still inquisitive to look out for what
dexter had on his /home directory. I got some CLUES from a note.txt
placed in there.

![](media/ICA/image14.png)

I navigated back to travis' account to explore further. Using the
command find / -perm -u=s -type f 2\>/dev/null, I searched for all files
on the entire system that have the SUID permission bit set. I tried
viewing the contents of /opt/get_access and realized it was
gibberish.

![](media/ICA/image15.png)

I attempted to run ./opt/get_access as an executable and further run
strings on it as seen below.

![](media/ICA/image16.png)

Moving on, I entered the command **echo \'bin/sh\' \> /tmp/cat** which
created a file /tmp/cat whose content was the text **bin/sh**. I then
made the **/tmp/cat** file executable, so the system can run it as a
program. Next, I hijacked the PATH variable using the command export
PATH=\"/tmp:\$PATH which puts **/tmp** first, and the system will find
my malicious **/tmp/cat** file before the real **/bin/cat**. So, when I
run cat, it will run my fake version instead. I then proceeded to run
the ./opt/get_access to spawn a root shell.

![](media/ICA/image17.png)

The program **get_access** located in the **/opt** directory may have
special privileges thus the \"SetUID" bit that allows it to run as the
root user. At some point, the **get_access** program internally calls
the **cat** command (As seen in the strings command). Because of our
hijacked PATH, it runs **/tmp/cat** instead of **/bin/cat.**

Gaining access needed me to elevate my shell privileges for an
interactive one of which I used the python command seen in the image
below. After, I changed directories to the root folder and cat the root
flag. Viola!

![](media/ICA/image18.png)

## Reflection

This project got my thoughts up on the fact that, a system\'s security
is only as strong as its weakest configuration. The successful
compromise was as a result of the chaining of two fundamental
misconfigurations. The exposure of sensitive database credentials
provided the initial foothold, while the misconfigured SetUID binary,
which trusted the user-controlled PATH environment, served as the direct
conduit to root privileges. Ultimately, it reinforces the necessity of
rigorous system hardening, strict adherence to the principle of least
privilege, and regular audits for misplaced credentials and insecure
file permissions.
