# Surviving the Linux Kernel Bug Fixing Mentorship Program - Spring 2025

## What it isn't

In order to set expectations, I thought I'd lead with what the program is not so as to cut to the chase. This mentorship is definitely not for the hand-holding type—i.e. don't expect a full-on instructor-led training, which this is not. There is no "curriculum" or a playbook. You are given a set of resources to go through at your own pace as you work your way towards graduation

## What it is

Now that we've covered what the program isn't, let's talk about what it actually is! Simply put, it's a primer to get your foot in the door. You're familiarized, to a certain extent, with how the kernel development ecosystem works. You're also introduced to general netiquette when interacting with reviewers/maintainers. While the kernel documentation has entire sections on submitting patches, it can get a little overwhelming to put everything you read into practice

This mentorship clarifies a few key differences between contributing to your average small-to-medium sized open-source projects, and much larger projects like the Linux kernel where there is so much intersection and moving parts. The program is led by Shuah Khan—one of the maintainers of various drivers/subsystems/frameworks like cpupower monitoring, kselftests, USB over IP, as well as the VIMC driver

Think of this program as the perfect guide on getting started with making contributions to the Linux kernel, and really, any large open-source project by association. Assistance is there when it's needed during weekly Zoom office hours, but don't count on office hours and/or Discord—you're expected to do your own research and keep up with everyone else doing the mentorship

## What I like about the program

There's a lot to cover here, but I'll try to keep it short for brevity:

- Resource sharing
- Hands-on demos during office hours
- Varying levels of expertise between mentees
- Tips & tricks straight from a subsystem maintainer
- Getting in touch; email/linux-kernel-mentees mailing list, Discord, Zoom, forums, and IRC
- Tips on finding bugs, and highlighting potential difficulties that might be associated with certain subsystems

## What I wish were different

- "My-tasks" dashboard could use a little love, and some clarification on acceptable file formats for uploads

## Setting yourself up for success

In order to get the best out of this learning opportunity, you really have to put both the time and effort into it. You get to choose whether you want to commit full-time or part-time (3 months vs 6 months), with the option to switch should time allow for it

- Respect others' time
- Go through mailing lists to get a feel of how the process goes, and how interactions should be
- Take the opportunity to network with developers/maintainers within the appropriate and acceptable bounds
- Stay organized; use task-tracking software if you have to so you're better at tracking your pending tasks and what have you
- Proofread your patches' subjects and descriptions, and make sure to include the relevant maintainers and/or lists in your submissions
- Carefully read the relevant kernel documentation on submitting patches, general do's and don't's, and the technical requirements for building custom kernels

## Simple yet powerful things to do to expedite your research

Let's face it, the Linux kernel source code is nothing short of humongous. Like where do you even begin, right? Ideally, you'd begin with the documentation—at least get familiar with the various subsystems that go into making the kernel what it is. Once you've familiarized yourself with the main subsystems, and where they can be found, you're gonna wanna get familiar with using `ctags` and/or `cscope`. Both are in the `make` targets for Linux, and I cannot recommend generating them enough! I don't know about other editors, but I use VIM, and it's an absolute best friend where navigating source code is concerned—especially with `ctags` involved! Once you have the dependencies installed for ctags (I personally use `universal-ctags`) and `cscope`, you're pretty much set for generating said target(s) with `make cscope` and/or `make tags`

Navigating source code with tags is a breeze with VIM. Imagine seeing a function called `driver_verify_signature` in `drivers/some_driver/some_file.c`. How do you find the function declaration or prototype? The most ineffecient way would be `grep -r driver_verify_signature drivers/some_driver/`. Reason why this is highly inefficient is because there's a whole lotta expensive IO taking place. What you're basically doing is opening each file in the specified directory/directories for reading, and looking for pattern matches on the fly

A better approach would be using `git grep` instead. It's a lot faster because you're searching indexed registry. You can also fine-tune your expressions with and/or operators; more on that in `man git grep`

Even better approaches would be making the most out of `ctags` and/or `cscope`. They're more or less similar in principle, but `cscope` is especially nice in the way it operates; it's interactive, and you can really fine-tune your search based on the nature of what it is you're looking for. You can also use it as a means to `grep` patterns all within the same interface. Pretty neat!

## Achieving faster build (and boot) times

I, for one, hate waiting for kernel builds as much as the next person does. If you're YOLOing your way through live-testing builds on metal, waiting for builds to finish is practically inevitable. Ideally, you want to make sure whatever patches you're about to submit don't break your live system. However, not all patches are created equal—you may not always want to test certain things on your metal setup. For example, you may be working on a patch for `btrfs` while you're using `ext4`, for example. It would be mad to have to switch up the entire file system between tests. That almost entails reinstalling your distro at least before you begin testing and after. Not ideal

That's where hypervisors come in handy. QEMU being the most common star of the show, but there's a new contender in town, and it delivers on the promise of speed! That contender is `virtme-ng`. If your patches are straight-forward enough, you can get away with having `vng` perform a minimal kernel build, and boot into the new kernel in as little time as possible. The trick is in the teensy-tiny init system. It's an absolute time-saver, and you'll really appreciate how it improves your workflow by a margin of |________________________________| this much. Trust me, you'll absolutely love it. A little gotcha there, however, is that you may or may not notice sporadic freezes/hangs as the [micro]VM boots up. If you do experience such hangs, pass `--disable-microvm`. My go-to invocation post-build is `vng -vp 2 -m 4G --disable-microvm --debug`; that is 2 CPU cores, 4G of RAM, verbosity (so you can read the kernel ring buffer as the VM boots up), and a GDB server for when you need to break into the kernel and do all sorts of debugging chicanery

## Programming language(s) of choice

It's no secret that C is the defacto language of choice for this program, and for generally having a good time understanding how the kernel works. I was accepted into this mentorship very shortly after I'd _just_ learned a _little bit_ of C, which honestly had me in a weird position. On the one hand, I could read and understand the code better, but I definitely lacked the nitty-gritty specifics and nuances of not only the language, syntactically speaking, but also kernelspace rules. However, it also forced me to actually learn and practice the language faster than trying to absorb it all in via books

If C's not exactly your cup of tea, there's Rust! There's a lot of work to be done in Rust, and contributions are always welcome. If neither of those choices tickle your fancy, you can make use of your experience with shell scripting and/or Python to contribute to kselftests and/or some of the custom tooling

If none of that does it for you, the simplest you can do without any programming knowledge whatsoever is contributing to documentation! We're not talking about simple typo fixes, however, but rather meaningful fixes to documentation. That is: fixing warnings/errors with documentation, _adding_ documentation where applicable, and/or translating existing documentation into other languages

Don't sweat it if you don't have any "programming-related" patches to offer though—meaningful contributions still count where applicable, and you're ultimately contributing whatever you can to make this project better for everyone else. That's quite something!

## My choice of tools

Here's a list of the tools I personally used during this mentorship:

- gdb/gdb-peda: debugging
- universal-ctags and cscope: navigating C code quickly
- virtme-ng: faster builds and kernel virtualization for testing
- neomutt: reading and sending emails, and reading mailing lists
- VIM: keep it simple, and Electron-free—the faster your text editor is, the faster _you_ are
- YouTrack: project management solution by JetBrains (a better alternative to Atlassian's JIRA imo for local deployment)

## Closing thoughts

If you're an aspiring kernel developer, or you generally just have a knack for debugging and fixing stuff, this program will set you up for success. Don't feel discouraged if you can't meet the graduation requirements in time; what matters is you learn what you're there to learn. There's a fair bit I'm selectively leaving out of this blog, and that is for you to discover should you choose to join any of the upcoming mentorship programs. Don't be afraid to ask questions during office hours; that's literally the reason why they're held

Once you've successfully completed your enrollment tasks and been enrolled into the program, you'll be required to write a blog like this one, with a chance to be featured on the LKMP (Linux Kernel Mentorship Program) website. There's a lot of learning ahead, and I recommend you get it all while you can. Links to Discord/Zoom will be provided to accepted mentees via email, so I will not be sharing those here. Everything you need for the mentorship will also be provided via email and/or during office hours, so don't worry about those either

## Acknowledgements

I owe a great deal of where I am today to the following amazing people:

- Kali Linux/Debian:
  - Steev Klimaszewski
  - Joseph O'Gorman
  - Arnaudr Rebillout
  - Ben Wilson
  - Sophie Brun
  - Raphael Hertzog
- The Linux Foundation:
  - Linus Torvalds
  - Shuah Khan
- Intel:
  - Saket Dumbre
  - Andy Shevchenko
- Pentesterlab:
  - Louis Nyffenegger
- OffSec:
  - [Redacted] :P love y'all

Be it changelogs, commit messages, public or private discussions, or emails—each one of the aforementioned individuals has helped me one way or another, directly or otherwise, get where I am today. I am grateful and honored to have gone through this journey with their continuous, unwavering support

## Helpful links

- [https://lore.kernel.org](https://lore.kernel.org)
- [https://docs.kernel.org](https://docs.kernel.org)
- [https://kernelnewbies.org](https://kernelnewbies.org)
- [https://wiki.linuxfoundation.org/lkmp](https://wiki.linuxfoundation.org/lkmp)
- [https://mentorship.lfx.linuxfoundation.org](https://mentorship.lfx.linuxfoundation.org)
