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
