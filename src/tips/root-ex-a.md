# Exhibit A: Mi Kali, su Kali <3

In this quick demo, we will show how an attacker can quickly (and effortlessly) gain a root shell on your system because you decided to clone a questionable Git repository and executed the script(s)/binaries it provided as root

## Enter: CNoEvil

CNoEvil is a fictitious repository that provides an "LDAP exploitation and scanning" script. The source code thereof will not be released, but I will walk you through what I did. The instructions shown in the README are as follows:

```sh
root@kali:~# git clone git@github.com:X0RW3LL/CNoEvil.git
root@kali:~# cd CNoEvil/
root@kali:~/CNoEvil# pip3 install -r requirements.txt
root@kali:~/CNoEvil# python3 ldapscan.py

LDAP scanner that utilizes state of the art
enumeration technologies and some fancy
military-grade AES-256 encryption that
obfuscates request payloads and hides
your traffic like the world is after you

Usage: ldapscanner.py [options] <IP>

Options:
     -h:     print this help message
     -e:     exploit mode
     -d:     DoS mode
     -l:     LDAP scanner
...
```

We can tell that the author implied running those steps as root as shown in the PS1 prompt `root@kali:~#`. Moreover, there's a considerable number of users who login as root by default anyway, and that's the audience I'm addressing. Now, let's have a look at the requirements.txt file:

```
urllib3==1.26.0
pytest-httpbin==1.0.0
requests>=2.28.2,!=2.26.0,!=2.27.1,!=2.27.0
./dist/requestd-2.28.2-py3-none-any.whl
ldap3>=2.5,!=2.5.2,!=2.5.0,!=2.6,!=2.5.1
pyOpenSSL>=21.0.0
colorama==0.4.6
```

Notice anything yet? The requirements list a typosquatted Python package `requestd` that was shipped along with the repo when it was cloned. That typosquatted malicious package does not need anything extra from the user's end; it fires the reverse shell as soon as the package is imported. Best part is: package is imported along with ANY selected argument (with the exception of `-h` to not raise suspicions). Even better: there's no explicit import statement for `requestd`--that import statement was rather obfuscated (however rudimentarily for the sake of this demo)

```sh
x0rw3ll@1984:~/dev/CNoEvil$ wc -l ldapscan.py
1812 ldapscan.py
x0rw3ll@1984:~/dev/CNoEvil$ grep import ldapscan.py
import os
from time import sleep
from multiprocessing import Process
    import sys
sale, or importing the Program or any portion of it.
make, use, sell, offer for sale, import and otherwise run, modify and
x0rw3ll@1984:~/dev/CNoEvil$ grep requestd ldapscan.py
x0rw3ll@1984:~/dev/CNoEvil$
```

You'd think an 1812-line script would be something solid, but no. You absolutely must scrutinize it to the core. If you inspect the source code, you'll find the script doesn't really do anything at all. No, really, it doesn't. It adds random sleeps, some bogus shell output, while it calls back to the attacker's machine. Let's look at an example snippet:

```py
def ldap_exploit(ip=''):
    print('[+] Initiating LDAP exploitation...')
    sleep(3)
    print('[+] Exploiting target: {}'.format(ip))
    sleep(1)
    print('[+] LDAP running on port 389')
    sleep(2.6)
    print('[+] Connected to LDAP Server successfully! Exploitation in progress...')
    print('[!] This might take a moment...DO NOT press any key until prompted!')
    sleep(0.5)
    if os.fork() != 0:
        return
    print('''
[+] Shell landed on target! Spawning shell...

Microsoft Windows [Version 10.0.16299.15]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\Windows\System32>''')
    print('[-] Host terminated the connection unexpectedly')
    enc_key = xor(IV)
```

It really is just a fake shell printer (trololol, amirite?), but you wouldn't know that without reading the script, now would you? Let's move on to the part where you finally get owned. You'd think that running the code in a Python virtual environment would save you, but that couldn't be further from the truth. Venvs serve a specific purpose, and it has absolutely nothing to do with security

## Demo time

In this demo, I am using my Kali on metal as the attacker machine (bottom terminal window), and a _containerized_ Kali based on my filesystem (top terminal window). My attacker IP is 172.20.10.1, and the would-be victim IP is 172.20.10.50. Moreover, the victim container is running a Python venv called `CNoEvil`, following the README instructions and running everything as root. The victim container has a lighter/blue background color for visual distinction

<img class="center" alt="Screenshot showing how running unverifiable scripts as root can give attackers complete control over a system" src="../../assets/img/screenshots/tips/root-ex-a.png"/>

## Moving forward

All the above being said and done, please heed this warning as you could very well become the next victim to such attacks

- Do not login as root 
- Do not execute scripts you cannot vet (or have them vetted by some reputable, trustworthy entity)
- Do not clone repositories as root/with sudo
