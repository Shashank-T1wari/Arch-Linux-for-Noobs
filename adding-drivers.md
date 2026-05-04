# Adding Drivers and Hardware Configuration

Here are the detailed steps to configure your hardware (Graphics, Audio, Bluetooth, and CPU Microcode) right after you boot into your newly installed Arch Linux system.

At this point, you should be looking at a basic text login screen (TTY).

---

## 1. First Login and Network Connection

### 1.1 Log into your user account
Type the username you created during installation and press Enter.
Type your password and press Enter.

### 1.2 Connect to the Internet
Since we installed and enabled NetworkManager during the base installation, we can use its text-based interface to connect to Wi-Fi.

Run:
```bash
nmtui
```

*What is nmtui?*
- It stands for NetworkManager Text User Interface.
- It provides a simple, keyboard-navigable menu to select Wi-Fi networks and enter passwords without needing to remember complex commands.

**Steps in nmtui:**
1. Select **Activate a connection**.
2. Find your Wi-Fi network in the list.
3. Select it, press Enter, and type your password.
4. Once connected (a star will appear next to the network), press Esc to go back and select **Quit**.

Test the connection:
```bash
ping -c 4 archlinux.org
```

(Output example)
```bash
PING archlinux.org (95.216.194.135) 56(84) bytes of data.
64 bytes from archlinux.org (95.216.194.135): icmp_seq=1 ttl=53 time=142 ms
64 bytes from archlinux.org (95.216.194.135): icmp_seq=2 ttl=53 time=142 ms
64 bytes from archlinux.org (95.216.194.135): icmp_seq=3 ttl=53 time=142 ms
64 bytes from archlinux.org (95.216.194.135): icmp_seq=4 ttl=53 time=142 ms

--- archlinux.org ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 142.112/142.188/142.234/0.177 ms
```
If you see replies, you are online.

---

## 2. Update the System
Before installing any new packages (drivers), it is critical to ensure your system's package database is up to date.

Run:
```bash
sudo pacman -Syu
```

(Output example)
```bash
:: Synchronizing package databases...
 core                  128.5 KiB   235 KiB/s 00:01 [######################] 100%
 extra                   8.1 MiB  3.90 MiB/s 00:02 [######################] 100%
:: Starting full system upgrade...
 there is nothing to do
```

*What does this do?*
- -S: Synchronize (install or update packages).
- y: Refresh the local package databases from the server mirrors.
- u: Upgrade all installed packages to their latest versions.
- Running this prevents errors where pacman tries to download an older version of a package that no longer exists on the mirror.

---

## 3. CPU Microcode

Check if you have intel-ucode installed.

Run:
```bash
pacman -Q intel-ucode
```

(Output example - If installed)
```bash
intel-ucode 20240312-1
```
*(If it is installed it will show the version number, and you can skip to the next section).*

(Output example - If missing)
```bash
error: package 'intel-ucode' was not found
```
*(If you see this error, you need to install it. Run the command below:)*

Run:
```bash
sudo pacman -S intel-ucode
```

*What is CPU Microcode?*
Microcode is essentially updateable firmware for your processor. The CPU manufacturer (Intel) provides these updates to patch security vulnerabilities (like Meltdown/Spectre) and fix hardware bugs.

**Important:** If you are installing this *after* generating your GRUB configuration, you must tell GRUB to load it.

Regenerate your GRUB configuration:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

(Output example)
```bash
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot: initramfs-linux-fallback.img
Warning: os-prober will not be executed to detect other bootable partitions.
done
```

*Why?* The bootloader (GRUB) needs to be told to load the microcode update into the CPU before the Linux kernel even starts. Generating the config file again will automatically detect the installed intel-ucode and add the necessary boot parameters.

---

## 4. Graphics Drivers (Hybrid Graphics / NVIDIA Optimus)

Your laptop has two GPUs:
1. **Intel UHD Graphics 630**: Low power, good for battery life and basic desktop rendering.
2. **NVIDIA GeForce GTX 1650 Mobile**: High power, needed for gaming or heavy 3D tasks.

This setup is called **NVIDIA Optimus**. We will set it up so the Intel GPU handles the main display, and the NVIDIA GPU is only used when needed (this is called PRIME offloading).

### 4.1 Install Intel Integrated Graphics Drivers
The Intel drivers are largely built into the Linux kernel natively, but we need userspace drivers for features like 3D acceleration (Vulkan/OpenGL).

Run:
```bash
sudo pacman -S mesa vulkan-intel
```

*What are these packages?*
- `mesa`: This is the open-source implementation of OpenGL and Vulkan. It translates graphical instructions from your apps into something your hardware understands.
- `vulkan-intel`: Specifically provides Vulkan API support for Intel Graphics. Vulkan is a modern, faster alternative to OpenGL.

### 4.2 Install NVIDIA Dedicated Graphics Drivers
NVIDIA requires proprietary drivers for optimal performance.

Because we are compiling the driver module from scratch specifically for your kernel, you **must** have the basic development tools (like compilers) installed first.

Run:
```bash
sudo pacman -S base-devel
```
*(Press **Enter** to install all packages in the group).*

Now, install the driver components:
Run:
```bash
sudo pacman -S nvidia-dkms nvidia-utils linux-headers
```

> **IMPORTANT: It is NOT frozen, be patient!**
> When you run this command, it might look like your terminal is completely stuck or hanging at a line that says something like:
> `==> dkms install --no-depmod nvidia...`
>
> **Do not cancel it!** It is not frozen. The `DKMS` package is currently compiling massive amounts of NVIDIA source code specifically for your Linux kernel. Because you are compiling a heavy driver from scratch, this step can take anywhere from **2 to 10 minutes** of pure silence depending on your CPU. Just grab a coffee and wait for the prompt to return.
>
> **Note on "Missing Firmware" Warnings:**
> Towards the end, you might see warnings like `WARNING: Possibly missing firmware for module 'qat_6xxx'` (or `wd719x`, `bfa`, etc.). You can **safely ignore these**. They occur during the `mkinitcpio` step because the generic Linux kernel checks for drivers for enterprise server hardware that your consumer laptop does not actually have.

> **Warning: If pacman truly freezes or hangs during download (Before DKMS)**
> If your terminal gets stuck in the middle of downloading or installing packages, **do not panic**. Here is the safe way to quit and restart:
>
> 1. **Try to cancel:** Press `Ctrl + C`. Wait a few seconds to see if pacman safely aborts.
> 2. **Force quit (if Ctrl+C failed):** Open a second terminal window (or if you are in a pure text screen, press `Alt + Right Arrow` or `Ctrl + Alt + F2` to open a new terminal). Log in, and run:
>    ```bash
>    sudo killall -9 pacman
>    ```
> 3. **Remove the lock file:** Pacman creates a "lock" file to prevent two installations from running at once. Because you forcefully killed it, it left the lock behind. You must delete it before pacman will work again:
>    ```bash
>    sudo rm /var/lib/pacman/db.lck
>    ```
> 4. **Clear corrupted downloads:** Since it hung, the files it was downloading might be broken. Clear the cache:
>    ```bash
>    sudo pacman -Sc
>    ```
>    *(Press `Y` when it asks to remove uninstalled packages).*
> 5. **Retry the installation safely:**
>    ```bash
>    sudo pacman -S nvidia-dkms nvidia-utils linux-headers
>    ```

*What are these packages?*
- `nvidia-dkms`: The main proprietary driver module that builds itself into your Linux kernel.
- `nvidia-utils`: Helper tools and libraries (like Vulkan support for NVIDIA) required for the driver to function.
- `linux-headers`: This is required for `nvidia-dkms` to actually build the driver successfully. Without headers, the DKMS build will fail.

### 4.3 Setting up Wayland Support for NVIDIA
Since your end goal is to use **Wayland** (and Hyprland), NVIDIA needs an extra compatibility layer because Wayland originally didn't support NVIDIA's proprietary way of drawing windows.

Run:
```bash
sudo pacman -S egl-wayland
```

*What does this do?*
It implements EGLStream, which allows NVIDIA's proprietary driver to talk properly with Wayland display servers (like Hyprland).

### 4.4 Configuring PRIME (NVIDIA Optimus)
By the default configuration matching Arch Wiki, installing the
nvidia driver automatically sets up PRIME render offloading for modern NVIDIA cards.
This means the Intel GPU runs your Desktop to save battery, but if you want to run a specific application (like a game) using the powerful NVIDIA card, you launch it with specific environment variables.

Example of how this will be used later in the terminal:
```bash
prime-run <application_name>
```
*(No need to execute this now, this is just for your understanding)*

---

## 5. Audio Drivers (PipeWire)

Linux audio used to be handled by ALSA (low level) and PulseAudio (high level). Today, **PipeWire** is the standard, modern replacement. It is much better at handling both audio and video streams, especially on Wayland, and has lower latency.

Run:
```bash
sudo pacman -S pipewire pipewire-audio pipewire-alsa pipewire-pulse pipewire-jack wireplumber
```

*What are these packages?*
| Package | Purpose |
|---------|---------|
| pipewire | The core multimedia routing server itself. |
| pipewire-audio | The audio processing component of PipeWire. |
| pipewire-alsa | A compatibility layer so older apps expecting ALSA still work. |
| pipewire-pulse | A compatibility layer so older apps expecting PulseAudio still work. |
| pipewire-jack | A compatibility layer for professional audio apps expecting JACK. |
| wireplumber | The session and policy manager for PipeWire. It essentially tells PipeWire what to do (e.g., routing sound to the correct speakers/headphones). |

*(Note: PipeWire uses systemd user services so it typically starts automatically when your user logs in. No need to enable it globally).*

---

## 6. Bluetooth Setup

Your laptop's Intel Bluetooth uses the btusb standard driver in the kernel, but you need the userspace tools to actually pair and connect devices.

Run:
```bash
sudo pacman -S bluez bluez-utils
```

*What are these packages?*
- `bluez`: The official Linux Bluetooth protocol stack. It provides the core bluetoothd daemon.
- `bluez-utils`: Provides command-line utilities like bluetoothctl which you can use to pair devices (though later you can install a GUI app for this).

**Enable the Bluetooth Service:**

Run:
```bash
sudo systemctl enable --now bluetooth.service
```

(Output example)
```bash
Created symlink /etc/systemd/system/dbus-org.bluez.service → /usr/lib/systemd/system/bluetooth.service.
Created symlink /etc/systemd/system/bluetooth.target.wants/bluetooth.service → /usr/lib/systemd/system/bluetooth.service.
```

*What does this do?*
- `enable`: Tells the system to start bluetooth every time the computer boots.
- `--now`: Starts the service immediately right now without needing a reboot.

---

## 7. Reboot and Verify

All your core hardware is now fully configured.

### 7.1 Reboot the system

Run:
```bash
reboot
```

Once you reboot and log back in, you need to check if your new drivers are actually loaded properly.

### 7.2 Verify Your Hardware

#### *7.2.1 Verify NVIDIA Driver*

Run:
```bash
nvidia-smi
```

(Output example)
```bash
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.54.14              Driver Version: 550.54.14      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce GTX 1650        Off |   00000000:01:00.0 Off |                  N/A |
| N/A   45C    P8              4W /   50W |       1MiB /   4096MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
```
*Expected Result:* A structured text table showing your NVIDIA GeForce GTX 1650, the current driver version, and power usage.

#### *7.2.2 Verify Intel Graphics*

Run:
```bash
lsmod | grep i915
```

(Output example)
```bash
i915                 4153344  0
drm_buddy              20480  1 i915
ttm                    98304  1 i915
drm_display_helper    225280  1 i915
intel_gtt              24576  1 i915
video                  77824  2 asus_wmi,i915
```
*Expected Result:* You should see a few lines starting with `i915` and other modules depending on it. This confirms the Intel graphics kernel module is loaded properly.

#### *7.2.3 Verify Audio Setup (PipeWire)*

Run:
```bash
wpctl status
```

(Output example)
```bash
PipeWire 'pipewire-0' [1.0.4, arch@archlinux, v0.3.x]
 └─ Clients:
        31. wireplumber                         [1.0.4, arch@archlinux, pid:821]
        56. wpctl                               [1.0.4, arch@archlinux, pid:1015]

Audio
 ├─ Devices:
 │
 ├─ Sinks:
 │
 ├─ Sources:
 │
 └─ Filters:

Video
 ├─ Devices:
 │
 ├─ Sinks:
 │
 ├─ Sources:
 │
 └─ Filters:
```

*Understanding the Output:*
- **What success looks like:** It prints `PipeWire 'pipewire-0'` and under `Clients:` it lists `wpctl` and `WirePlumber`. This proves the main audio services are properly running and talking to each other.
- **What about the RTKit errors and empty audio devices?** If you see yellow warnings like `RTKit error: org.freedesktop.DBus.Error.ServiceUnknown` at the top, and your `Audio -> Devices / Sinks / Sources` lists are completely blank (like the example above), **this is completely normal right now**. Bare text terminals (TTY) lack the session managers required to grant your user account access to the physical audio hardware or real-time priority (RTKit).
- **What failure looks like:** The command throws a fatal error entirely, such as `Could not connect to PipeWire`, meaning the daemon didn't start at all.

__CRITICAL POST-INSTALL REQUIREMENT:__
These empty audio lists and RTKit errors MUST be checked again and cleared **after** we install and boot into your graphical desktop (Hyprland). Once Hyprland starts your graphical session, DBus routing functions naturally and unlocks the required permissions. You must verify that the RTKit errors disappear and your laptop speakers populate the lists at that time.

#### *7.2.4 Verify Bluetooth Service*

Run:
```bash
systemctl status bluetooth
```

(Output example)
```bash
● bluetooth.service - Bluetooth service
     Loaded: loaded (/usr/lib/systemd/system/bluetooth.service; enabled; preset: disabled)
     Active: active (running) since Sun 2026-03-01 12:00:00 UTC; 5min ago
       Docs: man:bluetoothd(8)
   Main PID: 654 (bluetoothd)
     Status: "Running"
      Tasks: 1 (limit: 18950)
     Memory: 2.1M (peak: 2.4M)
```
*Expected Result:* Text showing `Active: active (running)` in green or plain text, proving the background daemon is alive. Prevent it from scrolling forever by pressing `q`.

> `What can go wrong here?`
> - **NVIDIA is missing:** `nvidia-smi` outputs "NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver", or "command not found".
> - **Audio is dead:** `wpctl status` outputs "Could not connect to PipeWire" (meaning the service genuinely failed to start).
> - **Bluetooth is dead:** `systemctl status bluetooth` shows as "inactive (dead)" or "failed", or it does not pair.
>
> `How to recover:`
> - **NVIDIA driver recovery:**
>   1. The Linux headers might be mismatched with your kernel, preventing DKMS from building. Rerun: `sudo pacman -S linux-headers`.
>   2. Force rebuild the modules: `sudo dkms autoinstall`.
>   3. If it is entirely broken, perform a clean reinstall. Remove the broken packages: `sudo pacman -Rns nvidia-dkms nvidia-utils`, then completely **repeat Section 4.2** and reboot.
>
> - **Audio (PipeWire) recovery:**
>   PipeWire runs as a *user* service (per user), not as a global system service like standard daemons.
>   1. Manually explicitly enable and start the required user services (Note: **Do NOT use `sudo`** for `--user` commands):
>      `systemctl --user enable --now pipewire.service pipewire-pulse.service wireplumber.service`
>   2. Check if WirePlumber (session manager) specifically crashed: `systemctl --user status wireplumber.service`.
>   3. If it failed, simply restart the services: `systemctl --user restart pipewire.service pipewire-pulse.service wireplumber.service`.
>
> - **Bluetooth recovery:**
>   1. Most commonly, you forgot to enable the service. Run: `sudo systemctl enable --now bluetooth.service`.
>   2. Check if the device is "soft-blocked" (disabled by software). Run: `rfkill list`. If Bluetooth shows `Soft blocked: yes`, unblock it with the command: `sudo rfkill unblock bluetooth`.
>   3. If your Intel Bluetooth kernel module failed to load initially, force it to load with: `sudo modprobe btusb` and restart the service.

Once you verify all hardware is responding, your base system is 100% prepared to have a graphical desktop interface (like Hyprland/Wayland) installed on top of it.

---

## 8. What are the Next Steps? (GUI Installation)

Now that your specific hardware components, most importantly your NVIDIA graphics drivers, are fully installed and configured, your system is perfectly primed to support a modern graphical environment.

**To begin installing Wayland, the Hyprland Window Manager, and the rest of your graphical interface, proceed to the next guide: [gui-installation.md](gui-installation.md).**

---

## 9. Official Arch Wiki References

For deeper reading and troubleshooting, here are the official Arch Linux Wiki pages used to construct this guide:

- **CPU Microcode:** [https://wiki.archlinux.org/title/Microcode](https://wiki.archlinux.org/title/Microcode)
- **Intel Graphics:** [https://wiki.archlinux.org/title/Intel_graphics](https://wiki.archlinux.org/title/Intel_graphics)
- **NVIDIA Drivers:** [https://wiki.archlinux.org/title/NVIDIA](https://wiki.archlinux.org/title/NVIDIA)
- **PRIME (NVIDIA Optimus):** [https://wiki.archlinux.org/title/PRIME](https://wiki.archlinux.org/title/PRIME)
- **Wayland Support:** [https://wiki.archlinux.org/title/Wayland#Requirements](https://wiki.archlinux.org/title/Wayland#Requirements)
- **PipeWire (Audio):** [https://wiki.archlinux.org/title/PipeWire](https://wiki.archlinux.org/title/PipeWire)
- **Bluetooth:** [https://wiki.archlinux.org/title/Bluetooth](https://wiki.archlinux.org/title/Bluetooth)
