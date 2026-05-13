# Hyprland Configuration & Customization

Welcome to the fun part! You now have a working graphical environment. However, because Hyprland is a "Window Manager" (and not a full pre-packaged desktop like Windows or macOS), **everything** about how it looks and behaves is controlled by a single text file.

This document will guide you through making your desktop actually look beautiful and functional.

---

## 1. First Launch: The "Getting Started" Wizard

When you boot into Hyprland for the very first time, you may be greeted by a graphical "Getting started" setup wizard running inside your terminal. It checks your system for core capabilities like file managers, clipboards, and status bars.

![alt text](images/image-3.png)

This wizard shows heavily recommended tools and detects missing ones. Let's break down the components it is asking for and explain the options.

### What We Already Installed (and Why)

You will see several items marked in **green** (Installed). We intentionally chose these packages during the GUI installation phase:

1. **Terminal (`kitty`)**:
   - *Why we chose it:* Kitty is an incredibly fast, highly customizable, GPU-accelerated terminal emulator. It can even render images and graphs directly inside the terminal window.
   - *Alternatives:* `alacritty` (faster but lacks some advanced image-rendering features), `wezterm` (very powerful, configured via Lua), `konsole` (KDE's default, good but heavier).
2. **Wallpaper (`hyprpaper`)**:
   - *Why we chose it:* Developed directly by the Hyprland team, `hyprpaper` offers native integration, excellent IPC support (allowing you to change wallpapers instantly via command line), and distinct wallpapers per monitor.
   - *Alternatives:* `swaybg` (older, more basic but extremely lightweight), `wpaperd`, or `mpvpaper` (for animated video wallpapers).
3. **Status bar / shell (`hyprpanel`)**:
   - *Why we chose it:* Instead of just a bar, `hyprpanel` is a cohesive ecosystem panel. It beautifully integrates a top bar, a sleek OSD (on-screen display for volume/brightness), and unified control widgets out of the box, looking phenomenal natively on Hyprland.
   - *Alternatives:* `waybar` (the standard customizable choice using CSS), `eww` (insanely customizable using Lisp-like widgets, but highly complex).
4. **Notification Daemon (`swaync`)**:
   - *Why we chose it:* SwayNotificationCenter (`swaync`) doesn't just pop up alerts; it provides a stunning, slide-out control center panel (like macOS or Windows 11) where you can toggle Wi-Fi, Do Not Disturb, and see past notifications.
   - *Alternatives:* `dunst` (a very lightweight, text-based popup daemon), `mako`.
5. **Application launcher (`rofi-wayland`)**:
   - *Why we chose it:* `rofi -show drun` is mature, lightweight, fast, and easy to script. It is one of the most stable launcher choices on Wayland Hyprland setups.
   - *Alternatives:* `Hyprlauncher`, `wofi`, or `fuzzel`.

*(For detailed video overviews of configuring these tools, search YouTube for "Linux Hyprland Rice" or look up channels like **DistroTube** or **The Linux Cast**).*

### Resolving "Missing" Core Components

You will also notice the wizard highlights several things in **red** (Missing). You need these for a fully functioning desktop.

1. **Authentication Agent (Missing)**
   - *What it does:* When a graphical app needs administrator/root privileges, this provides the secure popup box asking for your password.
   - *The Fix:* We will install and use `hyprpolkitagent`. We chose this because it is the officially developed polkit agent by the Hypr ecosystem, guaranteeing zero legacy X11 or heavy KDE dependencies.
   - *Other options:* `polkit-kde-agent`, `lxsession-polkit` or `gnome-polkit`.
2. **XDG Desktop Portal (Missing)**
   - *What it does:* Absolutely mandatory for modern Linux. It handles secure communication between apps and your system—specifically **screen sharing** (OBS, Discord, WebRTC) and standard **Open/Save File Dialog** menus.
   - *The Fix:* You must install `xdg-desktop-portal-hyprland`. No alternatives here—you need the specific Hyprland portal for screen sharing to work perfectly on this window manager.
3. **Clipboard (Missing)**
   - *What it does:* Manages copy and paste history for Wayland.
   - *The Fix:* Install `wl-clipboard` (the engine responsible for handling copy data) and optionally `cliphist` (a manager that saves a history of your past copied items so you can recall them with a shortcut).

---

## 2. Setting Default Apps (File Managers & Launchers)

If you hit "Next" in the wizard, it will ask you to select Default Apps.

![alt text](images/image-4.png)

Because a window manager is a "build-it-yourself" desktop, you must choose what app opens when you want to manage files or launch programs.

### File Manager Options

*A File Manager is the app you use to browse your folders (like Windows Explorer or macOS Finder).*

1. **`dolphin`**:
   - *Features:* The default file manager for the KDE Plasma desktop. It is the most feature-rich option available (split screen, heavy plugin support, excellent network drive handling).
   - *Pros/Cons:* Unmatched power, but very "heavy" as it relies on many KDE desktop dependencies.
2. **`thunar`**:
   - *Features:* The default for the XFCE desktop environment.
   - *Pros/Cons:* The perfect balance. It is incredibly fast, extremely lightweight, yet supports advanced features like bulk-renaming and custom right-click actions without needing a heavy desktop environment attached to it.
3. **`nautilus`**:
   - *Features:* The default for the GNOME desktop.
   - *Pros/Cons:* Very clean, minimalist, and touch-screen friendly. However, power users sometimes find it lacking in advanced layout options.
4. **`nemo`**:
   - *Features:* The default for Linux Mint (Cinnamon). It is a fork of an older version of Nautilus.
   - *Pros/Cons:* Retains heavily requested power-user features (like a compact list view and advanced context menus) that standard Nautilus removed.
5. **`pcmanfm`**:
   - *Features:* The default for LXDE/LXQt.
   - *Pros/Cons:* The absolute lightest and fastest file manager in existence. The trade-off is that it looks somewhat dated.

*(Installation Choice: We are proceeding with **Thunar** for a fast and lightweight daily file manager.)*

### Application Launcher Options

*A Launcher is the menu that pops up to search and open apps (similar to hitting the Windows Key).*

1. **`rofi -show drun`** (Our Installation Choice)
   - *Features:* Mature, handles app launching, window switching, and even custom script menus (like a power-off menu). Infinite customization.
   - *Pros/Cons:* Highly flexible via standard CSS, but requires more manual setup to look perfectly native.
2. **`Hyprlauncher`**
   - *Features:* A first-party multipurpose and versatile launcher for Hyprland.
   - *Pros/Cons:* Native integration and straightforward setup, but not required for this guide path.
3. **`wofi`**
   - *Features:* One of the first launchers built explicitly for the Wayland protocol. Very simple, neat GTK-based layout.
   - *Pros/Cons:* Easy to configure using standard CSS, but development has slowed down compared to newer forks.
4. **`fuzzel`**
   - *Features:* Strictly Wayland-native. It is designed to be an extremely fast, lightweight application launcher.
   - *Pros/Cons:* Less visually flashy than Rofi, but blazingly fast if you just want to type an app name and press enter.
5. **`anyrun`**
   - *Features:* Written in Rust. It functions like macOS Spotlight—it's highly modular. You can use it to launch apps, do math calculations (e.g., typing "3+3"), or do direct web searches.
6. **`tofi`**
   - *Features:* A highly minimalist launcher that relies strictly on text rendering directly to the screen. It is extremely fast and very lightweight.

*(Installation Choice: We are proceeding with **Rofi** for launch/search and scriptable workflows).*

---

## 3. Completing the Wizard & Generating Your Config

Assuming you are currently looking at the **"Default apps"** screen (In the image above), follow these steps:

1. Using your arrow keys or mouse, ensure **Terminal** is set to kitty.
2. Set **File Manager** to `thunar`.
3. Set **Launcher** to `rofi -show drun` (or `rofi-wayland` if the wizard asks for package name).
4. Click the **"Next"** button at the bottom right.

### 3.1 Basic Configuration (Autostarting)

![Basic Configuration](images/image-5.png)

Now you will encounter the **Basic configuration** screen. This highlights the most critical rule of configuring Hyprland: **You are in charge of autostarting components.**

According to official Arch Wiki guidelines, it is crucial that you use the correct dispatchers in your config:
- **`exec-once = appname`**: Use this for background daemons (like your wallpaper, polkit agent, and status bar). It guarantees the app launches *exactly once* when you first log in.
- **NEVER use `exec = appname` for services**: `exec` runs the command every single time you save your configuration file or reload Hyprland. This creates multiple duplicate instances, devours memory, and will crash your system.

Click **"Next"**.

### 3.2 The Hypr Ecosystem

![Hypr Ecosystem](images/image-6.png)

The wizard briefly explains the **Hypr Ecosystem**. Unlike bulky desktop environments (like GNOME or KDE) that force a bunch of apps on you, Hyprland lets you install elements separately. This confirms the exact path we took: manually picking tailored, native components like `hyprpaper`, `hyprpanel`, and `hyprpolkitagent` to keep the system lean and optimized.

Click **"Next"**.

### 3.3 Default Shortcuts & Finalizing

![That's it!](images/image-7.png)

The final **"That's it!"** screen gives you a cheat sheet of default keybinds. Because you don't have a taskbar yet, these are mandatory for navigation:
- `SUPER + Q`: Terminal
- `SUPER + E`: File Manager
- `SUPER + M`: Exit Hyprland completely.
- `SUPER + V`: Toggle floating mode for a window.

**Final Steps on the Wizard:**
1. Click the **"Finish"** button at the bottom right.
2. The wizard will officially write your `~/.config/hypr/hyprland.conf` file.
3. Once it closes, you will be deposited into your raw, blank Hyprland session.
4. You will likely see a yellow warning banner at the top of the screen stating: *"Warning: You're using an autogenerated config!"*

Now it is time to open the terminal and complete the missing installations. Press `SUPER + Q` (Windows Key + Q) to open your terminal (kitty).

---

## 4. Installing the Missing Components

Based on the red "Missing" tags from the wizard (Image 3), we will now systematically install each missing piece. We are following official Arch Linux and Hyprland Wiki guidelines.

*Note: We will use the terminal (kitty) for all these installations. Since you are in a live graphical session, you do not need to reboot between these installations—they take effect immediately.*

### 4.1 Authentication Agent (hyprpolkitagent)

We need a polkit agent to allow GUI apps (like partition managers) to ask for root passwords securely. The official Hyprland ecosystem provides its own native polkit agent.

Run:
```Bash
sudo pacman -S hyprpolkitagent
```

(Output example)
```Bash
resolving dependencies...
looking for conflicting packages...

Packages (2) polkit-1.24-1 hyprpolkitagent-0.1.2-1

Total Download Size:   0.45 MiB
Total Installed Size:  1.32 MiB

:: Proceed with installation? [Y/n]
```
*(Press Y and Enter)*

### 4.2 File Manager + TUI Manager (thunar + yazi)

We selected **Thunar** as the GUI file manager and **Yazi** as the terminal file manager.

Run:
```Bash
sudo pacman -S thunar thunar-volman gvfs tumbler file-roller yazi
```

Why these packages:
1. `thunar`: lightweight GUI file manager.
2. `thunar-volman`: removable drives and automount actions.
3. `gvfs`: trash, network mounts, and desktop file operations.
4. `tumbler`: thumbnail support in Thunar.
5. `file-roller`: archive integration for zip/tar actions.
6. `yazi`: fast terminal file manager for keyboard workflows.

### 4.3 XDG Desktop Portal

The XDG Desktop Portal allows apps (like Firefox or OBS) to record your screen or open file-picker menus in Wayland securely. We strictly need the official Hyprland portal.

Run:
```Bash
sudo pacman -S xdg-desktop-portal-hyprland xdg-desktop-portal
```

(Output example)
```Bash
resolving dependencies...
looking for conflicting packages...

Packages (4) sdbus-cpp-1.4.0-1 xdg-desktop-portal-1.18.2-1 xdg-desktop-portal-hyprland-1.3.1-1

Total Download Size:   2.30 MiB
Total Installed Size:  9.60 MiB

:: Proceed with installation? [Y/n]
```
*(Press Y and Enter)*

### 4.4 Clipboard Managers (wl-clipboard & cliphist)

Wayland requires dedicated clipboard engines. wl-clipboard handles the raw copying and pasting (like Ctrl+C), and cliphist remembers your copy history.

Run:
```Bash
sudo pacman -S wl-clipboard cliphist
```

(Output example)
```Bash
resolving dependencies...
looking for conflicting packages...

Packages (2) wl-clipboard-2.2.1-1 cliphist-0.5.0-1

Total Download Size:   1.02 MiB
Total Installed Size:  3.10 MiB

:: Proceed with installation? [Y/n]
```
*(Press Y and Enter)*

### 4.5 Installing the Actual Ricing Packages (Your Chosen Stack)

From the choices you already made above, your actual Hyprland rice stack is:
- `kitty` (terminal)
- `hyprpaper` (wallpaper daemon)
- `swaync` (notification center)
- `rofi-wayland` (launcher)
- `thunar` (GUI file manager)
- `yazi` (TUI file manager)

`hyprpanel` is optional in this path. Add it later after you validate your current baseline.

Install the stack (excluding swaync - see 4.6 below):

```Bash
sudo pacman -S kitty hyprpaper rofi-wayland thunar thunar-volman gvfs tumbler file-roller yazi brightnessctl
```

Verify each binary is available:

```Bash
command -v kitty hyprpaper rofi thunar yazi brightnessctl
```

If each command prints a path (for example `/usr/bin/rofi`), installation is complete.

### 4.6 Install swaync from AUR (IMPORTANT - Not in Official Repos)

**IMPORTANT:** `swaync` from official repos has D-Bus notification name conflicts. You **MUST** install it from AUR instead.

**Step 1: Install build tools**

Run:
```Bash
sudo pacman -S --needed base-devel git
```

**Step 2: Clone swaync from AUR**

Run:
```Bash
git clone https://aur.archlinux.org/swaync.git
cd swaync
```

(Output example)
```Bash
Cloning into 'swaync'...
remote: Enumerating objects: 124, done.
remote: Counting objects: 100% (124/124), done.
```

**Step 3: Build and install**

Run:
```Bash
makepkg -si
```

**What this does:**
- `makepkg` = Build the package from source
- `-s` = Install missing dependencies automatically
- `-i` = Install after building

(Output example - may take 2-5 minutes)
```Bash
==> Making package: swaync 0.12.6-1 (Sat May 04 21:30:00 IST 2026)
==> Checking runtime dependencies...
==> Building swaync...
[...build output...]
==> Installing package with pacman...
```

**Step 4: Verify installation**

Run:
```Bash
pacman -Q swaync
```

(Expected output)
```Bash
swaync 0.12.6-1
```

Run:
```Bash
which swaync
```

(Expected output)
```Bash
/usr/bin/swaync
```

### 4.7 Optional: Install hyprpanel from Git (AUR PKGBUILD)

If `hyprpanel` is not available via official repos on your system, install it from the AUR git repository with:

```Bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/hyprpanel.git
cd hyprpanel
makepkg -si
```

Verify it was installed:

```Bash
pacman -Q hyprpanel
command -v hyprpanel
```

Output example:
```Bash
hyprpanel 1.3.1-1
/usr/bin/hyprpanel
```

---

## 5. Final Package Verification Commands

Run this block to verify that your selected stack is installed:

```Bash
pacman -Q kitty rofi-wayland thunar thunar-volman gvfs tumbler file-roller yazi brightnessctl \
hyprpaper swaync hyprpolkitagent xdg-desktop-portal xdg-desktop-portal-hyprland wl-clipboard cliphist

**Note:** If you installed swaync from AUR (as recommended in 4.6), it should appear in this list.
```

Output Example:
```Bash
kitty 0.28.3-1
rofi 1.8.4-1
thunar 4.18.0-1
thunar-volman 4.18.0-1
gvfs 1.52.0-1
tumbler 0.3.0-1
file-roller 3.42.0-1
yazi 0.5.0-1
brightnessctl 1.0-1
hyprpaper 1.3.1-1
hyprpolkitagent 0.1.2-1
xdg-desktop-portal 1.18.2-1
xdg-desktop-portal-hyprland 1.3.1-1
wl-clipboard 2.2.1-1
cliphist 0.5.0-1
```

If you also installed `hyprpanel` from AUR, run:

```Bash
pacman -Q hyprpanel
```

Output Example:
```bash
hyprpaper 1.3.1-1
```

Quick runtime binary check:

```Bash
command -v kitty rofi thunar yazi hyprpaper swaync wl-paste
```

Output Example:
```Bash
/usr/bin/kitty
/usr/bin/rofi
/usr/bin/thunar
/usr/bin/yazi
/usr/bin/hyprpaper
/usr/bin/swaync
/usr/bin/wl-paste
```

Optional process check (what is currently running):

```Bash
pgrep -af "hyprpaper|hyprpanel|swaync|hyprpolkitagent|wl-paste|xdg-desktop-portal"
```

---

## 6. Activating All Services (Systemd - Best Practice)

Now that all packages are installed and verified, we need to activate them so they run every time you log into Hyprland.

**The best approach:** Use **systemd user services** for everything that has them. Systemd provides:
- ✅ Auto-restart if a service crashes
- ✅ Persistent across sessions (not restarted on Hyprland reload)
- ✅ No duplicate instances eating RAM
- ✅ Can start before Hyprland loads
- ✅ Easy to manage with standard commands

---

### 6.1 Enable All Systemd Services (Recommended)

Most of your services have systemd files. Enable them all at once:

**Enable all services:**

Run:
```bash
systemctl --user enable hyprpaper.service swaync.service xdg-desktop-portal.service xdg-desktop-portal-hyprland.service hyprpolkitagent.service cliphist.service
```

(Output example)
```bash
Created symlink /etc/systemd/user/multi-user.target.wants/hyprpaper.service → /usr/lib/systemd/user/hyprpaper.service.
Created symlink /etc/systemd/user/multi-user.target.wants/swaync.service → /usr/lib/systemd/user/swaync.service.
Created symlink /etc/systemd/user/multi-user.target.wants/xdg-desktop-portal.service → /usr/lib/systemd/user/xdg-desktop-portal.service.
Created symlink /etc/systemd/user/multi-user.target.wants/xdg-desktop-portal-hyprland.service → /usr/lib/systemd/user/xdg-desktop-portal-hyprland.service.
Created symlink /etc/systemd/user/multi-user.target.wants/hyprpolkitagent.service → /usr/lib/systemd/user/hyprpolkitagent.service.
Created symlink /etc/systemd/user/multi-user.target.wants/cliphist.service → /usr/lib/systemd/user/cliphist.service.
```

**What this does:**
- `systemctl --user` = Manage user-level services (not system-level)
- `enable` = Start these services every time you log in
- Creates symlinks so systemd knows to auto-start them

**Start all services immediately:**

Run:
```bash
systemctl --user start hyprpaper.service swaync.service xdg-desktop-portal.service xdg-desktop-portal-hyprland.service hyprpolkitagent.service cliphist.service
```

(Output example: usually silent, which means success)

---

### 6.2 Verify All Systemd Services Are Running

Run this to check all at once:

Run:
```bash
systemctl --user status hyprpaper.service swaync.service xdg-desktop-portal.service xdg-desktop-portal-hyprland.service hyprpolkitagent.service cliphist.service
```

(Output example - abbreviated for readability)
```bash
● hyprpaper.service - Hyprland Wallpaper utility
   Loaded: loaded (/usr/lib/systemd/user/hyprpaper.service; enabled; preset: disabled)
   Active: active (running) since Sat 2026-05-04 14:30:22 IST; 2min ago

● swaync.service - Simple Wayland Notification Center
   Loaded: loaded (/usr/lib/systemd/user/swaync.service; enabled; preset: disabled)
   Active: active (running) since Sat 2026-05-04 14:30:23 IST; 2min ago

● xdg-desktop-portal.service - Portal service
   Loaded: loaded (/usr/lib/systemd/user/xdg-desktop-portal.service; enabled; preset: disabled)
   Active: active (running) since Sat 2026-05-04 14:30:24 IST; 2min ago

● xdg-desktop-portal-hyprland.service - Hyprland Portal
   Loaded: loaded (/usr/lib/systemd/user/xdg-desktop-portal-hyprland.service; enabled; preset: disabled)
   Active: active (running) since Sat 2026-05-04 14:30:25 IST; 2min ago

● hyprpolkitagent.service - Hyprland Polkit Authentication Agent
   Loaded: loaded (/usr/lib/systemd/user/hyprpolkitagent.service; enabled; preset: disabled)
   Active: active (running) since Sat 2026-05-04 14:30:26 IST; 2min ago

● cliphist.service - Wayland Clipboard Manager
   Loaded: loaded (/usr/lib/systemd/user/cliphist.service; enabled; preset: disabled)
   Active: active (running) since Sat 2026-05-04 14:30:27 IST; 2min ago
```

**Expected Result for each service:**
- `Loaded:` shows `enabled`
- `Active:` shows `active (running)`

If any shows `inactive`, restart that specific service:

```bash
systemctl --user restart SERVICE_NAME.service
```

---

### 6.3 Activate wl-paste (Special Case - Not a Service)

`wl-paste` is a utility tool, not a service, so it needs to be activated via `exec-once` in your Hyprland config.

**Edit your config:**

Run:
```bash
nvim ~/.config/hypr/hyprland.conf
```

**Find or create an `# ========== AUTOSTART ==========` section and add:**

```bash
# ========== AUTOSTART (Tools) ==========

# Clipboard watching (wl-paste with cliphist)
exec-once = wl-paste --type text --watch cliphist store
exec-once = wl-paste --type image --watch cliphist store
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

**Reload Hyprland:**

Run:
```bash
hyprctl reload
```

(Output example)
```bash
Reloading the config...
```

---

### 6.4 Check hyprpanel (Optional)

`hyprpanel` may or may not have a systemd service depending on your version. Check:

Run:
```bash
ls /usr/lib/systemd/user/hyprpanel.service
```

**If file exists:**

Enable it:
```bash
systemctl --user enable hyprpanel.service
systemctl --user start hyprpanel.service
```

**If file doesn't exist:**

Add to your `hyprland.conf` instead:

```bash
exec-once = hyprpanel
```

---

### 6.5 Final Process Check - All Services

After systemd services are enabled and wl-paste is added to config, verify everything:

Run:
```bash
pgrep -af "hyprpaper|hyprpanel|swaync|hyprpolkitagent|wl-paste|cliphist|xdg-desktop-portal"
```

(Output example)
```bash
/usr/bin/hyprpaper
/usr/bin/hyprpanel --config ~/.config/hyprpanel
/usr/bin/swaync
/usr/lib/hyprpolkitagent/hyprpolkitagent
wl-paste --type text --watch cliphist store
wl-paste --type image --watch cliphist store
cliphist store
/usr/lib/xdg-desktop-portal
/usr/lib/xdg-desktop-portal-hyprland
/usr/lib/xdg-desktop-portal-gtk
```

**All processes should be running!** ✅

---

### 6.6 Comparison: Systemd vs exec-once

**Enable the service to start on login:**

Run:
```bash
systemctl --user enable hyprpolkitagent.service
```

(Output example)
```bash
Created symlink /etc/systemd/user/multi-user.target.wants/hyprpolkitagent.service → /usr/lib/systemd/user/hyprpolkitagent.service.
```

**What this does:**
- `systemctl --user` = Manage user-level services (not system-level)
- `enable` = Start this service every time you log in
- `hyprpolkitagent.service` = The polkit authentication agent service

**Start the service immediately:**

Run:
```bash
systemctl --user start hyprpolkitagent.service
```

(Output example: usually silent, which means success)

**Verify it's running:**

Run:
```bash
systemctl --user status hyprpolkitagent.service
```

(Output example)
```bash
● hyprpolkitagent.service - Hyprland Polkit Authentication Agent
   Loaded: loaded (/usr/lib/systemd/user/hyprpolkitagent.service; enabled; preset: disabled)
   Active: active (running) since Sat 2026-05-04 14:30:22 IST; 2min ago
     Main PID: 1234 (hyprpolkitagent)
   Status: "Ready"
   CGroup: /user.slice/user-1000.slice/user@1000.service/hyprpolkitagent.service
           └─1234 /usr/lib/hyprpolkitagent/hyprpolkitagent
```

**Expected Result:**
- `Loaded:` shows `enabled`
- `Active:` shows `active (running)`

---

### 6.6 Comparison: Systemd vs exec-once

| Aspect | Systemd Services | exec-once (Tools) |
|--------|:----------------:|:----------------:|
| **Auto-restart** | ✅ Yes, if crashes | ❌ Only on Hyprland reload |
| **Duplicate instances** | ❌ Never | ⚠️ Can duplicate on reload |
| **Boot behavior** | ✅ Starts with user session | ❌ Only in Hyprland |
| **Resource use** | ✅ Persistent, clean | ⚠️ Can accumulate if reload |
| **Easy to manage** | ✅ Standard commands | ✅ Text-based config |
| **Used for** | Services (daemons) | Tools with arguments |

---

### 6.7 Your Service Breakdown

Here's exactly what you're running on your system:

**Via Systemd (Auto-managed):**
- ✅ `hyprpaper` - Wallpaper daemon
- ✅ `swaync` - Notification center
- ✅ `xdg-desktop-portal` - Portal base
- ✅ `xdg-desktop-portal-hyprland` - Screen sharing support
- ✅ `hyprpolkitagent` - Polkit auth agent
- ✅ `cliphist` - Clipboard history storage

**Via exec-once (Tools with arguments):**
- ⚙️ `wl-paste --type text --watch cliphist store` - Watch text clipboard
- ⚙️ `wl-paste --type image --watch cliphist store` - Watch image clipboard
- ⚙️ `hyprpanel` (if you have it, or no systemd file)

---

### 6.8 Individual Service Details

#### **hyprpaper (Wallpaper Daemon)**

Check status:
```bash
systemctl --user status hyprpaper.service
```

Expected output: `active (running)`

View config location:
```bash
cat ~/.config/hypr/hyprpaper.conf
```

#### **swaync (Notification Center)**

Check status:
```bash
systemctl --user status swaync.service
```

Expected output: `active (running)`

Test notification:
```bash
notify-send "Test" "This is a test notification"
```

#### **xdg-desktop-portal (Screen Sharing & Dialogs)**

Check status:
```bash
systemctl --user status xdg-desktop-portal-hyprland.service
```

Expected output: `active (running)`

Test screen sharing (will work in Discord/OBS after this)

#### **hyprpolkitagent (Password Prompts)**

Check status:
```bash
systemctl --user status hyprpolkitagent.service
```

Expected output: `active (running)`

Test: Try to open a GUI app that needs root (like `thunar` pointing to `/root`)

#### **cliphist (Clipboard History)**

Check status:
```bash
systemctl --user status cliphist.service
```

Expected output: `active (running)`

View stored clipboard history:
```bash
cliphist list
```

---

### 6.9 Troubleshooting: Service Not Working

#### **Problem: Service shows `inactive (dead)` but is enabled**

**Solution: Start it manually**

Run:
```bash
systemctl --user start SERVICE_NAME.service
```

Check if it starts:
```bash
systemctl --user status SERVICE_NAME.service
```

If still fails, see error logs:
```bash
journalctl --user -eu SERVICE_NAME.service
```

(Shows detailed error messages)

---

#### **Problem: Service crashes immediately**

**Step 1: Check error logs**

Run:
```bash
journalctl --user -eu SERVICE_NAME.service --tail -30
```

(Shows last 30 lines of error output)

**Step 2: Check if binary exists**

Run:
```bash
which SERVICE_NAME
```

or for nested binaries:

```bash
pacman -Q SERVICE_NAME
```

**Step 3: Disable and try exec-once instead**

If systemd service keeps failing, disable it and use exec-once:

Run:
```bash
systemctl --user disable SERVICE_NAME.service
```

Then add to `~/.config/hypr/hyprland.conf`:

```bash
exec-once = SERVICE_NAME
```

---

#### **Problem: swaync fails with "Could not acquire notification name" error**

**This is the most common issue. Root cause: swaync from official repos has D-Bus conflicts.**

**Error example (in logs):**
```bash
swaync[1234]: Could not acquire notification name. Please close any other notification daemon like mako or dunst
```

**Solution 1: Reinstall from AUR (RECOMMENDED)**

The official repo version has D-Bus state issues. A clean AUR build fixes this completely.

Run:
```bash
# Step 1: Kill all notification daemons
pkill -9 swaync dunst mako notify-daemon

# Step 2: Remove old installation
sudo pacman -R swaync

# Step 3: Clear package cache
sudo pacman -Sc

# Step 4: Install fresh from AUR
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/swaync.git
cd swaync
makepkg -si

# Step 5: Enable and start
systemctl --user enable swaync.service
systemctl --user start swaync.service

# Step 6: Verify
systemctl --user status swaync.service
```

**Verify it works:**
```bash
notify-send "Test" "Notification test"
```

(You should see a notification in the top-right corner)

---

#### **Problem: Services not starting after reboot**

**Step 1: Verify services are enabled**

Run:
```bash
systemctl --user list-unit-files | grep enabled
```

(Shows all enabled user services)

**Step 2: Re-enable them**

Run:
```bash
systemctl --user enable hyprpaper.service swaync.service xdg-desktop-portal.service xdg-desktop-portal-hyprland.service hyprpolkitagent.service cliphist.service
```

**Step 3: Reboot and verify**

Run:
```bash
reboot
```

After reboot:
```bash
pgrep -af "hyprpaper|swaync|xdg-desktop-portal|hyprpolkitagent|cliphist"
```

---

### 6.10 Final Verification (Fresh Reboot Test)

The best test is a complete logout and login cycle:

**Logout from Hyprland:**

Press `SUPER + SHIFT + Q` (or use exit keybind from wizard)

**Log back in to Hyprland**

Check all services immediately:

Run:
```bash
pgrep -af "hyprpaper|hyprpanel|swaync|hyprpolkitagent|wl-paste|cliphist|xdg-desktop-portal"
```

**All processes should appear within 2-3 seconds!**

If any are missing:
1. Check service status with `systemctl --user status SERVICE_NAME.service`
2. Check logs with `journalctl --user -eu SERVICE_NAME.service`
3. Refer to section 6.9 troubleshooting

---

### 6.11 Quick Reference: Service Management Commands

```bash
# Enable a service to start on login
systemctl --user enable SERVICE_NAME.service

# Start a service immediately
systemctl --user start SERVICE_NAME.service

# Check if service is running
systemctl --user status SERVICE_NAME.service

# Stop a service
systemctl --user stop SERVICE_NAME.service

# Disable a service (won't start on login)
systemctl --user disable SERVICE_NAME.service

# View service errors
journalctl --user -eu SERVICE_NAME.service

# List all enabled user services
systemctl --user list-unit-files | grep enabled

# Restart a service
systemctl --user restart SERVICE_NAME.service
```

---

### 6.12 What Each Service Does (Reference)

| Service | Purpose | If Missing |
|---------|---------|-----------|
| **hyprpaper** | Sets desktop wallpaper on startup | Desktop has default/black background |
| **swaync** | Notification daemon + control center | No notifications or popup alerts |
| **xdg-desktop-portal** | Base portal for system integration | File dialogs may not work properly |
| **xdg-desktop-portal-hyprland** | Hyprland-specific screen sharing support | Can't share screen in Discord/OBS/WebRTC |
| **hyprpolkitagent** | Asks for password when admin access needed | GUI apps crash when needing root permissions |
| **cliphist** | Stores clipboard history for retrieval | Can't access past copied items |
| **wl-paste** | Watches clipboard and sends to cliphist | Clipboard history won't capture new items |

---

### 6.13 Systemd Service Cheat Sheet

**After enabling services, they will:**
- ✅ Auto-start every time you log in
- ✅ Restart automatically if they crash
- ✅ Not duplicate instances on Hyprland reload
- ✅ Persist until you disable them
- ✅ Log to journalctl for easy debugging

**To verify at any time:**

```bash
systemctl --user status hyprpaper.service swaync.service xdg-desktop-portal-hyprland.service hyprpolkitagent.service cliphist.service
```

If anything shows `inactive`, use the troubleshooting steps in section 6.9.

---

**Congratulations! All services are now properly configured and managed by systemd.** 🎉

You now have:
- ✅ Wallpaper daemon running
- ✅ Notification center active
- ✅ Screen sharing enabled
- ✅ Password prompts working
- ✅ Clipboard history tracking
- ✅ Polkit authentication agent ready

**Next Steps:** Follow the Hyprland wiki configuration order from section 7 onwards.

---

**Step 1: Verify the binary exists**

Run:
```bash
ls -la /usr/lib/hyprpolkitagent/hyprpolkitagent
```

(Expected output: file path and permissions)

**If file not found:**

Reinstall the package:
```bash
sudo pacman -S hyprpolkitagent
```

**Step 2: Check systemd service status**

Run:
```bash
systemctl --user status hyprpolkitagent.service
```

**If status shows `inactive`:**

Start it manually:
```bash
systemctl --user start hyprpolkitagent.service
```

**Step 3: Check for errors**

Run:
```bash
journalctl --user -eu hyprpolkitagent.service
```

(Shows error messages if the service fails to start)

**If you see permission errors:**

Try running directly with full path:
```bash
/usr/lib/hyprpolkitagent/hyprpolkitagent &
```

---

#### **Problem: hyprpaper not showing wallpaper**

**Step 1: Verify hyprpaper is running**

Run:
```bash
pgrep hyprpaper
```

**Step 2: Check if hyprpaper config exists**

Run:
```bash
cat ~/.config/hypr/hyprpaper.conf
```

(Expected output: config file content)

**If file doesn't exist, create it:**

```bash
mkdir -p ~/.config/hypr
nvim ~/.config/hypr/hyprpaper.conf
```

Add this basic config:

```bash
preload = /path/to/your/wallpaper.png
wallpaper = HDMI-1,/path/to/your/wallpaper.png
```

Replace `HDMI-1` with your actual display name (find it with `hyprctl monitors`)

Replace `/path/to/your/wallpaper.png` with an actual wallpaper file.

---

#### **Problem: wl-paste clipboard not working**

**Step 1: Verify wl-clipboard is installed**

Run:
```bash
pacman -Q wl-clipboard
```

(Expected output: version number)

**If not installed:**

```bash
sudo pacman -S wl-clipboard cliphist
```

**Step 2: Test copying and pasting manually**

In your terminal, type something and copy it:

```bash
echo "test clipboard" | wl-copy
```

Then paste it back:

```bash
wl-paste
```

(Expected output: `test clipboard`)

**If wl-paste returns empty:**

Your clipboard might be broken. Try:

```bash
sudo pacman -Syu wl-clipboard
```

Then log out and log back in.

---

#### **Problem: xdg-desktop-portal not working (screen sharing fails)**

**Step 1: Verify the portal is running**

Run:
```bash
pgrep -f xdg-desktop-portal-hyprland
```

**Step 2: Check environment variables**

Run:
```bash
echo $XDG_CURRENT_DESKTOP
```

(Expected output: `Hyprland`)

**If output is empty or wrong:**

Add this to `~/.config/hypr/hyprland.conf`:

```bash
env = XDG_CURRENT_DESKTOP,Hyprland
```

Then reload:

```bash
hyprctl reload
```

**Step 3: Reinstall the portal**

```bash
sudo pacman -S xdg-desktop-portal-hyprland xdg-desktop-portal
```

---

### 6.6 Final Verification (After Reboot)

The best test is a fresh reboot. After rebooting and logging into Hyprland, run:

```bash
pgrep -af "hyprpaper|hyprpanel|hyprpolkitagent|wl-paste|xdg-desktop-portal"
```

**All services should appear immediately without manual starting.** If they don't, check the systemd service status as shown in section 6.3.

---

### 6.7 What Each Service Does

| Service | Purpose | If Missing |
|---------|---------|-----------|
| **hyprpaper** | Sets desktop wallpaper | Desktop looks plain/default |
| **hyprpanel** | Top status bar (time, battery, workspaces) | No status bar visible at top |
| **hyprpolkitagent** | Asks for password when admin apps run | Apps crash when needing root access |
| **wl-paste + cliphist** | Copy/paste history | Copy-paste broken or history unavailable |
| **xdg-desktop-portal** | Screen sharing (Discord, OBS) + file dialogs | Can't share screen, file open menus broken |

---
