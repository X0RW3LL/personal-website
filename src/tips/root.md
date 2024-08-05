# The Root of All Evil

## Prologue

The year is 2024 AD. We have flying cars (sort of), Quantum technology, potential cures for cancer, new and powerful space telescopes, and users still use sudo/root for EVERYTHING. It's as if the entire world moved on, and a significant subset of users decided to stick to the past and horrible opinions/advice some _other_ misinformed users left some 5, 10, or 15 years ago. If you're given this link, you __need__ to read on. Stick around because there _are_ real demos which will deliver the point straight to your proverbial doorstep

## What the hell is root, sudo, or any of that witchcraft?

Let's think of your operating system as a car for a moment. For the sake of simplicity, we're doing good ol' pre-2000s tech, so no Engine Start/Stop buttons, nothing fancy; as simple as it is to insert the key and switch. As an "unprivileged user", you're free to turn on the cabin lights, turn the steering wheel to a certain degree, move a few things around here and there, but not much beyond that. What if you need to, say, start moving? Well, you switch the key to ignition, start the car and off you go. That right there is a "privileged" operation, meaning you need to 1) have the key, and 2) actually start the car

Now, imagine how absurd it would be to start the engine just to turn on the cabin lights, only to switch it off after. This is ~~your brain on drugs~~ what you're doing when you do something like `sudo ssh user@host`, `root@kali~:# echo "I am going to do EVERYTHING as root because I can"`

Back to starting the engine just to turn on the cabin lights analogy. Let's think about why that's a bad idea:

- Fuel, oil, and coolant are wasted
- Environment's polluted over something that never warranted that action
- Gears, belts, spark plugs, and any/all parts involved in the process are worn out (however insignificant this might be, it ultimately adds up on the long run)
- Battery life's degraded
- ... (you get the idea by now)

Now, let's leave the garage and come back to technicalities. Remember: turning on the cabin lights is any action your unprivileged user can do, and starting the engine and driving is anything your privileged user(s) can perform. In techy-techy terms, privileged actions require privileged access

But what _are_ privileged actions? Here are a few examples:

- Managing system services
- Modifying system files
- Managing packages
- Sending RAW packets
- Modifying files belonging to other users/groups
- Capturing packets
- [Un]loading kernel modules
- Modifying kernel command-line params
- Modifying boot entries

Anything that does _not_ __require__ privileged access **SHOULD NOT** be given one. Here are some examples:

- Starting the browser to navigate the World Wide Web (if you do this, please throw your laptop out the window until you understand why that's terrible)
- Reading, writing, executing, and/or removing (user|group)-owned files
- SSH, SMB, FTP, TELNET, SMTP, SNMP, HTTP, ... access
- Managing __user__ services
- Running most programs (unless otherwise prompted for privileged access)
- ... (the list goes on)

## What's actually so bad if I perform everything I want while privileged?

Where do we even begin? Let's populate a list of things:

- Dear baddies, here's my entire computer at your disposal, with love xoxo (see [Exhibit A: Mi Kali, su Kali <3](root-ex-a.md))
- Accidental data loss (`sudo rm -rf /oops/i/didnt/mean/to/remove/the/entire/etc/directory111one1eleven1`) (see [Exhibit B: O' Filesystem, where art thou?](root-ex-b.md))
- Broken permissions? Who needs 'em anyway, amirite?
- Wait, wtf is `DISPLAY :0`? And why can't root just "open" it?
- Oh, you killed the wrong PID? Sucks to be you

Let's face it: we __ALL__ make mistakes. Some of these mistakes might be forgivable, and some others...not so much. Deleting a text file that just says "test" is meh, but deleting an entire directory like /usr, /boot, /etc, or whatever else by mistake? That's an unforgivably expensive mistake with no one to blame but yourself. You might think "oh, I definitely know better than all those losers that don't know what they're doing", and you'd still be absolutely wrong. Everyone makes mistakes; you, me, that guy over there, that girl on the other side of the globe, those people studying arctic climate--_everyone_. You're certainly not better, but if you let your ego take you down that joyride, by all means enjoy it until you don't.

And yes, some programs WILL NOT actually work as sudo/root because of the different shell/environment variables available to your user vs sudo/root. The most critical thing that can happen to you, from a security standpoint, is running unknown binaries/code 1) as root, and 2) without FULLY understanding what they do. Sit tight; this is gonna be demoed next

## Ugh, I just hate having to type my password every N minutes

Stop being lazy. I'm not gonna sugarcoat anything here. I'd rather type my complex password 300 times a day than take the risk with any of the aforementioned mishaps/dangers. The "convenience" factor is just not worth it for me. If you absolutely must, maybe extend your token validity (NOT recommended, but up to you), or turn to other means of authentication (again, up to you; we're not covering those here)

## Closing thoughts

You're a supposed "security professional". If I were your client, and you told me that running this web application as root is terrible practice whilst having `root@kali:~#` in your screenshots, I'd never be inclined to do any future business with you. Plain and simple. If you're not practicing what you're preaching, your legitimacy will be questioned. Again, I'm not shying away from calling out bad practices. If you wanna get places in this industry, learn to do things _correctly_. If not for anything else, at least for yourself and your own systems' sake, and more importantly, for doing things the right way. Additionally, this entire topic was already covered some 5 years ago in [this blog post](https://www.kali.org/blog/kali-default-non-root-user) by the Kali dev team. Please make a habit of reading docs and blog posts; they're there to help you
