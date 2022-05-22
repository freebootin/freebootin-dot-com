---
title: Over The Wire Bandit Writeup
data: 05/18/22
keywords: [wargames]
---
## Preface
These are my writeups for the [Bandit](https://overthewire.org/wargames/bandit) wargame. This does contain spoilers, but not passwords.

---
>
> SPOILERS AHEAD 
>
---

## Level 0

You are supplied with a username `bandit0`, a host `bandit.labs.overthewire.org` a port number `2220`, and a password `bandit0`.

- First `ssh` into the system using `ssh bandit0@bandit.labs.overthewire.org -p 2220`. Enter the password when prompted to login.
- Read `readme` using `cat readme`.

---

## Level 1

The password for the next level is stored in a file called `-` located in the home directory.

You cannot open `-` directly with `cat` like in level0.

- To open `-` use a relative path `./-`. 
- Use `cat ./-`.
- Copy the password and logout.

---

## Level 2

The password for the next level is stored in a file called `spaces in this filename` located in the home directory. 

This is fundimentally the same problem as level 1. This time surround the filename in quotes. `Bash` autocompletion will actually do this for you.

- Login with `ssh`.
- Read `spaces in this filename` using `cat "spaces in this filename"`.
- Copy password and logout.

---

## Level 3

The password for the next level is stored in a hidden file in the `inhere` directory.

- `ssh` into the server.
- `cd inhere` to get into the inhere directory.
- You can list out all files including hidden files using `ls -a`. This shows the file `.hidden`.
- `cat .hidden` gets you the password.

---

## Level 4

The password for the next level is stored in the only human-readable file in the `inhere` directory.

- Move into the `inhere` directory with `cd inhere`.
- `ls` shows 10 files in `inhere`.
- `file ./-file00` shows that `-file00` contains `data`, which is not human readable.
- Using `for file in *; do file ./"${file}"; done` I can run the `file` command on all the files in `inhere`. 
- They all contain `data` except `-file07` which contains `ASCII Text` which should be human-readable.
- `cat ./-file07` nets me the password.

---

## Level 5

The password for the next level is stored in a file somewhere under the `inhere` directory and has all of the following properties:

  human-readable
  1033 bytes in size
  not executable

- `ssh` into the server.
- `cd inhere` to get into the `inhere` directory.
- `ls -a` shows 18 directories, and `ls`ing one shows 9 files.
- Checking every file by hand would take awhile. Instead I used the `find` command.
- Using `find -type f -size 1033c ! -executable` returns one file `maybehere07/.file2`, and it has the password in it.

`find -type f -size 1033c ! -executable` breaks down to:
- `-type f` look for a file.
- `-size 1033c` it should be 1033 bytes in size.
- `! -executable` file should NOT(!) be executable.

---

## Level 6

The password for the next level is stored somewhere on the server and has all of the following properties:
  owned by user bandit7
  owned by group bandit6
  33 bytes in size

This is very similar to the last level.

- `ssh` into the server.
- `ls -a` shows no files or directories in my home directory. Password can't be here.
- `cd ..` to move up a directory into `/home/`.
- To search all these I used `find -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null`. The `2>/dev/null` sends any error messages into the void. This gets rid of any `Permission failed` messages.
- `/home/` does not contain the file. Move up one more `cd ..`.
- Running my `find` command in `/` yields `./var/lib/dpkg/info/bandit7.password`, which contains the password.

---

## Level 7

The password for the next level is stored in the file data.txt next to the word millionth.

- `ssh` into the server.
- `ls` shows that `data.txt` is in my home directory.
- Use `grep millionth data.txt` searches the file for any lines containing 'millionth' and returns the password.

---

## Level 8

The password for the next level is stored in the file data.txt and is the only line of text that occurs only once.

- `ssh` into the server.
- `ls` shows `data.txt` is in my home directory.
- Looking at `data.txt` it contains thousands of possible passwords.
- To sort through them I used `sort data.txt | uniq -u`. This outputs the only unique line in the file, the password.

---

## Level 9

The password for the next level is stored in the file data.txt in one of the few human-readable strings, preceded by several '=' characters.

- `ssh` into the server.
- `ls` shows `data.txt` is in my home directory.
- The `strings` utility will output all the ascii strings it finds in a file.
- Using `strings data.txt | grep ===` gets me all the strings with at least 3 '='s in them. This yields the password.

---

## Level 10

The password for the next level is stored in the file data.txt, which contains base64 encoded data.

- `ssh` into the server.
- `data.txt` is in the home directory.
- Use `base64 --decode data.txt` to decode `data.txt`, giving the password.

---

## Level 11

The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

- `ssh` into the server.
- `data.txt` is in the home directory.
- Inspecting the contents shows gibberish.  The level goal tells you how to read it though. You could manually move every character 13 positions or you could use `tr`, a find an replace utility.
- Use `tr a-mA-Mn-zN-Z n-zN-Za-mA-M < data.txt` to "decrypt" the password.  The wierd string after `tr` is the transform.  It tells `tr` that it should take all charactor inputs in the range a-m and change it to an equivilent position in the range n-z, and A-M to N-Z, etc.

---

## Level 12

The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed.

This level was more annoying than difficult. 

- `ssh` into the server.
- `data.txt` is found in the home directory.
- `data.txt` is a hex dump of another file.  To get the original file back you need to 'un-dump' it. This is done with `xxd -r data.txt` the `-r` option reverses the dump. 
- Reverse the dump and put it in a new file `data`.  Running `file data` shows that this is a `gzip` archive.
- What follows next is repeatedly unzipping archives. There are three kinds used `gzip`, `bzip2` and `tar`. After each is uncompressed use `file` to see which command you need next. 
- Commands used: `gunzip <file>` for gzip, `bzip2 -d <file>`, for bzip2, and `tar -xf <file>` for tarballs.
- There are 9 file in total to decompress before finding the password.

---

## Level 13

The password for the next level is stored in /etc/bandit_pass/bandit14 and can only be read by user bandit14.

- `ssh` into the server.
- In the home directory you find `sshkey.private`. This is a private ssh key that you can use to access the bandit14 user on the current machine.
- Using `ssh bandit14@localhost -i sshkey.private` you can log on as bandit14 and get the password from `/etc/bandit_pass/bandit14`. Here localhost is the name of your current machine (the bandit server) and `-i` points `ssh` to what key file you want it to use.

---

## Level 14

The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

- `ssh` into the server.
- To send something to a port on a host the easiest way is through `netcat` or `nc`.  `netcat` opens a TCP or UDP connection between you and a host:port, which you can then use to send data and text.
- Using `nc localhost 30000` you can open a connection to port 30000, then send the password from the previous level and a server on port 30000 will send you the next password.

---

## Level 15

The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL encryption.

This is a lot like level 14, except a different client program is used.

- `ssh` into the server.
- Trying to use `nc` doesn't work as `nc` does not implement SSL encryption. 
- Browsing through the `man` pages of the suggested commands, it looks like `s_client` should be able to replicate `nc`'s function, but with encryption.
- Using `openssl s_client -connect localhost:30001` gets me the same function that `nc` got for me in the last level.  Enter the password for the last level and the server gives you the password for the next.

---

## Level 16

The credentials for the next level can be retieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don't. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

- `ssh` into the server.
- Now we have to find which ports from 31k to 32k are open. We can do that with `nmap` using `nmap -PS localhost -p 31000-32000`. This gave me ports 31046, 31518, 31691, 31790, and 31960. 
- We can use the same command from the last level to try to access each of these ports using SSL `openssl s_client -connect localhost:port`
- This can be scripted using `for port in 31046 31518 31691 31790 31960; do openssl s_client -connect localhost:"${port}"; done`. If the port doesn't speak SSL the connection will be refused and the script will try the next port.  When a port does connect with SSL it will prompt you to enter a password. Then it gives you a response and closes the connection.  Keep entering the password until a port gives you a SSL certificate.

---

## Level 17

There are 2 files in the home directory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new.

- `ssh` into the server.
- We can use `diff` to view differences between two files.
- Use `diff passwords.old passwords.new`, this shows us the password from passwords.new.

---

## Level 18

The password for the next level is stored in a file **readme** in the homedirectory. Unfortunately, someone has modified **.bashrc** to log you out when you log in with SSH.

- `ssh` into the server...get bumped out.
- Turns out `ssh` can run commands remotely: `ssh bandit18@bandit.labs.overthewire.org -p 2220 [command]`.
- Knowing this, and knowing that **readme** is in the home directory we can use `ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme` to get the password.

---

## Level 19

To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.

- `ssh` into the server.
- Running the `bandit20-do` binary tells you that it runs a command as the bandit20 user.
- Using this binary we can view the password. `bandit20-do cat /etc/bandit_pass/bandit20`.

---

## Level 20

There is a setuid binary in the home directory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

