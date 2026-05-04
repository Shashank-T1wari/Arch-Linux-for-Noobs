# Graphical User Interface (GUI) Installation & Configuration

Before we install the graphical interface, we need to make a few critical tweaks specifically because you have an **NVIDIA graphics card** and are targeting **Wayland**.

---

## 1. NVIDIA Wayland Preparations

Wayland requires the kernel to take explicit control of the graphics card very early in the boot process. Without these steps, Hyprland might crash on launch or give you a black screen.

### 1.1 Enable DRM Kernel Mode Setting (KMS)
We need to pass a specific parameter to the kernel via the GRUB bootloader to tell NVIDIA to allow direct rendering.

Run:
```bash
sudo nano /etc/default/grub
```

Find the line starting with `GRUB_CMDLINE_LINUX=`. We previously added LUKS encryption settings here. You need to append `nvidia_drm.modeset=1` inside the quotes.

It should look something like this:
```text
GRUB_CMDLINE_LINUX="rd.luks.name=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx=cryptroot root=/dev/mapper/cryptroot nvidia_drm.modeset=1"
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

Now, regenerate your GRUB configuration so the change takes effect:

Run:
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

### 1.2 Early Module Loading (Initramfs)
To prevent the screen from flickering or going black before the login screen appears, we want the NVIDIA drivers to load as fast as possible (during the initramfs phase, before the root filesystem is even fully mounted).

Run:
```bash
sudo nano /etc/mkinitcpio.conf
```

Find the `MODULES=()` line. It is usually empty or only contains a few items. Change it to exactly this:

```text
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

*(Note: We DO NOT put `i915` (Intel) here because we want the NVIDIA driver explicitly loaded early for Wayland compatibility).*

Save and exit.

Now, regenerate the initramfs image:

Run:
```bash
sudo mkinitcpio -P
```

(Output example)
```bash
==> Building image from preset: /etc/mkinitcpio.d/linux.preset: 'default'
==> Using default configuration file: '/etc/mkinitcpio.conf'
  -> -k /boot/vmlinuz-linux -c /etc/mkinitcpio.conf -g /boot/initramfs-linux.img
==> Starting build: '6.8.1-arch1-1'
  -> Running build hook: [base]
  -> Running build hook: [systemd]
  /* Lines truncated */
WARNING: Possibly missing firmware for module: 'qat_6xxx'
==> Image generation successful
```

> **What does this WARNING mean? (Should I panic?)**
> No, this warning is **completely safe to ignore**.
>
> *Here is what is happening:* When `mkinitcpio` builds your boot image, it scans the standard Linux kernel for all possible modules. It sees the driver for `qat_6xxx` (which is Intel QuickAssist Technology hardware usually found only in high-end enterprise servers or specific networking appliances). Since you have a consumer laptop, you physically do not possess this enterprise hardware, so the system didn't download the extra firmware for it. The warning is just the build script saying "Hey, if you actually had a qat_6xxx card plugged in, it wouldn't work." But since you don't, it has zero impact on your system.

### 1.3 Temporary Reboot (Recommended)
Before we move on to installing the actual GUI components, it is a very good idea to reboot your system right now to ensure the NVIDIA tweaks didn't break your standard terminal environment.

Run:
```bash
reboot
```

After the reboot, unlock your drive and log in again with your normal user account. If everything works as usual and you reach the text prompt, the Wayland foundation is solid.

### 1.4 Verify NVIDIA Wayland Preparations

Once logged back in, verify that your kernel changes actually took effect.

#### *1.4.1 Verify DRM Kernel Mode Setting*

Run:
```bash
sudo cat /sys/module/nvidia_drm/parameters/modeset
```

(Output example)
```bash
Y
```
*Expected Result:* It should output `Y` (Yes). This confirms that the kernel correctly read your GRUB boot parameter and DRM direct rendering is active.

#### *1.4.2 Verify Early Module Loading*

Run:
```bash
lsmod | grep nvidia
```

(Output example)
```bash
nvidia_drm             81920  1
nvidia_modeset       1597440  1 nvidia_drm
nvidia_uvm           3526656  0
nvidia              62537728  2 nvidia_uvm,nvidia_modeset
video                  77824  2 i915,nvidia_modeset
```
*Expected Result:* You should see all four modules (`nvidia_drm`, `nvidia_modeset`, `nvidia_uvm`, and `nvidia`) listed here. This proves the system loaded them properly on boot.

NOTE: If you see `i915` (Intel) listed here, it is normal and expected. It does not interfere with NVIDIA's operation in Wayland mode, as long as the NVIDIA modules are also present and `nvidia_drm.modeset=1` is active.

---

## 2. Installing the GUI & Packages

Now we assemble your desktop. Because you chose a Window Manager (**Hyprland**) rather than a full Desktop Environment, you must install each piece of the graphical experience individually.

Run:
```bash
sudo pacman -S hyprland kitty waybar rofi-wayland swaybg dunst polkit-kde-agent qt5-wayland qt6-wayland
```

*What are these packages?*
| Package | Purpose |
|---------|---------|
| `hyprland` | The core Wayland compositor/window manager itself. |
| `kitty` | A fast, GPU-accelerated terminal emulator to type commands inside the GUI. |
| `waybar` | The customizable status bar (typically at the top) showing clock, battery, and workspaces. |
| `rofi-wayland` | The application launcher. Pressing your designated shortcut pulls up a menu to search/open apps. |
| `swaybg` | A lightweight utility designed to draw a wallpaper image on your screen. |
| `dunst` | The notification daemon. Handles popup alerts (like low battery or chat messages). |
| `polkit-kde-agent` | A secure popup box that asks for your user password when a GUI app needs administrator (root) access. |
| `qt5-wayland` / `qt6-wayland` | Compatibility packages that help apps built with the Qt framework (like VLC or OBS) run natively and smoothly in Wayland instead of falling back to legacy X11 mode. |

(Output example)
```bash
resolving dependencies...
looking for conflicting packages...

Packages (48) /* List of dependencies */ ... hyprland-0.39.1-1 kitty-0.34.1-1 waybar-0.10.0-1 ...

Total Download Size:   154.21 MiB
Total Installed Size:  582.44 MiB

:: Proceed with installation? [Y/n]
```
*(Press `Y` and `Enter`)*

*(Note: Depending on your internet speed, downloading 154+ MiB of packages may take a few moments. Let it finish completely).*

### 2.1 Verify GUI Package Installation
To be absolutely sure all the critical components were installed properly, you can query pacman to list them.

Run:
```bash
pacman -Q hyprland kitty waybar rofi-wayland swaybg dunst polkit-kde-agent qt5-wayland qt6-wayland
```

(Output example)
```bash
hyprland 0.39.1-1
kitty 0.34.1-1
waybar 0.10.0-1
rofi-wayland 1.7.5+wayland3-1
swaybg 1.2.1-1
dunst 1.11.0-1
polkit-kde-agent 6.0.4-1
qt5-wayland 5.15.13+kde+r63-1
qt6-wayland 6.7.0-1
```
*Expected Result:* It should list the version numbers of all requested packages. If any package says `package was not found`, it means the installation failed or was interrupted, and you should re-run the pacman command.

---

## 3. The Display Manager (Login Screen)

You need a way to log in graphically. We will use **SDDM** (Simple Desktop Display Manager) because it is officially recommended for Wayland, highly reliable, and supports Wayland sessions natively without legacy Xorg dependencies.

Run:
```bash
sudo pacman -S sddm
```

*(Note: When you run this command, Pacman will ask you to choose from 11 different providers for `ttf-font`. This is required for SDDM to render text properly. Here is a breakdown of your options:)*

- **`gnu-free-fonts`** (Default): Extremely basic, old-school textbook fonts. They work, but they look a bit dated.
- **`noto-fonts`**: (Highly Recommended) Built by Google ("No Tofu"). It is an incredibly clean, modern UI font designed to support every single language and symbol in the world. This is the gold standard for modern Linux desktops.
- **`ttf-bitstream-vera`**: A classic, slightly wider, old Linux terminal/desktop font.
- **`ttf-croscore`**: These are Chrome OS fonts designed to be identical in width/size to Microsoft's Arial and Times New Roman.
- **`ttf-dejavu`**: Another very popular classic Linux font based on Bitstream Vera. It's perfectly fine but not as modern as Noto or Roboto.
- **`ttf-droid`**: The old, legacy Android font (before Google switched to Roboto).
- **`ttf-ibm-plex`**: A very cool, crisp, professional, and slightly "techy" looking font made by IBM. Great for coding setups.
- **`ttf-input`**: A font explicitly designed for programming.
- **`ttf-input-nerd`**: The programming font above, but patched with hundreds of special icons (Nerd fonts) like folder icons, apple logos, git branching symbols, etc. (Very popular for advanced customized terminals, but perhaps too specific just for the login screen right now).
- **`ttf-liberation`**: Very standard fonts designed by Red Hat to clone the look of Arial/Times New Roman. Good for office documents.
- **`ttf-roboto`**: (Also Highly Recommended) Google's current Android font. Very beautiful, clean, and legible for user interfaces.

*We highly recommend typing **`2`** to select `noto-fonts` (or whichever number corresponds to it) and pressing **Enter**. After selecting the font, press `Y` and `Enter` to proceed with the installation.*

Enable SDDM so it starts automatically on boot:

Run:
```bash
sudo systemctl enable sddm.service
```

(Output example)
```bash
Created symlink /etc/systemd/system/display-manager.service → /usr/lib/systemd/system/sddm.service.
```

> `What can go wrong here?`
> - **Failed to enable:** If you get an error saying `Failed to enable unit: Unit file /etc/systemd/system/display-manager.service already exists`, it means another display manager was previously installed.
> - **How to recover:** Run `sudo systemctl disable display-manager.service --force` to clear old configs, then retry the `enable sddm.service` command above.

### 3.1 Verify SDDM Service
Before rebooting, ensure systemd has appropriately registered SDDM to start up.

Run:
```bash
systemctl status sddm
```

(Output example)
```bash
● sddm.service - Simple Desktop Display Manager
     Loaded: loaded (/usr/lib/systemd/system/sddm.service; enabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:sddm(1)
             man:sddm.conf(5)
```
*Expected Result:* Look at the `Loaded:` line. It must say `enabled`. It is perfectly normal and expected for it to say `Active: inactive (dead)` right now, because SDDM won't actually launch until you restart your computer. Prevent it from scrolling forever by pressing `q` to exit.

---

## 4. Booting into Hyprland for the First Time

You have all the building blocks installed.

Run:
```bash
reboot
```

### What to expect on reboot:
1. Your computer will turn on.
2. GRUB will load normally (with our new direct rendering flags).
3. You will decrypt your drive as usual.
4. **Instead of a pure text TTY**, you should now arrive at a graphical login screen (**SDDM**).
5. Before typing your password, look for a "Session" dropdown menu (usually at the bottom left or top left). Ensure **Hyprland** is selected.
6. Type your password and press Enter.

### Welcome to Hyprland!
When you log in, you might see a warning message at the top of an otherwise empty screen regarding an autogenerated config file. **This is normal.**

By default, Hyprland gives you a bare-minimum, unconfigured environment.

#### *What is the "SUPER" Key?*
**`SUPER` is simply your Windows Key.**
In the Linux Window Manager world, they don't call it the "Windows" key, they call it the "Super" or "Modifier" key. So when the instructions say `SUPER + Q`, you just hold down your **Windows Key** and press **Q**. That will instantly pop open your kitty terminal!

Here are your default shortcuts right now:
- Press `SUPER + Q` to open your terminal (which by default is explicitly set to `kitty`).
- Press `SUPER + M` to exit Hyprland and return to the login screen.

> **CRITICAL POST-INSTALL CHECK:**
> Remember the audio check from `adding-drivers.md`? Now that you are literally running a graphical session (Hyprland), the permissions lock is bypassed.
> 1. Press `SUPER + Q` to open Kitty.
> 2. Run `wpctl status`.
> 3. Verify that the previous "RTKit errors" are gone and your actual laptop speakers finally appear under `Audio -> Devices / Sinks / Sources`.

---

## 5. What are the Next Steps? (The Possibilities)

Because you are using a Window Manager instead of a pre-packaged desktop like Windows or MacOS, **you have 100% control over every single pixel on your screen**. Your graphical environment is entirely dictated by a single text file.

Here are the different things you can (and will) configure next:

- **Wallpapers:** We will use the `swaybg` package we installed to load any image you want every time you log in, filling your background instead of leaving it plain grey.
- **Waybar (Status Bar):** You can put it at the top, bottom, or sides. By manipulating its raw text configuration, you can use CSS (like designing a website) to make it transparent, rounded, or totally custom. Thousands of people share beautiful pre-made Waybar themes online on sites like GitHub or Reddit's `r/unixporn`.
- **App Launcher (Rofi):** You will bind a shortcut so that when you press `SUPER + R` (Windows Key + R), a highly visual app search menu will pop up in the center of your screen, letting you launch software without touching the terminal.
- **Animations & Blur:** You can dive into the config file to radically change the aesthetics: adding Apple-style rounded window corners, making your terminal transparent so it looks like frosted glass (blur), adjusting the gaps between windows, and adding desktop drop shadows.
- **Hardware Keys:** Out of the box, your specific laptop volume up/down, mute, and brightness keys do not work. We have to explicitly map those physical keyboard keys to actual system commands in the backend so they function naturally.

**To begin configuring all of these features and actually making your desktop usable, proceed to the next guide: [hyprland-configuration.md](hyprland-configuration.md).**

---

## 6. Official Arch Wiki References

For deeper reading and troubleshooting, here are the official Arch Linux Wiki pages used to construct this graphical setup guide:

- **NVIDIA Wayland / DRM Modeset:** [https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting](https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting)
- **Early Module Loading (mkinitcpio):** [https://wiki.archlinux.org/title/NVIDIA#Early_loading](https://wiki.archlinux.org/title/NVIDIA#Early_loading)
- **Hyprland:** [https://wiki.archlinux.org/title/Hyprland](https://wiki.archlinux.org/title/Hyprland)
- **SDDM (Display Manager):** [https://wiki.archlinux.org/title/SDDM](https://wiki.archlinux.org/title/SDDM)
- **Waybar:** [https://wiki.archlinux.org/title/Waybar](https://wiki.archlinux.org/title/Waybar)
- **Polkit & Authentication Agents:** [https://wiki.archlinux.org/title/Polkit#Authentication_agents](https://wiki.archlinux.org/title/Polkit#Authentication_agents)
- **Qt Wayland Integration:** [https://wiki.archlinux.org/title/Wayland#Qt](https://wiki.archlinux.org/title/Wayland#Qt)
