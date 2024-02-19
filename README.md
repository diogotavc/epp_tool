## What is this?

### epp_tool

[epp_tool](https://github.com/diogotavc/epp_tool) ~~(very creative, indeed)~~ aims to provide a very simple service to set, during boot, the epp bias to something more conservative (defaults to performance), and to provide a very quick way to toggle between modes afterwards.

### EPP bias?

Kernel 6.3 now supports a new scalling driver, amd_pstate=active (read about it on the [Linux Kernel documentation](https://docs.kernel.org/admin-guide/pm/amd-pstate.html#active-mode) or on the [ArchWiki](https://wiki.archlinux.org/title/CPU_frequency_scaling#amd_pstate)), on modern AMD ~~(and Intel, but I don't really care to adapt my script)~~ processors. The TL;DR is that the driver provides a hint to the hardware whether to bias toward performance or energy efficiency.

Whilst there's also a new guided mode, I've seen better results with the slightly older active mode. My testing was very superficial and rudimentary, so take it with a grain of salt.

### Why would I need it?

Whilst I haven't tested on other machines, I've seen tangible gains in battery life on my machine, so I decided to make a repository for it. Some improvements and updates may or may not come, so use this as a base, if you need it for yourself.

## How to get it started?

### Enabling the correct scaling driver

Before beginning, make sure you've enabled the correct scaling driver:

```cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver```

If the output is `amd_pstate_epp`, you're all set. If not, you will have to add the correct kernel argument for it to be enabled. You can enable it manually, for GRUB, or use `grubby --update-kernel=ALL --args="amd_pstate=active"` (not forgetting to remove the old arguments). The procedure is similar for EFISTUB, systemd-boot and so on.

### Moving and enabling the service

Time to copy the [unit file](epp_tool.service) to the correct location (`/etc/systemd/system/` for Fedora). If you want to change the bias to something other than `balance_power`, make sure to modify the service accordingly.

To make sure systemd is aware of the new service, reload its service files with `systemctl daemon-reload`. You may now enable the service with `systemctl enable epp_tool`.

To verify it's working properly, you can reboot or start the service with `systemctl start epp_tool`. The output of `cat /sys/devices/system/cpu/cpufreq/policy0/energy_performance_preference` should now match what was set by the service.

### Changing the bias after booting

If you don't want to mock around with the terminal every time you want to toggle between modes, you can use the [script](epp_tool). It's very readable, as it uses zenity to display a simple GUI. Create a custom keyboard shortcut pointing to it, for quick access.

If you do want to mock around with the terminal, then just `echo $epp | sudo tee /sys/devices/system/cpu/cpufreq/policy*/energy_performance_preference`, epp being whatever mode you want.

---

### Missing features:

- It currently only works for my machine, and may not properly work on others (i.e. if they have a different CPU model, firmware or kernel).
- More power related features, not possible to implement with Zenity, like battery charging threshold control and power draw monitoring, among others.
- A config file for better control over the default modes, for when certain conditions are met and for the other features, which are yet to be implemented.

### Available modes:

You may check using `cat /sys/devices/system/cpu/cpufreq/policy0/energy_performance_available_preferences/`. However, for my machine, I've got the following:

```default performance balance_performance balance_power power```

### Tested on:

- Lenovo ThinkPad E14 Gen 3 (20Y7CTO1WW)
    - Fedora Workstation 39
    - AMD Ryzen 7 5700U