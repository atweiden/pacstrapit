PacstrapIt!
===========

Features
--------

- Btrfs on LUKS


Instructions
------------

1. Burn LiveCD/LiveUSB with latest [Arch ISO](https://www.archlinux.org/download/)
2. Boot from LiveCD/LiveUSB
3. Download pacstrapit: `curl -k https://codeload.github.com/atweiden/{pacstrapit}/{tar.gz}/{0.0.14} -o "#1-#3.#2"`
4. Extract: `tar xvzf pacstrapit-0.0.14.tar.gz`
5. **Customize variables**

WARNING: failure to give appropriate values could cause catastrophic
data loss and system instability.

Defaults:

<table>
<tr><td>Target partition</td><td>/dev/sda</td><tr>
<tr><td>Type of processor</td><td>Intel</td><tr>
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

> `cd pacstrapit-0.0.14 && $EDITOR pacstrapit`

Done. Ready to run `pacstrapit`:

> `./pacstrapit`

With logging:

> `./pacstrapit 2>&1 | tee pacstrapit.log`

If the script exits with an error, it's best to reboot and start fresh.


To Do
-----

- `cp $srcdir/sudoers /mnt/etc/sudoers`
- tmpfs for vim, firefox, chromium
- anything-sync-daemon
- profile-sync-daemon

Licensing
---------

This is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE file.
