# Source code analysis

Let's analyze the source code for the `tpm_crb` driver to get a better understanding of the flow. Immediately following the `crb_fixup_cmd_size` function [shown earlier](addressing-tpm.md) is the `crb_map_io` function, which, as the name implies, maps the IO resources for the command response buffer. I will only include snippets that are immediately relevant to what we're researching. This is _not_ the complete function declaration

```c
/* 
 * drivers/char/tpm/tpm_crb.c
 * Annotated functions are there for later cross-referencing
 */

static int crb_map_io(struct acpi_device *device, struct crb_priv *priv, // ------> crb_map_io() {
		      struct acpi_table_tpm2 *buf)
{
	struct list_head acpi_resource_list;
	struct resource iores_array[TPM_CRB_MAX_RESOURCES + 1] = { {0} };
	void __iomem *iobase_array[TPM_CRB_MAX_RESOURCES] = {NULL};
	struct device *dev = &device->dev;
	struct resource *iores;
	void __iomem **iobase_ptr;
	int i;
	u32 pa_high, pa_low;
	u64 cmd_pa;
	u32 cmd_size;
	__le64 __rsp_pa;
	u64 rsp_pa;
	u32 rsp_size;
    int ret;

    ret = __crb_cmd_ready(dev, priv);                                    // ------> __crb_cmd_ready();
    if (ret)
      goto out_relinquish_locality;

    pa_high = ioread32(&priv->regs_t->ctrl_cmd_pa_high);
    pa_low  = ioread32(&priv->regs_t->ctrl_cmd_pa_low);
    cmd_pa = ((u64)pa_high << 32) | pa_low;
    cmd_size = ioread32(&priv->regs_t->ctrl_cmd_size);

    iores = NULL;
    iobase_ptr = NULL;
    for (i = 0; iores_array[i].end; ++i) {
    }

    if (iores)
      cmd_size = crb_fixup_cmd_size(dev, iores, cmd_pa, cmd_size);       // ------> _dev_err() {

    dev_dbg(dev, "cmd_hi = %X cmd_low = %X cmd_size %X\n",               // ------> dev_printk_emit() {
      pa_high, pa_low, cmd_size);

    priv->cmd = crb_map_res(dev, iores, iobase_ptr, cmd_pa, cmd_size);   // ------> crb_map_res() {
    if (IS_ERR(priv->cmd)) {
      ret = PTR_ERR(priv->cmd);
      goto out;
    }

	memcpy_fromio(&__rsp_pa, &priv->regs_t->ctrl_rsp_pa, 8);             // ------> memcpy_fromio();
	rsp_pa = le64_to_cpu(__rsp_pa);
	rsp_size = ioread32(&priv->regs_t->ctrl_rsp_size);

	iores = NULL;
	iobase_ptr = NULL;
	for (i = 0; resource_type(iores_array + i) == IORESOURCE_MEM; ++i) {
		if (rsp_pa >= iores_array[i].start &&
		    rsp_pa <= iores_array[i].end) {
			iores = iores_array + i;
			iobase_ptr = iobase_array + i;
			break;
		}
	}

	if (iores)
		rsp_size = crb_fixup_cmd_size(dev, iores, rsp_pa, rsp_size);     // ------> _dev_err() {

	if (cmd_pa != rsp_pa) {
		priv->rsp = crb_map_res(dev, iores, iobase_ptr,                  // ------> crb_map_res() {
					rsp_pa, rsp_size);
		ret = PTR_ERR_OR_ZERO(priv->rsp);
		goto out;
	}
}
```

At a very high level, only zooming into functions immediately preceding the calls to `crb_fixup_cmd_size`, the following takes place:

- Request tpm crb device to enter ready state
- Validate IO resources (command). If not NULL, fix up the command size (first call to `_dev_err()`)
- Emit device debug `prinkt` statement (command)
- Map command buffer resources
- Copy memory area from IO (response)
- Validate IO resources (response). If not NULL, fix up the response size (second call to `_dev_err()`)
- Map response buffer resources

The basic idea here is that 1) the ACPI region should cover the entire command response buffer as reported by the registers, and 2) command and response buffer sizes must be identical

We now have a good enough reference that will help us trace these functions, or rather `crb_map_io()` specifically, during bootup. Question is: how do you debug something to which you don't have access? That's where the beauty of dynamic debugging comes into play. We'll instruct `ftrace` to trace kernel-space functions (remember: TPM gets initialized very early on), filtering that one function we're after, and save a snapshot of the output to the filesystem once it's ready. That's what we'll be doing next!
