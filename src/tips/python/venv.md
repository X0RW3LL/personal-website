# Handling externally-managed Python packages with virtual environments

Following up on the [previous entry](installing_packages.md), let's talk about the situation when you need a package for a single use, but don't want do deal with it sitting on your drive doing absolutely nothing but wasting storage and bandwidth (however little). Alternatively, a situation when the package you need is not packaged by your distro. Or really, installing older/newer versions of packages. This is where virtual environments (venv) come in handy

We'll cover this very quickly because, really, there isn't all too much to it to get the point

## Creating a virtual environment

```py
~$ python3 -c 'import requests; print(requests)'
<module 'requests' from '/usr/lib/python3/dist-packages/requests/__init__.py'>

~$ pip3 show requests # Version and Location are highlighted for relevance
Name: requests
===============
Version: 2.32.3
===============
Summary: Python HTTP for Humans.
Home-page: https://requests.readthedocs.io
Author: Kenneth Reitz
Author-email: me@kennethreitz.org
License: Apache-2.0
========================================
Location: /usr/lib/python3/dist-packages
========================================
Requires: certifi, charset-normalizer, idna, urllib3
Required-by: apache-libcloud, autorecon, b4, censys, crackmapexec, dropbox, faraday-agent-dispatcher, faraday-plugins, faradaysec, netexec, pooch, pwntools, pyExploitDb, pypsrp, python-gitlab, pywinrm, requests-file, requests-futures, requests-toolbelt, sherlock-project, Sphinx, theHarvester, tldextract, wafw00f, websockify

~$ python3 -m venv requests_external # choice of venv name is up to you, but try to keep it descriptive nonetheless

~$ cd requests_external
~/requests_external$ source bin/activate
(requests_external) ~/requests_external$ pip3 show requests
WARNING: Package(s) not found: requests # we are now inside the venv as denoted by the venv name inside the parenthesis, so requests is not installed

(requests_external) ~/requests_external$ pip3 install requests
Collecting requests
  Using cached requests-2.32.3-py3-none-any.whl.metadata (4.6 kB)
Collecting charset-normalizer<4,>=2 (from requests)
  Downloading charset_normalizer-3.4.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (35 kB)
Collecting idna<4,>=2.5 (from requests)
  Using cached idna-3.10-py3-none-any.whl.metadata (10 kB)
Collecting urllib3<3,>=1.21.1 (from requests)
  Downloading urllib3-2.3.0-py3-none-any.whl.metadata (6.5 kB)
Collecting certifi>=2017.4.17 (from requests)
  Downloading certifi-2025.1.31-py3-none-any.whl.metadata (2.5 kB)
Using cached requests-2.32.3-py3-none-any.whl (64 kB)
Downloading certifi-2025.1.31-py3-none-any.whl (166 kB)
Downloading charset_normalizer-3.4.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (145 kB)
Using cached idna-3.10-py3-none-any.whl (70 kB)
Downloading urllib3-2.3.0-py3-none-any.whl (128 kB)
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2025.1.31 charset-normalizer-3.4.1 idna-3.10 requests-2.32.3 urllib3-2.3.0

(requests_external) ~/requests_external$ pip3 show requests
Name: requests
===============
Version: 2.32.3
===============
Summary: Python HTTP for Humans.
Home-page: https://requests.readthedocs.io
Author: Kenneth Reitz
Author-email: me@kennethreitz.org
License: Apache-2.0
======================================================================
Location: /home/x0rw3ll/requests_external/lib/python3.12/site-packages
======================================================================
Requires: certifi, charset-normalizer, idna, urllib3
Required-by:

(requests_external) ~/requests_external$ deactivate

~/requests_external$ # back to system-wide python/pip environment
```

Notice how we now have two packages with the same name and version, but not the location? That's because we're inside the virtual environment. The venv is only concerned with the packages installed in its location, not system-wide packages. The key is to remember to _activate_ the venv beforehand so the correct PATH environment is set accordingly. As long as you can see `(<venv name>)` prefixing your PS1 prompt, you can use python/pip installed in that venv anywhere in your file system. Once you're done, simply execute `deactivate` to get out of that virtual environment

## Removing virtual environments

Once you no longer need the venv, you can simply `rm -r /path/to/venv`. You don't need to uninstall packages from the venv or anything like that to remove it; literally just remove the directory and you're good to go

<div class="warning" style="color:red">
<em><strong>Python virtual environments are not meant to provide any protections against malicious packages. They simply provide isolation between PATH environments, so please do not confuse virtual environments with virtual machines; those are two completely different things</strong></em>
</div>

_That's all, folks!_
