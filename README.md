PacstrapIt!
===========

Features
--------

- Btrfs on LUKS


Requirements
------------

You:

1. have no need for dual-booting Windows
2. are comfortable working on the command line.
3. are starting from a brand new or completely blank hard drive (`shred -fvz -n 7 /dev/sdX`)


Instructions
------------

1. Burn LiveCD/LiveUSB with latest [Arch ISO](https://www.archlinux.org/download/)
2. Boot from LiveCD/LiveUSB
3. Download pacstrapit: `curl -k https://codeload.github.com/atweiden/{pacstrapit}/{tar.gz}/{0.3.4} -o "#1-#3.#2"`
4. Extract: `tar xvzf pacstrapit-0.3.4.tar.gz`
5. **Customize variables**

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
<tr><td>User name</td><td>live</td></tr>
<tr><td>User password</td><td>secret</td></tr>
<tr><td>LUKS name</td><td>luksroot</td></tr>
<tr><td>LUKS password</td><td>secret</td></tr>
<tr><td>Hostname</td><td>luksiso</td></tr>
</table>

> `cd pacstrapit-0.3.4 && $EDITOR pacstrapit`

Done.

To run `pacstrapit`:

> `./pacstrapit`

To run `pacstrapit` with logging:

> `./pacstrapit 2>&1 | tee pacstrapit.log`

If the script exits with an error, it's best to reboot and start fresh
with a blank disk (`shred -fvz -n 0 /dev/sdX`).


Optional: sshify
----------------

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

Record the exact URL for each keyfile inside the variables section of
`pacstrapit`.

Optional: select
----------------

Select packages include:

<table>
<tr><td>ack</td><td>grep replacement.</td></tr>
<tr><td>ansible</td><td>Radically simple IT automation platform.</td></tr>
<tr><td>archey</td><td>Display the Arch Linux logo and basic system info.</td></tr>
<tr><td>arch-wiki-markdown-git</td><td>Offline Arch Wiki in Markdown.</td></tr>
<tr><td>avahi</td><td>Multicast/unicast DNS-SD framework.</td></tr>
<tr><td>bluez</td><td>Bluetooth support.</td></tr>
<tr><td>bluez-firmware</td><td>Firmware for Broadcom BCM203x and STLC2300 Bluetooth chips.</td></tr>
<tr><td>colordiff</td><td>Syntax highlighting for diff.</td></tr>
<tr><td>create_ap</td><td>Create a NATed/Bridged Software Access Point (aka WiFi).</td></tr>
<tr><td>cronwhip</td><td>Run missed cronjobs.</td></tr>
<tr><td>darkhttpd</td><td>Small and secure static webserver.</td></tr>
<tr><td>devtools</td><td>Tools for Arch Linux package maintainers.</td></tr>
<tr><td>downgrade</td><td>Downgrade software to an earlier version.</td></tr>
<tr><td>dvd+rw-tools</td><td>DVD burning tools.</td></tr>
<tr><td>easytether-rpm</td><td>EasyTether Internet access drivers.</td></tr>
<tr><td>elinks</td><td>Text mode web browser.</td></tr>
<tr><td>geturl-git</td><td>Filepicker.io CLI tool to get a public link for any file.</td></tr>
<tr><td>git</td><td>Fast distributed version control system.</td></tr>
<tr><td>gnupg1</td><td>GNU Privacy Guard.</td></tr>
<tr><td>haveged</td><td>Entropy harvesting daemon.</td></tr>
<tr><td>imgurbash</td><td>Imgur.com CLI uploader.</td></tr>
<tr><td>ipw2100-fw</td><td>Intel Centrino Drivers firmware for IPW2100.</td></tr>
<tr><td>ipw2200-fw</td><td>Firmware for the Intel PRO/Wireless 2200BG.</td></tr>
<tr><td>ix</td><td>Client for the ix.io pastebin.</td></tr>
<tr><td>jq</td><td>Command-line JSON processor.</td></tr>
<tr><td>libfaketime</td><td>Report arbitrary dates and times to programs.</td></tr>
<tr><td>libusb-compat</td><td>USB device support.</td></tr>
<tr><td>lrzip</td><td>Multi-threaded file compression.</td></tr>
<tr><td>lynx</td><td>Text mode Web browser.</td></tr>
<tr><td>macchanger</td><td>Change your NIC's MAC address.</td></tr>
<tr><td>mercurial</td><td>Scalable distributed SCM tool.</td></tr>
<tr><td>mlocate</td><td>Locate files easily.</td></tr>
<tr><td>mosh</td><td>Mobile shell, survives disconnects.</td></tr>
<tr><td>ncurses</td><td>Text mode user interface support.</td></tr>
<tr><td>openvpn</td><td>An easy-to-use, robust, and highly configurable VPN (Virtual Private Network).</td></tr>
<tr><td>p7zip</td><td>7zip file archiver.</td></tr>
<tr><td>pacmatic</td><td>Pacman with less surprises.</td></tr>
<tr><td>perl-image-exiftool</td><td>Alter EXIF data easily.</td></tr>
<tr><td>pkgcacheclean</td><td>Clean the Pacman cache to save on disk space.</td></tr>
<tr><td>proxychains-ng</td><td>Run programs from behind a proxy server.</td></tr>
<tr><td>pwsafe</td><td>CLI password manager.</td></tr>
<tr><td>python2-pbp</td><td>Simple crypto tool.</td></tr>
<tr><td>quixand</td><td>Create single-use unrecoverable encrypted sandboxes.</td></tr>
<tr><td>reptyr</td><td>Utility for taking an existing running program and attaching it to a new terminal.</td></tr>
<tr><td>rfkill</td><td>Tool for enabling and disabling wireless devices.</td></tr>
<tr><td>rtorrent</td><td>Rakshasa's BitTorrent client.</td></tr>
<tr><td>sfk</td><td>Swiss File Knife.</td></tr>
<tr><td>socat</td><td>Multipurpose relay.</td></tr>
<tr><td>sshuttle</td><td>Poor man's VPN.</td></tr>
<tr><td>ssss</td><td>CLI tool for Shamir's Secret Sharing Scheme.</td></tr>
<tr><td>steghide</td><td>Embed a message in a file.</td></tr>
<tr><td>subrepo</td><td>Git & Mercurial subrepos done right.</td></tr>
<tr><td>tcplay</td><td>CLI TrueCrypt.</td></tr>
<tr><td>the_silver_searcher</td><td>Code searching tool.</td></tr>
<tr><td>tor</td><td>Anonymizing overlay network.</td></tr>
<tr><td>torsocks</td><td>Safely torify applications.</td></tr>
<tr><td>tree</td><td>Show the contents of a directory in a depth-indented list of files.</td></tr>
<tr><td>ttf-monaco</td><td>Monospaced sans-serif typeface.</td></tr>
<tr><td>usb_modeswitch</td><td>Activate switchable USB devices.</td></tr>
<tr><td>usbmuxd</td><td>USB Multiplex Daemon.</td></tr>
<tr><td>youtube-dl</td><td>YouTube.com CLI video downloader.</td></tr>
</table>

Additional packages are available.

<table>
<tr><td>Analysis</td><td>Sysadmin tools for system analysis</td></tr>
<tr><td>Clojure</td><td>Packages for programming in Clojure</td></tr>
<tr><td>Elixir</td><td>Packages for programming in Elixir</td></tr>
<tr><td>Fonts</td><td>Fonts with internationalization</td></tr>
<tr><td>Games</td><td>minecraft, papersplea.se, radegast, steam</td></tr>
<tr><td>Go</td><td>Packages for programming in Go</td></tr>
<tr><td>Gobi</td><td>Gobi 3G / mobile network connection firmware</td></tr>
<tr><td>Haskell</td><td>Packages for programming in Haskell</td></tr>
<tr><td>Ledger</td><td>CLI accounting</td></tr>
<tr><td>Lua</td><td>Packages for programming in Lua</td></tr>
<tr><td>Markdown</td><td>Packages for writing in Markdown</td></tr>
<tr><td>Nginx</td><td>Nginx webserver packages</td></tr>
<tr><td>Nimrod</td><td>Packages for programming in Nimrod</td></tr>
<tr><td>Node</td><td>Packages for programming in Node.js</td></tr>
<tr><td>Perl</td><td>Packages for programming in Perl</td></tr>
<tr><td>Python</td><td>Packages for programming in Python</td></tr>
<tr><td>Ruby</td><td>Packages for programming in Ruby</td></tr>
</table>

Selecting all packages (GUI included) adds over 1.0 GB to the download
and over 4.0 GB to the system image size. This is something to be aware
of if installing on a slow Internet connection or to a small USB stick.


To Do
-----

- Add keyfile-encrypted swap partition
- Add `_raid` variable to setup [Btrfs RAID](https://wiki.archlinux.org/index.php/Btrfs#Multi-device_filesystem_and_RAID_feature)
- Add command line flags (shflags)
- Add [menu](https://github.com/jamielinux/bashmount)


Licensing
---------

This is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE file.
