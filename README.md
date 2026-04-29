# nixos-xmonad

My personal NixOS and XMonad configuration.

## Structure

- **nixos/** — system configuration used for `/etc/nixos`
- **xmonad/** — XMonad configuration used for `~/.xmonad`

## Usage

Clone the repo:

```bash
git clone git@github.com:Vittalbigamudra/nixos-xmonad.git ~/dotfiles
```

Link the configs:

```bash
sudo ln -sf ~/dotfiles/nixos /etc/nixos
ln -sf ~/dotfiles/xmonad ~/.xmonad
```

Rebuild NixOS:

```bash
sudo nixos-rebuild switch
```

Restart XMonad:

```bash
xmonad --recompile && xmonad --restart
```


## Handling kworkers on NixOS

In Linux, `kworkers` (kernel workers) are background threads that perform various tasks for the kernel, such as handling interrupts, I/O, or system timers. Because they are kernel-level processes, they generally cannot—and **should not**—be "killed" like a standard user-space application using `kill -9`.

On NixOS, attempting to manage kernel behavior declaratively involves configuring the kernel parameters or hardware settings rather than targeting the threads themselves. 

---

### Why are kworkers spiking?
If you are seeing high CPU usage from `kworker`, it is usually a symptom of a hardware interrupt or a specific driver working overtime. Common culprits include:
1.  **ACPI Interrupts:** Frequent "GPE" (General Purpose Events).
2.  **Disk I/O:** High wait times on storage.
3.  **Polling:** Drivers checking hardware status too frequently.

---

### Method 1: Disabling Buggy ACPI Interrupts (Common Fix)
If a specific interrupt is causing a `kworker` storm, you can disable it via a boot parameter. First, identify the culprit by running `grep . /sys/firmware/acpi/interrupts/gpe*`. 

Once found (e.g., `gpe13`), add this to your `configuration.nix`:

```nixos
boot.kernelParams = [ "acpi_mask_gpe=0x13" ];
```

---

### Method 2: Managing Power/Performance (The "Throttle" Approach)
If the workers are related to power management or frequency scaling, you can declaratively adjust the CPU governor or power saving features.

```nixos
powerManagement.cpuFreqGovernor = "powersave"; # or "performance"
services.tlp.enable = true; # Advanced power management
```

---

### Method 3: Blacklisting Drivers
If the `kworker` activity is tied to a specific piece of hardware you don't use (like a buggy webcam or card reader), you can prevent the driver from loading:

```nixos
boot.blacklistedKernelModules = [ "module_name" ];
```

---

### Method 4: Real-time Kernel (For Determinism)
If you need to control how kernel tasks are prioritized relative to user tasks, you can switch to a kernel with the `preempt-rt` patches. This doesn't "kill" them, but it changes how they are scheduled.

```nixos
boot.kernelPackages = pkgs.linuxPackages_rt;
```

---

### Summary Warning
> [!CAUTION]
> Forcefully stopping kernel threads is not possible via standard process signals because it would likely result in an immediate **Kernel Panic** or a frozen system. The declarative NixOS approach is to address the *source* of the work (drivers, interrupts, or power settings) rather than the workers themselves.
