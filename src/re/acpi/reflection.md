# Reflection

So far, we've been able to address not one, but two ACPI-related warnings. I've largely left out some explanations/gotchas, and I did so on purpose for the following reasons:

- Remember: this is not a guide, but a writeup on how I addressed some issues related to _my_ hardware, which may very well be different from yours
- I can't claim to be an expert on any of this, so I'd rather not deliver wrong information based on my own limited knowledge/interpretation
- You'll come across a wealth of <u>accurate</u> information by following the linked documentaion/specification

When I started this journey, I had to go through the process of finding information, hunting for resources, lots of trial and error, and many a sleepless night. Ultimately, however, it was really worth it

My main goal with this project is to get anyone interested in doing their own research. If this helps in any way, I'll be very glad. If I get criticised to the bone for it, I'll also be very glad because that means I'll not only learn what I did or understood wrong, but also more people can learn from my mistakes. See, we're not perfect, and that's okay. What matters is how we respond to criticism in a manner that's both healthy and productive

Moving forward, we now have a couple of issues still dangling, one of which involves making a certain change to the nouveau_acpi driver in the kernel source code. I don't do C myself, but I've had to learn just enough C for me to able to go forward with addressing the aforementioned issue. Ideally, however, we don't want half-baked solutions; we want solid, verifiable solutions. I am still learning though, and there's a lot of value for me in doing things this way I am, so the tradeoff is justifiable for me. We'll stop by TPM first, however, before we can get to nouveau

Let's go ahead and continue on to addressing TPM
