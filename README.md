btrfs on luks
=============

<table>
<tr><td>Last tested</td><td>2014-11-27 with archlinux-2014.11.01-dual.iso</td></tr>
</table>

`pacstrapit` is meant for provisioning physical hardware. It assumes
you have no need for dual-booting Windows, and that you are comfortable
working on the command line.


Instructions
------------

1. Burn LiveCD/LiveUSB with latest [Arch ISO](https://www.archlinux.org/download/)
2. Boot from LiveCD/LiveUSB
3. Connect to the Internet: `wifi-menu -o`
4. Download pacstrapit: `curl -k https://codeload.github.com/atweiden/{pacstrapit}/{tar.gz}/{0.10.0} -o "#1-#3.#2"`
5. Extract: `tar xvzf pacstrapit-0.10.0.tar.gz`
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
<tr><td>Hosts allowed</td><td>192.168.0. (LAN - SSH is disabled by default)</td></tr>
</table>

> `cd pacstrapit-0.10.0 && $EDITOR pacstrapit`

Done.


Usage
-----

Run `pacstrapit` for a capable headless persistent USB installation
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
./pacstrapit start --select    "base gui python nim"      \
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
> The `base` bundle adds CLI, vim, grsec and bitcoin pkgs
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
$ sudo pacman -S electrum
```

Generate an Electrum wallet.

```bash
$ electrum create -o
```

Pick an Electrum address for signing.

```bash
$ address_for_signing=$(electrum listaddresses | sed -n '2p' | tr -d '[:punct:]' | awk '{print $1}')
```

```bash
$ echo ${address_for_signing} > ~/keys/id_secp256k1.pub
```

```bash
$ electrum signmessage ${address_for_signing} "$(cat ~/keys/id_ed25519.pub)" > ~/keys/id_ed25519.pub.sig
```

Upload `id_ed25519.pub`, `id_ed25519.pub.sig` and `id_secp256k1.pub` to
GitHub, or any other file hosting service reachable via `curl`.

Save the exact URL for each keyfile to the corresponding
variable (`_electrum_pubkey_default`, `_user_pubkey_default`,
`_user_pubkey_sig_default`).


Default packages
----------------

Minimum packages installed (excluding dependencies, `base` and
`base-devel` groups):

<table>
<tr><td><a href="https://projects.archlinux.org/abs.git">abs</a></td><td>Utilities to download and work with the Arch Build System (ABS).</td></tr>
<tr><td><a href="https://projects.archlinux.org/arch-install-scripts.git">arch-install-scripts</a></td><td>Scripts to aid in installing Arch.</td></tr>
<tr><td><a href="https://github.com/vianney/arch-luks-suspend">arch-luks-suspend</a></td><td>Lock encrypted root volume on suspend.</td></tr>
<tr><td><a href="https://bash-completion.alioth.debian.org/">bash-completion</a></td><td>Programmable completion for the bash shell.</td></tr>
<tr><td><a href="https://btrfs.wiki.kernel.org/index.php/Main_Page">btrfs-progs</a></td><td>Btrfs filesystem utilities.</td></tr>
<tr><td><a href="https://pkgs.fedoraproject.org/cgit/ca-certificates.git">ca-certificates</a></td><td>Common CA certificates.</td></tr>
<tr><td><a href="https://fedorahosted.org/cronie/">cronie</a></td><td>Daemon that runs specified programs at scheduled times and related tools.</td></tr>
<tr><td><a href="https://www.isc.org/software/dhcp">dhclient</a></td><td>Standalone DHCP client.</td></tr>
<tr><td><a href="http://invisible-island.net/dialog/">dialog</a></td><td>Tool to display dialog boxes from shell scripts.</td></tr>
<tr><td><a href="https://github.com/jedisct1/dnscrypt-proxy">dnscrypt-proxy</a></td><td>Tool for securing communications between a client and a DNS resolver.</td></tr>
<tr><td><a href="https://www.gnu.org/software/ed/ed.html">ed</a></td><td>Line-oriented text editor.</td></tr>
<tr><td><a href="https://www.kernel.org/pub/software/network/ethtool/">ethtool</a></td><td>Utility for controlling network drivers and hardware.</td></tr>
<tr><td><a href="http://expect.sourceforge.net/">expect</a></td><td>Tool for automating interactive applications.</td></tr>
<tr><td><a href="http://www.rodsbooks.com/gdisk/">gptfdisk</a></td><td>Text-mode partitioning tool that works on GUID Partition Table (GPT) disks.</td></tr>
<tr><td><a href="https://www.gnu.org/software/grub/">grub-bios</a></td><td>GNU GRand Unified Bootloader.</td></tr>
<tr><td><a href="http://www.issihosts.com/haveged">haveged</a></td><td>Entropy harvesting daemon using CPU timings.</td></tr>
<tr><td><a href="https://downloadcenter.intel.com/SearchResult.aspx?lang=eng&keyword=%22microcode%22">intel-ucode</a></td><td>Microcode update files <a href="https://www.archlinux.org/news/changes-to-intel-microcodeupdates/">for Intel CPUs</a>.</td></tr>
<tr><td><a href="http://www.linuxfoundation.org/collaborate/workgroups/networking/iproute2">iproute2</a></td><td>IP Routing Utilities.</td></tr>
<tr><td><a href="https://www.netfilter.org/projects/iptables/">iptables</a></td><td>Linux kernel packet control tool.</td></tr>
<tr><td><a href="http://wireless.kernel.org/en/users/Documentation/iw">iw</a></td><td>nl80211 based CLI configuration utility for wireless devices.</td></tr>
<tr><td><a href="http://www.kbd-project.org/">kbd</a></td><td>Keytable files and keyboard utilities.</td></tr>
<tr><td><a href="https://kernel.org/pub/linux/utils/kernel/kexec/">kexec-tools</a></td><td>Load another kernel from the currently executing Linux kernel.</td></tr>
<tr><td><a href="http://net-tools.sourceforge.net/">net-tools</a></td><td>Configuration tools for Linux networking.</td></tr>
<tr><td><a href="http://roy.marples.name/projects/openresolv/index">openresolv</a></td><td>resolv.conf management framework (resolvconf).</td></tr>
<tr><td><a href="http://www.openssh.com/portable.html">openssh</a></td><td>SSH connectivity tools.</td></tr>
<tr><td><a href="https://github.com/rmarquis/pacaur">pacaur</a></td><td>Fast workflow AUR helper.</td></tr>
<tr><td><a href="https://www.python.org/">python2</a></td><td>High-level scripting language v2.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/reflector/">reflector</a></td><td>Retrieve and filter the latest Pacman mirror list.</td></tr>
<tr><td><a href="https://rsync.samba.org/">rsync</a></td><td>A file transfer program to keep remote files in sync.</td></tr>
<tr><td><a href="http://sourceforge.net/projects/sshpass/">sshpass</a></td><td>Fool ssh into accepting an interactive password non-interactively.</td></tr>
<tr><td><a href="https://github.com/Nefelim4ag/systemd-swap">systemd-swap</a></td><td>Create hybrid swap space from zram swaps, swap files and swap partitions.</td></tr>
<tr><td><a href="http://tmux.sourceforge.net/">tmux</a></td><td>Terminal multiplexer.</td></tr>
<tr><td><a href="https://www.info-zip.org/UnZip.html">unzip</a></td><td>Extract and view files in zip archives.</td></tr>
<tr><td><a href="https://www.gnu.org/software/wget/wget.html">wget</a></td><td>Retrieve files from the Web.</td></tr>
<tr><td><a href="http://www.hpl.hp.com/personal/Jean_Tourrilhes/Linux/Tools.html">wireless_tools</a></td><td>Wireless tools.</td></tr>
<tr><td><a href="https://projects.archlinux.org/wpa_actiond.git/">wpa_actiond</a></td><td>Daemon that connects to wpa_supplicant and handles connect and disconnect events.</td></tr>
<tr><td><a href="http://w1.fi/wpa_supplicant/">wpa_supplicant</a></td><td>Utility providing key negotiation for WPA wireless networks.</td></tr>
<tr><td><a href="https://www.info-zip.org/Zip.html">zip</a></td><td>Create and modify zipfiles.</td></tr>
<tr><td><a href="http://www.zsh.org/">zsh</a></td><td>Very advanced and programmable shell.</td></tr>
</table>


Optional packages
-----------------

Base packages (use `--bundle "base"` to include):

<table>
<tr><td><a href="http://beyondgrep.com/">ack</a></td><td>Grep replacement.</td></tr>
<tr><td><a href="https://github.com/ansible/ansible">ansible</a></td><td>Radically simple IT automation platform.</td></tr>
<tr><td><a href="https://github.com/brain0/appjail">appjail</a></td><td>Sandboxing tool to protect private data from untrusted applications.</td></tr>
<tr><td><a href="https://github.com/seblu/archversion">archversion</a></td><td>Archlinux Version Controller.</td></tr>
<tr><td><a href="http://kmkeen.com/arch-wiki-lite/">arch-wiki-lite</a></td><td>Offline <a href="https://wiki.archlinux.org/">Arch Wiki</a>, easily searched and viewable on console.</td></tr>
<tr><td><a href="https://github.com/vstakhov/asignify">asignify</a></td><td>Yet another signify tool.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/autochown">autochown</a></td><td>Monitor multiple directories using glob patterns and automatically adjust file ownership and permissions.</td></tr>
<tr><td><a href="http://www.avahi.org/">avahi</a></td><td>Multicast/unicast DNS-SD framework.</td></tr>
<tr><td><a href="http://www.isc.org/software/bind/">bind-tools</a></td><td>The ISC DNS tools.</td></tr>
<tr><td><a href="http://www.bluez.org/">bluez</a></td><td>Bluetooth support.</td></tr>
<tr><td><a href="http://www.bluez.org/">bluez-firmware</a></td><td>Firmware for Broadcom BCM203x and STLC2300 Bluetooth chips.</td></tr>
<tr><td><a href="http://ccrypt.sourceforge.net/">ccrypt</a></td><td>Command-line utility for encrypting and decrypting files and streams.</td></tr>
<tr><td><a href="https://bbs.archlinux.org/viewtopic.php?pid=1490719">check-pacman-mtree</a></td><td>Locate files that have changed on disk (size/md5/sha256).</td></tr>
<tr><td><a href="http://www.colordiff.org/">colordiff</a></td><td>Syntax highlighting for diff.</td></tr>
<tr><td><a href="https://github.com/shyiko/commacd">commacd</a></td><td>A faster way to move around Bash.</td></tr>
<tr><td><a href="http://www.agroman.net/corkscrew/">corkscrew</a></td><td>A tool for tunneling SSH through HTTP proxies.</td></tr>
<tr><td><a href="https://github.com/oblique/create_ap">create_ap</a></td><td>Create a NATed/Bridged Software Access Point (aka WiFi).</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/cronwhip">cronwhip</a></td><td>Run missed cronjobs.</td></tr>
<tr><td><a href="https://www.cryfs.org/">cryfs</a></td><td> Cryptographic filesystem for the cloud.</td></tr>
<tr><td><a href="https://unix4lyfe.org/darkhttpd/">darkhttpd</a></td><td>Small and secure static webserver.</td></tr>
<tr><td><a href="https://projects.archlinux.org/devtools">devtools</a></td><td>Tools for package maintainers.</td></tr>
<tr><td><a href="https://github.com/pbrisbin/downgrade">downgrade</a></td><td>Downgrade software to an earlier version.</td></tr>
<tr><td><a href="https://github.com/joowani/dtags">dtags</a></td><td>Directory tags for lazy programmers.</td></tr>
<tr><td><a href="http://fy.chalmers.se/~appro/linux/DVD+RW">dvd+rw-tools</a></td><td>DVD burning tools.</td></tr>
<tr><td><a href="http://www.mobile-stream.com/easytether/drivers.html">easytether</a></td><td><a href="http://easytether.org/">EasyTether</a> Internet access drivers.</td></tr>
<tr><td><a href="http://elinks.or.cz/">elinks</a></td><td>Text mode web browser.</td></tr>
<tr><td><a href="http://entrproject.org/">entr</a></td><td>Run arbitrary commands when files change.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/facadefs">facadefs</a></td><td>FUSE-based filesystem sandbox.</td></tr>
<tr><td><a href="https://github.com/Keruspe/facron">facron</a></td><td>fanotify cron system.</td></tr>
<tr><td><a href="https://github.com/drmikehenry/findx">findx</a></td><td>Wrapper to extend the Unix find command.</td></tr>
<tr><td><a href="http://www.vanheusden.com/f-irc/">f-irc</a></td><td>User-friendly IRC client for the console/terminal.</td></tr>
<tr><td><a href="https://l3net.wordpress.com/projects/firejail/">firejail</a></td><td>Sandbox any type of process.</td></tr>
<tr><td><a href="https://github.com/junegunn/fzf">fzf</a></td><td>Fuzzy finder for your shell.</td></tr>
<tr><td><a href="https://github.com/atweiden/fzf-extras">fzf-extras</a></td><td>Extra keybindings for fzf.</td></tr>
<tr><td><a href="https://github.com/uams/geturl">geturl</a></td><td><a href="https://filepicker.io">Filepicker.io</a> CLI tool to get a public link for any file.</td></tr>
<tr><td><a href="https://git-scm.com/">git</a></td><td>Fast distributed version control system.</td></tr>
<tr><td><a href="https://github.com/google/git-appraise">git-appraise</a></td><td>Distributed code review system for Git repositories.</td></tr>
<tr><td><a href="https://github.com/visionmedia/git-extras">git-extras</a></td><td>Git utilities.</td></tr>
<tr><td><a href="https://git-lfs.github.com/">git-lfs</a></td><td>An open source Git extension for versioning large files.</td></tr>
<tr><td><a href="https://jorisroovers.github.io/gitlint/">gitlint</a></td><td>Git commit message linter.</td></tr>
<tr><td><a href="https://www.gnupg.org/">gnupg1</a></td><td>GNU Privacy Guard.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/hexgrep">hexgrep</a></td><td>Versatile grep-like binary stream and file search tool.</td></tr>
<tr><td><a href="https://github.com/jkbr/httpie">httpie</a></td><td>cURL for humans.</td></tr>
<tr><td><a href="https://hub.github.com/">hub</a></td><td>CLI interface for Github.</td></tr>
<tr><td><a href="https://github.com/jeffkaufman/icdiff">icdiff</a></td><td>Improved colored diff.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/idemptables">idemptables</a></td><td>Idempotent iptables wrapper for appending and deleting rules.</td></tr>
<tr><td><a href="https://imgur.com/tools/">imgurbash</a></td><td><a href="https://imgur.com">Imgur.com</a> CLI uploader.</td></tr>
<tr><td><a href="http://ipset.netfilter.org/">ipset</a></td><td>Store multiple IP addresses or port numbers and match against the collection with iptables in one swoop.</td></tr>
<tr><td><a href="http://ipw2100.sourceforge.net/">ipw2100-fw</a></td><td>Intel Centrino Drivers firmware for IPW2100.</td></tr>
<tr><td><a href="http://ipw2200.sourceforge.net/">ipw2200-fw</a></td><td>Firmware for the Intel PRO/Wireless 2200BG.</td></tr>
<tr><td><a href="http://irssi.org/">irssi</a></td><td>Modular text mode IRC client with Perl scripting.</td></tr>
<tr><td><a href="https://github.com/cryptodotis/irssi-otr">irssi-otr</a></td><td>OTR support for Irssi.</td></tr>
<tr><td><a href="https://github.com/irssi/scripts.irssi.org">irssi-script-sasl</a></td><td>Freenode SASL support for Irssi.</td></tr>
<tr><td><a href="http://ix.io/">ix</a></td><td>Client for the <a href="http://ix.io/">ix.io</a> pastebin.</td></tr>
<tr><td><a href="https://stedolan.github.io/jq/">jq</a></td><td>CLI JSON processor.</td></tr>
<tr><td><a href="https://keybase.io/">keybase</a></td><td>CLI tool for GPG with <a href="https://keybase.io/">keybase.io</a>.</td></tr>
<tr><td><a href="https://github.com/mafm/ledger.py">ledger.py</a></td><td>CLI double-entry accounting.</td></tr>
<tr><td><a href="https://github.com/wolfcw/libfaketime">libfaketime</a></td><td>Report arbitrary dates and times to programs.</td></tr>
<tr><td><a href="http://libusb.sourceforge.net/">libusb-compat</a></td><td>USB device support.</td></tr>
<tr><td><a href="http://lrzip.kolivas.org/">lrzip</a></td><td>Multi-threaded file compression.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/lsgrp">lsgrp</a></td><td>A simple command-line utility to list all members of a group.</td></tr>
<tr><td><a href="http://lynx.isc.org/">lynx</a></td><td>Text mode Web browser.</td></tr>
<tr><td><a href="https://ftp.gnu.org/gnu/macchanger/">macchanger</a></td><td>Change your NIC's MAC address.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/makedep">makedep</a></td><td>Convert Pacman optional dependencies to real dependencies.</td></tr>
<tr><td><a href="http://invisible-island.net/mawk/">mawk</a></td><td>Minimal-featured Awk designed for speed of execution over functionality.</td></tr>
<tr><td><a href="https://github.com/visit1985/mdp">mdp</a></td><td>A command-line based Markdown presentation tool.</td></tr>
<tr><td><a href="http://mercurial.selenic.com/">mercurial</a></td><td>Scalable distributed SCM tool.</td></tr>
<tr><td><a href="https://fedorahosted.org/mlocate/">mlocate</a></td><td>Locate files easily.</td></tr>
<tr><td><a href="https://www.gutenberg.org/ebooks/3202">moby-thesaurus</a></td><td>The Project Gutenberg Etext of Moby Thesaurus II by Grady Ward.</td></tr>
<tr><td><a href="https://joeyh.name/code/moreutils/">moreutils</a></td><td>A growing collection of the unix tools that nobody thought to write thirty years ago.</td></tr>
<tr><td><a href="https://mosh.mit.edu/">mosh</a></td><td>Mobile shell, survives disconnects.</td></tr>
<tr><td><a href="https://myrepos.branchable.com/">myrepos</a></td><td>Multiple Repository management tool.</td></tr>
<tr><td><a href="http://cm.bell-labs.com/who/bwk/">nawk</a></td><td>The One True Awk.</td></tr>
<tr><td><a href="https://www.gnu.org/software/ncurses/">ncurses</a></td><td>Text mode user interface support.</td></tr>
<tr><td><a href="https://github.com/shibumi/nyan">nyan</a></td><td>Simple netcat wrapper.</td></tr>
<tr><td><a href="https://openvpn.net/index.php/open-source.html">openvpn</a></td><td>An easy-to-use, robust, and highly configurable VPN.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/ottools">ottools</a></td><td>Tools for encrypting files with one-time pads.</td></tr>
<tr><td><a href="http://p7zip.sourceforge.net/">p7zip</a></td><td>7zip file archiver.</td></tr>
<tr><td><a href="http://jjacky.com/pacdep">pacdep</a></td><td>List package dependencies.</td></tr>
<tr><td><a href="https://github.com/archlinuxfr/package-query">package-query</a></td><td>Query ALPM and AUR.</td></tr>
<tr><td><a href="https://github.com/keenerd/packer">packer</a></td><td>Bash wrapper for Pacman and AUR.</td></tr>
<tr><td><a href="https://github.com/protist/paclog">paclog</a></td><td>List recent commits for Arch Linux packages.</td></tr>
<tr><td><a href="http://kmkeen.com/pacmatic/">pacmatic</a></td><td>Pacman with less surprises.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/pacnew_scripts">pacnew_scripts</a></td><td>A collection of scripts to help merge changes in .pacnew files.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/pacserve">pacserve</a></td><td>Easily share Pacman packages between computers.</td></tr>
<tr><td><a href="https://github.com/crossroads1112/bin/tree/master/pacupg">pacupg</a></td><td>Script that wraps package and AUR upgrades in btrfs snapshots as well as providing an easy way to roll them back.</td></tr>
<tr><td><a href="https://github.com/cheusov/paexec">paexec</a></td><td>Parallel executor.</td></tr>
<tr><td><a href="https://www.gnu.org/software/parallel/">parallel</a></td><td>Execute jobs in parallel.</td></tr>
<tr><td><a href="https://github.com/EtiennePerot/parcimonie.sh">parcimonie-sh</a></td><td>Safely refresh your GnuPG keyring.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/pbget">pbget</a></td><td>Retrieve PKGBUILDs and local source files from Git, ABS and the AUR for makepkg.</td></tr>
<tr><td><a href="https://pdfgrep.org/">pdfgrep</a></td><td>A tool to search text in PDF files.</td></tr>
<tr><td><a href="https://metacpan.org/pod/distribution/Image-ExifTool/exiftool">perl-image-exiftool</a></td><td>Alter EXIF data easily.</td></tr>
<tr><td><a href="https://github.com/junegunn/pipe-logger">pipe-logger</a></td><td>Log rotation of stdout & stderr.</td></tr>
<tr><td><a href="https://github.com/flonatel/pipexec">pipexec</a></td><td>Handle pipe of commands like a single command.</td></tr>
<tr><td><a href="https://github.com/falconindy/pkgbuild-introspection">pkgbuild-introspection</a></td><td>Tools for generating .AURINFO files and PKGBUILD data extraction.</td></tr>
<tr><td><a href="https://bbs.archlinux.org/viewtopic.php?pid=841774">pkgcacheclean</a></td><td>Clean the Pacman cache to save on disk space.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/pkg_scripts">pkg_scripts</a></td><td>A collection of pacman and package-related utilities.</td></tr>
<tr><td><a href="https://github.com/Xfennec/progress">progress</a></td><td>Show running coreutils basic commands and display stats.</td></tr>
<tr><td><a href="https://github.com/rofl0r/proxychains">proxychains-ng</a></td><td>Run programs from behind a proxy server.</td></tr>
<tr><td><a href="https://puppetlabs.com/puppet/puppet-open-source">puppet</a></td><td>Server automation framework and application.</td></tr>
<tr><td><a href="https://github.com/stef/pbp">python2-pbp</a></td><td>Simple crypto tool.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/python3-aur">python3-aur</a></td><td>AUR-related modules and helper utilities.</td></tr>
<tr><td><a href="https://harelba.github.io/q/">q</a></td><td>Run SQL directly on CSV or TSV files.</td></tr>
<tr><td><a href="https://fukuchi.org/works/qrencode/">qrencode</a></td><td>Encode data in a QR code.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/quickserve">quickserve</a></td><td>A simple HTTP server for quickly sharing files.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/quixand">quixand</a></td><td>Create single-use unrecoverable encrypted sandboxes.</td></tr>
<tr><td><a href="http://ranger.nongnu.org/">ranger</a></td><td>A simple, Vim-like file manager.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/recollect">recollect</a></td><td>Keep local copies of remote files updated.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/repo-add_and_sign">repo-add_and_sign</a></td><td>Easily create signed Pacman package repositories.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/repoman">repoman</a></td><td>The pacman of repository managers.</td></tr>
<tr><td><a href="https://github.com/nelhage/reptyr">reptyr</a></td><td>Utility for taking an existing running program and attaching it to a new terminal.</td></tr>
<tr><td><a href="http://wireless.kernel.org/en/users/Documentation/rfkill">rfkill</a></td><td>Tool for enabling and disabling wireless devices.</td></tr>
<tr><td><a href="http://utopia.knoware.nl/~hlub/uck/rlwrap/">rlwrap</a></td><td>Add readline-style editing and history to programs.</td></tr>
<tr><td><a href="https://rakshasa.github.io/rtorrent/">rtorrent</a></td><td>Rakshasa's BitTorrent client.</td></tr>
<tr><td><a href="https://github.com/saltstack/salt">salt-raet</a></td><td>Central system and configuration manager.</td></tr>
<tr><td><a href="http://sdcv.sourceforge.net/">sdcv</a></td><td>StarDict Console Version.</td></tr>
<tr><td><a href="https://github.com/saitoha/seq2gif">seq2gif</a></td><td>Convert a ttyrec record into a gif animation directly.</td></tr>
<tr><td><a href="http://stahlworks.com/dev/?tool=sfk">sfk</a></td><td>Swiss File Knife.</td></tr>
<tr><td><a href="https://github.com/naquad/shmig">shmig</a></td><td>Database migration tool.</td></tr>
<tr><td><a href="https://sift-tool.org/">sift</a></td><td>A fast and powerful open source alternative to grep.</td></tr>
<tr><td><a href="https://github.com/aperezdc/signify">signify</a></td><td>Sign and verify signatures on files.</td></tr>
<tr><td><a href="http://snapper.io/">snapper</a></td><td>The ultimate snapshot tool for Linux.</td></tr>
<tr><td><a href="http://www.dest-unreach.org/socat/">socat</a></td><td>Multipurpose relay.</td></tr>
<tr><td><a href="https://www.tarsnap.com/spiped.html">spiped</a></td><td>Secure pipe daemon.</td></tr>
<tr><td><a href="https://github.com/apenwarr/sshuttle">sshuttle</a></td><td>Poor man's VPN.</td></tr>
<tr><td><a href="http://point-at-infinity.org/ssss/">ssss</a></td><td>CLI tool for Shamir's Secret Sharing Scheme.</td></tr>
<tr><td><a href="http://steghide.sourceforge.net/">steghide</a></td><td>Embed a message in a file.</td></tr>
<tr><td><a href="http://blog.rusty.io/2010/01/24/submodules-and-subrepos-done-right/">subrepo</a></td><td>Git & Mercurial subrepos done right.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/synclinks">synclinks</a></td><td>A tool that synchronizes hierarchies of symlinks.</td></tr>
<tr><td><a href="https://github.com/bwalex/tc-play">tcplay</a></td><td>CLI TrueCrypt implementation.</td></tr>
<tr><td><a href="https://aur.archlinux.org/packages/terminfo-italics/">terminfo-italics</a></td><td>Common terminfo formats patched to support italics.</td></tr>
<tr><td><a href="https://github.com/monochromegane/the_platinum_searcher">the_platinum_searcher</a></td><td>Code searching tool similar to the_silver_searcher.</td></tr>
<tr><td><a href="https://github.com/ggreer/the_silver_searcher">the_silver_searcher</a></td><td>Code searching tool.</td></tr>
<tr><td><a href="http://xyne.archlinux.ca/projects/timedatectl-restorer">timedatectl-restorer</a></td><td>Restore system time across reboots when the internal clock is dead.</td></tr>
<tr><td><a href="https://www.torproject.org/">tor</a></td><td>Anonymizing overlay network.</td></tr>
<tr><td><a href="https://code.google.com/p/torsocks">torsocks</a></td><td>Safely torify applications.</td></tr>
<tr><td><a href="https://github.com/Tox/toxic">toxic</a></td><td>Text mode instant messaging client for Tox.</td></tr>
<tr><td><a href="http://mama.indstate.edu/users/ice/tree/">tree</a></td><td>Show the contents of a directory in a depth-indented list of files.</td></tr>
<tr><td><a href="http://0xcc.net/ttyrec/">ttyrec</a></td><td>A tty recorder and player.</td></tr>
<tr><td><a href="http://gittup.org/tup/">tup</a></td><td>A fast, file-based build system.</td></tr>
<tr><td><a href="https://www.gnu.org/software/units/units.html">units</a></td><td>Convert quantities expressed in various systems of measurement to their equivalents in other systems of measurement.</td></tr>
<tr><td><a href="http://www.draisberghof.de/usb_modeswitch/">usb_modeswitch</a></td><td>Activate switchable USB devices.</td></tr>
<tr><td><a href="https://marcan.st/blog/iphonelinux/usbmuxd/">usbmuxd</a></td><td>USB Multiplex Daemon.</td></tr>
<tr><td><a href="https://github.com/joewalnes/websocketd">websocketd</a></td><td>Turn any application that uses STDIO/STDOUT into a WebSocket server.</td></tr>
<tr><td><a href="http://aspell.net/">words</a></td><td>A collection of International 'words' files for /usr/share/dict.</td></tr>
<tr><td><a href="http://www.xinetd.org/">xinetd</a></td><td>A secure replacement for inetd.</td></tr>
<tr><td><a href="https://rg3.github.io/youtube-dl/">youtube-dl</a></td><td><a href="https://www.youtube.com/">YouTube.com</a> CLI video downloader.</td></tr>
</table>

Additional packages are available.

<table>
<tr><td>AS</td><td>Packages for ActionScript programming</td></tr>
<tr><td>Analysis</td><td>Packages for system analysis</td></tr>
<tr><td>Android</td><td>Packages for Android programming</td></tr>
<tr><td>Assembly</td><td>Packages for programming in Assembly</td></tr>
<tr><td>BEAM</td><td>Packages for programming in Erlang / Elixir</td></tr>
<tr><td>C</td><td>Packages for programming in C / C++ / Obj-C</td></tr>
<tr><td>Crystal</td><td>Packages for programming in Crystal</td></tr>
<tr><td>D</td><td>Packages for programming in D</td></tr>
<tr><td>DotNet</td><td>Packages for programming in .NET</td></tr>
<tr><td>Fonts</td><td>Fonts with internationalization</td></tr>
<tr><td>Go</td><td>Packages for programming in Go</td></tr>
<tr><td>Gobi</td><td>Gobi 3G / mobile network connection firmware</td></tr>
<tr><td>Grsec</td><td>Packages for Grsecurity / PaX</td></tr>
<tr><td>Haskell</td><td>Packages for programming in Haskell</td></tr>
<tr><td>Haxe</td><td>Packages for programming in Haxe</td></tr>
<tr><td>Julia</td><td>Packages for programming in Julia</td></tr>
<tr><td>JVM</td><td>Packages for programming in Java / Clojure / Groovy / Scala / Kotlin</td></tr>
<tr><td>LaTeX</td><td>Packages for writing in LaTeX</td></tr>
<tr><td>Lisp</td><td>Packages for programming in Lisp / Scheme / Racket</td></tr>
<tr><td>Lua</td><td>Packages for programming in Lua</td></tr>
<tr><td>Markdown</td><td>Packages for writing in Markdown</td></tr>
<tr><td>Nim</td><td>Packages for programming in Nim</td></tr>
<tr><td>OCaml</td><td>Packages for programming in OCaml</td></tr>
<tr><td>Perl</td><td>Packages for programming in Perl</td></tr>
<tr><td>Perl6</td><td>Packages for programming in Perl6</td></tr>
<tr><td>PHP</td><td>Packages for programming in PHP</td></tr>
<tr><td>Python</td><td>Packages for programming in Python</td></tr>
<tr><td>RST</td><td>Packages for writing in reStructuredText</td></tr>
<tr><td>Ruby</td><td>Packages for programming in Ruby</td></tr>
<tr><td>Rust</td><td>Packages for programming in Rust</td></tr>
<tr><td>Swift</td><td>Packages for programming in Swift</td></tr>
<tr><td>Vala</td><td>Packages for programming in Vala</td></tr>
<tr><td>Vbox</td><td>Packages for VirtualBox</td></tr>
<tr><td>Webdev</td><td>HTML, Haml, Jade, Jinja, Mustache/Handlebars, Slim; CSS, LESS, Sass, SCSS, Stylus; JS/Node</td></tr>
</table>

Selecting all packages (GUI included) adds over 1.1 GB to the download
and over 5.5 GB to the system image size. This is something to be aware
of if installing on a slow Internet connection or to a small USB stick.
An 8GB USB stick should work perfectly well, but free space would be
very limited with all GUI packages included.


Optional: dotfiles
------------------

Set `_dotfiles` to 1 for:

![openbox](https://i.imgur.com/zDzVno0.png)

[Dotfiles](https://github.com/atweiden/dotfiles)


Licensing
---------

This is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE file.
