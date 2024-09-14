# Exhibit F: Merciless killing

## Dive right in

Last, but not least: making typos while killing processes. Imagine you want to kill PID 2944, but ended up killing PID 294 instead? The following happens

<img class="center" alt="Screenshot showing Bash process being killed by mistake and kicking the root user out of their shell" src="../../assets/img/screenshots/tips/root-ex-f.png"/>

Notice how the colors changed? That signifies the login shell, i.e. Bash, was killed and thereby kicking the root user out of its session. Imagine you did this in the middle of something very important, like halfway through upgrading the system, flashing hardware, or what have you; it can be catastrophic, especially when there isn't really anything to ask whether you _really_ meant to kill a specific process
