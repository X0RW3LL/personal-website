# Telltale signs of broken ACPI

## Preface

You've just installed a Linux distro on bare metal. You just made it to the GRUB menu, or systemd-boot, or whatever bootloader of your choice, selected the kernel to boot, and were ready to rock and roll

Depending on your boot parameters, bootloader, plymouth, or logging preferences, you may or may not have noticed one or more along the lines of the following:
```
ACPI Warning: \_SB.PCI0.GPP0.PEGP._DSM: Argument #4 type mismatch - Found [Buffer], ACPI requires [Package] (20230628/nsarguments-61)
tpm_crb MSFT0101:00: [Firmware Bug]: ACPI region does not cover the entire command/response buffer. [mem 0xbd6bb000-0xbd6bbfff flags 0x200] vs bd6bb000 4000
tpm_crb MSFT0101:00: [Firmware Bug]: ACPI region does not cover the entire command/response buffer. [mem 0xbd6bf000-0xbd6bffff flags 0x200] vs bd6bf000 4000
ACPI Error: Divide by zero (20230628/utmath-478)
ACPI Error: Aborting method \_SB.ATKD.WMNB due to previous error (AE_AML_DIVIDE_BY_ZERO) (20230628/psparse-529)
```

The above output may or may not match your system's; different manufacturers do things differently. You may get similar output, no ACPI-related errors/warnings at all (lucky you, but that's _probably_ never going to be the case; adhering to the specification may not always be 100% to the dot, unfortunately), _different_ errors/warnings, or a different number of warnings/errors

In this section, we are going to explore how to conduct research, dump ACPI tables, disassembly, compiling modified/custom tables, and upgrading the new tables via initrd

## How do I know my ACPI tables are broken?

Besides the example output above, you might notice unexpected behaviors on Linux as opposed to an otherwise smooth-running system on Windows. You might notice:

- Severly degraded battery life
- Always-on/loud laptop fans
- Overheating
- TPM issues
- LEDs not behaving as expected
- General hardware-related issues
- Explicit BIOS errors/firmware bugs/broken ACPI output in `dmesg`

This is only a list of potential issues that might be [in]directly ACPI-related. There is not a silver bullet to all issues, however, as you will soon get to understand why. There might be some trivialities that the end-user might be able to address with time, patience, effort, and experience, and there may be issues whose solutions may never see the light of day unless manufacterers step in. Unlike common, higher-level programming languages where function and variable names may hint or downright explicitly say what they do/mean, ASL (ACPI Source Language), however, is different. The ACPI specification dictates what the syntax must, should, must not, or should not be like. For instance, consider the following function name `brightness_ctl_lvl`: pretty self-explanatory, right? If we were to infer what the function did just by reading its name, we'd probably have a good guess that it would return the brightness control levels, or something along those lines. In ASL, however, that control method is called `_BCL`. ACPICA (ACPI Component Architecture) project's tools include a disassembler that can help annotate some of the device-specific methods, devices, and so on. The real limitation is obscure methods that only the manufacturer knows about; take `P8XH`, for instance...What in the fresh blue hell does it mean? Who knows. You're bound by:

- The manufacturer's obscurity
- Your imagination
- Your ability to unpack acronyms within the right context
- Your ability to reverse-engineer

Some of those 4-character method names might be easier to identify than others, like `NTFY`; simply notify. Others? Maybe not so much. This is why we need to accept the fact that not everything will be easy, if at all possible, to address

What we will be doing here is essentially a case study with some of the things I was able to address on my system. You might come out of it learning absolutely nothing, inspired to do your own research, or fixing an issue that's always bothered you. Again I remind you: proceed at your own risk should you wish to chase it down that far
