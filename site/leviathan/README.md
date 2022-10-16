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
