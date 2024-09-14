# Exhibit D: Look how they massacred my perms

## The basics

Let's talk about permission bits for a moment. Assume we have a file with the following permission bets set: `-rwxrwxrwx`. It should already be obvious there's some sort of a pattern here, but in case that wasn't clear enough, let's dissect it

We notice there are 4 main segments to these bits

```
# See chmod(1) for more information

r: read
w: write
x: execute (or search for directories)
X: execute/search only if the file is a directory or already has execute permission for some user
s: set user or group ID on execution
t: restricted deletion flag or sticky bit

- [rwx] [rwx] [rwx]
```

The first field can either be a `-` or a `d` for file, or directory, respectively

The first segment of `[rwx]` is owner permissions. Second is group, and third is "others". The account that first created the file would be its owner, and if the account belongs to a certain group, the file might also belong to that same group; meaning other members of the same group might be able to read the file as well, given correct permission bits are set. The final segment refers to others, meaning anyone else who's not 1) the owner, and 2) part of the group that owns the file

If you've ever looked up file permissions, you must have come across something like `chmod 0777 <file>`. What do these numbers mean? This would be what's known as numeric mode. Permission bits can be represented by octal digits, ranging from 0-7, derived by adding up the bits with the values 4, 2, and 1. First digit (`0` in this case) selects the set user ID, set group ID, and restricted deletion/sticky attributes. Second digit selects the permissions for the user who owns the file (read (4), write (2), and execute (1)). Third selects permissions for other users in the same group, and fourth selects permissions for other users not in the file's group

In the case of `-rwxrwxrwx`, we agreed that we have 3 segments; `rwx` for each. Let's add them up; `r = 4, w = 2, x = 1`

`4+2+1=7`, therefore the first digit becomes 7. Same for the second and third, ergo `777`. First digit, when omitted, is assumed to be a leading zero

Now, what if the file permissions are as follows: `-rw-------`. The segments are: `rw-`, `---`, and `---`. Add them up: `4+2+0=6`. Omitted digits are zeros, remember? Therefore, octal permissions are `0600` in this case, meaning only the file owner has read and write permissions to the file, and everyone else cannot read, write, or execute/search for the file. Now you know why OpenSSH private keys are set with these permission bits by default; no one else should ever be able to read them but their respective owner alone. We've got the basics covered, yes? Let's move on

## Why permissions are important

Your file system is not up for public demonstration, hey. Every user or group on the system should be able to enjoy the right to privacy to their files and how they want to control them. For example, I think we can all agree that the shadow file `/etc/shadow` should never, ever, be publicly readable by anyone waltzing in on the system without proper access rights, yes? It's the database that holds the hashed passwords of every user on the system after all. Imagine how easy it would have been if everyone could read that database, let alone modify it. From a security standpoint, permission bits can be considered the bare minimum anyone can adopt to place any sort of restrictive access controls to files and directories in the file system

## Demo I: Fictitious perms for demonstration purposes

For this demo, I've set up 3 scripts with different permission bits set. Each script checks its own permissions, and if they match, it calls the next script which does the same. This specific demo will highlight one thing many like to do, with little-to-zero idea what it does or why: `sudo chmod -R 777 /path/to/some/directory`

Before we go ahead, this is something you should never do unless you absolutely know what you're doing and why you're doing it. Do not follow random "advice" you find on forums, even if some of the answers swear it worked for them. Just because something works, it doesn't mean it was done _correctly_

Let's have a look at what the scripts do, and what their permissions are

```sh
x0rw3ll@1984:~/testing-perms$ ll
total 12
-rwx------ 1 x0rw3ll x0rw3ll 393 Sep 14 14:44 test1.sh
-rwxrw-r-- 1 x0rw3ll x0rw3ll 393 Sep 14 14:44 test2.sh
-rwx--x--x 1 x0rw3ll x0rw3ll 385 Sep 14 14:44 test3.sh
x0rw3ll@1984:~/testing-perms$ for i in `ls`; do echo $i; cat $i; echo; done
test1.sh
#!/usr/bin/bash

r="\e[31m"
g="\e[32m"
b="\e[34m"
e="\e[0m"
perms=`stat -c %a $0`
filename=`echo $0 | cut -d '/' -f2`

printf "\n%b[!] Executing $filename%b\n" $b $e

if [ $perms == 700 ]
then
  printf "%b[+] $filename has correct permission bits set: $perms%b\n" $g $e
  ./test2.sh
else
  printf "%b[-] $filename has incorrect permission bits set: $perms -- Expected: 700%b\n" $r $e
  exit 1
fi

test2.sh
#!/usr/bin/bash

r="\e[31m"
g="\e[32m"
b="\e[34m"
e="\e[0m"
perms=`stat -c %a $0`
filename=`echo $0 | cut -d '/' -f2`

printf "\n%b[!] Executing $filename%b\n" $b $e

if [ $perms == 764 ]
then
  printf "%b[+] $filename has correct permission bits set: $perms%b\n" $g $e
  ./test3.sh
else
  printf "%b[-] $filename has incorrect permission bits set: $perms -- Expected: 764%b\n" $r $e
  exit 1
fi

test3.sh
#!/usr/bin/bash

r="\e[31m"
g="\e[32m"
b="\e[34m"
e="\e[0m"
perms=`stat -c %a $0`
filename=`echo $0 | cut -d '/' -f2`

printf "\n%b[!] Executing $filename%b\n" $b $e

if [ $perms == 777 ]
then
  printf "%b[+] $filename has correct permission bits set: $perms%b\n\n" $g $e
else
  printf "%b[-] $filename has incorrect permission bits set: $perms -- Expected: 777%b\n\n" $r $e
  exit 1
fi

x0rw3ll@1984:~/testing-perms$
```

Fairly straight forward. Let's see what happens when we execute

<img class="center" alt="Screenshot showing running test1.sh, with expected output as correct permission bits are set" src="../../assets/img/screenshots/tips/root-ex-d_pgood.png"/>

Now, let's do the stupid thing; `chmod 777 *` inside the `testing-perms` directory, and see how that plays out


<img class="center" alt="Screenshot showing running test1.sh, with unexpected output as incorrect permission bits are set" src="../../assets/img/screenshots/tips/root-ex-d_pbad.png"/>

Bear in mind you'll likely never run into a situation where you'll be given some handy guide with all the correct file permissions outlined therein. Often times, permissions will be provided as-is, and should not be modified unless, again, you know what you're doing and why. Imagine you did something incredibly stupid like `sudo chmod -R 0777 /`; this is 100% guaranteed to break your system beyond actual repair. There's just no way to keep track of which file(s) had which permissions, not to mention that it would take literal ages to go through _every single file_ that was affected. It gets worse; at least `sudo` prompts you for a password (given the token's expired). Imagine running that as root, without paying attention to what file/directory you're modifying. Good luck with that!

## Demo II: OpenSSH keys

This one is more functional, and is often a point of headaches. The principal is the same, really, but I'm inviting you to think about what happens when you, say, `sudo wget http://target/main.php?include=/path/to/id_rsa`. Or when you download an OpenSSH private key file without remembering to set the proper permissions to it. If you've ever connected to, say, an AWS EC2 instance, you might remember it instructing you to set the key perms to 0400 before connecting. 0400 is more restrictive than 0600 since only the read bit is set, but no write is allowed

<img class="center" alt="Screenshot showing two SSH connection attempts, one failing due to bad permissions set for the key file, and the other succeeding when permission bits are fixed" src="../../assets/img/screenshots/tips/root-ex-d_ssh.png"/>

## Closing thoughts

Hopefully those demos give you better insights into avoiding bad habits. Permissions are no joke, and can be a massive security risk if set incorrectly. Moreoever, you should always read error output; error messages are there to help you diagnose issues and troubleshoot them
