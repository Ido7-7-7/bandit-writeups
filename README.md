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


## level 14 -> level 15
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


## level 16 -> level 17

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














