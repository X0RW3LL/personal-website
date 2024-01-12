# ACPI

## So what the heck is that character soup ACPI?

ACPI stands for **A**dvanced **C**onfiguration and **P**ower **I**nterface

## Okay, but what _is_ ACPI?

> ACPI can first be understood as an architecture-independent power management and configuration framework that forms a subsystem within the host OS. This framework establishes a hardware register set to define power states (sleep, hibernate, wake, etc). The hardware register set can accommodate operations on dedicated hardware and general purpose hardware.[^note]

## Simpler...

To put it very simply, it's the subsystem that handles your computer's power management. If you've ever wondered how LEDs, fans, and other devices/controllers know when to turn on/off, return their status, and a host of other functions, that's ACPI for you. It lets you, well not __you__; the operating system, and you by association, more or less interface with the motherboard

## Interface with the motherboard, you say?

Yes. This brings us to this much needed

<div class="warning" style="color:red">
<strong>DISCLAIMER</strong>: This section is considered intermediate-to-advanced level. If you are unsure what any of this is, please do not attempt messing with your ACPI tables. If you are not careful, you might break your distro at best, and/or cause damage to your hardware at worst. I am not liable or responsible for anything you choose to do at your own risk. You have been warned.
</div>

Now that we got the formalities out of the way, let's dive in

[^note]: [https://uefi.org/specs/ACPI/6.5/Frontmatter/Overview.html](https://uefi.org/specs/ACPI/6.5/Frontmatter/Overview.html)
