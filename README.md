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
