# OverTheWire Bandit Write-ups
My solutions and write-ups for the OverTheWire Bandit wargame (levels 0–33).

## Level 0 → Level 1
To solve Level 0, I connected to the server using SSH. After logging in, I searched for the readme file using `ls` and read its contents with `cat`. I found the password inside the file, which i used for the next level.

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

## Level 1 → Level 2
For Level 1, I had to figure out how to open a file named `-`. I solved it by researching how dashed filenames work and then using the following command:

```bash
cat ./-
```

## Level 2 → Level 3
For Level 2, the challenge was to read a file with both a dash and spaces in its name. After researching how to handle filenames like this, I used the following command:

```bash
cat -- "filename"
```

## Level 3 → Level 4
For Level 3, I needed to figure out how to find the hidden file. After running `ls` and seeing nothing, I used the following command:

`ls -a`  

After finding the file, I noticed its name contained dots and dashes, so to cat the file, I used the command: 

```bash
cat -- "...File-name"
```

## Level 4 → Level 5
For level 4, The password was hidden among several files, where most files contained binary data. I used the find command with the file command:  

```bash
find . -type f -exec file {} +  
```

to identify the file containing human-readable text, after finding the file i used `cat` to reveal the password.


## Level 5 → Level 6
For this level, I was given three properties of the password file: it had to be human-readable, exactly 1033 bytes in size, and not executable. 

following that, i used the command:   

```bash
find . -size 1033c
```

and got the path of the file which i used this command next: 

```bash
cat ./maybehere07/.file2
```

to finally get the password for the next level. 


## Level 6 → Level 7 
For this level, I was given three properties: the file was owned by user bandit7, owned by group bandit6, and was 33 bytes in size. The password was also in an unknown place somewhere on the server. 

I used this command: 

```bash
find / -user bandit7 -size 33c -group bandit6 2>/dev/null 
```

doing this command gave me the path of the file containing the password, so i continued by doing: 

```bash
cat /var/lib/dpkg/info/bandit7.password 
```

to get the password itself for level 7.


## Level 7 → Level 8
for this level the password was stored in a large list containing many word-password pairs, it was a random word and a random password next to it but the password i was looking for was next to the word "millionth"

so i used the command:  

```bash
grep "millionth" data.txt 
```

to get the password for the next level

## level 8 → level 9
The password was stored in data.txt and was the only line that appeared once. I used:  

```bash
sort data.txt | uniq -u
```

to find the unique line containing the password.

## level 9 → level 10
for this level i was told "The password for the next level is stored in the file data.txt in one of the few human-readable strings, preceded by several ‘=’ characters." being told this i used the command: 

```bash
strings data.txt -n 20
```

The command extracted human-readable strings from the file, revealing the string containing the password.

## level 10 → level 11
for this level the password was stored in the file data.txt, which contains base-64 encoded data so to decode it i used the command:  

```bash
base64 -d data.txt
```

which gave me the password.

## level 11 → level 12
In Level 11, the password for the next level is stored in data.txt, where all lowercase (a–z) and uppercase (A–Z) letters have been rotated by 13 positions.

I decrypted data.txt using the command: 

```bash
tr 'A-Za-z' 'N-ZA-Mn-za-m' < data.txt
```

This reverses the 13-position letter shift (ROT13) and reveals the password for Level 12.


## level 12 → level 13
For Bandit Level 12, the challenge required unpacking a heavily nested archive containing a mix of gzip, bzip2, and tar archives. I inspected each layer using the `file` command, added the correct file extension when needed, and extracted each layer using `gunzip`, `bunzip2`, and `tar -xf`, I successfully worked through the repetitive process until the final ASCII text file containing the password was revealed.


## level 13 → level 14
For Bandit Level 13, the challenge required using the provided SSH private key to log in as bandit14 instead of using a password.
At first, I kept getting authentication errors because I was trying to connect from within the Bandit server. I spent some time troubleshooting before realizing that the connection had to be made from my local machine instead. 
After copying the private key locally, setting the correct permissions with `chmod 400`, and connecting with:

```bash
ssh -i /home/kali/sshkey.private -p 2220 bandit14@bandit.labs.overthewire.org 
```

i was able to log in as bandit14.


## level 14 → level 15
For Bandit Level 14, the challenge required sending the current level's password to a service running on localhost port 30000. I first spent some time investigating an SSH public key I came across, but later realized it was only related to SSH authentication and was not part of the solution. After understanding that localhost referred to the Bandit server itself, I used `netcat` to connect to port 30000.

The next step was finding the current level's password, which was stored in `/etc/bandit_pass/bandit14`. I first tried accessing the file while still logged in as bandit13, because the previous level's description pointed me toward that location, but I received a permission denied error. I realized that only bandit14 had permission to read the file, so I used the SSH key from the previous level to log in as bandit14 and retrieve the password.

After sending the password through `netcat`, the service returned the password for Level 15. This level taught me more about SSH authentication, localhost, Linux permissions and communicating with services through specific ports.


## level 15 -> level 16
For this level, the challenge required sending the current level's password to port 30001 on localhost using SSL/TLS encryption. Before starting, I researched the commands suggested in the level description to understand their purpose and see usage examples. After I understood them, I realized that `openssl s_client` was the right tool for creating a secure SSL/TLS connection.

Using an example I found, I modified the command to fit my goal by replacing the host and port with `localhost:30001`. When I first connected, a lot of connection information was displayed, so I added the `-quiet` flag to suppress the extra output and make it easier to interact with. After establishing the connection, I submitted level 15's password and received the password for the next level. 

The command I used was: 

```bash
openssl s_client -quiet -connect localhost:30001
```


## level 16 → level 17

For this level, the challenge required finding the correct port between `31000` and `32000` that was running an SSL/TLS service and submitting the current level's password to it. Before starting, I researched the suggested commands again to refresh my understanding of how they worked.

Since the challenge required finding a specific port in a range, I used an `nmap` scan on `localhost`:

```bash
nmap localhost -p 31000-32000
```

The scan showed five open ports, so I tested each one using:

```bash
openssl s_client -connect localhost:<port>
```

Some of the ports failed to establish an SSL/TLS connection. Two of the ports accepted the SSL/TLS connection, but when I sent the current password, both returned a `KEYUPDATE` message.

To figure out which one was the correct service, I tested both connections further. On one of them, when I sent random text, the same text was returned back to me, showing that it was not the service I was looking for. On the other port, I received the message:

```text
Wrong! Please enter the correct current password.
```

This showed me that I had found the correct port because the service was actually checking the password, but there was still an issue with how the SSL/TLS connection was handling the exchange.

To troubleshoot this, I researched the `openssl s_client` documentation using:

```bash
man openssl-s_client
```

From the manual, I learned about the `-ign_eof` option, which keeps the TLS session open after receiving an EOF signal. I realized that this was needed to keep the connection active long enough for the password exchange to complete.

I then used:

```bash
openssl s_client -connect localhost:<port> -ign_eof
```

After sending the correct password, the service returned a private SSH key instead of a normal password. This key was required for the next level.


## Level 17 → Level 18

For this level, the challenge required finding the only line that was different between `passwords.old` and `passwords.new`. Before starting, I researched the suggested commands from the level description and learned that `diff` was the correct tool for comparing the two files.

Since I had received an SSH key from the previous level, I first retrieved the current level's password from:

```bash
/etc/bandit_pass/bandit17
```

After that, I used `diff` to compare the two files:

```bash
diff passwords.old passwords.new
```

The command displayed the only changed line between the files, which contained the password for Level 18.


## Level 18 → Level 19

For this level, the challenge required reading the password stored in the `readme` file located in the home directory. However, the `.bashrc` file had been modified to automatically log me out when connecting through SSH.

At first, I tried troubleshooting the SSH connection using the verbose flag:

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 -vvv
```

However, this did not provide any useful information. I then researched how `.bashrc` is executed during SSH logins and learned that instead of opening an interactive shell, I could execute commands directly through SSH.

After looking into SSH command execution, I tested running commands remotely by adding the command at the end of the SSH connection:

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "ls"
```

This displayed the contents of the home directory and showed that a `readme` file existed. I then used:

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat /etc/bandit_pass/bandit19"
```

The command displayed the password for the next level, allowing me to continue to Level 19.



## Level 19 → Level 20

For this level, the challenge required using a SetUID binary located in the home directory to access the password for the next level.

After logging in, I searched the directory and found an executable called:

```bash
bandit20-do
```

I first checked its permissions using:

```bash
ls -l
```

The output showed:

```text
-rwsr-x---
```

I noticed the `s` permission, which indicated that the file had the SetUID bit enabled. After researching SetUID, I learned that this means the program runs with the permissions of its owner instead of the user executing it. Since the file was owned by `bandit20`, the program could execute commands with `bandit20`'s permissions.

I then checked what type of file it was using:

```bash
file bandit20-do
```

The output showed that it was an ELF executable. I initially tried reading it using `cat`, but the output was unreadable because it was a compiled binary rather than a normal text file.

To understand what the program did, I used:

```bash
strings -n 8 bandit20-do
```

This revealed a message explaining that the program could run commands as another user:

```text
Run a command as another user.
Example: %s whoami
```

I tested this by running:

```bash
./bandit20-do whoami
```

The output confirmed that the command was being executed as `bandit20`.

Since the program could execute commands with `bandit20`'s permissions, I used it to read the password file for the next level:

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

The command returned the password for Level 20.

This level taught me how SetUID binaries work and how programs with elevated permissions can perform actions that the current user cant.


## Level 20 → Level 21

For this level, the challenge required using a SetUID binary in the home directory that connects to a specified port on localhost. The program reads a line of text from the connection and compares it with the password from the previous level. If the password is correct, it sends back the password for the next level.

Before starting, I researched the commands suggested in the level description to understand which tools could be useful. After finding the `suconnect` executable in the home directory, I inspected it using:

```bash
file suconnect
```

This showed that it was an ELF executable. I then used:

```bash
strings suconnect
```

to extract human readable text from the binary. This revealed information such as error messages and usage instructions, which helped me understand how the program works.

To get a better and clearer explanation of how to use the program, I ran it without any arguments:

```bash
./suconnect
```

This displayed the usage information:

```text
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
```

From this, I understood that I needed to create a TCP connection where `suconnect` could connect and receive the current level's password.

After some troubleshooting and researching, I realized that I needed to create a listener using `netcat`. I opened another terminal, logged into Bandit Level 20, and started a listener:

```bash
nc -l localhost 4444
```

Then, from the second terminal, I connected the SetUID program to the same port:

```bash
./suconnect 4444
```

At first, I was confused because typing into the `suconnect` terminal did not produce any output, even when testing with random text. After closing the listener, I received an error message stating that the password did not match. This helped me realize that the connection was working, but I was sending the password from the wrong side.

I then sent the Level 20 password through the `netcat` listener, and the program responded with:

```text
Read: <current password>
Password matches, sending next password
```

The password for the next level was then transmitted through the connection.

This level helped me understand how programs communicate through TCP connections, how SetUID binaries can interact with other processes, and the difference between a program connecting to a service and a listener waiting for a connection.



## Level 21 → Level 22

For this level, the challenge required investigating a program that was running automatically through `cron`, the Linux time based job scheduler. The goal was to find the cron configuration and understand what command was being executed.

First, I navigated to the cron configuration directory:

```bash
cd /etc/cron.d
```

 And I listed the available cron jobs:

```bash
ls
```

I first tried using `crontab` on the files, but I received permission errors. Instead, I inspected the files directly and found one that looked relevant:

```bash
cat cronjob_bandit22
```

The output showed:

```text
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

This showed that a script called `cronjob_bandit22.sh` was being executed automatically as the user `bandit22`.

I then inspected the script:

```bash
cat /usr/bin/cronjob_bandit22.sh
```

The script contained:

```bash
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/<file>
```

From this, I understood that the script was reading the password for `bandit22` and writing it into a temporary file.

I then read the generated file:

```bash
cat /tmp/<file>
```

This revealed the password for the next level.

This level helped me understand how cron jobs work in Linux, how scheduled tasks can execute scripts automatically, and how inspecting automated processes can reveal useful information about system behavior.



## Level 22 → Level 23

For this level, the challenge required investigating another cron job running automatically. The goal was to understand the script being executed and find where it stored the password.

I first checked the cron directory:

```bash
cd /etc/cron.d
```

After listing the files, I found a relevant cron job:

```bash
ls
```

I inspected the file I found:

```bash
cat cronjob_bandit23
```

The output showed:

```text
@reboot bandit23 /usr/bin/cronjob_bandit23.sh &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh &> /dev/null
```

This showed that the script was being executed automatically as the user `bandit23`.

I then inspected the script:

```bash
cat /usr/bin/cronjob_bandit23.sh
```

The script contained:

```bash
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

At first, I thought I needed to modify or complete something in the script. To test how it worked, I manually executed it:

```bash
bash /usr/bin/cronjob_bandit23.sh
```

The script ran successfully, but it created a file containing the password for `bandit22` instead of `bandit23`. After reviewing the script again, I realized this happened because the script used:

```bash
myname=$(whoami)
```

Since I was logged in as `bandit22`, `whoami` was `bandit22`. However, the actual cron job runs the script as `bandit23`, meaning the generated filename and password location would be different.

To calculate the filename created by the cron job, I ran the command from the script and replaced the username with `bandit23`:

```bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
```

This returned the hash


The script would therefore store the password in:

```bash
/tmp/<hash>
```

I then read the file:

```bash
cat /tmp/<hash>
```

This revealed the password for the next level.

This level helped me understand how cron jobs run with the permissions of their assigned user, how scripts can behave differently depending on the current user, and how to analyze scripts to predict their output.



## Level 23 → Level 24

For this level, the challenge required investigating another cron job running automatically. This level was different from previous ones because it required creating my own shell script that would be executed by the cron job.

I first navigated to the cron directory:

```bash
cd /etc/cron.d
```

After listing the files, I found the relevant cron job:

```bash
ls
```

I inspected it:

```bash
cat cronjob_bandit24
```

The output showed:

```text
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
```

This showed that a script called `cronjob_bandit24.sh` was being executed automatically as the user `bandit24`.

I then inspected the script:

```bash
cat /usr/bin/cronjob_bandit24.sh
```

The script contained:

```bash
#!/bin/bash

shopt -s nullglob

myname=$(whoami)

cd /var/spool/"$myname"/foo || exit
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." ] && [ "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" "./$i")"
        if [ "${owner}" = "bandit23" ] && [ -f "$i" ]; then
            timeout -s 9 60 "./$i"
        fi
        rm -rf "./$i"
    fi
done
```

After analyzing the script, I understood that it executes scripts placed inside:

```bash
/var/spool/bandit24/foo
```

and then deletes them afterward. The script only executes files owned by `bandit23`.

I then checked the permissions of the directory:

```bash
ls -ld /var/spool/bandit24/foo
```

The output showed:

```text
drwxrwx-wx
```

After researching Linux directory permissions, I understood that although I could not list the contents of the directory, I could still create files inside it because I had write and execute permissions.

My next step was creating a script that would be executed by the cron job. Before creating the final version, I tested the idea by creating a simple script that saved the output of:

```bash
whoami
```

into a temporary file to confirm that the cron job was executing my script.

After confirming that it worked, I created the final script:

```bash
touch script.sh
nano script.sh
```

The script contained:

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/41
```

I then made it executable:

```bash
chmod +x script.sh
```

After waiting for the cron job to execute the script, the file was removed as expected. I then checked the temporary file:

```bash
cat /tmp/41
```

This revealed the password for the next level.

This level helped me understand how cron jobs can execute scripts, how Linux directory permissions work, and how automated processes can perform actions with different user privileges. cool level.



## Level 24 → Level 25

For this level, the challenge required communicating with the daemon running on port `30002`. The service required two pieces of information: the current level's password and a secret 4 digit PIN code. Since there wasnt a way to retrieve the PIN directly, the solution was to brute-force all possible combinations from `0000` to `9999`.

Instead of manually sending thousands of attempts, I decided to create a script to automate the process.

I created a new script:

```bash
touch 1.sh
nano 1.sh
```

The script contained:

```bash
#!/bin/bash

password="<current-levels-pass>"

for pin in {0000..9999}; do
    echo "$password $pin"
done | nc localhost 30002
```
The | operator takes the output from the loop and sends it as input to nc, allowing all generated password/PIN combinations to be sent directly to the daemon.

The script loops through every possible 4 digit PIN combination and combines it with the current level's password. The output is then piped into `netcat`, which sends each attempt to the daemon running on port `30002`.

I then gave the script executable permissions:

```bash
chmod +x /tmp/1.sh
```

And ran it:

```bash
bash /tmp/1.sh
```

The script automatically sent all 10,000 possible PIN combinations to the daemon. When the correct PIN was sent, the daemon returned the password for the next level.

This level helped me understand the usefulness of scripting for automation, how pipes can connect the output of one command to another program, and how brute-force techniques can be automated when the search space is small enough. 9/10 level.




## Level 25 → Level 26

For this level, the challenge required figuring out why logging into `bandit26` immediately disconnected me and finding a way to escape the restricted shell.

For this level they gave me an ssh private key for bandit26, so I first copied it to my local machine and tried using it to connect to `bandit26`:

```bash
ssh -i sshkey.private -p 2220 bandit26@bandit.labs.overthewire.org
```

The authentication worked, but I was immediately disconnected. I also tried connecting with the verbose flag:

```bash
ssh -vvv -i sshkey.private -p 2220 bandit26@bandit.labs.overthewire.org
```

The output confirmed that authentication was successful, so I knew the private key was not the problem. Something else was causing the connection to close.

I also tried using SSH forced commands, but that did not work either. After some research, I checked the entry for `bandit26` in `/etc/passwd`:

```bash
cat /etc/passwd
```

I found:

```text
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
```

This showed that the login shell for `bandit26` was not `/bin/bash`, but `/usr/bin/showtext`.

I then inspected the file:

```bash
cat /usr/bin/showtext
```

It contained:

```bash
#!/bin/sh

export TERM=linux

exec more ~/text.txt
exit 0
```

This explained why I was being disconnected. Instead of starting a normal shell, the SSH connection was executing `showtext`, which launched `more` to display the `text.txt` file.

I then researched how `more` behaves when there is not enough space to display the entire file. I found that if the terminal window is small enough, `more` pauses instead of immediately finishing. This gave me an interactive `more` session that I could work with and control.

I reduced the size of my terminal window and connected to `bandit26` again. This time, instead of being immediately disconnected, I was left inside `more`.

From there, I researched ways to escape from `more` and found that `more` can be used to launch `vi`. I opened `vi` and changed its shell setting:

```vim
:set shell=/bin/bash
```

I then started the shell from `vi`:

```vim
:shell
```

This gave me access to a Bash shell as `bandit26`.

Finally, I read the password for the current level:

```bash
cat /etc/bandit_pass/bandit26
```

This revealed the password needed for level 26.

This level was probably one of the more frustrating ones for me because I initially focused on the SSH authentication instead of looking at what happened after authentication. It taught me how to investigate a user's configured shell, how programs like `more` can behave differently depending on terminal size, and how one interactive program can sometimes be used to reach another shell. messy level overall.





## Level 26 → Level 27

For this level, the goal was to use the SetUID binary in the home directory to access the password for `bandit27`.

After getting the shell from the previous level, I listed the files in the home directory:

```bash
ls
```

I found the `bandit27-do` executable. Because it was a SetUID binary owned by `bandit27`, I understood that commands executed through it would run with `bandit27`'s permissions.

I used it to read the password file:

```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```

This revealed the password for Level 27.





## Level 27 → Level 28

For this level, the challenge required cloning a Git repository from the OverTheWire server to my own machine and finding the password for the next level.

I first read through the recommended material to understand what Git was and how `git clone` worked. The instructions also made it clear that I needed to clone the repository from my local machine rather than from the OverTheWire server.

I used:

```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
```

I had to do a little research to figure out where to specify the SSH port. I also learned that the port could be included directly in the SSH URL after the hostname:

```text
:2220
```

After running the command, Git cloned the repository into a new `repo` directory on my local machine.

I entered the repository:

```bash
cd repo
```

and listed its contents:

```bash
ls
```

I found a `README` file, so I read it:

```bash
cat README
```

This revealed the password for level 28.

This level helped me understand the basics of cloning Git repositories over SSH and how Git can be used to retrieve files from a remote repository. Overall cool level.




## Level 28 → Level 29

For this level, the challenge was similar to the previous one. I had to clone another Git repository from the OverTheWire server to my local machine and find the password for the next level.

I cloned the repository using:

```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
```

After entering the repository, I found a `README.md` file and read it:

```bash
cat README.md
```

It contained:

```text
# Bandit Notes
Some notes for level29 of bandit.

## credentials

- username: bandit29
- password: xxxxxxxxxx
```

The password was hidden, so I started researching what I could do with an already cloned Git repository. I learned that Git keeps a history of previous commits, including changes made to files. So i figured i should use that to my advantage.

I first checked the commit history:

```bash
git log
```

This showed three commits:

```text
fix info leak
add missing data
initial commit of README.md
```

The commit message `fix info leak` stood out to me, so I decided to investigate the changes made in those commits.

I used:

```bash
git log -p
```

This displayed the actual changes made in each commit. The important part was this:

```text
- password: <the password>
+ password: xxxxxxxxxx
```

This revealed the password for Level 29.

This level taught me that deleting or changing information in a Git repository does not necessarily remove it from the repository's history. Even though the password was no longer visible in the current version of the README, it was still present in an earlier commit.





## Level 29 → Level 30

For this level, the challenge was similar to the previous two levels. I had to clone the Git repository to my local machine and find the password for the next level.

I cloned the repository using:

```bash
git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
```

After entering the repository, I found a `README.md` file and read it:

```bash
cat README.md
```

It contained:

```text
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: <no passwords in production!>
```

The password for the next level wasn't in the README, so I knew I needed to investigate what else I could find in the repository.

After researching what I could do with a cloned Git repository, I came across Git branches. I learned that branches are different versions of a repository that can be used for things like developing and testing changes.

I checked which branches existed using:

```bash
git branch
```

This showed:

```text
  dev
  master
* sploits-dev
```

The `*` showed that I was currently on the `sploits-dev` branch.

I switched to the `master` branch and checked the README, but it was the same as the current branch. I then switched to the `dev` branch:

```bash
git switch dev
```

After checking the README again:

```bash
cat README.md
```

I found:

```text
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: <THE PASSWORD!>
```

This revealed the password for Level 30.

This level taught me how Git branches work and how different branches can contain different versions of the same files. It also showed me that when investigating a Git repository, and I shouldn't only look at the current branch. Cool level.




## Level 30 → Level 31

For this level, the challenge was again based on investigating a Git repository. I started by cloning the repository to my local machine:

```bash
git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo
```

Inside the repository, there was only one file. When I read it, it contained:

```text
just an empty file... muahaha
```

I then tried the two techniques I had used in the previous levels. I checked the branches:

```bash
git branch
```

and the commit history:

```bash
git log
```

Neither of these gave me anything useful.

I started researching other Git commands that could be useful for investigating an already cloned repository. I tried a few commands, including:

```bash
git reflog
git stash list
git status -u
```

but none of them revealed anything useful.

While researching other things I could check in the repository, I was lucky and quickly came across Git tags. I learned that tags can be used to point to a specific commit, so I figured it was worth checking if the repository had any.
I checked for any tags using:

```bash
git tag
```

This revealed a tag called:

```text
secret
```

I then inspected the tag:

```bash
git show secret
```

The output revealed the password for Level 31.

This level taught me that when investigating a Git repository, there can be useful information in places other than the files or normal commit history. In this case, the password was hidden behind a Git tag. Nice.



## Level 31 → Level 32

For this level, the challenge was again based on a Git repository. I started by cloning the repository to my local machine:

```bash
git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo
```

After entering the repository, I found a `README.md` file containing instructions:

```bash
cat README.md
```

The contents:

```text
This time your task is to push a file to the remote repository.

Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master
```

From this, I understood that I needed to create a file called `key.txt`, put `May I come in?` inside it, and push it to the `master` branch.

I created the file and added the required content:

```bash
touch key.txt
nano key.txt
```

I then researched how to push a file to a remote git repository and learned that I first needed to add and commit the file.

I started by adding it:

```bash
git add key.txt
```

However, Git rejected it, and gave me a hint to use `-f` if I "really wanted to add the file". So I tried:

```bash
git add -f key.txt
```

This time it worked.

I then tried to create a commit:

```bash
git commit -m "need to push"
```

but Git gave me an error saying that it didn't know who I was:

```text
Author identity unknown

Please tell me who you are.
```

After researching the error, I learned that Git needs a username and email attached to commits. Since I was using a fresh Kali installation, I hadn't configured these yet.

I set them using these two commands:

```bash
git config --global user.email "emailsmtsmt@example.com"
git config --global user.name "A_name"
```

I then created the commit:

```bash
git commit -m "pushing file"
```

Finally, I pushed the commit to the `master` branch:

```bash
git push origin master
```

The remote server received and validated the file, and then revealed the password for the next level.

```text
Well done! Here is the password for the next level:
<THE PASSWORD!>
```

This level taught me the basic workflow of adding, committing, and pushing files with Git. It also helped me understand Git configuration. Interesting level.





## Level 32 → Level 33

For this level, the first thing I noticed after logging in was:

```text
WELCOME TO THE UPPERCASE SHELL
```

When I tried running normal commands, they were all rejected with a `sh: 1: LS: Permission denied` error. For example, trying:

```bash
ls
```

resulted in:

```text
sh: 1: LS: Permission denied
```

I started messing around with different inputs to see what the shell was actually doing. I tried things like quotes, backticks, ;, and a few other shell characters. Some of them gave me different errors instead of just Permission denied, which made me realize that my input was being passed to another shell and interpreted as shell syntax.

For example:

```bash
"ls
```

returned:

```text
sh: 2: Syntax error: Unterminated quoted string
```

and:

```bash
`ls
```

with a backquote substitution produced:

```text
sh: 2: Syntax error: EOF in backquote substitution
```

I kept trying different shell syntax and doing some research on how sh handles commands. Eventually I understood that the first word I entered was being treated as the command to run, but the uppercase shell was changing normal commands to uppercase before passing them to sh.

When researching what else I could try, I came across shell parameters such as $0, $1, $2, $@, and $_. I learned that $0 represents the name or path of the currently running shell or script.

This gave me the idea to try $0 because the whole point of this shell is to turn every command I type into uppercase. Since normal commands like ls were being changed to uppercase, I was getting Permission denied because LS isn't the same executable as ls.

So when trying $0, the UPPERCASE shell doesn't change it because there are no letters to uppercase, so sh receives $0, expands it to the name of the current shell (sh), and executes that shell, giving me a normal shell.

Once I had a normal shell again, I could use normal Linux commands. I did `whoami` saw that I'm bandit 33 and then I proceeded with doing `cat /etc/bandit_pass/bandit33` to get the password for level 33.

This was probably one of the more interesting levels for me because I had to actually experiment and mess around with the shell instead of just finding a file or running a command. It also helped me understand how shells interpret input and taught me more about things like shell parameters and command substitution. Very hard level overall.



## Final Thoughts 

Currently, this is the last level of Bandit. I hope these write-ups helped you or gave you some insight into how I personally solved the challenges.

I had a lot of fun solving these levels. Some were frustrating, some took a lot of trial and error, but they all helped me get more comfortable with Linux and sharpen my command line skills. 
