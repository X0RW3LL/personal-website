# Part 1: Driver

To start things off, we'll keep the debug output in mind because it's a crucial pointer for figuring out what's happening

```
ACPI Warning: \_SB.PCI0.GPP0.PEGP._DSM: Argument #4 type mismatch - Found [Buffer], ACPI requires [Package] (20230628/nsarguments-61)
ACPI Debug:  "------- NVOP --------"
ACPI Debug:  "------- NVOP --------"
ACPI Debug:  "------- NVOP 0x1A --------"
```

It looks like the `NVOP` control method gets called twice, with the second call showing additional debug output with the argument `0x1A`. Why is this significant? We'll see as soon as we look at the `drivers/gpu/drm/nouveau/nouveau_acpi.c` file

```c
// SPDX-License-Identifier: MIT
#include <linux/pci.h>
#include <linux/acpi.h>
#include <linux/slab.h>
#include <linux/mxm-wmi.h>
#include <linux/vga_switcheroo.h>
#include <drm/drm_edid.h>
#include <acpi/video.h>

#include "nouveau_drv.h"
#include "nouveau_acpi.h"

#define NOUVEAU_DSM_LED 0x02
#define NOUVEAU_DSM_LED_STATE 0x00
#define NOUVEAU_DSM_LED_OFF 0x10
#define NOUVEAU_DSM_LED_STAMINA 0x11
#define NOUVEAU_DSM_LED_SPEED 0x12

#define NOUVEAU_DSM_POWER 0x03
#define NOUVEAU_DSM_POWER_STATE 0x00
#define NOUVEAU_DSM_POWER_SPEED 0x01
#define NOUVEAU_DSM_POWER_STAMINA 0x02

#define NOUVEAU_DSM_OPTIMUS_CAPS 0x1A
#define NOUVEAU_DSM_OPTIMUS_FLAGS 0x1B
```

Sure enough, there are DSM-related definitions, and then there's `#define NOUVEAU_DSM_OPTIMUS_CAPS 0x1A`

NOUVEAU...DSM...OPTIMUS...0x1A\
**N**ou**V**eau...**OP**timus...0x1A

`NVOP 0x1A`! Sounds very promising, no? Let's go ahead and search for instances of the previously defined constant in the file. First hit we get looks like the following

```c
/* Must be called for Optimus models before the card can be turned off */
void nouveau_switcheroo_optimus_dsm(void)
{
	u32 result = 0;
	if (!nouveau_dsm_priv.optimus_detected || nouveau_dsm_priv.optimus_skip_dsm)
		return;

	if (nouveau_dsm_priv.optimus_flags_detected)
		nouveau_optimus_dsm(nouveau_dsm_priv.dhandle, NOUVEAU_DSM_OPTIMUS_FLAGS,
				    0x3, &result);

	nouveau_optimus_dsm(nouveau_dsm_priv.dhandle, NOUVEAU_DSM_OPTIMUS_CAPS,
		NOUVEAU_DSM_OPTIMUS_SET_POWERDOWN, &result);

}
```

Something I haven't mentioned earlier is that if you read the entire kernel ring buffer (`dmesg`), line by line, you'll find references to `switcheroo` which can definitely add more context to your search when hunting for things in the kernel. For example, consider the following output

```
$ dmesg -t | grep 'DSM'
VGA switcheroo: detected Optimus DSM method \_SB_.PCI0.GPP0.PEGP handle
nouveau: detected PR support, will not use DSM
```

We already have a good head start in that we know those are related, and it's something to do with the kernel driver sending an unexpected data type (Buffer) to the PEGP Device-Specific Method. Additionally, the reason I mentioned it being a non-issue is that `_DSM` is not even used since a **P**ower **R**esource was detected. For context, `switcheroo`, as the name implies, handles switching graphics. With all that being said, let's navigate to the `nouveau_optimus_dsm` function

```c
static int nouveau_optimus_dsm(acpi_handle handle, int func, int arg, uint32_t *result)
{
	int i;
	union acpi_object *obj;
	char args_buff[4];
	union acpi_object argv4 = {
		.buffer.type = ACPI_TYPE_BUFFER,
		.buffer.length = 4,
		.buffer.pointer = args_buff
	};

	/* ACPI is little endian, AABBCCDD becomes {DD,CC,BB,AA} */
	for (i = 0; i < 4; i++)
		args_buff[i] = (arg >> i * 8) & 0xFF;

	*result = 0;
	obj = acpi_evaluate_dsm_typed(handle, &nouveau_op_dsm_muid, 0x00000100,
				      func, &argv4, ACPI_TYPE_BUFFER);
	if (!obj) {
		acpi_handle_info(handle, "failed to evaluate _DSM\n");
		return AE_ERROR;
	} else {
		if (obj->buffer.length == 4) {
			*result |= obj->buffer.pointer[0];
			*result |= (obj->buffer.pointer[1] << 8);
			*result |= (obj->buffer.pointer[2] << 16);
			*result |= (obj->buffer.pointer[3] << 24);
		}
		ACPI_FREE(obj);
	}

	return 0;
}
```

We can see that `argv4` is indeed an `ACPI_TYPE_BUFFER` object, which is passed to the Device-Specific Method via the call to `acpi_evaluate_dsm_typed`. The returned object is a buffer whose pointers are bitshifted and bitwise OR-assigned. Here's the thing: when you're sending data to/from ACPI, data types should match. That is, if you send a buffer object, you're expecting to receive a buffer object. If we recall the warning message, we did indeed send a buffer object, but the Device-Specific Method, specifically for argument #4, the expected data type was supposed to be a package object. The problem is, however, that the returned data type is a _buffer_ object, and the object the method expects to receive should be a _package_ object

So, how do we address this? It's simple, really; change the data type we're sending, right? Not so much...For us to understand how our data is being sent, we must look at how `acpi_evaluate_dsm_typed` works. If you already have experience compiling the kernel, then you probably know how indespinsable tags are. If you don't know what that is, `ctags -R 2>/dev/null` at the root of the Linux repo will generate those tags for you. This is how we can jump back and forth between function declarations. With that in mind, and whilst browsing the file using Vim, we can position the cursor on the function, and press `Ctrl+]` to jump to its declaration (which may or may not be in the same file). In this case, `acpi_evaluate_dsm_typed` is defined in the `acpi_bus.h` header file

```c
acpi_evaluate_dsm_typed(acpi_handle handle, const guid_t *guid, u64 rev,
			u64 func, union acpi_object *argv4,
			acpi_object_type type)
{
	union acpi_object *obj;

	obj = acpi_evaluate_dsm(handle, guid, rev, func, argv4);
	if (obj && obj->type != type) {
		ACPI_FREE(obj);
		obj = NULL;
	}

	return obj;
}
```

Judging by the name, and what the function does, it calls the `acpi_evaluate_dsm` function with `argv4` passed as an argument, storing the result in `obj`. Immediately after, it checks whether 1) an object was returned, and 2) if its type does _not_ match the type we sent in the initial call, in which case it would free the object and return NULL. So we know we can't use this function because we'll be sending a package and receiving a buffer in exchange, thereby nullifying the returned object (if any). In our case, we already know what to expect out of this exchange, and we _mean_ for it to be processed that way. This function gave us a very good pointer, and that is using the `acpi_evaluate_dsm` function instead since it doesn't perform any type checking

The tricky bit here is figuring out how we can send a _packaged_ buffer object. That is, a package object, containing a buffer object. Quickest way to draw inspiration is to `git grep ACPI_TYPE_PACKAGE`, and go after various function declarations in hopes of finding one that has more or less the same scenario. This process can be a bit time-consuming, and so for brevity's sake, I will leave it to you as an exercise

The solution is relatively simple once you get things in the right order. I struggled with this bit myself because of my extremely limited knowledge of C, which was when I had to read a bit more at least to understand basic concepts. Let me walk you through the steps needed:

- Keep the buffer `obj` as-is; we're still sending that
- Declare a new package object, containing one element _pointing_ to the buffer object
- Use `acpi_evaluate_dsm` instead of `acpi_evaluate_dsm_typed`, with argument #4 pointing to the newly declared package object as opposed to the `argv4` buffer object

Let's see the modified code. I will use `diff` formatting here just so the changes are easier to identify

```diff
static int nouveau_optimus_dsm(acpi_handle handle, int func, int arg, uint32_t *result)
{
	int i;
	union acpi_object *obj;
	char args_buff[4];
	union acpi_object argv4 = {
		.buffer.type = ACPI_TYPE_BUFFER,
		.buffer.length = 4,
		.buffer.pointer = args_buff
	};
+   union acpi_object pkg = {
+      .package.type = ACPI_TYPE_PACKAGE,
+      .package.count = 1,
+      .package.elements = &argv4
+   };

	/* ACPI is little endian, AABBCCDD becomes {DD,CC,BB,AA} */
	for (i = 0; i < 4; i++)
		args_buff[i] = (arg >> i * 8) & 0xFF;

	*result = 0;
+   obj = acpi_evaluate_dsm(handle, &nouveau_op_dsm_muid, 0x00000100,
+		func, &pkg);

	if (!obj) {
		acpi_handle_info(handle, "failed to evaluate _DSM\n");
		return AE_ERROR;
	} else {
		if (obj->buffer.length == 4) {
			*result |= obj->buffer.pointer[0];
			*result |= (obj->buffer.pointer[1] << 8);
			*result |= (obj->buffer.pointer[2] << 16);
			*result |= (obj->buffer.pointer[3] << 24);
		}
		ACPI_FREE(obj);
	}

	return 0;
}
```

That's it, really. At this point, we can commit the changes changes and recompile the entire kernel, or simply compile and install only the nouveau module. We are not done yet, however; we MUST modify the ACPI table to inform it of the new changes. More on that in the next part, so let's get on with it
