<div align="center">
  <h1>Manage and sync all your passwords and one-time-passwords on Linux</h1>
  <a href="https://youtu.be/CwHCPvuJKgE">
    <img width="320px" height="180px" src="https://img.youtube.com/vi/CwHCPvuJKgE/mqdefault.jpg" style="border-radius: 1rem;" />
    <p>Watch the YouTube Tutorial</p>
  </a>
</div>

## Requirements

**Debian**

```bash
apt install git gnupg pass rofi pass-extension-otp zbar-tools
```

**Fedora**

```bash
dnf install git gnupg pass pass-otp rofi zbar
```

**Arch**

```bash
pacman -S git gnupg pass pass-otp rofi zbar wl-clipboard
```

## Setup `pass`

1. You need a GPG key

   ```bash
   gpg --list-keys
   gpg --full-gen-key
   ```

2. Backup your GPG key

   ```bash
   # backup
   gpg -o private.gpg --export-options backup --export-secret-keys <gpg_key_fingerprint>

   # restore
   gpg --import-options restore --import private.gpg
   ```

3. Initialize password store

   ```bash
   pass init <gpg_key_fingerprint>
   pass git init
   ```

4. Manage passwords

   ```bash
   pass help
   ```

   Insert given password

   ```bash
   pass insert <pass-name>
   ```

   Generate standard password

   ```bash
   pass generate <pass-name>
   ```

   Generate password with no symbols and custom length (standard length is 25)

   ```bash
   pass generate --no-symbols <pass-name> <pass-length>
   ```

   Edit a password

   ```bash
   pass edit <pass-name>
   ```

   Remove a password

   ```bash
   pass rm <pass-name>
   ```

   Rename a password

   ```bash
   pass mv <old-path> <new-path>
   ```

## Setup `passmenu`

1. Clone this repository

   ```bash
   git clone git@github.com:DennisKasper/pass
   cd pass
   ```

2. Install `passmenu` script

   ```bash
   sudo cp ./passmenu /usr/bin/
   ```

3. Assign hotkey to `passmenu`

   > In GNOME it can be done like this:

   - Settings 🠖 Keyboard 🠖 Shortcuts 🠖 Custom 🠖 Add
   - Enter `passmenu` as the "Command"
   - And set a "Shortcut" (e.g. `Ctrl` + `Alt` + `P`)

4. Fix menu not focusing

   > If you start the menu and it won't be focused, you need to disable Wayland and switch to X11. But don't worry it's very easy:

   - Edit `/etc/gdm/custom.conf`
   - Change `#WaylandEnable=false` to `WaylandEnable=false`
   - Reboot your computer
   - If you have a laptop and your touch gestures are broken afterwards, just install [X11 Gestures](https://extensions.gnome.org/extension/4033/x11-gestures) extension and [touchegg](https://github.com/JoseExposito/touchegg#installation)

## Synchronization

I recommend syncing your passwords through an encrypted Git repository. You can read more about the reasoning in my [blog post](https://flolu.de/blog/linux-password-manager-and-sync).

1. Install [git-remote-gcrypt](https://spwhitton.name/tech/code/git-remote-gcrypt)

   - Debian: `apt install git-remote-gcrypt`
   - Fedora: `dnf install git-remote-gcrypt`
   - Arch: `pacman -S git-remote-gcrypt`

2. Add encrypted remote

   ```bash
   pass git remote add <remote_name> gcrypt::<remote_url>
   pass git config remote.<remote_name>.gcrypt-participants "<key_fingerprint>"
   pass git config remote.<remote_name>.gcrypt-signingkey "<key_fingerprint>"
   ```

3. Push changes

   ```bash
   pass git push origin main
   ```

4. Pull changes

   ```bash
   git clone gcrypt::<remote_url>
   git pull origin main
   ```
