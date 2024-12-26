# apt-patterns

Let's talk about advanced package management. In this section, we'll visit one of apt's lesser known features—apt-patterns. If you've ever found yourself jumping hoops trying to manage multiple packages, doing stuff like `apt purge $(dpkg -l | grep whatever | cut -d ' ' -f3 | tr '\n' ' ')`, you're gonna find this entry useful. Let's see how it works

## What's a PATTERN?

First things first: semantics. An apt PATTERN is different than a RegEx pattern, and we need to be very mindful of that distinction because it _can_ get confusing when first reading the man-pages. An apt PATTERN is syntax used with apt, and is usually prefixed with special characters, namely a question mark `?` for long-form syntax, and a tilde `~` for short-form syntax. An example would be `?installed`, versus `~i`. With that in mind, `~i` and the likes, be it long or short form, is the PATTERN to which we refer in this section. Again, not to be confused with REGEX, which [APT-PATTERNS(7)](https://people.debian.org/~jak/apt-doc/man/apt-patterns.7.html) will highlight as such. Now that we got the semantics out of the way, let's put that into practice

## Querying packages

Assume you're shopping for packages from the repository. Quickest thing you can do is `apt search <package>`, and that's great and all, but sometimes we need more than just a quick fuzzy search. That is, we might be looking for specific package archs, versions, exact names, or even sections. The great thing about apt-patterns is the fact that we can specify whether we want to look exclusively for installed packages, and really just have more fine-grained control over the search. Let's say we want to search for _installed_ packages that include `json` in the name

```sh
$ apt list '~i ~njson'
libcjson1/kali-rolling,now 1.7.18-3+b1 amd64 [installed,automatic]
libcpanel-json-xs-perl/kali-rolling,now 4.38-1+b1 amd64 [installed,automatic]
libjson-c-dev/kali-rolling,now 0.18+ds-1 amd64 [installed]
libjson-c5/kali-rolling,now 0.18+ds-1 amd64 [installed]
...
```

`~n` stands for name. We could also use the long-form syntax as follows:

```sh
$ apt list '~i ?name(json)'
libcjson1/kali-rolling,now 1.7.18-3+b1 amd64 [installed,automatic]
libcpanel-json-xs-perl/kali-rolling,now 4.38-1+b1 amd64 [installed,automatic]
libjson-c-dev/kali-rolling,now 0.18+ds-1 amd64 [installed]
libjson-c5/kali-rolling,now 0.18+ds-1 amd64 [installed]
...
```

Now, let's try to find packages that are _not_ installed with `json` in the name. The logical negation operator works as well as you'd expect; simply prefix the pattern with it

```sh
$ apt list '!~i ?name(json)'
dwarf2json/kali-rolling 0.6.0~git20200714-0kali1 amd64
dwarf2json/kali-rolling 0.6.0~git20200714-0kali1 i386
ffjson/kali-rolling 0.0~git20181028.e517b90-1.1 amd64
ffjson/kali-rolling 0.0~git20181028.e517b90-1.1 i386
gir1.2-json-1.0/kali-rolling 1.10.6+ds-1 amd64
gir1.2-json-1.0/kali-rolling 1.10.6+ds-1 i386
gir1.2-jsonrpc-1.0/kali-rolling 3.44.1-1 amd64
gir1.2-jsonrpc-1.0/kali-rolling 3.44.1-1 i386
...
```

As we can see, the results include `i386` packages since I have a multi-arch setup. Let's include only amd64 packages

```sh
$ apt list '!~i ?name(json) ~ramd64'
dwarf2json/kali-rolling 0.6.0~git20200714-0kali1 amd64
ffjson/kali-rolling 0.0~git20181028.e517b90-1.1 amd64
gir1.2-json-1.0/kali-rolling 1.10.6+ds-1 amd64
gir1.2-jsonrpc-1.0/kali-rolling 3.44.1-1 amd64
gjh-asl-json/kali-rolling 0.0+git20210628.867c5da-1 amd64
golang-easyjson/kali-rolling 0.7.7-1+b8 amd64
...
```

Now, let's assume you're just window-shopping, metaphorically, for packages under certain [sections](https://www.debian.org/doc/debian-policy/ch-archive.html#s-subsections). Say you want to figure out what games are available in the repos, specifically those with the amd64 arch

```sh
$ apt list '~sgames ~ramd64'
0ad-data-common/kali-rolling,kali-rolling 0.0.26-1 all
0ad-data/kali-rolling,kali-rolling 0.0.26-1 all
0ad/kali-rolling 0.0.26-8 amd64
1oom/kali-rolling 1.11-1 amd64
2048-qt/kali-rolling 0.1.6-2+b3 amd64
2048/kali-rolling 1.0.0-1 amd64
3dchess/kali-rolling 0.8.1-22 amd64
...
```

## Querying dependencies

If you're coming from `aptitude` where it's easy to just `aptitude why <package>` and it tells you _why_ a package is installed, i.e. what packages _DEPEND_ on said `<package>`, this is gonna be helpful. Granted it's not going to be exactly the same as `aptitude` does it, but it's close enough with a vanilla pattern; reason being that `aptitude` will also list `Suggests` and `Provides`, not strictly `Depends`. I will leave those as an exercise to the reader, but it shouldn't be too difficult to figure out how to combine multiple patterns in one go. Not to mention `man apt-patterns` will be your trusty resource, as always

Let's try to figure out the reverse dependencies of, say, `libx86-1`. There are 3 ways to do it; `aptitude why libx86-1` (if `aptitude` is installed), `apt rdepends libx86-1` (which can get pretty crowded and is not friendly with package management without Bash-Fu), and `apt list '~i ~D(?exact-name(libx86-1))'`

`aptitude` is great because it will trace the reverse dependencies all the way to the top. This will make more sense when seen in output, so let's compare how each command does it

```sh
$ aptitude why libx86-1
i   kali-linux-default  Depends    kali-linux-headless
i A kali-linux-headless Depends    i2c-tools
i A i2c-tools           Recommends read-edid
i A read-edid           Depends    libx86-1 (>= 1.1+ds1)

$ apt rdepends libx86-1
libx86-1
Reverse Depends:
  Depends: read-edid (>= 1.1+ds1)

$ apt list '~i ~D(?exact-name(libx86-1))'
read-edid/now 3.0.2-1.1 amd64 [installed,local]
```

Notice how `aptitude` traced rdeps all the way to the top? i.e. `kali-linux-default` depends on `kali-linux-headless`, which in turn depends on `i2c-tools`, which recommends `read-edid`, which is what depends _directly_ on `libx86-1`. In the other examples using `apt`, you don't get that nice relational table highlighting the dependency list, but you do get the actual reverse depency nonetheless. The difference between `apt rdepends` and `apt list` then becomes a matter of cooperability with managing packages. `apt rdepends` can be used in cases where you quickly want to check the reverse dependencies of certain packages, whereas `apt list ...` is used when you want to do something with _those_ queried reverse dependencies. We'll look at a similar scenario in this following section

## Package management

Here comes the risky part; actually managing packages. Please pay close attention because you can easily mess this one up when first trying it out. In this section, we'll figure out how to handle multiple packages based on specific criteria as specified by PATTERNs. I build custom kernels very frequently, and I tend to keep previous releases in case things go wrong so I'm not stuck with an unbootable kernel. Once I'm certain the new kernel release works as expected, I purge the previous one(s). I could type up `apt purge --auto-remove linux-image-* linux-headers-*`, but that's just too much unnecessary typing. Let's do this efficiently with apt-patterns. For starters, it's always a good idea to just `list` packages first to make sure we have the intended ones in place before passing destructive verbs like `remove` or `purge`. At the time of writing this entry, I'm on the `6.13.0-rc4` kernel, and I still have previous release candidates 1, 2, and 3. I also have the debug symbols installed, so I'll want those purged as well. Let's list those _installed_ packages, specifying the name RegEx

```sh
$ apt list '~i ~n"^linux-(image|headers)-6.13.0-rc[1-3]"'
linux-headers-6.13.0-rc1/now 6.13.0-rc1-00695-gf941996d0cb8-5 amd64 [installed,local]
linux-headers-6.13.0-rc2/now 6.13.0-rc2-00401-g48c84277de4c-2 amd64 [installed,local]
linux-headers-6.13.0-rc3/now 6.13.0-rc3-00674-gffe093fb67f8-3 amd64 [installed,local]
linux-image-6.13.0-rc1-dbg/now 6.13.0-rc1-00695-gf941996d0cb8-5 amd64 [installed,local]
linux-image-6.13.0-rc1/now 6.13.0-rc1-00695-gf941996d0cb8-5 amd64 [installed,local]
linux-image-6.13.0-rc2-dbg/now 6.13.0-rc2-00401-g48c84277de4c-2 amd64 [installed,local]
linux-image-6.13.0-rc2/now 6.13.0-rc2-00401-g48c84277de4c-2 amd64 [installed,local]
linux-image-6.13.0-rc3-dbg/now 6.13.0-rc3-00674-gffe093fb67f8-3 amd64 [installed,local]
linux-image-6.13.0-rc3/now 6.13.0-rc3-00674-gffe093fb67f8-3 amd64 [installed,local]
```

Note that when we want extended RegEx in the query, we'll ideally want to wrap it in quotation marks. In the above example, we're looking for packages beginning (`^`) with `linux-`, followed by either `image` or `headers`, `6.13.0-rc`, followed by any one digit between 1 and 3 (inclusive). Sure enough, we have all the packages we need, and because the output is a listing, it's extremely easy to handle those packages with apt as-is. For extra caution, however, dear reader, I recommend you simulate the action before going live with it; meaning `apt -s <VERB> <packages>`. I've already done that in the past, so I know what the syntax is and what packages have those names. Here's how I'll purge those in one go

```sh
$ sudo apt purge --auto-remove '~i ~n"^linux-(image|headers)-6.13.0-rc[1-3]"'
[sudo] password for x0rw3ll:
REMOVING:
  linux-headers-6.13.0-rc1*  linux-headers-6.13.0-rc3*  linux-image-6.13.0-rc1-dbg*  linux-image-6.13.0-rc2-dbg*  linux-image-6.13.0-rc3-dbg*
  linux-headers-6.13.0-rc2*  linux-image-6.13.0-rc1*    linux-image-6.13.0-rc2*      linux-image-6.13.0-rc3*

Summary:
  Upgrading: 0, Installing: 0, Removing: 9, Not Upgrading: 0
  Freed space: 7,121 MB

Continue? [Y/n]
(Reading database ... 788000 files and directories currently installed.)
Removing linux-headers-6.13.0-rc1 (6.13.0-rc1-00695-gf941996d0cb8-5) ...
Removing linux-headers-6.13.0-rc2 (6.13.0-rc2-00401-g48c84277de4c-2) ...
Removing linux-headers-6.13.0-rc3 (6.13.0-rc3-00674-gffe093fb67f8-3) ...
Removing linux-image-6.13.0-rc1 (6.13.0-rc1-00695-gf941996d0cb8-5) ...
update-initramfs: Deleting /boot/initrd.img-6.13.0-rc1
Removing kernel version 6.13.0-rc1 from systemd-boot...
Generating grub configuration file ...
Found background image: /usr/share/images/desktop-base/desktop-grub.png
Found linux image: /boot/vmlinuz-6.13.0-rc4
Found initrd image: /boot/initrd.img-6.13.0-rc4
Found linux image: /boot/vmlinuz-6.13.0-rc3
Found initrd image: /boot/initrd.img-6.13.0-rc3
Found linux image: /boot/vmlinuz-6.13.0-rc2
Found initrd image: /boot/initrd.img-6.13.0-rc2
Found linux image: /boot/vmlinuz-6.12.0
Found initrd image: /boot/initrd.img-6.12.0
Found linux image: /boot/vmlinuz-6.11.2-amd64
Found initrd image: /boot/initrd.img-6.11.2-amd64
Warning: os-prober will be executed to detect other bootable partitions.
Its output will be used to detect bootable binaries on them and create new boot entries.
Adding boot menu entry for UEFI Firmware Settings ...
done
Removing linux-image-6.13.0-rc1-dbg (6.13.0-rc1-00695-gf941996d0cb8-5) ...
...
Removing linux-image-6.13.0-rc3-dbg (6.13.0-rc3-00674-gffe093fb67f8-3) ...
(Reading database ... 758746 files and directories currently installed.)
Purging configuration files for linux-image-6.13.0-rc2 (6.13.0-rc2-00401-g48c84277de4c-2) ...
Removing kernel version 6.13.0-rc2 from systemd-boot...
Purging configuration files for linux-image-6.13.0-rc3 (6.13.0-rc3-00674-gffe093fb67f8-3) ...
Removing kernel version 6.13.0-rc3 from systemd-boot...
Purging configuration files for linux-image-6.13.0-rc1 (6.13.0-rc1-00695-gf941996d0cb8-5) ...
Removing kernel version 6.13.0-rc1 from systemd-boot...
```

Couldn't have been easier, don't you think? It might take you a little while just to get used to it, but once you do, it's literally all you'll ever need for the most part. It gets things going quickly, and the fact that you can work with the output straight from `apt` without any one-liners makes it a very attractive choice for system administration. Do note that this section is really only a very basic, rough guide to using apt-patterns correctly. There is so much more for you to discover in the man-pages, which I cannot recommend enough!
