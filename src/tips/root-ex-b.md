# Exhibit B: O' Filesystem, where art thou?

## Enter: rm -rf / --no-preserve-root

We've all seen it, and we've laughed at all the memes, but we cannot disregard the fact that it is a very real thing that can (and will) happen to everyone. I'm not talking specifically about `rm -rf / --no-preserve-root`, but the fact that everyone's bound to accidentally delete something. Let's destroy some filesystems, shall we?

I will be demoing removing the entire filesystem inside the ephemeral container, and that's enough to demonstrate the extent of the damage caused. Apply this to any directory that shouldn't have been mistakenly deleted in the first place, and you get the point

<img class="center" alt="Screenshot showing the filesystem being completely destroyed by executing rm -rf --no-preserve-root /" src="../../assets/img/screenshots/tips/root-ex-b.png"/>
