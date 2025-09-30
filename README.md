# ubuntu-desktop-devbox

## Step 1: install ubuntu

no-USB to install Ubuntu 24.04 Desktop over an existing Windows install by booting the Ubuntu ISO directly from your internal drive, then wiping Windows during the Ubuntu install.

**Method:** boot the ISO from disk using GRUB2Win (Windows 10/11)

1) Prep Windows

- Disable Fast Startup:
  - Control Panel → Hardware and Sound → Power Options → Choose what the power buttons do → Change settings that are currently unavailable → uncheck “Turn on fast startup”.
- If BitLocker is enabled, suspend it:
  - Control Panel → BitLocker Drive Encryption → Suspend.
- Download Ubuntu 24.04.1 Desktop ISO:
  - Save it as C:\iso\ubuntu-24.04.1-desktop-amd64.iso (create C:\iso if needed).
- Verify the ISO (optional but recommended):
  - Open PowerShell as admin:
    ```bash
    certutil -hashfile "C:\iso\ubuntu-24.04.1-desktop-amd64.iso" SHA256
    ```
2) Install and configure GRUB2Win

- Install GRUB2Win (it’s free and UEFI-friendly).
- Launch GRUB2Win → “Manage Boot Menu” → “Add A New Entry” → choose “ISO Boot”.
- Point it to C:\iso\ubuntu-24.04.1-desktop-amd64.iso and select UEFI mode. Save the entry.
- If you don’t see the ISO wizard, add a custom entry and paste this:
  ```
  menuentry "Ubuntu 24.04 Desktop ISO" {
    insmod part_gpt
    insmod fat
    insmod ntfs
    insmod iso9660
    insmod loopback
    set isofile="/iso/ubuntu-24.04.1-desktop-amd64.iso"
    search --no-floppy --file $isofile --set=root
    loopback loop $root$isofile
    linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=$isofile noprompt noeject ---
    initrd (loop)/casper/initrd
  }
  ```
  Notes:
  - The path must match exactly where you placed the ISO: C:\iso\… is /iso/… in GRUB.
  - This works with Secure Boot on most systems; if it doesn’t boot, temporarily disable Secure Boot in BIOS, then re-enable it after Ubuntu is installed.

3) Reboot and start the Ubuntu installer
- Reboot → pick the “GRUB2Win” entry → choose “Ubuntu 24.04 Desktop ISO”.
- In the Ubuntu live environment, click “Install Ubuntu”.
- Recommended choices for a dev machine:
  - Updates and third-party software: check both boxes.
  - Installation type: Minimal installation (you’ll add tools via Ansible later).
  - Disk: choose “Erase disk and install Ubuntu” to replace Windows completely.
  - Optional: enable encryption if you want full-disk encryption (you’ll set a passphrase).

4) BIOS storage mode (only if the disk isn’t detected)
- If the installer doesn’t see your NVMe/SATA drive, it’s likely set to Intel RST/RAID.
- Enter BIOS/UEFI and set “Storage/SATA mode” to AHCI (disable Intel RST/Optane).
- Boot the ISO again via GRUB2Win and proceed with the install.

5) Finish install
- Let the installer run; it loads into RAM, so erasing Windows is safe at this point.
- When finished, remove any GRUB2Win entries from BIOS if you like. After the wipe, Windows is gone anyway; Ubuntu will set the default UEFI entry to “ubuntu”.

6) First boot checklist
- Update and reboot:
  - sudo apt update && sudo apt full-upgrade -y && sudo reboot

### Troubleshooting

- ISO entry doesn’t appear or won’t boot:
  - Ensure the ISO path is exactly C:\iso\… and the filename matches.
  - Try temporarily disabling Secure Boot; re-enable after Ubuntu is installed.
- Installer can’t see the disk:
  - Switch BIOS storage to AHCI (disable RST/RAID/Optane), then retry.
- Black screen after install (rare with NVIDIA):
  - At GRUB, press E on the Ubuntu entry, append nomodeset to the linux line, boot; then install NVIDIA drivers via “Additional Drivers”.


## Step 2: Make the setup reproducible (Ansible, local run)
- Install basics:
  ```
  sudo apt update
  sudo apt install -y git ansible python3-apt
  ```
- Create or clone your config repo (from our earlier plan). Example quick-start:
  ```
  git clone https://github.com/yourname/ubuntu-desktop-devbox.git
  cd ubuntu-desktop-devbox
  ansible-galaxy collection install -r requirements.yml
  ansible-playbook -K site.yml
  ```