# មគ្គុទ្ទេសក៍ Caelestia Shell — ពីដើមដល់ចប់

> សរសេរដោយ Mr-Reaz
> កែរចុងក្រោយ: មិថុនា 2026

---

## មាតិកា

1. [សេចក្ដីផ្ដើម](#១-សេចក្ដីផ្ដើម)
2. [ការតំឡើង Arch Linux + Hyprland + Caelestia](#២-ការតំឡើង-arch-linux--hyprland--caelestia)
3. [របៀបកែប្រែកូដ](#៣-របៀបកែប្រែកូដ)
4. [ឯកសារសំខាន់ៗដែលត្រូវដឹង](#៤-ឯកសារសំខាន់ៗដែលត្រូវដឹង)
5. [ការធ្វើ Version ថ្មី](#៥-ការធ្វើ-version-ថ្មី)
6. [ការដាក់ទៅ GitHub](#៦-ការដាក់ទៅ-github)
7. [សំនួរញឹកញាប់](#៧-សំនួរញឹកញាប់)

---

## ១. សេចក្ដីផ្ដើម

### តើ Caelestia Shell ជាអ្វី?

Caelestia Shell ជា **desktop shell UI** សម្រាប់ Hyprland (Wayland compositor) លើ Linux។ វាសរសេរដោយ QML (Qt) និង C++។ វាមានមុខងារដូចជា៖

| ម៉ូឌុល | ឯកសារ | ពិពណ៌នា |
|---------|--------|-----------|
| Bar/Taskbar | `modules/bar/` | របារខាងក្រោម, workspaces, clock, tray |
| Launcher | `modules/launcher/` | ប្រអប់ស្វែងរកកម្មវិធី, calculator, wallpaper |
| Lock Screen | `modules/lock/` | អេក្រង់ចាក់សោ, password, fingerprint |
| Nexus (Settings) | `modules/nexus/` | ការកំណត់, audio, bluetooth, network |
| Dashboard | `modules/dashboard/` | ផ្ទាំងគ្រប់គ្រង, media, performance |
| Notifications | `modules/notifications/` | ការជូនដំណឹង |
| OSD | `modules/osd/` | បង្ហាញកម្រិតសំឡេង/ពន្លឺ |
| Sidebar | `modules/sidebar/` | របារចំហៀងសម្រាប់ notifications |
| Session | `modules/session/` | ប្រអប់ shutdown/reboot/logout |
| Background | `modules/background/` | ផ្ទាំងខាងក្រោយ, wallpaper, visualiser |
| Window Info | `modules/windowinfo/` | ព័ត៌មានបង្អួច |
| Utilities | `modules/utilities/` | toasts, recording, quick toggles |

### តើត្រូវការអ្វីខ្លះ?

| អ្វីដែលត្រូវការ | ពិពណ៌នា |
|-----------------|-----------|
| Laptop/PC | 4GB RAM+, GPU ទ្រទ្រង់ Vulkan |
| USB drive | 8GB+ សម្រាប់ boot Arch installer |
| Internet | ទាញយក packages |
| Time | ~1 ម៉ោងសម្រាប់ដំឡើងដំបូង |

---

## ២. ការតំឡើង Arch Linux + Hyprland + Caelestia

### ២.១ ត្រៀម USB Bootable

```bash
# Windows (PowerShell as Admin):
# 1. ដោត USB ចូល
# 2. រកមើល USB disk number:
Get-Disk

# 3. សរសេរ ISO ចូល USB (ប្រយ័ត្ន replace X ជាមួយ disk number របស់ USB):
# ទាញយក Rufus: https://rufus.ie
# ឬប្រើ command:
# diskpart
# list disk
# select disk X
# clean
# create partition primary
# format fs=fat32 quick
# active
# assign
# exit
# រួច copy ឯកសារ ISO ចូល USB (extract ដោយ 7zip ឬដុតដោយ Rufus)
```

### ២.២ កំណត់ BIOS

1. Reboot → ចុច F2/Del/F12 (អាស្រ័យលើម៉ាស៊ីន)
2. បើក **UEFI boot** (បិទ Legacy/CSM)
3. **Secure Boot** → **Disable**
4. Boot order → USB first
5. Save & Exit

### ២.៣ ដំឡើង Arch Linux

ពេល boot ចូល USB រួច ដំណើរការ command ទាំងនេះ៖

```bash
# ពិនិត្យ internet
ping -c 3 google.com

# ពិនិត្យ UEFI
ls /sys/firmware/efi/efivars   # ត្រូវតែមាន files

# មើល partitions
lsblk
```

#### ចែក Partition (Manual)

```bash
# បើចង់ដាក់ dual boot ជាមួយ Windows:
# យើងបាន shrink D: ទុករួចហើយ ត្រូវតែមាន free space ទំនេរ
fdisk /dev/nvme0n1    # ឬ /dev/sda
```

ធ្វើតាមនេះក្នុង `fdisk`:

| Command | អត្ថន័យ |
|---------|-----------|
| `g` | បង្កើត GPT partition table |
| `n` → Enter → `+1G` → `t` → `1` | EFI partition (1GB) → type EFI |
| `n` → Enter → Enter | Root partition (free space នៅសល់) |
| `w` | សរសេររួចចេញ |

```bash
# Format
mkfs.fat -F32 /dev/nvme0n1p1    # EFI
mkfs.ext4 /dev/nvme0n1p2         # root

# Mount
mount /dev/nvme0n1p2 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot

# បើមាន Windows partition ចង់ mount:
mkdir /mnt/windows
mount /dev/nvme0n1p3 /mnt/windows   # Windows partition
```

#### ដំឡើង base system

```bash
pacstrap -K /mnt base linux linux-firmware base-devel git neovim networkmanager sudo cmake ninja

genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt
```

#### កំណត់រចនាសម្ព័ន្ធ

```bash
# Time
ln -sf /usr/share/zoneinfo/Asia/Phnom_Penh /etc/localtime
hwclock --systohc

# Language
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Hostname
echo "arch" > /etc/hostname
cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch.localdomain arch
EOF

# Password
passwd    # ដាក់ root password
useradd -mG wheel caelestia
passwd caelestia
echo "%wheel ALL=(ALL:ALL) ALL" >> /etc/sudoers

# Bootloader (systemd-boot)
bootctl install
cat > /boot/loader/loader.conf <<EOF
default arch.conf
timeout 4
console-mode max
editor no
EOF

# បើ dual boot ជាមួយ Windows បន្ថែម:
cat > /boot/loader/entries/arch.conf <<EOF
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p2) rw
EOF

cat > /boot/loader/entries/windows.conf <<EOF
title   Windows 11
efi     /EFI/Microsoft/Boot/bootmgfw.efi
EOF

# Services
systemctl enable NetworkManager systemd-resolved
```

### ២.៤ ដំឡើង Hyprland + Caelestia

```bash
# ក្នុង chroot:
pacman -Syu --noconfirm --needed \
  hyprland qt6-declarative qt6-base \
  ddcutil brightnessctl networkmanager lm_sensors fish pipewire wireplumber \
  libqalculate ttf-cascadia-code-nerd material-symbols-fonts noto-fonts \
  swappy aubio libcava

# AUR helper
cd /tmp
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si --noconfirm --needed

# Caelestia
yay -S --noconfirm --needed quickshell-git caelestia-shell-git

# ចាកចេញ
exit   # ចាកចេញពី chroot
umount -R /mnt
reboot
```

### ២.៥ ចាប់ផ្ដើម Caelestia

ពេល reboot រួច login ជា `caelestia`៖

```bash
# ចាប់ផ្ដើម Hyprland
Hyprland

# ក្នុង Hyprland បើក terminal (Super+Q):
caelestia shell -d
```

បើចង់ auto-start ពេល login ចូល Hyprland៖

```bash
echo "exec-once = caelestia shell -d" >> ~/.config/hypr/hyprland.conf
```

---

## ៣. របៀបកែប្រែកូដ

### ៣.១ ទាញយកកូដពី GitHub

```bash
# បើក terminal ក្នុង Hyprland (Super+Q)
cd ~/.config
git clone https://github.com/Mr-Reaz/Reaz.git quickshell/caelestia
cd quickshell/caelestia

# Build ម្ដងដំបូង
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/ -DINSTALL_QSCONFDIR=~/.config/quickshell/caelestia
cmake --build build
sudo cmake --install build
sudo chown -R $USER ~/.config/quickshell/caelestia
```

### ៣.២ កែ QML (UI) — reload ស្វ័យប្រវត្តិ

**QML** ជា Qt Markup Language — ងាយស្រួលកែរូបរាង។ ព្រោះ `shell.qml` មាន `watchFiles: true` → កែរួច save → reload ខ្លួនឯង។

#### ឧទាហរណ៍៖ កែពណ៌ Bar

File: `modules/bar/Bar.qml`

```qml
// រកពាក្យ "color" ឬ "background" ក្នុង file
// ឧ. កែពណ៌ background:
Rectangle {
    color: "#1e1e2e"   // ផ្លាស់ប្ដូរតម្លៃនេះ
}
```

#### ឧទាហរណ៍៖ កែទំហំ Clock

File: `modules/bar/components/Clock.qml`

```qml
// កែ font size:
text: "HH:mm"
font.pixelSize: 16    // បង្កើន/បន្ថយតម្លៃនេះ
```

#### ឧទាហរណ៍៖ បន្ថែម button ថ្មី

File: `modules/bar/Bar.qml`

```qml
// បន្ថែមក្នុង Row {}:
IconButton {
    icon: "home"
    onClicked: print("Home clicked!")
}
```

### ៣.៣ កែ Config — ដោយមិនប៉ះកូដ

File: `~/.config/caelestia/shell.json`

```json
{
    "bar": {
        "persistent": true,
        "entries": [
            { "id": "logo", "enabled": true },
            { "id": "workspaces", "enabled": true },
            { "id": "spacer", "enabled": true },
            { "id": "clock", "enabled": true },
            { "id": "statusIcons", "enabled": true }
        ]
    },
    "launcher": {
        "vimKeybinds": true,
        "favouriteApps": ["firefox", "code", "thunar"]
    },
    "appearance": {
        "rounding": { "scale": 1.2 },
        "font": { "scale": 1.1 }
    }
}
```

កែ `shell.json` → save → Caelestia reload ស្វ័យប្រវត្តិ។

### ៣.៤ កែ C++ (Backend) — ត្រូវ rebuild

C++ files នៅក្នុង `plugin/src/Caelestia/` គ្រប់គ្រង៖

| File | មុខងារ |
|------|---------|
| `Services/cpu.cpp` | អាន CPU usage |
| `Services/gpu.cpp` | អាន GPU info |
| `Services/lyrics.cpp` | ទាញយក lyrics |
| `Services/storage.cpp` | អាន disk usage |
| `Blobs/blobshape.cpp` | ការ animation blob |
| `Config/configobject.cpp` | ការអាន config |
| `Internal/hyprextras.cpp` | Hyprland IPC |
| `appdb.cpp` | បញ្ជីកម្មវិធី |

ឧទាហរណ៍៖ កែពេលវេលា update CPU

File: `plugin/src/Caelestia/Services/cpu.cpp`

```cpp
// រក update interval:
m_timer.setInterval(1000);   // ផ្លាស់ប្ដូរតម្លៃនេះ (ms)
```

ក្រោយកែ C++ ត្រូវ rebuild៖

```bash
cd ~/.config/quickshell/caelestia
cmake --build build
sudo cmake --install build
```

រួច restart Caelestia៖

```bash
caelestia shell -d    # ឬ kill ចាស់ហើយរត់ថ្មី
```

### ៣.៥ កែ Services (QML Backend)

File: `services/` — logic ខាងក្រោយសរសេរដោយ QML៖

| File | មុខងារ |
|------|---------|
| `services/Audio.qml` | គ្រប់គ្រង audio (PipeWire) |
| `services/Network.qml` | គ្រប់គ្រង network |
| `services/Brightness.qml` | គ្រប់គ្រងពន្លឺអេក្រង់ |
| `services/Colours.qml` | គ្រប់គ្រង color scheme |
| `services/Wallpapers.qml` | គ្រប់គ្រង wallpaper |
| `services/Weather.qml` | ទាញយកអាកាសធាតុ |
| `services/Bluetooth.qml` | Bluetooth manager |

ឧទាហរណ៍៖ កែ weather location

File: `services/Weather.qml`

```qml
// រក URL ឬ API key:
const API_URL = "https://api.openweathermap.org/data/2.5/weather"
const CITY = "Phnom Penh"   // ផ្លាស់ប្ដូរទីក្រុង
```

### ៣.៦ កែ Components (Reusable UI)

File: `components/` — widgets ដែលប្រើឡើងវិញ៖

| File | មុខងារ |
|------|---------|
| `controls/IconButton.qml` | Button មាន icon |
| `controls/StyledSlider.qml` | Slider |
| `controls/StyledSwitch.qml` | Switch toggle |
| `controls/StyledInputField.qml` | Input field |
| `controls/Menu.qml` | Dropdown menu |
| `effects/Colouriser.qml` | ការលាបពណ៌ |
| `effects/Elevation.qml` | Shadow effect |

### ៣.៧ កែ Lock Screen

File: `modules/lock/`

| File | មុខងារ |
|------|---------|
| `Lock.qml` | Main lock screen |
| `center/Clock.qml` | នាឡិកាលើ lock screen |
| `center/PasswordInput.qml` | ប្រអប់បញ្ចូលលេខសម្ងាត់ |
| `center/ProfilePic.qml` | រូបថត profile |
| `Pam.qml` | PAM authentication |
| `Media.qml` | Media player លើ lock screen |
| `weather/BriefInfo.qml` | អាកាសធាតុ |

### ៣.៨ កែ Nexus (Settings/Control Center)

File: `modules/nexus/`

| File | មុខងារ |
|------|---------|
| `Nexus.qml` | Main settings window |
| `NavPane.qml` | របារនាំផ្លូវ |
| `pages/AudioPage.qml` | ទំព័រ audio settings |
| `pages/BluetoothPage.qml` | ទំព័រ bluetooth |
| `pages/NetworkPage.qml` | ទំព័រ network (WiFi) |
| `pages/WallpaperAndStyle.qml` | ទំព័រ wallpaper + style |
| `pages/PanelsPage.qml` | កំណត់ bar/dashboard/launcher |

### ៣.៩ កែ Launcher

File: `modules/launcher/`

| File | មុខងារ |
|------|---------|
| `Content.qml` | Main launcher window |
| `AppList.qml` | បញ្ជីកម្មវិធី |
| `WallpaperList.qml` | បញ្ជី wallpaper |
| `items/AppItem.qml` | Item កម្មវិធីនីមួយៗ |
| `items/CalcItem.qml` | Calculator |
| `services/Apps.qml` | Service សម្រាប់រកកម្មវិធី |
| `services/Schemes.qml` | Color schemes |

---

## ៤. ឯកសារសំខាន់ៗដែលត្រូវដឹង

### រចនាសម្ព័ន្ធថត

```
~/.config/quickshell/caelestia/
├── shell.qml                    # Entry point (main)
├── modules/                     # UI modules
│   ├── bar/                     # Taskbar
│   ├── launcher/                # App launcher
│   ├── lock/                    # Lock screen
│   ├── nexus/                   # Settings (control center)
│   ├── dashboard/               # Dashboard
│   ├── notifications/           # Notifications
│   ├── sidebar/                 # Sidebar
│   ├── osd/                     # On-screen display
│   ├── session/                 # Shutdown/logout
│   ├── background/              # Wallpaper
│   ├── windowinfo/              # Window info
│   └── utilities/               # Toasts, quick toggles
├── services/                    # Backend logic (QML)
├── components/                  # Reusable widgets
│   ├── controls/                # Buttons, sliders, switches
│   ├── effects/                 # Visual effects
│   ├── containers/              # Windows, lists
│   └── images/                  # Image caching
├── plugin/src/Caelestia/        # C++ backend
│   ├── Services/                # CPU, GPU, storage, lyrics
│   ├── Config/                  # Config reading
│   ├── Blobs/                   # Blob animation
│   ├── Internal/                # Hyprland IPC
│   ├── Images/                  # Image processing
│   └── Models/                  # File system model
├── utils/                       # Helper utilities
├── assets/                      # Images, fonts, scripts
├── CMakeLists.txt               # Build config
└── README.md                    # This file
```

### ឯកសារសំខាន់ដែលកែញឹកញាប់

| ឯកសារ | តួនាទី | កែអ្វីបាន |
|---------|----------|-----------|
| `~/.config/caelestia/shell.json` | Config | រាល់ការកំណត់ទាំងអស់ |
| `modules/bar/Bar.qml` | Taskbar | Layout, ពណ៌, ទំហំ |
| `modules/bar/components/workspaces/Workspaces.qml` | Workspaces | រូបរាង workspace |
| `modules/bar/components/Clock.qml` | នាឡិកា | Format, font, ពណ៌ |
| `modules/launcher/Content.qml` | Launcher | រូបរាង launcher |
| `modules/lock/Lock.qml` | Lock screen | រូបរាង lock screen |
| `modules/lock/center/Clock.qml` | Lock screen clock | នាឡិកា, date |
| `modules/nexus/Nexus.qml` | Settings | ទំព័រ settings |
| `modules/dashboard/Dash.qml` | Dashboard | រូបរាង dashboard |
| `services/Colours.qml` | Color scheme | ពណ៌ប្រព័ន្ធ |
| `services/Wallpapers.qml` | Wallpaper | របៀប wallpaper ដំណើរការ |

---

## ៥. ការធ្វើ Version ថ្មី

### ៥.១ ការដាក់ឈ្មោះ Version

```
v1.0.0  → Major.Minor.Patch

- Major: កែរចនាសម្ព័ន្ធធំ (breaking changes)
- Minor: បន្ថែម feature ថ្មី
- Patch: កែកំហុស (bug fix)
```

### ៥.២ ត្រៀមបង្កើត Version ថ្មី

```bash
# ប្ដូរទៅ git repo
cd ~/.config/quickshell/caelestia

# ពិនិត្យស្ថានភាព
git status
git log --oneline -5

# បង្កើត branch ថ្មី (បើចង់)
git checkout -b v2.0-dev
```

### ៥.៣ កែប្រែសំខាន់ៗសម្រាប់ Version ថ្មី

#### កែឈ្មោះ Shell

File: `shell.qml` (កំពូល)

```qml
// កែ​ pragma ឬ metadata
// ឬផ្លាស់ប្ដូរ import paths
```

#### កែ Logo

File: `components/Logo.qml`
File: `assets/logo.svg` (ផ្លាស់ប្ដូររូប logo)

```qml
// កែរូបរាង logo
Image {
    source: "assets/logo.svg"
    width: 32
    height: 32
}
```

#### កែ Fonts

File: `plugin/src/Caelestia/Config/font.hpp`

```cpp
// កែ default font family
static constexpr auto DEFAULT_FONT = "GoogleSansFlex";
```

File: `components/StyledText.qml`

```qml
// កែ default text style
font.family: Tokens.font.body.medium.family
font.pixelSize: Tokens.font.body.medium.size
```

#### កែពណ៌ Brand

File: `plugin/src/Caelestia/Config/tokens.hpp`

```cpp
// កែ primary/accent color
static constexpr auto PRIMARY = "#9ccbfb";   // ផ្លាស់ប្ដូរតម្លៃ
```

#### កែ Animation

File: `plugin/src/Caelestia/Config/anim.hpp`

```cpp
// កែល្បឿន animation
static constexpr qreal DURATION_SCALE = 1.0; // បង្កើន/បន្ថយ
```

### ៥.៤ ដាក់ Version ថ្មី

```bash
# កែរួច commit
git add .
git commit -m "v2.0.0: បន្ថែម feature ថ្មី"

# ដាក់ tag
git tag -a v2.0.0 -m "Version 2.0.0"

# Push
git push origin main
git push origin v2.0.0
```

---

## ៦. ការដាក់ទៅ GitHub

### ៦.១ បង្កើត Repo ថ្មី

```bash
# បើក terminal:
echo "# Caelestia Shell - My Version" > README.md
git init
git checkout -b main
git add .
git commit -m "Initial commit"

# បន្ថែម remote
git remote add origin https://github.com/ឈ្មោះអ្នក/ឈ្មោះrepo.git

# Push
git push -u origin main
```

### ៦.២ កំណត់ License

បើចង់ដាក់ license ថ្មី៖

- MIT → អនុញ្ញាតអោយគេយកទៅប្រើដោយសេរី
- GPL → ត្រូវតែបើកចំហ (open source)
- Apache 2.0 → សុវត្ថិភាពសម្រាប់ក្រុមហ៊ុន

ឬ **All Rights Reserved** (គ្មាន license) → គ្មាននរណាអាចយកទៅប្រើបានទេ។

### ៦.៣ ដាក់ Release

លើ GitHub:
1. ចូល repo → Releases → Create a new release
2. Tag: `v1.0.0`
3. Title: `Version 1.0.0`
4. Description: សរសេរអ្វីដែលកែប្រែ
5. Publish release

---

## ៧. សំនួរញឹកញាប់

### សំនួរ: ដំឡើងហើយមិនឃើញ Caelestia?

ដំណើរការ:
```bash
caelestia shell -d    # ចាប់ផ្ដើម shell
```

បើឃើញ error:
```bash
qs -c caelestia    # មើល log
```

### សំនួរ: កែ QML ហើយមិន reload?

មើល `shell.qml` ថាមាន `settings.watchFiles: true` ឬទេ។ បើអត់ បន្ថែមវា។

### សំនួរ: កែ C++ ហើយមិនឃើញប្រែប្រួល?

ត្រូវ rebuild ជានិច្ច៖
```bash
cd ~/.config/quickshell/caelestia
cmake --build build
sudo cmake --install build
caelestia shell -d
```

### សំនួរ: ចង់ reset config ទាំងអស់?

```bash
rm -rf ~/.config/caelestia
rm -rf ~/.config/quickshell/caelestia
```

រួចដំឡើងថ្មីម្ដងទៀត។

### សំនួរ: ចង់បន្ថែម shortcut ថ្មី?

File: `modules/Shortcuts.qml`

```qml
CustomShortcut {
    sequence: "Super+Shift+N"
    onTriggered: {
        // ដំណើរការអ្វីមួយ
        print("Shortcut triggered!")
    }
}
```

### សំនួរ: ចង់បន្ថែម Widget ថ្មី?

1. បង្កើត file ថ្មីក្នុង `modules/` (ឧ. `modules/mywidget/MyWidget.qml`)
2. Import វាក្នុង `shell.qml`:

```qml
import "modules/mywidget"

ShellRoot {
    MyWidget {}
    // ... component ផ្សេងៗ
}
```

### សំនួរ: បើកមិនរួច / VM មិន boot?

- ពិនិត្យ BIOS → UEFI enabled, Secure Boot disabled
- ពិនិត្យ USB boot order
- ពិនិត្យ ISO file ខូចឬទេ (re-download)

---

## សេចក្ដីបញ្ចប់

Caelestia Shell ជា desktop shell ដែលមានភាពបត់បែនខ្ពស់។ អ្នកអាចកែប្រែបានគ្រប់យ៉ាងតាមចិត្ត — ពីពណ៌, font, layout, animation រហូតដល់ backend logic។

អ្វីដែលសំខាន់ត្រូវចាំ:
- **QML** → កែរួច save → reload ស្វ័យប្រវត្តិ
- **C++** → កែរួច rebuild ជានិច្ច
- **Config** → កែ `shell.json`
- **Git** → commit ទុកជាប្រវត្តិ

សូមសំណាងល្អក្នុងការកែប្រែ! 🚀
