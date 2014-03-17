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
3. Download pacstrapit: `curl -k https://codeload.github.com/atweiden/{pacstrapit}/{tar.gz}/{0.1.1} -o "#1-#3.#2"`
4. Extract: `tar xvzf pacstrapit-0.1.1.tar.gz`
5. **Customize variables**

WARNING: failure to give appropriate values could cause catastrophic
data loss and system instability.

Defaults:

<table>
<tr><td>Target partition</td><td>/dev/sda</td><tr>
<tr><td>Type of processor</td><td>other</td><tr>
<tr><td>Type of graphics card</td><td>Intel</td><tr>
<tr><td>Type of hard drive</td><td>HDD</td><tr>
<tr><td>Locale</td><td>en_US</td><tr>
<tr><td>Keymap</td><td>us</td><tr>
<tr><td>Timezone</td><td>America/Los_Angeles (PST)</td><tr>
<tr><td>Root password</td><td>secret</td><tr>
<tr><td>User name</td><td>live</td><tr>
<tr><td>User password</td><td>secret</td><tr>
<tr><td>LUKS name</td><td>luksroot</td><tr>
<tr><td>LUKS password</td><td>secret</td><tr>
<tr><td>Hostname</td><td>luksiso</td><tr>
</table>

> `cd pacstrapit-0.1.1 && $EDITOR pacstrapit`

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
$ for _pkg in python2-ecdsa python2-pbkdf2 python2-slowaes; do
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


To Do
-----

- create 0.1.0 release of dotfiles
- package electrum-headless electrum-lite-headless python2-scrypt
- replace manual electrum headless with electrum-headless pkg installation
- vim re:center debug boxes
- organize pkg sections
- command line flags (shflags)
- Add keyfile-encrypted swap partition
- Add `_raid` variable to setup [Btrfs RAID](https://wiki.archlinux.org/index.php/Btrfs#Multi-device_filesystem_and_RAID_feature)


Licensing
---------

This is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE file.
