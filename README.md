# alpine
Guide for Alpine Linux Desktop


# setup-alpine
| Question                      | Option                                                                                   |
|-------------------------------|------------------------------------------------------------------------------------------|
| Keyboard layout / variant     | fi-classic                                                                               |
| Hostname                      | t480                                                                                     |
| Which interface to initialize | eth0 → dhcp → manual config? n (Wi‑Fi instead: pick wlan0, then enter SSID + passphrase) |
| Root password                 | set a strong one                                                                         |
| Timezone                      | Europe/Helsinki                                                                          |
| Proxy                         | none                                                                                     |
| NTP                           | chrony                                                                                   |
| APK mirror                    | 1                                                                                        |
| Setup a user?                 | alice → full name → password → SSH key: none                                             |
| SSH server                    | none                                                                                     |
| Which disk                    | nvme0n1                                                                                  |
| How to use it                 | sys (choose crypt instead if you want LUKS full‑disk encryption)                         |
| Erase disk?                   | y                                                                                        |

Reboot after `setup-alpine`

# setup-desktop
`setup-desktop gnome`

# Packages to install
```
apk update && apk upgrade

# --- Intel firmware + CPU microcode (Wi-Fi 8265, Intel Bluetooth, i915 GuC/HuC) ---
apk add linux-firmware-intel linux-firmware-i915 intel-ucode

# --- NetworkManager + Wi-Fi/Bluetooth plugins + CLI/TUI ---
apk add networkmanager networkmanager-wifi networkmanager-cli networkmanager-tui networkmanager-bluetooth

# --- Bluetooth stack ---
apk add bluez rfkill

# --- Firewall ---
apk add ufw

# --- Flatpak + GNOME Software plugin + portals ---
apk add flatpak gnome-software-plugin-flatpak xdg-desktop-portal xdg-desktop-portal-gnome xdg-desktop-portal-gtk

# --- Podman (rootless) + Docker-compatible CLI + compose ---
apk add podman podman-docker podman-compose crun slirp4netns passt shadow-subids

# --- Firmware updates + Thunderbolt ---
apk add fwupd fwupd-efi bolt

# --- Power management + Intel graphics / video acceleration ---
apk add tlp tlp-rdw mesa-vulkan-intel intel-media-driver libva-utils mesa-utils

# --- Terminal / system tools ---
apk add curl wget nano bash bash-completion git tree file lsof rsync unzip zip \
        man-db man-pages bind-tools ethtool iw pciutils usbutils lm-sensors smartmontools \
        nvme-cli powertop util-linux util-linux-misc lsblk mount umount doas-sudo-shim fastfetch alsa-utils
```

## Optional packages
```
# --- Fonts + GNOME virtual-FS extras ---
apk add font-noto font-noto-emoji font-noto-cjk gvfs-mtp gvfs-smb

# Bibata cursor
mkdir -p ~/.local/share/icons
wget https://github.com/ful1e5/Bibata_Cursor/releases/download/v2.0.7/Bibata.tar.xz
doas tar -xvf Bibata.tar.xz -C /usr/share/icons/

# Gnome utils
apk add gnome-tweaks gnome-shell-extensions papirus-icon-theme

# Flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install flathub com.mattjakeman.ExtensionManager
flatpak install flathub org.telegram.desktop
flatpak install flathub com.brave.Browser
flatpak install flathub org.nmap.Zenmap
flatpak install flathub com.mikrotik.WinBox
```

# System configuration
```
# rc services
rc-update add elogind default
rc-update add gdm default
rc-update add dbus default
rc-update add cgroups default
rc-update add networkmanager default
rc-update add bluetooth default
rc-update add ufw default
rc-update add tlp default
rc-update add chronyd default

# Firewall
ufw default deny incoming
ufw default allow outgoing
ufw enable

# Network, DHCP
nano /etc/network/interfaces

auto lo
iface lo inet loopback
#iface lo inet6 loopback

#auto eth0
#iface eth0 inet dhcp
#iface eth0 inet6 auto

#auto eth1
#iface eth1 inet dhcp
#iface eth1 inet6 auto

# Audio
alsactl store
rc-update add alsa default
```

# User in GNOME environment
```
doas apk update && doas apk upgrade # doas works (wheel + persist)

# Rootless Podman test
podman run --rm docker.io/library/alpine echo "Podman rootless works"
podman info | grep -i rootless

# Firmware updates
doas fwupdmgr refresh --force
doas fwupdmgr get-updates
doas fwupdmgr update

# Thunderbolt dock/devices
boltctl list
doas boltctl enroll <UUID>   # authorize a dock once

# Hardware checks
doas dmesg | grep -i microcode     # should show "Updated early"
cat /sys/power/mem_sleep           # want [deep]  (BIOS "Sleep State: Linux")
sensors
doas tlp-stat -b

# GNOME configurations
dconf load /org/gnome/ < gnome_settings.dconf
```

# GNOME extensions
https://extensions.gnome.org/extension/615/appindicator-support/  
https://extensions.gnome.org/extension/1160/dash-to-panel/  
https://extensions.gnome.org/extension/3628/arcmenu/  
https://extensions.gnome.org/extension/5461/osd-volume-number/  

# Cleanup after setup
```
doas apk del gnome-user-share apache2 apache2-webdav apache2-openrc mod_dnssd epiphany yelp yelp-xsl gnome-user-docs gnome-tour gnome-initial-setup
doas rm -f /etc/wpa_supplicant/wpa_supplicant.conf
```
