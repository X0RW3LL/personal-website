# Finding and installing Python packages correctly

This is an important, and a rather quick one, so pay close attention

For the longest time ever, the de-facto go-to method to install Python packages, as suggested on many an online forum, is `pip install <package>`. This is no longer the case, however, since [PEP-668](https://peps.python.org/pep-0668/) (See also: [Pip install and Python's externally managed](https://www.kali.org/blog/python-externally-managed/))

## PEP-668 TL;DR

Assume the following scenario: your system has the `python3-requests` package installed system-wide. You then decide to _also_ execute `pip install requests`, or worse, `sudo pip install requests` (don't do that or recommend anyone do it; it's why we now have PEP-668 because of how terrible that is. See [Exhibit C: The case against /usr/local](../root-ex-c.md))

Your system now has two packages with the same name, found in different paths. This is where conflicts exist, especially when you have different versions of the same package installed in different directories. PEP-668 finally puts an end to that, preventing misinformed users from inadvertently damaging their environment. I've already covered that in detail in the linked article above, so we're not going to reinvent the wheel here

## How to actually find and install packages correctly

The recommendation is to use your system's package manager. Python packages have the `python3-` prefix, and so this is with which you should be leading your search

If you know the package name, like requests for example, you can query the package index using `apt policy python3-requests` to verify that the package indeed exists (i.e. is packaged by your distro), and find out what the candidate version is

If you _don't_ know the package name, however, you can `apt search <package name> | grep -E '^python3-'`. Following [apt-patterns](../apt-patterns.md), however, you can do something like `apt list '~n^python3-' | grep <package name>`.

If you're just catalogue-shopping packages for inspiration, you can do `apt list '~spython ~n<package name>'`. This will page through packages that are available. If you wanna take it a step further, you can use the same syntax with the operation `show` instead of `list`. This is a whole lot quicker than searching for the package, copying its name, then performing `apt show <package name>` (or going berzerk with Bash-fu loops; it's awfully slow, trust me)

Here's what that looks like in action: (output is truncated for brevity)

```sh
$ apt list '~spython ~nrequests'
python3-aws-requests-auth/kali-rolling,kali-rolling 0.4.3-4 all
python3-grequests/kali-rolling,kali-rolling 0.7.0-3 all
python3-requests-cache/kali-rolling,kali-rolling 1.2.1-1 all
python3-requests-file/kali-rolling,kali-rolling,now 2.1.0-1 all [installed,automatic]
python3-requests-futures/kali-rolling,kali-rolling,now 1.0.2-1 all [installed,automatic]
python3-requests-kerberos/kali-rolling,kali-rolling 0.14.0-4 all
python3-requests-mock/kali-rolling,kali-rolling 1.12.1-3 all
python3-requests-ntlm/kali-rolling,kali-rolling,now 1.1.0-3 all [installed,automatic]
python3-requests-oauthlib/kali-rolling,kali-rolling 1.3.1-1 all
python3-requests-toolbelt/kali-rolling,kali-rolling,now 1.0.0-4 all [installed,automatic]
python3-requests-unixsocket/kali-rolling,kali-rolling 0.3.0-6 all
python3-requests/kali-rolling,kali-rolling,now 2.32.3+dfsg-1 all [installed,automatic]
python3-requestsexceptions/kali-rolling,kali-rolling 1.4.0-5 all
python3-txrequests/kali-rolling,kali-rolling 0.9.6-2 all

$ apt show '~spython ~nrequests'
Package: python3-requests
Version: 2.32.3+dfsg-1
Priority: optional
Section: python
Source: requests
Maintainer: Debian Python Team <team+python@tracker.debian.org>
Installed-Size: 247 kB
Depends: python3-certifi, python3-charset-normalizer, python3-idna, python3-urllib3 (>= 1.21.1), python3:any, ca-certificates, python3-chardet (>= 3.0.2)
Suggests: python3-cryptography, python3-idna (>= 2.5), python3-openssl, python3-socks, python-requests-doc
Breaks: awscli (<< 1.11.139)
Homepage: https://requests.readthedocs.io/
Download-Size: 71.9 kB
APT-Manual-Installed: no
APT-Sources: https://kali.download/kali kali-rolling/main amd64 Packages
Description: elegant and simple HTTP library for Python3, built for human beings
 Requests allow you to send HTTP/1.1 requests. You can add headers, form data,
 multipart files, and parameters with simple Python dictionaries, and access
 the response data in the same way.
 ...
 This package contains the Python 3 version of the library.

Package: python3-requests-oauthlib
Version: 1.3.1-1
Priority: optional
Section: python
Source: python-requests-oauthlib
Maintainer: Debian Python Team <team+python@tracker.debian.org>
Installed-Size: 94.2 kB
Depends: python3-oauthlib, python3-requests, python3:any
Homepage: https://github.com/requests/requests-oauthlib
Download-Size: 21.1 kB
APT-Sources: https://kali.download/kali kali-rolling/main amd64 Packages
Description: module providing OAuthlib auth support for requests (Python 3)
...
```
