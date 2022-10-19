---
title: Leviathan Wargame
date: 10/15/22
keywords: [wargame hacking]
---
## Leviathan

From the wargame [Leviathan](https://overthewire.org/wargames/leviathan).

### Level 0

Login credentials are given to you to start. After logging I used `ls -a` to see if anything interesting was in the home directory. There was a directory called `.backup` which contained a file `bookmarks.html`. I opened `bookmarks.html` in vim and searched for "leviathan". This turned up a bookmark entry that contained the password for level 1.

### Level 1

After logging in I ran `ls -al` to see what's in the home directory. The only interesting thing was a suid binary called `check`. When I ran `check` it prompted me for a password, and responded with "Sorry, wrong password" when I entered something random.

Now `ls` showed that this binary is owned by leviathan2, so if I can get the password then I can probably use it to get the next password.

First I ran `strings` on `check` to see if there are any obvious strings that might be the password. I found "love" and "secrf". Neither of these worked so next ran `hexdump -C ./check` and scrolled around to see if anything else jumped out at me. Again I found "love" and "secrf", but near these two where also "sex"and "god". I didn't find these  with `strings` because it defaults to displaying strings that are at least 4 characters long.

Running `check` again, this time entering "sex" got a me a shell (/bin/sh appears near the password section in the hex dump so its probably that). A quick `whoami` shows that I am now leviathan2. Next I ran `cat /etc/leviathan_pass/leviathan2` which gives me the next password. 

### Level 2

Logging in and running `ls -al` I found an suid executable named `printfile`. I tried the obvious `./printfile /etc/leviathan_pass/leviathan3` and got a response, "You cant have that file...". So, I tried `printfile` on itself and my password file `/etc/leviathan_pass/leviathan2`. When used on itself it dumps the ascii form of the binary, and when used on my password file I get a permission error. That makes sense as `printfile` is not owned by me but by `leviathan3` so they shouldn't be able to view my password.

At this point I was stumped for the rest of the day. I tried running `printfile` through `gdb` and some of the other analysis/debugging tools supplied by the enviroment but I couldn't find anything. After searching for other tools I came across `strace` and `ltrace`. These tools can be used to check what system calls (strace) and library calls (ltrace) a program is calling.

Running ltrace on printfile looking at the password file gave me this:
`
leviathan2@gibson:/etc/leviathan_pass$ ltrace ~/printfile /etc/leviathan_pass/leviathan3
__libc_start_main(0x80491e6, 2, 0xffffd584, 0 <unfinished ...>
access("/etc/leviathan_pass/leviathan3", 4)                              = -1
puts("You cant have that file..."You cant have that file...
)                                       = 27
+++ exited (status 1) +++
`

Checking the man page for the `access` call tells you that a return value of -1 means that the process does not have permission to access the file in the first argument. This seems odd as the suid binary should have access to this file as it is run as the user leviathan3. Well, further reading into `access` shows that `access` check the permissions of the *user* running the binary, not the binary itself. So that seems to be the problem.

Next I ran ltrace on printfile on itself:

`
leviathan2@gibson:/etc/leviathan_pass$ ltrace ~/printfile ~/printfile
__libc_start_main(0x80491e6, 2, 0xffffd584, 0 <unfinished ...>
access("/home/leviathan2/printfile", 4)                                  = 0
snprintf("/bin/cat /home/leviathan2/printf"..., 511, "/bin/cat %s", "/home/leviathan2/printfile") = 35
geteuid()                                                                = 12002
geteuid()                                                                = 12002
setreuid(12002, 12002)                                                   = 0
system("/bin/cat /home/leviathan2/printf"...ELF4H64

---Output of cat omitted for clarity---

--- SIGCHLD (Child exited) ---
<... system resumed> )                                                   = 0
+++ exited (status 0) +++
`

Now here you can see that `access` has returned a 0, meaning access granted. Now I can also see the `system` command being executed. This tells me exactly what command the program is running, and it looks like it is passing the string created by `snprintf` earlier.

If you look at the man page for `cat` you see that it can take any number of files as arguments and it will display them all concatenated together, and to do this it has to open both files. Now even though `printfile` is just passing a string to `cat`, that string is only made up of the first argument to `printfile`, any other arguments are discarded. One cool thing, and this took a long while to figure out, if you pass a file to `printfile` that contains a space, then it will try to call `cat` on both files. 

Here I passed `printfile` another file I created called `dog bird` that my user owns. You can see `access` returns 0, allowing me to continue then you can see it runs `cat` on both files.

`
leviathan2@gibson:/tmp/tmp.FRyEyTJ4PQ$ ltrace ~/printfile dog\ bird
__libc_start_main(0x80491e6, 2, 0xffffd5a4, 0 <unfinished ...>
access("dog bird", 4)                                                    = 0
snprintf("/bin/cat dog bird", 511, "/bin/cat %s", "dog bird")            = 17
geteuid()                                                                = 12002
geteuid()                                                                = 12002
setreuid(12002, 12002)                                                   = 0
system("/bin/cat dog bird"/bin/cat: dog: No such file or directory
/bin/cat: bird: No such file or directory
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                   = 256
+++ exited (status 0) +++
`

One last thing to notice is that `access` is only testing the file I gave it `dog bird`, but it is NOT running `cat` against that file, but rather two nonexistent files `dog` and `bird`, and it is looking for them in my current directory. This is all the information you need to get the password.

The final solution is to create a file in your temp directory that ends in `\ leviathan3`. Next `cd` into `/etc/leviathan_pass`, finally run `~/printfile [full path to temp file]`.

`
leviathan2@gibson:/etc/leviathan_pass$ ~/printfile /tmp/tmp.FRyEyTJ4PQ/solution\ leviathan3
/bin/cat: /tmp/tmp.FRyEyTJ4PQ/solution: Permission denied
********** <- password will be here
`

### Level 3

Logging in there is a suid binary `level3`. Running it prompts you for a password then exits when you enter the wrong one.

`
leviathan3@gibson:~$ ./level3
Enter the password> nothing
bzzzzzzzzap. WRONG
`

Checking `strings` and `hexdump` reveals some odd strings, but none of them work as the password. Running `level3` with `ltrace` shows:

`
leviathan3@gibson:~$ ltrace ./level3
__libc_start_main(0x80492bf, 1, 0xffffd5f4, 0 <unfinished ...>
strcmp("h0no33", "kakaka")                                               = -1
printf("Enter the password> ")                                           = 20
fgets(Enter the password> kakaka
"kakaka\n", 256, 0xf7fab620)                                       = 0xffffd3cc
strcmp("kakaka\n", "snlprintf\n")                                        = -1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                                               = 19
+++ exited (status 0) +++
`

The first `strcmp` checks two of the strings you can find parts of in the hexdump but entering either as the password does not get me anywhere. More interesting is the second `strcmp` that compares what I entered as the password to "snlprintf". Using "snlprintf" as the password gets me a shell. Running `id` shows that my uid is now "leviathan4". Now I just cat out /etc/leviathan_pass/leviathan4 and get the password.

`
leviathan3@gibson:~$ ./level3
Enter the password> snlprintf
[You've got shell]!
$ id
uid=12004(leviathan4) gid=12003(leviathan3) groups=12003(leviathan3)
$ cat /etc/leviathan_pass/leviathan4
********** <- password appears here
`

### Level 4

After logging in the only interesting thing in the home directory is a hidden directory called `.trash`. Inside `.trash` is a suid binary called `bin`. Running `bin` outputs a series of eleven binary strings. Using `ltrace` shows that `bin` is opening `/etc/leviathan_pass/leviathan5`, which has the password. Now every password so far has been ten ascii characters long. I guessed that those binary strings are just the password in binary plus a newline. So I ran the binary through a binary to ascii text converter I found online and tried the resulting string as the password to level 5. It worked.

### Level 5

Logging in there is a suid binary in the home directory, `leviathan5`. Running it gives an output of "Cannot find /tmp/file.log". Now on this box you can't read from `/tmp`, but you can write to it. Creating the log file with `touch /tmp/file.log` and then rerunning `leviathan5` gives me no output, but my temp file appears to be deleted.

When I run `leviathan5` after creating `/tmp/file.log` I get the following from `ltrace`:

`
leviathan5@gibson:/tmp/tmp.lQz0mHaHeG$ ltrace ~/leviathan5
__libc_start_main(0x8049206, 1, 0xffffd5b4, 0 <unfinished ...>
fopen("/tmp/file.log", "r")                                              = 0x804d1a0
fgetc(0x804d1a0)                                                         = '\377'
feof(0x804d1a0)                                                          = 1
fclose(0x804d1a0)                                                        = 0
getuid()                                                                 = 12005
setuid(12005)                                                            = 0
unlink("/tmp/file.log")                                                  = 0
+++ exited (status 0) +++
`

Those calls to `fgetc` and `feof` are probably where I need to go next.

So those two functions are getting single characters from `/tmp/file.log` and checking for EOF respectively. Now if I write something in `/tmp/file.log` then `leviathan5` outputs it to STDIN. This gave me the idea to try making `/tmp/file.log` into a symbolic link to `/etc/leviathan_pass/leviathan6` to see if it would print out the password. I ran `ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log`, then `~/leviathan5` and it worked. On to next level.

### Level 6

On logging in the only thing in the home directory is the suid `leviathan7`. Running it I get "usage: ./leviathan6 <4 digit code>". I could try the same `ltrace` shenanagins I've been doing, or I could run a one liner like, `for i in $(seq -f "%04g" 0 9999); do ~/leviathan6 $i; done` and try all 10,000 combinations. Running my one liner got me a shell which I used to cat out /etc/levianthan_pass/leviathan7 for the password.

### Level 7

There is no puzzle, just a congratulations message.
