# Linux workstation security checklist
This is a set of recommendations used by the Linux Foundation for their systems
administrators. All of LF employees are remote workers and we use this set of
guidelines to ensure that a sysadmin's system passes core security requirements
in order to reduce the risk of it becoming the attack vector against the rest
of our infrastructure.

Even if your systems administrators are not remote workers, chances are that
they perform a lot of their work either from a portable laptop in a work
environment, or set up their home systems to access work infrastructure for
after-hours/emergency support. In either case, you can adapt this set of
recommendations to suit your environment.

This, by no means, is an exhaustive "workstation hardening" document, but
rather an attempt at a set of baseline recommendations to avoid most glaring
security errors without introducing too much inconvenience. You may read this
document and think it is way too paranoid, while someone else may think this
barely scratches the surface. Security is just like driving on a highway --
anyone going slower than you is an idiot, while anyone driving faster than you
is a crazy person. These guidelines are merely a basic set of highway safety
rules that is neither exhaustive, nor a replacement for experience, vigilance,
and common sense.

## Choosing the right hardware
We do not mandate that our admins use a specific vendor or a specific model, so
this section addresses core considerations when choosing a work system.

### Checklist
- [ ] System supports SecureBoot _(CRITICAL)_
- [ ] System has no firewire, thunderbolt or ExpressCard ports _(MODERATE)_
- [ ] System has a TPM chip _(LOW)_

### Considerations
#### SecureBoot
Despite its controversial nature, SecureBoot offers prevention against many
attacks targeting workstations (Rootkits, "Evil Maid," etc), without
introducing too much extra hassle. It will not stop a truly dedicated attacker,
plus there is a pretty high degree of certainty that state security agencies
have ways to defeat it (probably by design), but having SecureBoot is better
than having nothing at all.

Alternatively, you may set up [Anti Evil Maid][1] which offers more wholesome
protection against the type of attacks that SecureBoot is supposed to prevent,
but it will require more effort to set up and maintain.

#### Firewire, thunderbolt, and ExpressCard ports
Firewire is a silly standard that, by design, allows any connecting device full
direct memory access to your system ([see Wikipedia][2]). Thunderbolt and
ExpressCard are guilty of the same sin, though some later implementations of
Thunderbolt attempt to mitigate this vulnerability. It is best if the system
you are getting has none of these ports, but it is not critical, as they
usually can be turned off via UEFI or disabled in the kernel itself.

#### TPM Chip
Trusted Platform Module (TPM) is a crypto chip bundled with the motherboard
separately from the core processor, which can be used for additional platform
security (such as to store full-disk encryption keys), but is not normally used
for day-to-day workstation operation. At best, this is a nice-to-have, unless
you have a specific need to use TPM for your workstation security.

## Pre-boot environment
This is a set of recommendations for your workstation before you even start
with OS installation.

### Checklist
- [ ] UEFI boot mode is used (not legacy BIOS) _(CRITICAL)_
- [ ] Password is required to enter UEFI configuration _(CRITICAL)_
- [ ] SecureBoot is enabled _(CRITICAL)_
- [ ] UEFI-level password is required to boot the system _(LOW)_

### Considerations
#### UEFI and SecureBoot
UEFI, with all its warts, offers a lot of goodies that legacy BIOS doesn't,
such as SecureBoot. Most modern systems come with UEFI mode on by default.

Make sure a strong password is required to enter UEFI configuration mode. Pay
attention, as many manufacturers quietly limit the length of the password you
are allowed to use, so you may need to choose high-entropy short passwords vs.
long passphrases (see below for more on passphrases).

Depending on the Linux distribution you decide to use, you may or may not have
to jump through additional hoops in order to import your distribution's
SecureBoot key that would allow you to boot the distro. Many distributions have
partnered with Microsoft to sign their released kernels with a key that is
already recognized by most system manufacturers, therefore saving you the
trouble of having to deal with key importing.

As an extra measure, before someone is allowed to even get to the boot
partition and try some badness there, let's make them enter a password. This
password should be different from your UEFI management password, in order to
prevent shoulder-surfing. If you shut down and start a lot, you may choose to
not bother with this, as you will already have to enter a LUKS passphrase and
this will save you a few extra keystrokes.

## Distro choice considerations
Chances are you'll stick with a fairly widely-used distribution such as Fedora,
Ubuntu, Arch, Debian, or one of their close spin-offs. In any case, this is
what you should consider when picking a distribution to use.

### Checklist
- [ ] Has a robust MAC/RBAC implementation (SELinux/AppArmor/PaX) _(CRITICAL)_
- [ ] Publishes security bulletins _(CRITICAL)_
- [ ] Provides timely security patches _(CRITICAL)_
- [ ] Provides cryptographic verification of packages _(CRITICAL)_
- [ ] Fully supports UEFI and SecureBoot _(CRITICAL)_
- [ ] Has robust native full disk encryption support _(CRITICAL)_

### Considerations
#### SELinux, AppArmor, and GrSecurity/PaX
Mandatory Access Controls (MAC) or Role-Based Access Controls are an extension
of the basic user/group security mechanism used in legacy POSIX systems. Most
distributions these days either already come bundled with a MAC/RBAC
implementation (Fedora, Ubuntu), or provide a mechanism to add it via an
optional post-installation step (Gentoo, Arch, Debian). Obviously, it is highly
advised that you pick a distribution that comes pre-configured with a MAC/RBAC
system, but if you have strong feelings about a distribution that doesn't come
with one enabled by default, do plan to configure it post-installation.

Distributions that do not provide any MAC/RBAC mechanisms should be strongly
avoided, as traditional POSIX user- and group-based security mechanisms should
be considered insufficient in this day and age. If you would like to start out
with a MAC/RBAC workstation, AppArmor and PaX are generally considered easier
to learn than SELinux. Furthermore, on a workstation, where there are few or no
externally listening daemons, and where user-run applications pose the highest
risk, GrSecurity/PaX will _probably_ offer more security benefits than SELinux.

#### Distro security bulletins
Most widely used distributions have a mechanism to deliver security bulletins
to its users, but if you are fond of something esoteric, check whether the
developers have a documented mechanism of alerting the users about security
vulnerabilities and patches. Absence of such mechanism is a major warning sign
that the distribution is not mature enough to be considered for a primary admin
workstation.

#### Timely and trusted security updates
Most widely used distributions deliver security updates, but is worth checking
to ensure that critical package updates are provided in a timely fashion. Avoid
using spin-offs and "community rebuilds" for this reason, as they routinely
delay security updates due to having to wait for the upstream distribution to
release it first.

You'll be hard-pressed to find a distribution that does not use cryptographic
signatures on packages, updates metadata, or both. That being said, fairly
widely used distributions have been known to go for years before introducing
this basic security measure (Arch, I'm looking at you), so this is a thing
worth checking.

#### Distros supporing UEFI and SecureBoot
Check that the distribution supports UEFI and SecureBoot. Find out whether it
requires importing an extra key or whether it signs its boot kernels with a key
already trusted by systems manufacturers (e.g. via an agreement with
Microsoft). Some distributions do not support UEFI/SecureBoot but offer
alternatives to ensure tamper-proof or tamper-evident boot environments
([Qubes-OS][3] uses Anti Evil Maid, mentioned earlier). If a distribution
doesn't support SecureBoot and has no mechanisms to prevent boot-level attacks,
look elsewhere.

#### Full disk encryption
Full disk encryption is a requirement for securing data at rest, and is
supported by most distributions. As an alternative, systems with
self-encrypting hard drives may be used (normally implemented via the on-board
TPM chip) and offer comparable levels of security plus faster operation, but at
a considerably higher cost.

## Distro installation guidelines
All distributions are different, but here are general guidelines:

### Checklist
- [ ] Use full disk encryption (LUKS) with a robust passphrase _(CRITICAL)_
- [ ] Make sure swap is also encrypted _(CRITICAL)_
- [ ] Require a password to edit bootloader (can be same as LUKS) _(CRITICAL)_
- [ ] Set up a robust root password (can be same as LUKS) _(CRITICAL)_
- [ ] Use an unprivileged account, part of administrators group _(CRITICAL)_
- [ ] Set up a robust user-account password, different from root _(CRITICAL)_

### Considerations
#### Full disk encryption
Unless you are using self-encrypting hard drives, it is important to configure
your installer to fully encrypt all the disks that will be used for storing
your data and your system files. It is not sufficient to simply encrypt the
user directory via auto-mounting cryptfs loop files (I'm looking at you,
Ubuntu), as this offers no protection for system binaries or swap, which is
likely to contain a slew of sensitive data. The recommended encryption strategy
is to encrypt the LVM device, so only one passphrase is required during the
boot process.

The `/boot` partition will always remain unencrypted, as the bootloader needs
to be able to actually boot the kernel before invoking LUKS/dm-crypt. The
kernel image itself should be protected against tampering with a cryptographic
signature checked by SecureBoot.

In other words, `/boot` should always be the only unencrypted partition on your
system.

#### Choosing good passphrases
Modern Linux systems have no limitation of password/passphrase length, so the
only real limitation is your level of paranoia and your stubbornness. If you
boot your system a lot, you will probably have to type at least two different
passwords: one to unlock LUKS, and another one to log in, so having long
passphrases will probably get old really fast. Pick passphrases that are 2-3
words long, easy to type, and preferably from rich/mixed vocabularies.

Examples of good passphrases (yes, you can use spaces):
- nature abhors roombas
- 12 in-flight Jebediahs
- perdon, tengo flatulence

You can also stick with non-vocabulary passwords that are at least 10-12
characters long, if you prefer that to typing passphrases.

Unless you have concerns about physical security, it is fine to write down your
passphrases and keep them in a safe place away from your work desk.

#### Root, user passwords and the admin group
I recommend that you use the same passphrase for your root password as you use
for your LUKS encryption (unless you share your laptop with other trusted
people who should be able to unlock the drives, but shouldn't be able to become
root). If you are the sole user of the laptop, then having your root password
be different from your LUKS password has no meaningful security advantages.
Generally, you can use the same passphrase for your UEFI administration, disk
encryption, and root account -- knowing any of these will give an attacker full
control of your system anyway, so there is little security benefit to have them
be different on a single-user workstation.

You should have a different, but equally strong password for your regular user
account that you will be using for day-to-day tasks. This user should be member
of the admin group (e.g. `wheel` or similar, depending on the distribution),
allowing you to perform `sudo` to elevate privileges.

In other words, if you are the sole user on your workstation, you should have 2
distinct, robust, equally strong passphrases you will need to remember:

**Admin-level**, used in the following locations:
- UEFI administration
- Bootloader (GRUB)
- Disk encryption (LUKS)
- Workstation admin (root user)

**User-level**, used for the following:
- User account and sudo
- Master password for the password manager

All of them, obviously, can be different if there is a compelling reason.

## Post-installation hardening
Post-installation security hardening will depend greatly on your distribution
of choice, so it is futile to provide detailed instructions in a general
document such as this one. However, here are some steps you should take:

### Checklist
- [ ] Globally disable firewire and thunderbolt modules _(CRITICAL)_
- [ ] Check your firewalls to ensure all incoming ports are filtered _(CRITICAL)_
- [ ] Make sure root mail is forwarded to an account you check _(CRITICAL)_
- [ ] Check to ensure sshd service is disabled by default _(MODERATE)_
- [ ] Set up an automatic OS update schedule, or update reminders _(MODERATE)_
- [ ] Configure the screensaver to auto-lock after a period of inactivity _(MODERATE)_
- [ ] Set up logwatch _(MODERATE)_
- [ ] Install and use rkhunter _(LOW)_
- [ ] Install an Intrusion Detection System _(PARANOID)_

### Considerations
#### Blacklisting modules
To blacklist a firewire and thunderbolt modules, add the following lines to a
file in `/etc/modprobe.d/blacklist-dma.conf`:

    blacklist firewire-core
    blacklist thunderbolt

The modules will be blacklisted upon reboot. It doesn't hurt doing this even if
you don't have these ports (but it doesn't do anything either).

#### Root mail
By default, root mail is just saved on the system and tends to never be read.
Make sure you set your `/etc/aliases` to forward root mail to a mailbox that
you actually read, otherwise you may miss important system notifications and
reports:

    # Person who should get root's mail
    root:          bob@example.com

Run `newaliases` after this edit and test it out to make sure that it actually
gets delivered, as some email providers will reject email coming in from
nonexistent or non-routable domain names. If that is the case, you will need to
play with your mail forwarding configuration until this actually works.

#### Firewalls, sshd, and listening daemons
The default firewall settings will depend on your distribution, but many of
them will allow incoming `sshd` ports. Unless you have a compelling legitimate
reason to allow incoming ssh, you should filter that out and disable the `sshd`
daemon.

    systemctl disable sshd.service
    systemctl stop sshd.service

You can always start it temporarily if you need to use it.

In general, your system shouldn't have any listening ports apart from
responding to ping. This will help safeguard you against network-level 0-day
exploits.

#### Automatic updates or notifications
It is recommended to turn on automatic updates, unless you have a very good
reason not to do so, such as fear that an automatic update would render your
system unusable (it's happened in the past, so this fear is not unfounded). At
the very least, you should enable automatic notifications of available updates.
Most distributions already have this service automatically running for you, so
chances are you don't have to do anything. Consult your distribution
documentation to find out more.

You should apply all outstanding errata as soon as possible, even if something
isn't specifically labeled as "security update" or has an associated CVE code.
All bugs have potential of being security bugs and erring on the side of newer,
unknown bugs is _generally_ a safer strategy than sticking with old, known
ones.

#### Watching logs
You should have a keen interest in what happens on your system. For this
reason, you should install `logwatch` and configure it to send nightly activity
reports of everything that happens on your system. This won't prevent a
dedicated attacker, but is a good safety-net feature to have in place.

Note, that many systemd distros will no longer automatically install a syslog
server that `logwatch` needs (due to systemd relying on its own journal), so
you will need to install and enable `rsyslog` to make sure your `/var/log` is
not empty before logwatch will be of any use.

#### Rkhunter and IDS
Installing `rkhunter` and an intrusion detection system (IDS) like `aide` or
`tripwire` will not be that useful unless you actually understand how they work
and take the necessary steps to set them up properly (such as, keeping the
databases on external media, running checks from a trusted environment,
remembering to update the hash databases after performing system updates and
configuration changes, etc). If you are not willing to take these steps and
adjust how you do things on your own workstation, these tools will introduce
hassle without any tangible security benefit.

I do recommend that you install `rkhunter` and run it nightly. It's fairly easy
to learn and use, and though it will not deter a sophisticated attacker, it may
help you catch your own mistakes.

## Personal workstation backups
Workstation backups tend to be overlooked or done in a haphazard, often unsafe
manner.

### Checklist
- [ ] Set up encrypted workstation backups to external storage _(CRITICAL)_
- [ ] Use zero-knowledge backup tools for cloud backups _(MODERATE)_

### Considerations
#### Full encrypted backups to external storage
It is handy to have an external hard drive where one can dump full backups
without having to worry about such things like bandwidth and upstream speeds
(in this day and age most providers still offer dramatically asymmetric
upload/download speeds). Needless to say, this hard drive needs to be in itself
encrypted (again, via LUKS), or you should use a backup tool that creates
encrypted backups, such as `duplicity` or its GUI companion, `deja-dup`. I
recommend using the latter with a good randomly generated passphrase, stored in
a password manager. If you travel with your laptop, leave this drive at home to
have something to come home to in case your laptop is lost or stolen.

In addition to your home directory, you should also back up `/etc` and
`/var/log` for various forensic purposes.

Above all, avoid copying your home directory onto unencrypted storage, even as
a quick way to move your files around between systems, as you will most
certainly forget to erase it once you're done.

#### Selective zero-knowledge backups off-site
Off-site backups are also extremely important and can be done either to your
employer, if they offer space for it, or to a cloud provider. You can set up a
separate duplicity/deja-dup profile to only include most important files in
order to avoid transferring huge amounts of data that you don't really care to
back up off-site (internet cache, music, downloads, etc).

Alternatively, you can use a zero-knowledge backup tool, such as
[SpiderOak][5], which offers an excellent Linux GUI tool and has additional
useful features such as synchronizing content between multiple systems and
platforms.

## Best practices
What follows is a curated list of best practices that we think you should
adopt. It is most certainly non-exhaustive, but attempts to offer practical
advice that strikes a workable balance between security and overall usability.

### Browsing
There is no question that the web browser will be the piece of software with
the largest and the most exposed attack surface on your system. It is a tool
written specifically to download and execute untrusted, frequently hostile
code. It attempts to shield you from this by employing multiple mechanisms
such as sandboxes and code inspection, but they have all been previously
defeated on multiple occasions. You should learn to treat browsing websites as
the most insecure activity you'll engage in on any given day.

There are several ways you can reduce the impact of a compromised browser, but
the truly effective ways will require significant changes in the way you
operate your workstation.

#### 1: Use two different browsers
This is the easiest to do, but only offers minor security benefits. Not all
browser compromises give an attacker full unfettered access to your system --
sometimes they are limited to allowing one to read local browser storage,
steal active sessions from other tabs, capture input entered into the browser,
etc. Using two different browsers, one for work/high security sites, and
another for everything else will help prevent minor compromises from giving
attackers access to the whole cookie jar. The main inconvenience will be the
amount of memory consumed by two different browser processes.

Here's what we recommend:

##### Firefox for work and high security sites
Use it to access work-related sites, where extra care should be taken to
ensure that data like cookies, sessions, login information, keystrokes, etc,
should most definitely not fall into an attacker's hands. You should NOT use
this browser for accessing any other sites except select few.

You should install the following Firefox add-ons:

- [ ] NoScript _(CRITICAL)_
  - NoScript prevents active content from loading, unless specifically
    whitelisted. It is a great hassle to use inside your default browser
    (though offers really good security benefits), so we recommend only
    enabling it on the browser you use to access work-related sites.
- [ ] Ghostery _(CRITICAL)_
  - Ghostery will prevent most external trackers and ad platforms from being
    loaded on the pages, which will help prevent compromises on these tracking
    sites from affecting your browser (trackers and ad sites are very commonly
    targeted by attackers, as they allow rapid infection of multiple systems
    worldwide).
- [ ] HTTPS Everywhere _(CRITICAL)_
  - This EFF-developed Add-on will ensure that all your sites are accessed
    over a secure connection, even if a link you click is using http:// (great
    to avoid a number of attacks like SSL-strip).
- [ ] Certificate Patrol _(MODERATE)_
  - This tool will alert you if the site you're accessing has recently changed
    the TLS certificates -- especially if it wasn't nearing expiration dates
    or if it is now using a different certification authority. Note, that this
    will generate a lot of false-positives.

You should leave Firefox as your default browser for opening links, as
NoScript will prevent most active content from loading or executing.

##### Chrome/Chromium for everything else
Chromium developers are ahead of Firefox in adding a lot of nice security
features (at least [on Linux][6]), such as seccomp sandboxes, kernel user
namespaces, etc, which act as an added layer of isolation between the sites
you visit and the rest of your system. Chromium is the upstream open-source
project, and Chrome is Google's proprietary binary build based on it (insert
the usual paranoid caution about not using it for anything you don't want
Google to know about).

It is recommended that you install Ghostery and HTTPS Everywhere extensions in
Chrome as well and give it a distinct theme from Firefox to indicate that this
is your "untrusted sites" browser.

#### 2: Use two different browsers, one inside a dedicated VM
This is a similar recommendation to the above, except you will add an extra
step of running Chrome inside a dedicated VM that you access via a fast
protocol that allows you to share clipboards and forwards sound events (e.g.
Spice or RDP). This will add an excellent layer of isolation between the
untrusted browser and the rest of your work environment, ensuring that
attackers who manage to fully compromise your browser will then have to break
out of the VM isolation layer in order to get to the rest of your system.

This is a surprisingly workable configuration, but requires a lot of RAM and
fast processors that can handle the increased load. It will also require an
important amount of dedication on the part of the admin who will need to
adjust their work practices accordingly.

#### 3: Fully separate your work and play environments via virtualization
See [Qubes-OS project][3], which strives to provide a high-security
workstation environment via compartmentalizing your applications into separate
fully isolated VMs.

### Password managers

### Team communication

### SELinux on the workstation
- [CRITICAL] Make sure SELinux is enforcing on your workstation
- [CRITICAL] Never `setenforce 0`, use `semanage permissive -a somedomain_t`
- [CRITICAL] Never blindly run `audit2allow`, always check
- [MODERATE] Switch your account to SELinux user `staff_u` (use `usermod -Z`)
- (use `sudo -r sysadm_r` when performing sudo)

[1]: https://github.com/QubesOS/qubes-antievilmaid
[2]: https://en.wikipedia.org/wiki/IEEE_1394#Security_issues
[3]: https://qubes-os.org/
[4]: https://xkcd.com/936/
[5]: https://spideroak.com/
[6]: https://code.google.com/p/chromium/wiki/LinuxSandboxing