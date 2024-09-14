# Exhibit E: Can't see me

## About DISPLAY

Let's talk about environment variables once more, specifically the `DISPLAY` variable this time. This variable, when set, tells programs what the current display is so they can draw their graphics on _that_ display. Easy, right? Usually, it's set to `:0`, but it can also be something like `:0.0`. You can also have multiple displays, and instruct each program to draw its graphics by setting the `DISPLAY` variable beforehand. So who has the authority to those displays? The user running them!

After booting up the system, you're greetd (SWIDT?) with the login manager where you're prompted for your username and password. Once you're logged in, and you can already see the desktop environment, your username with which you logged in now controls that display that's set. It would be such a shame if someone else were to login, remotely, to your system as another user and be able to just see what you're looking at, right? Not only is it a massive security breach, obviously, but it also doesn't make sense. Well, what if _you_, being _physically_ on the computer, switch users to root, and try running graphical applications? It's still the same. From your system's perspective, those are still very much two completely different users

This one is going to be very short and straight forward: a few examples of programs that can and will break when trying to run everything as sudo/root

## IMPORTANT: Browser security

I believe I'd already alluded to that bit earlier, but I'm going to go ahead and emphasize it once more: browsers should __never__ be run as root. Your browser is your gateway to the World ~~Wide~~ Wild Web. You'll see this in action as we try to run Chromium as a privileged user shortly

<img class="center" alt="Screenshot showing graphical programs failing to launch in a privileged context as the DISPLAY environment variable is not propagated" src="../../assets/img/screenshots/tips/root-ex-e.png"/>

## General word of caution

Whenever possible, and unless otherwise _explicitly_ stated, do not try running graphical programs as a privileged user. One more thing to keep in mind is that not all environment variables are propagated to/shared with other users. This latter point explains why you can't just open or see graphical programs run as root/sudo since their `DISPLAY` environment variable is not set to your logged in user's
