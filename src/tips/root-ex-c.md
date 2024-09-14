# Exhibit C: The case against /usr/local

## Prologue

This is a special one. Probably one everyone hates the most. The crown jewel of the most common issues with privileged execution of commands. It's no secret that Python is one of the most ~~in~~famous languages out there, and it's no secret either that almost everyone knows what `pip` is; Python's package manager. In the past, `pip` might not have been such an issue (for the most part). Nowadays, however, `pip` is synonymous with nothing but "trouble"

## Import Paths

Let's talk about your `$PATH` environment variable for a moment. When you type the name of a program, say, `nmap`, how does the shell know to autocomplete the binary name, let alone launch it?

Autocompletion technicalities aside, at a very basic level, the shell will start searching the `$PATH` environment variable for possible locations that hold said binary. This is certainly better than having to type, say, `/usr/bin/nmap` or `/usr/sbin/poweroff`. Neat, right?

Now, Python works on the same principle, more or less. When you import a library, Python will look for the default path where packages get installed (i.e. `/usr/lib/pythonX/dist-packages/`), where `X` is the version. Let's see that in action with a library everyone knows; `requests`. `python3-requests` is installed on Kali by default as it's a dependency for a considerable number of tools, so it's a good candidate for this demo

```sh
x0rw3ll@1984:~$ systemd-detect-virt
none
x0rw3ll@1984:~$ pip3 show requests
Name: requests
Version: 2.31.0
Summary: Python HTTP for Humans.
Home-page: https://requests.readthedocs.io
Author: Kenneth Reitz
Author-email: me@kennethreitz.org
License: Apache 2.0
Location: /usr/lib/python3/dist-packages
Requires:
Required-by: censys, crackmapexec, dropbox, faraday-agent-dispatcher, faraday-plugins, faradaysec, netexec, pyExploitDb, pypsrp, python-gitlab, pywinrm, requests-file, requests-toolbelt, theHarvester, tldextract
x0rw3ll@1984:~$ python3 -c 'import requests; print(requests)'
<module 'requests' from '/usr/lib/python3/dist-packages/requests/__init__.py'>
x0rw3ll@1984:~$
```

As we can see from the above output, this is running on metal, with no externally-managed packages installed via `pip`. The default path, as previously mentioned, is indeed `/usr/lib/python3/dist-packages`. We can also confirm this with `pip list --path /usr/lib/python3/dist-packages`

```sh
x0rw3ll@1984:~$ pip list --path /usr/lib/python3/dist-packages/
Package                        Version
------------------------------ -------------------------
aardwolf                       0.2.8
adblockparser                  0.7
aesedb                         0.1.3
aiocmd                         0.1.2
aioconsole                     0.7.0
...
zope.deprecation               5.0
zope.event                     5.0
zope.interface                 6.4
zstandard                      0.23.0.dev0
```

`pip` happily lists all the packages installed in that location, which is everything currenlty installed with its package name prefixed with `python3-`

## Enter: Trouble

We'll now switch contexts; we'll spin up an ephemeral container based on the host file system so that we retain the same packages and everything in its place, and not mess up the actual host with potentially breaking changes. Additionally, we'll be running everything as root for maximum effect. I will be using `run0` instead of `sudo` to switch users so systemd can give us the nice, bright red background color. We'll use pip to install `requests` _again_ with switches instructing it to break system packages, and ignore currently installed packages. This is done for demonstration purposes only, and should not be used for the trouble that ensues. Recall from the output above showing `requests`that the currently installed version is `2.31.0`

<img class="center" alt="Screenshot showing externally-managed requests package being installed to /usr/local/lib/python3.11/dist-packages" src="../../assets/img/screenshots/tips/root-ex-c-req1.png"/>

Note how pip now reports the package being installed to `/usr/local/lib/python3.11/dist-packages` instead of `/usr/lib/python3/dist-packages`? Let's double-check whether the system-wide installation of `python3-requests` still exists

```sh
root@1984:~# pip list --path /usr/lib/python3/dist-packages | egrep '^requests '
requests           2.31.0
root@1984:~# pip list --path /usr/local/lib/python3.11/dist-packages | grep requests
requests           2.32.3
root@1984:~#
```

Now we have a real problem: we have two different versions of `requests`, namely `2.31.0` and `2.32.3` installed in two different locations; `/usr/lib/python3/dist-packages` and `/usr/local/lib/python3.11/dist-packages`. What does this mean? Well, different programs/scripts will be extremely unreliable when it comes to importing `requests`. They might end up importing one version or the other, depending on who's calling, where, under which context, etc. Moreover, some tools will have _exactly equal_ Depends. That means that the tool is designed to work with a specific version of a library. This might be due to deprecated APIs, or other decisions made by the tool developer(s)


## Demo

Let's see that in action with a package that will indeed throw some functionality-breaking errors; `impacket`. Here's what we have so far (before installing the externally-managed `impacket` package)

```sh
root@1984:~# apt policy python3-impacket; apt rdepends python3-impacket; pip show impacket
python3-impacket:
  Installed: 0.11.0+git20240410.ae3b5db-0kali1
  Candidate: 0.11.0+git20240410.ae3b5db-0kali1
  Version table:
 *** 0.11.0+git20240410.ae3b5db-0kali1 500
        500 https://kali.download/kali kali-rolling/main amd64 Packages
        500 https://kali.download/kali kali-rolling/main i386 Packages
        100 /var/lib/dpkg/status
python3-impacket
Reverse Depends:
  Depends: netexec (>= 0.11.0+git20240410)
  Depends: wig-ng
  Depends: spraykatz
  Depends: smbmap
  Depends: set
  Depends: redsnarf
  Depends: python3-pywerview
  Recommends: python3-pcapy
  Depends: python3-masky
  Depends: python3-lsassy
  Depends: python3-dploot
  Depends: polenum
  Depends: patator
  Recommends: openvas-scanner
  Depends: offsec-pwk
  Depends: impacket-scripts (>= 0.11.0)
  Depends: koadic
  Depends: kali-linux-headless
  Depends: autorecon (>= 0.10.0)
  Depends: hekatomb
  Depends: enum4linux-ng
  Depends: crackmapexec
  Depends: coercer
  Depends: certipy-ad
  Depends: bloodhound.py
Name: impacket
Version: 0.12.0.dev1
Summary: Network protocols Constructors and Dissectors
Home-page: https://www.coresecurity.com
Author: SecureAuth Corporation
Author-email:
License: Apache modified
Location: /usr/lib/python3/dist-packages
Requires:
Required-by: crackmapexec, dploot, lsassy, netexec
root@1984:~#
```

As we can see, `python3-impacket` has quite a number of reverse dependencies that may very well end up breaking. Let's break some!

After installing the package with pip as root, we get the following information
```sh
root@1984:~# pip show impacket
Name: impacket
Version: 0.11.0
Summary: Network protocols Constructors and Dissectors
Home-page: https://www.coresecurity.com
Author: SecureAuth Corporation
Author-email:
License: Apache modified
Location: /usr/local/lib/python3.11/dist-packages
Requires: charset-normalizer, dsinternals, flask, future, ldap3, ldapdomaindump, pyasn1, pycryptodomex, pyOpenSSL, six
Required-by: crackmapexec, dploot, lsassy, netexec
root@1984:~#
```

Right off the bat, besides the obvious location, we now have a _downgraded_ version of `impacket`. Why is that? For starters, PyPI might not have been updated with the latest release of the package, while the Debian Python Team has taken the lead on that one, building the `0.12.0.dev1` release, as opposed to `0.11.0`. Let's now try running some of our favorite impacket examples and see what happens

<img class="center" alt="Screenshot showing impacket-ntlmrelayx breaking due to conflicting versions of impacket being installed in two different locations" src="../../assets/img/screenshots/tips/root-ex-c-impacket.png"/>

Sure enough, we definitely broke system packages! Even worse, the above error output doesn't even say much about what's _actually_ wrong; it just complained that `NTLMRelayxConfig` has no attribute `setAddComputerSMB`. This attribute could have been added in the newer release of the package, or a result of conflicting import paths; one would have to really dig into it, line by line, to figure out where/what the problem is

## Fixing the mess

The million-dollar question is: how does one fix this dependency hell? The answer is quite simple, really. All we need to do is filter those packages located at `/usr/local/lib/python*/dist-packages`, and uninstall them with elevated privileges much like they were originally installed. At this point, saving the package list to a file can be a good idea in case we want to install some of those packages properly later. For the purpose of this demo, I am going to have a bunch of externally-managed packages installed via `pip` so we can take a look at automating an otherwise tedious process

```sh
┌──(test㉿1984)-[~]
└─$ pip list --path /usr/local/lib/python3.11/dist-packages/
Package            Version
------------------ ---------
aesedb             0.1.6
aiosmb             0.4.11
aiowinreg          0.0.12
asn1crypto         1.5.1
asyauth            0.0.21
asysocks           0.2.13
blinker            1.8.2
certifi            2024.8.30
cffi               1.17.1
chardet            5.2.0
charset-normalizer 3.3.2
click              8.1.7
colorama           0.4.6
cryptography       43.0.1
dnspython          2.6.1
dsinternals        1.2.4
Flask              3.0.3
future             1.0.0
h11                0.14.0
idna               3.8
impacket           0.11.0
itsdangerous       2.2.0
Jinja2             3.1.4
ldap3              2.9.1
ldapdomaindump     0.9.4
lsassy             3.1.12
markdown-it-py     3.0.0
MarkupSafe         2.1.5
mdurl              0.1.2
minidump           0.0.24
minikerberos       0.4.4
msldap             0.5.12
netaddr            1.3.0
oscrypto           1.3.0
prompt_toolkit     3.0.47
pyasn1             0.6.1
pycparser          2.22
pycryptodomex      3.20.0
Pygments           2.18.0
pyOpenSSL          24.2.1
pypykatz           0.6.10
requests           2.32.3
rich               13.8.1
six                1.16.0
tabulate           0.9.0
tqdm               4.66.5
unicrypto          0.0.10
urllib3            2.2.3
wcwidth            0.2.13
Werkzeug           3.0.4
winacl             0.1.9
```

As we can see, there's a considerable number of externally-managed packages that need to be dealt with. Since we're all about automation, let's get creative with a one-liner that does just that

```sh
┌──(test㉿1984)-[~]
└─$ pip list --path /usr/local/lib/python3.11/dist-packages/ | cut -d ' ' -f1 | egrep -v '^Package|---*' | tr '\n' ' '
aesedb aiosmb aiowinreg asn1crypto asyauth asysocks blinker certifi cffi chardet charset-normalizer click colorama cryptography dnspython dsinternals Flask future h11 idna impacket itsdangerous Jinja2 ldap3 ldapdomaindump lsassy markdown-it-py MarkupSafe mdurl minidump minikerberos msldap netaddr oscrypto prompt_toolkit pyasn1 pycparser pycryptodomex Pygments pyOpenSSL pypykatz requests rich six tabulate tqdm unicrypto urllib3 wcwidth Werkzeug winacl
```

We used `cut -d ' ' -f1` to simply grab the first thing that's not a space, which happens to be the package names. We then `egrep -v '^Package|---*'` to filter out irrelevant output that would break the uninstall process since `Package` and `---------` are obviously not valid Python packages. Finally, we used `tr '\n' ' '` to translate newlines into spaces instead. Now that we got the desired output, let's incorporate it into the final `pip` command

```sh
┌──(test㉿1984)-[~]
└─$ sudo pip uninstall -y $(pip list --path /usr/local/lib/python3.11/dist-packages/ | cut -d ' ' -f1 | egrep -v '^Package|---*' | tr '\n' ' ')
Found existing installation: aesedb 0.1.6
Uninstalling aesedb-0.1.6:
  Successfully uninstalled aesedb-0.1.6
Found existing installation: aiosmb 0.4.11
Uninstalling aiosmb-0.4.11:
  Successfully uninstalled aiosmb-0.4.11
Found existing installation: aiowinreg 0.0.12
Uninstalling aiowinreg-0.0.12:
  Successfully uninstalled aiowinreg-0.0.12
...
```

To confirm, we can run the listing again, and sure enough, all those externally-managed packages are now a thing of the past

## Closing thoughts

Luckily, `pip` is now becoming more a thing of the past, and I do hope it gets sunset soon. Switches like `--break-system-packages` have been added as a deterrent to stop users from, well, __breaking system packages__. I cannot stress enough how terrible an idea it is to keep running everything as a privileged user all the time. Again, it does way more harm than good, and even if you do know what you're doing, you're still very much prone to making mistakes; we're all human, remember? We __do__ make mistakes. Should you need to install Python packages, search the package repos for them first using `apt search`. If they exist, they will be prefixed with `python3-`. If they don't exist, you can always create virtual environments that will take care of path separation for you, and avoid breaking your currently installed packages

## Further reading

- [https://www.kali.org/blog/kali-linux-2023-1-release/#python-updates--changes](https://www.kali.org/blog/kali-linux-2023-1-release/#python-updates--changes)
- [https://peps.python.org/pep-0668/](https://peps.python.org/pep-0668/)
- [https://docs.python.org/3/library/venv.html](https://docs.python.org/3/library/venv.html)
