PacstrapIt!
===========

Features
--------

- Btrfs on LUKS


Instructions
------------

1. Burn LiveCD/LiveUSB with latest [Arch ISO](https://www.archlinux.org/download/)
2. Boot from LiveCD/LiveUSB
3. Download pacstrapit: `curl -k https://codeload.github.com/atweiden/{pacstrapit}/{tar.gz}/{0.0.19} -o "#1-#3.#2"`
4. Extract: `tar xvzf pacstrapit-0.0.19.tar.gz`
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

> `cd pacstrapit-0.0.19 && $EDITOR pacstrapit`

Done. Ready to run `pacstrapit`:

> `./pacstrapit`

With logging:

> `./pacstrapit 2>&1 | tee pacstrapit.log`

If the script exits with an error, it's best to reboot and start fresh.


Optional: sshify
----------------

Before setting `_ssh` to `1`...

**On the control machine**:

`mkdir -p ~/keys`

Generate ssh keys.

```
$ ssh-keygen -t ed25519 -b 521 -f ~/keys/id_ed25519
```

Obtain Electrum.

```
cd && curl -k https://codeload.github.com/spesmilo/{electrum}/{tar.gz}/{${_electrum_version}} -o "#1-#3.#2"
tar xvzf electrum-${_electrum_version}.tar.gz
cd electrum-${_electrum_version}
find . -type f -print0 | xargs -0 sed -i 's#/usr/bin/python#/usr/bin/python2#g'
find . -type f -print0 | xargs -0 sed -i 's#/usr/bin/env python#/usr/bin/env python2#g'
```

Pick an Electrum address for signing.

```
$ address_for_signing=$(./electrum listaddresses | sed -n '2p' | tr -d '[:punct:]' | awk '{print $1}')
```

```
echo ${address_for_signing} > ~/keys/electrum.pub
```

```
$ ./electrum signmessage ${address_for_signing} "$(cat ~/keys/id_ed25519.pub)" > ~/keys/id_ed25519.pub.sig
```

Upload `id_ed25519.pub`, `id_ed25519.pub.sig` and `electrum.pub` to
GitHub, or any other file hosting service reachable via `curl`.

Record the exact URL for each keyfile inside the variables section of
`pacstrapit`.


Licensing
---------

This is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE file.
