<div align="center">
  <h1>Manage and sync all your passwords and one-time-passwords on Linux</h1>
  <a href="https://youtu.be/CwHCPvuJKgE">
    <img width="320px" height="180px" src="https://img.youtube.com/vi/CwHCPvuJKgE/mqdefault.jpg" style="border-radius: 1rem;" />
    <p>Watch the YouTube Tutorial</p>
  </a>
</div>

## Requirements

**Debian**

```console
apt install git gnupg pass rofi pass-extension-otp zbar-tools wl-clipboard
```

**Fedora**

```console
dnf install git gnupg pass pass-otp rofi zbar wl-clipboard
```

**Arch**

```console
pacman -S git gnupg pass pass-otp rofi zbar wl-clipboard
```

## Setup `pass`

1. You need a GPG key

   ```console
   gpg --list-keys
   gpg --full-gen-key
   ```

2. Backup your GPG key

   ```console
   # backup
   gpg -o private.gpg --export-options backup --export-secret-keys <gpg_key_fingerprint>

   # restore
   gpg --import-options restore --import private.gpg
   ```

   > Note
   >
   > The trust level may need to be set when restoring the key

3. Initialize password store

   ```console
   pass init <gpg_key_fingerprint>
   pass git init
   ```

4. Manage passwords

   ```console
   pass help
   ```

   Insert given password

   ```console
   pass insert <pass-name>
   ```

   Generate standard password

   ```console
   pass generate <pass-name>
   ```

   Generate password with no symbols and custom length (standard length is 25)

   ```console
   pass generate --no-symbols <pass-name> <pass-length>
   ```

   Edit a password

   ```console
   pass edit <pass-name>
   ```

   Remove a password

   ```console
   pass rm <pass-name>
   ```

   Rename a password

   ```console
   pass mv <old-path> <new-path>
   ```

5. Manage one-time-passwords

   Insert an OTP key from a QR code screenshot

   ```console
   zbarimg -q --raw <qr-code.png> | pass otp insert <pass-name>
   ```

   Or insert an OTP key manually (prompts for the `otpauth://` URI)

   ```console
   pass otp insert <pass-name>
   ```

   Generate the current code

   ```console
   pass otp <pass-name>
   ```

   > Note
   >
   > `passmenu` copies the OTP code instead of the password for every entry that contains an `otpauth://` URI. So store OTP keys in their own entries (e.g. `otp/<pass-name>`) instead of appending them to a password entry.

## Setup `passmenu`

1. Clone this repository

   ```console
   git clone git@github.com:denniskasper/pass
   cd pass
   ```

2. Install `passmenu` script

   ```console
   sudo cp ./passmenu /usr/bin/
   ```

3. Assign hotkey to `passmenu`

   > In GNOME it can be done like this:

   - Settings 🠖 Keyboard 🠖 Keyboard Shortcuts 🠖 Custom 🠖 Add
   - Enter `passmenu` as the "Command"
   - And set a "Shortcut" (e.g. `Ctrl` + `Alt` + `Shift` + `P`)

> Note
>
> GNOME on Wayland doesn't support the layer-shell protocol that rofi needs, and an XWayland popup won't get keyboard focus. So when `passmenu` detects a GNOME Wayland session (`WAYLAND_DISPLAY` is set and `XDG_CURRENT_DESKTOP` contains `GNOME`), it runs rofi with `-x11 -normal-window` and passes GNOME's X11 DPI (`Xft.dpi`) as `-dpi`, so the menu is not tiny on scaled displays (this needs `xrdb`, on Debian part of `x11-xserver-utils`; without it the menu still works, just unscaled). No need to disable Wayland. Other Wayland compositors that support layer-shell use plain `rofi -dmenu`. Any arguments you pass to `passmenu` are forwarded to rofi.

## Synchronization

I recommend syncing your passwords through an encrypted Git repository. You can read more about the reasoning in my [blog post](https://flolu.de/blog/linux-password-manager-and-sync).

1. Install [git-remote-gcrypt](https://spwhitton.name/tech/code/git-remote-gcrypt)

   - Debian: `apt install git-remote-gcrypt`
   - Fedora: `dnf install git-remote-gcrypt`
   - Arch: `pacman -S git-remote-gcrypt`

2. Add encrypted remote

   ```console
   pass git remote add <remote_name> gcrypt::<remote_url>
   pass git config remote.<remote_name>.gcrypt-participants "<key_fingerprint>"
   pass git config remote.<remote_name>.gcrypt-signingkey "<key_fingerprint>"
   ```

3. Push changes

   ```console
   pass git push <remote_name> main
   ```

4. Pull changes

   ```console
   pass git pull <remote_name> main
   ```

   > Note
   >
   > To set up the password store on a new computer, see [Recover password-store](#recover-password-store)

## Recover password-store

1. Restore private GPG key

   ```console
   gpg --import-options restore --import private.gpg
   ```

   > Note
   >
   > The trust level may need to be set when restoring the key

2. Git clone the private repository

   ```console
   git clone gcrypt::<private_remote_url> ~/.password-store
   ```
