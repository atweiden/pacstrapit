pacstrap it: install arch the easy way
======================================

<table>
<tr><td>Last tested</td><td>2014-04-14 with archlinux-2014.04.01-dual.iso</td></tr>
</table>


Features
--------

- Btrfs on LUKS done for you
- Supports persistent USB installation
- Interactive and cmdline configuration options


Assumptions
-----------

You:

1. have no need for dual-booting Windows
2. are comfortable working on the command line
3. are starting from a brand new or completely blank hard drive (`shred -fvz -n 7 /dev/sdX`)


Instructions
------------

1. Burn LiveCD/LiveUSB with latest [Arch ISO](https://www.archlinux.org/download/)
2. Boot from LiveCD/LiveUSB
3. Connect to the Internet: `wifi-menu -o`
4. Download pacstrapit: `curl -k https://codeload.github.com/atweiden/{pacstrapit}/{tar.gz}/{0.5.16} -o "#1-#3.#2"`
5. Extract: `tar xvzf pacstrapit-0.5.16.tar.gz`
6. Customize defaults (recommended even if using cmdline flags or environment variables)

WARNING: failure to give appropriate values could cause catastrophic
data loss and system instability.

Defaults:

<table>
<tr><td>Target partition</td><td>/dev/sda</td></tr>
<tr><td>Type of processor</td><td>other</td></tr>
<tr><td>Type of graphics card</td><td>Intel</td></tr>
<tr><td>Type of hard drive</td><td>HDD</td></tr>
<tr><td>Locale</td><td>en_US</td></tr>
<tr><td>Keymap</td><td>us</td></tr>
<tr><td>Timezone</td><td>America/Los_Angeles (PST)</td></tr>
<tr><td>Root password</td><td>secret</td></tr>
<tr><td>User name</td><td>guru</td></tr>
<tr><td>User password</td><td>secret</td></tr>
<tr><td>LUKS name</td><td>lux</td></tr>
<tr><td>LUKS password</td><td>secret</td></tr>
<tr><td>Hostname</td><td>luxor</td></tr>
<tr><td>Hosts allowed</td><td>192.168.0. (LAN)</td></tr>
</table>

> `cd pacstrapit-0.5.16 && $EDITOR pacstrapit`

Done.

Usage
-----

Run `pacstrapit` for a capable cmdline persistent USB installation
in /dev/sdb:

```bash
./pacstrapit start --bundle    "base"                     \
                   --username  "newusername"              \
                   --userpass  "your new user's password" \
                   --rootpass  "your root password"       \
                   --lukspass  "your LUKS password"       \
                   --hostname  "yourhostname"             \
                   --partition "/dev/sdb"                 \
                   --processor "other"                    \
                   --graphics  "intel"                    \
                   --disk      "usb"                      \
                   --luksname  "infinity"                 \
                   --locale    "en_US"                    \
                   --keymap    "us"                       \
                   --timezone  "America/Los_Angeles"
```

...use the `select` option instead of `bundle` for more control:

```bash
./pacstrapit start --select    "base gui python nimrod"   \
                   --dotfiles                             \
                   --username  "newusername"              \
                   --userpass  "your new user's password" \
                   --rootpass  "your root password"       \
                   --lukspass  "your LUKS password"       \
                   --hostname  "yourhostname"             \
                   --partition "/dev/sdb"                 \
                   --processor "other"                    \
                   --graphics  "intel"                    \
                   --disk      "usb"                      \
                   --luksname  "infinity"                 \
                   --locale    "en_US"                    \
                   --keymap    "us"                       \
                   --timezone  "America/Los_Angeles"
```

Tip:

> Omit `--bundle` and `--select` to create a very minimal headless system.
>
> The `base` bundle adds CLI, vim, and bitcoin pkgs
>
> The `lite` bundle includes the base bundle plus GUI pkgs and dotfiles
>
> The `full` bundle includes all pkgs and dotfiles

Run `pacstrapit` interactively:

```bash
./pacstrapit start -i
```

Tip:

> Interactive mode allows you to manually select all important
> options. Use concealed mode (`--concealed` | `-c`) if you're only
> interested in manually inputting your user password, root password
> and LUKS password without echoing it to console.

...with ssh access to target machine enabled:

```bash
./pacstrapit start -i -s
```

...with verbose output:

```bash
./pacstrapit start -i -s -V
```

Run `pacstrapit` using default variables in config section, but with
interactive password input:

```bash
./pacstrapit start --concealed
```

...with logging:

```bash
./pacstrapit start 2>&1 | tee pacstrapit.log
```

Run `pacstrapit` with username and password set from environment variables:

```bash
export USERNAME="sky"
export USERPASS="sailing"
./pacstrapit start
```

Run `pacstrapit` for a lightweight remote server installation:

```
./pacstrapit start --username     'newusername'  \
                   --hostname     'yourhostname' \
                   --partition    '/dev/xvda'    \
                   --processor    'intel'        \
                   --graphics     'intel'        \
                   --disk         'ssd'          \
                   --luksname     'infinity'     \
                   --locale       'en_US'        \
                   --keymap       'us'           \
                   --timezone     'UTC'          \
                   --hostsallowed '*'            \
                   --ssh                         \
                   --dotfiles                    \
                   --concealed                   \
                   --verbose
```

Run `pacstrapit` for a workstation installation:

```bash
./pacstrapit start --bundle    "full"                \
                   --username  "newusername"         \
                   --hostname  "yourhostname"        \
                   --partition "/dev/sda"            \
                   --processor "intel"               \
                   --graphics  "nvidia"              \
                   --disk      "ssd"                 \
                   --luksname  "infinity"            \
                   --locale    "en_US"               \
                   --keymap    "us"                  \
                   --timezone  "America/Los_Angeles" \
                   --concealed
```


If the script exits with an error, it's best to reboot and start fresh
with a blank disk (`shred -fvz -n 0 /dev/sdX`).


Optional: sshd
--------------

If you intend to enable SSH access, you'll want to reset the
values of `_electrum_pubkey_default`, `_user_pubkey_default`, and
`_user_pubkey_sig_default`.

Before setting `_ssh` to `1`...

**On the control machine**:

Make keys directory.

```bash
$ mkdir -p ~/keys
```

Generate ssh keys.

```bash
$ ssh-keygen -t ed25519 -b 521 -f ~/keys/id_ed25519
```

Get Electrum.

```bash
$ sudo pacman -Sy --needed python2-ecdsa --noconfirm
$ for _pkg in python2-pbkdf2 python2-slowaes; do
    mkdir -p ~/.src && cd ~/.src
    rm -rf $_pkg ${_pkg}.tar.gz && mkdir -p $_pkg
    curl -k -O https://aur.archlinux.org/packages/${_pkg:0:2}/$_pkg/$_pkg.tar.gz
    tar -xvzf ${_pkg}.tar.gz --strip 1 -C $_pkg
    cd $_pkg
    makepkg -Acsi --noconfirm
    cd ..
  done
$ cd && curl -k https://codeload.github.com/spesmilo/{electrum}/{tar.gz}/{${_electrum_version}} -o "#1-#3.#2"
$ tar xvzf electrum-${_electrum_version}.tar.gz
$ cd electrum-${_electrum_version}
$ find . -type f -print0 | xargs -0 sed -i 's#/usr/bin/python#/usr/bin/python2#g'
$ find . -type f -print0 | xargs -0 sed -i 's#/usr/bin/env python#/usr/bin/env python2#g'
```

Generate an Electrum wallet.

```bash
$ ./electrum create -o
```

Pick an Electrum address for signing.

```bash
$ address_for_signing=$(./electrum listaddresses | sed -n '2p' | tr -d '[:punct:]' | awk '{print $1}')
```

```bash
$ echo ${address_for_signing} > ~/keys/id_secp256k1.pub
```

```bash
$ ./electrum signmessage ${address_for_signing} "$(cat ~/keys/id_ed25519.pub)" > ~/keys/id_ed25519.pub.sig
```

Upload `id_ed25519.pub`, `id_ed25519.pub.sig` and `id_secp256k1.pub` to
GitHub, or any other file hosting service reachable via `curl`.

Save the exact URL for each keyfile to the corresponding
variable (`_electrum_pubkey_default`, `_user_pubkey_default`,
`_user_pubkey_sig_default`).

Optional: select
----------------

Base packages include:

<table>
<tr><td><a href="http://beyondgrep.com/">ack</a></td><td>Grep replacement.</td></tr>
<tr><td><a href="http://www.ansible.com/">ansible</a></td><td>Idempotent anti-conflaguration discombobulator.</td></tr>
<tr><td><a href="http://kmkeen.com/arch-wiki-lite/">arch-wiki-lite</a></td><td>Offline <a href="https://wiki.archlinux.org/">Arch Wiki</a>, easily searched and viewable on console.</td></tr>
<tr><td><a href="http://www.avahi.org/">avahi</a></td><td>Multicast/unicast DNS-SD framework.</td></tr>
<tr><td><a href="http://www.bluez.org/">bluez</a></td><td>Bluetooth support.</td></tr>
<tr><td><a href="http://www.bluez.org/">bluez-firmware</a></td><td>Firmware for Broadcom BCM203x and STLC2300 Bluetooth chips.</td></tr>
<tr><td><a href="http://www.colordiff.org/">colordiff</a></td><td>Syntax highlighting for diff.</td></tr>
<tr><td><a href="https://github.com/oblique/create_ap">create_ap</a></td><td>Create a NATed/Bridged Software Access Point (aka WiFi).</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/cronwhip">cronwhip</a></td><td>Run missed cronjobs.</td></tr>
<tr><td><a href="http://dmr.ath.cx/net/darkhttpd/">darkhttpd</a></td><td>Small and secure static webserver.</td></tr>
<tr><td><a href="https://projects.archlinux.org/devtools">devtools</a></td><td>Tools for package maintainers.</td></tr>
<tr><td><a href="https://github.com/pbrisbin/downgrade">downgrade</a></td><td>Downgrade software to an earlier version.</td></tr>
<tr><td><a href="http://fy.chalmers.se/~appro/linux/DVD+RW">dvd+rw-tools</a></td><td>DVD burning tools.</td></tr>
<tr><td><a href="http://www.mobile-stream.com/easytether/drivers.html">easytether-rpm</a></td><td>EasyTether Internet access drivers.</td></tr>
<tr><td><a href="http://elinks.or.cz/">elinks</a></td><td>Text mode web browser.</td></tr>
<tr><td><a href="https://github.com/uams/geturl">geturl-git</a></td><td><a href="https://filepicker.io">Filepicker.io</a> CLI tool to get a public link for any file.</td></tr>
<tr><td><a href="http://git-scm.com/">git</a></td><td>Fast distributed version control system.</td></tr>
<tr><td><a href="http://www.gnupg.org/">gnupg1</a></td><td>GNU Privacy Guard.</td></tr>
<tr><td><a href="https://imgur.com/tools/">imgurbash</a></td><td><a href="https://imgur.com">Imgur.com</a> CLI uploader.</td></tr>
<tr><td><a href="http://ipw2100.sourceforge.net/">ipw2100-fw</a></td><td>Intel Centrino Drivers firmware for IPW2100.</td></tr>
<tr><td><a href="http://ipw2200.sourceforge.net/">ipw2200-fw</a></td><td>Firmware for the Intel PRO/Wireless 2200BG.</td></tr>
<tr><td><a href="http://ix.io/">ix</a></td><td>Client for the <a href="http://ix.io/">ix.io</a> pastebin.</td></tr>
<tr><td><a href="https://stedolan.github.io/jq/">jq</a></td><td>CLI JSON processor.</td></tr>
<tr><td><a href="https://keybase.io/">keybase</a></td><td>CLI tool for GPG with <a href="https://keybase.io/">keybase.io</a>.</td></tr>
<tr><td><a href="https://github.com/mafm/ledger.py">ledger.py</a></td><td>CLI double-entry accounting.</td></tr>
<tr><td><a href="http://www.code-wizards.com/projects/libfaketime/">libfaketime</a></td><td>Report arbitrary dates and times to programs.</td></tr>
<tr><td><a href="http://libusb.sourceforge.net/">libusb-compat</a></td><td>USB device support.</td></tr>
<tr><td><a href="http://lrzip.kolivas.org/">lrzip</a></td><td>Multi-threaded file compression.</td></tr>
<tr><td><a href="http://lynx.isc.org/">lynx</a></td><td>Text mode Web browser.</td></tr>
<tr><td><a href="http://ftp.gnu.org/gnu/macchanger">macchanger</a></td><td>Change your NIC's MAC address.</td></tr>
<tr><td><a href="http://mercurial.selenic.com/">mercurial</a></td><td>Scalable distributed SCM tool.</td></tr>
<tr><td><a href="https://fedorahosted.org/mlocate/">mlocate</a></td><td>Locate files easily.</td></tr>
<tr><td><a href="https://mosh.mit.edu/">mosh</a></td><td>Mobile shell, survives disconnects.</td></tr>
<tr><td><a href="http://www.gnu.org/software/ncurses/">ncurses</a></td><td>Text mode user interface support.</td></tr>
<tr><td><a href="http://openvpn.net/index.php/open-source.html">openvpn</a></td><td>An easy-to-use, robust, and highly configurable VPN (Virtual Private Network).</td></tr>
<tr><td><a href="http://p7zip.sourceforge.net/">p7zip</a></td><td>7zip file archiver.</td></tr>
<tr><td><a href="http://kmkeen.com/pacmatic/">pacmatic</a></td><td>Pacman with less surprises.</td></tr>
<tr><td><a href="https://github.com/EtiennePerot/parcimonie.sh">parcimonie-sh-git</a></td><td>Safely refresh your GnuPG keyring.</td></tr>
<tr><td><a href="https://metacpan.org/pod/distribution/Image-ExifTool/exiftool">perl-image-exiftool</a></td><td>Alter EXIF data easily.</td></tr>
<tr><td><a href="https://bbs.archlinux.org/viewtopic.php?pid=841774">pkgcacheclean</a></td><td>Clean the Pacman cache to save on disk space.</td></tr>
<tr><td><a href="https://github.com/rofl0r/proxychains">proxychains-ng</a></td><td>Run programs from behind a proxy server.</td></tr>
<tr><td><a href="http://nsd.dyndns.org/pwsafe/">pwsafe</a></td><td>CLI password manager.</td></tr>
<tr><td><a href="https://github.com/stef/pbp">python2-pbp</a></td><td>Simple crypto tool.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/quixand">quixand</a></td><td>Create single-use unrecoverable encrypted sandboxes.</td></tr>
<tr><td><a href="https://github.com/nelhage/reptyr">reptyr</a></td><td>Utility for taking an existing running program and attaching it to a new terminal.</td></tr>
<tr><td><a href="http://wireless.kernel.org/en/users/Documentation/rfkill">rfkill</a></td><td>Tool for enabling and disabling wireless devices.</td></tr>
<tr><td><a href="http://libtorrent.rakshasa.no/">rtorrent</a></td><td>Rakshasa's BitTorrent client.</td></tr>
<tr><td><a href="http://stahlforce.com/dev/?tool=sfk">sfk</a></td><td>Swiss File Knife.</td></tr>
<tr><td><a href="http://www.dest-unreach.org/socat/">socat</a></td><td>Multipurpose relay.</td></tr>
<tr><td><a href="https://www.tarsnap.com/spiped.html">spiped</a></td><td>Secure pipe daemon.</td></tr>
<tr><td><a href="https://github.com/apenwarr/sshuttle">sshuttle</a></td><td>Poor man's VPN.</td></tr>
<tr><td><a href="http://point-at-infinity.org/ssss/">ssss</a></td><td>CLI tool for Shamir's Secret Sharing Scheme.</td></tr>
<tr><td><a href="http://steghide.sourceforge.net/">steghide</a></td><td>Embed a message in a file.</td></tr>
<tr><td><a href="http://blog.rusty.io/2010/01/24/submodules-and-subrepos-done-right/">subrepo</a></td><td>Git & Mercurial subrepos done right.</td></tr>
<tr><td><a href="https://github.com/bwalex/tc-play">tcplay</a></td><td>CLI TrueCrypt implementation.</td></tr>
<tr><td><a href="https://github.com/ggreer/the_silver_searcher">the_silver_searcher</a></td><td>Code searching tool.</td></tr>
<tr><td><a href="https://www.torproject.org/">tor</a></td><td>Anonymizing overlay network.</td></tr>
<tr><td><a href="https://code.google.com/p/torsocks">torsocks</a></td><td>Safely torify applications.</td></tr>
<tr><td><a href="http://mama.indstate.edu/users/ice/tree/">tree</a></td><td>Show the contents of a directory in a depth-indented list of files.</td></tr>
<tr><td><a href="https://en.wikipedia.org/wiki/Monaco_(typeface)">ttf-monaco</a></td><td>Monospaced sans-serif typeface.</td></tr>
<tr><td><a href="http://www.draisberghof.de/usb_modeswitch/">usb_modeswitch</a></td><td>Activate switchable USB devices.</td></tr>
<tr><td><a href="http://marcansoft.com/blog/iphonelinux/usbmuxd/">usbmuxd</a></td><td>USB Multiplex Daemon.</td></tr>
<tr><td><a href="https://rg3.github.io/youtube-dl/">youtube-dl</a></td><td><a href="https://www.youtube.com/">YouTube.com</a> CLI video downloader.</td></tr>
</table>

Additional packages are available.

<table>
<tr><td>Analysis</td><td>Packages for system analysis</td></tr>
<tr><td>Clojure</td><td>Packages for programming in Clojure</td></tr>
<tr><td>Elixir</td><td>Packages for programming in Elixir</td></tr>
<tr><td>Fonts</td><td>Fonts with internationalization</td></tr>
<tr><td>Go</td><td>Packages for programming in Go</td></tr>
<tr><td>Gobi</td><td>Gobi 3G / mobile network connection firmware</td></tr>
<tr><td>Haskell</td><td>Packages for programming in Haskell</td></tr>
<tr><td>Lua</td><td>Packages for programming in Lua</td></tr>
<tr><td>Markdown</td><td>Packages for writing in Markdown</td></tr>
<tr><td>Nimrod</td><td>Packages for programming in Nimrod</td></tr>
<tr><td>Node</td><td>Packages for programming in Node.js</td></tr>
<tr><td>Perl</td><td>Packages for programming in Perl</td></tr>
<tr><td>Python</td><td>Packages for programming in Python</td></tr>
<tr><td>Ruby</td><td>Packages for programming in Ruby</td></tr>
</table>

Selecting all packages (GUI included) adds over 1.0 GB to the download
and over 5.5 GB to the system image size. This is something to be aware
of if installing on a slow Internet connection or to a small USB stick.
An 8GB USB stick should work perfectly well, but free space would be
very limited with all GUI packages included.


Optional: dotfiles
------------------

Set `_dotfiles` to 1 for:

![openbox](https://i.imgur.com/zDzVno0.png)

#### Openbox Settings

**Keybindings**

*The Windows symbol key is the 'Super' key. For example,
`Windows+spacebar` opens the main menu*

Program shortcuts:

- `Super+space`: main menu
- `Super+t`    : LXTerminal (T)erminal
- `Super+f`    : PCManFM (F)ile manager
- `Super+e`    : Leafpad (E)ditor
- `Super+m`    : VLC (M)edia player
- `Super+w`    : Chromium (W)eb browser
- `Super+q`    : Force (Q)uit
- `Super+g`    : (g)Vim
- `Super+l`    : (L)ock screen
- `Super+r`    : Calculato(R)
- `Alt+F2`     : gmrun dialog (program launcher)
- `Alt+F3`     : dmenu (HUD-like program launcher with simple fuzzy completion)

Desktop shortcuts:

- `Super+a` : toggle maximize the current window
- `Super+h` : toggle fully expand the current window horizontally
- `Super+v` : toggle fully expand the current window vertically
- `Super+c` : center the current window
- `Super+<UP>`   : move the current window up
- `Super+<DOWN>` : move the current window down
- `Super+<RIGHT>`: move the current window to the right
- `Super+<LEFT>` : move the current window to the left
- `Alt+Super+<ARROW>` : resize the current window
- `Alt+Shift+<ARROW>` : move the current window to a different desktop
- `Super+<F1-F4>` : switch to desktop #1-4

#### How It Works

- Log in
- Run `startx` to launch Openbox


Notes
-----

- If you are doing a persistent USB install, it's recommended to boot
  the Arch LiveCD/USB in i686 mode to ensure maximum hardware
  compatibilty.
- If `wifi-menu -o` throws an `invalid interface` error, reboot the
  LiveCD/USB.


To Do
-----

- Add man page
- Handle cmdline flag-related error messages gracefully
- Add keyfile-encrypted swap partition
- Add `_raid` variable to setup [Btrfs RAID](https://wiki.archlinux.org/index.php/Btrfs#Multi-device_filesystem_and_RAID_feature)


Licensing
---------

This is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE file.
