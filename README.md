PacstrapIt!
===========

Features
--------

- Btrfs on LUKS

Instructions
------------

1. Burn CD with latest [Arch ISO](https://www.archlinux.org/download/)
2. Insert CD into drive
3. Boot from Arch ISO
4. Download pacstrapit

> `curl -k https://codeload.github.com/atweiden/{pacstrapit}/{tar.gz}/{0.0.1} -o "#1-#3.#2"`

5. Extract

> `tar xvzf pacstrapit-0.0.1.tar.gz`

6. **Customize variables**

*Warning: failure to give appropriate values could cause catastrophic
data loss and boot failures.*

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

> `cd pacstrapit-0.0.1 && $EDITOR pacstrapit`

7. Run pacstrapit

`./pacstrapit`

To Do
-----

- tmpfs for vim, firefox, chromium
- anything-sync-daemon
- profile-sync-daemon

Licensing
---------

This is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE file.
