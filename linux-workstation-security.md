# Linux workstation security checklist

Updated: 2017-01-23

### Target audience

This document is aimed at teams of systems administrators who use Linux
workstations to access and manage your project's IT infrastructure.

If your systems administrators are remote workers, you may use this
set of guidelines to help ensure that their workstations pass core security
requirements in order to reduce the risk that they become attack vectors
against the rest of your IT infrastructure.

Even if your systems administrators are not remote workers, chances are that
they perform a lot of their work either from a portable laptop in a work
environment, or set up their home systems to access the work infrastructure
for after-hours/emergency support. In either case, you can adapt this set of
recommendations to suit your environment.

### Limitations

This, by no means, is an exhaustive "workstation hardening" document, but
rather an attempt at a set of baseline recommendations to avoid most glaring
security errors without introducing too much inconvenience. You may read this
document and think it is way too paranoid, while someone else may think this
barely scratches the surface. Security is just like driving on the highway --
anyone going slower than you is an idiot, while anyone driving faster than you
is a crazy person. These guidelines are merely a basic set of core safety
rules that is neither exhaustive, nor a replacement for experience, vigilance,
and common sense.

We're sharing this document as a way to [bring the benefits of open-source
collaboration to IT policy documentation][18]. If you find it useful, we hope
you'll contribute to its development by making a fork for your own
organization and sharing your improvements.

### Structure

Each section is split into two areas:

- The checklist that can be adapted to your project's needs
- Free-form list of considerations that explain what dictated these decisions

#### Checklist priority levels

The items in each checklist include the priority level, which we hope will
help guide your decision:

- _(ESSENTIAL)_ items should definitely be high on the consideration list.
  If not implemented, they will introduce high risks to your workstation
  security.
- _(NICE)_ to have items will improve the overall security, but will
  affect how you interact with your work environment, and probably require
  learning new habits or unlearning old ones.
- _(PARANOID)_ is reserved for items we feel will significantly improve your
  workstation security, but will require a lot of adjustment to the
  way you interact with your operating system.

Remember, these are only guidelines. If you feel these priority levels do not
reflect your project's commitment to security, you should adjust them as you
see fit.

### Checklist

- [ ] System supports SecureBoot _(ESSENTIAL)_
- [ ] System has no firewire, thunderbolt or ExpressCard ports _(NICE)_
- [ ] System has a TPM chip _(NICE)_

### Considerations

#### SecureBoot

Despite its controversial nature, SecureBoot offers prevention against many
attacks targeting workstations (Rootkits, "Evil Maid," etc), without
introducing too much extra hassle. It will not stop a truly dedicated attacker,
plus there is a pretty high degree of certainty that state security agencies
have ways to defeat it (probably by design), but having SecureBoot is better
than having nothing at all.

Alternatively, you may set up [Anti Evil Maid][1] which offers a more
wholesome protection against the type of attacks that SecureBoot is supposed
to prevent, but it will require more effort to set up and maintain.

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

- [ ] UEFI boot mode is used (not legacy BIOS) _(ESSENTIAL)_
- [ ] Password is required to enter UEFI configuration _(ESSENTIAL)_
- [ ] SecureBoot is enabled _(ESSENTIAL)_

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

## Distro choice considerations

Chances are you'll stick with a fairly widely-used distribution such as Fedora,
Ubuntu, Arch, Debian, or one of their close spin-offs. In any case, this is
what you should consider when picking a distribution to use.

### Checklist

- [ ] Has a robust MAC/RBAC implementation (SELinux/AppArmor/GrSecurity) _(ESSENTIAL)_
- [ ] Publishes security bulletins _(ESSENTIAL)_
- [ ] Provides timely security patches _(ESSENTIAL)_
- [ ] Provides cryptographic verification of packages _(ESSENTIAL)_
- [ ] Fully supports UEFI and SecureBoot _(ESSENTIAL)_
- [ ] Has robust native full disk encryption support _(ESSENTIAL)_

### Considerations

#### SELinux, AppArmor, and GrSecurity/PaX

Mandatory Access Controls (MAC) or Role-Based Access Controls (RBAC) are an
extension of the basic user/group security mechanism used in legacy POSIX
systems. Most distributions these days either already come bundled with a
MAC/RBAC implementation (Fedora, Ubuntu), or provide a mechanism to add it via
an optional post-installation step (Gentoo, Arch, Debian). Obviously, it is
highly advised that you pick a distribution that comes pre-configured with a
MAC/RBAC system, but if you have strong feelings about a distribution that
doesn't have one enabled by default, do plan to configure it
post-installation.

Distributions that do not provide any MAC/RBAC mechanisms should be strongly
avoided, as traditional POSIX user- and group-based security should be
considered insufficient in this day and age. If you would like to start out
with a MAC/RBAC workstation, AppArmor and GrSecurity/PaX are generally
considered easier to learn than SELinux. Furthermore, on a workstation, where
there are few or no externally listening daemons, and where user-run
applications pose the highest risk, GrSecurity/PaX will offer more security
benefits than just SELinux.

## Distro installation guidelines

All distributions are different, but here are general guidelines:

### Checklist

- [ ] Use full disk encryption (LUKS) with a robust passphrase _(ESSENTIAL)_
- [ ] Make sure swap is also encrypted _(ESSENTIAL)_
- [ ] Set up a robust root password (can be same as LUKS) _(ESSENTIAL)_
- [ ] Use an unprivileged account, part of administrators group _(ESSENTIAL)_
- [ ] Set up a robust user-account password, different from root _(ESSENTIAL)_

### Considerations

#### Full disk encryption

Unless you are using self-encrypting hard drives, it is important to configure
your installer to fully encrypt all the disks that will be used for storing
your data and your system files. It is not sufficient to simply encrypt the
user directory via auto-mounting cryptfs loop files (I'm looking at you, older
versions of Ubuntu), as this offers no protection for system binaries or swap,
which is likely to contain a slew of sensitive data. The recommended
encryption strategy is to encrypt the LVM device, so only one passphrase is
required during the boot process.

The `/boot` partition will usually remain unencrypted, as the bootloader needs
to be able to boot the kernel itself before invoking LUKS/dm-crypt. Some
distributions support encrypting the `/boot` partition as well (e.g.
[Arch][16]), and it is possible to do the same on other distros, but likely at
the cost of complicating system updates. It is not critical to encrypt
`/boot` if your distro of choice does not natively support it, as the kernel
image itself leaks no private data and will be protected against tampering
with a cryptographic signature checked by SecureBoot.

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

Weak passphrases are combinations of words you're likely to see in published
works or anywhere else in real life, and you should avoid using them, as
attackers are starting to include such simple passphrases into their
brute-force strategies. Examples of passphrases to avoid:

- Mary had a little lamb
- you're a wizard, Harry
- to infinity and beyond

You can also stick with non-vocabulary passwords that are at least 10-12
characters long, if you prefer that to typing passphrases.

Unless you have concerns about physical security, it is fine to write down your
passphrases and keep them in a safe place away from your work desk.

#### Root, user passwords and the admin group

We recommend that you use the same passphrase for your root password as you
use for your LUKS encryption (unless you share your laptop with other trusted
people who should be able to unlock the drives, but shouldn't be able to
become root). If you are the sole user of the laptop, then having your root
password be different from your LUKS password has no meaningful security
advantages.  Generally, you can use the same passphrase for your UEFI
administration, disk encryption, and root account -- knowing any of these will
give an attacker full control of your system anyway, so there is little
security benefit to have them be different on a single-user workstation.

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

- [ ] Check your firewalls to ensure all incoming ports are filtered _(ESSENTIAL)_
- [ ] Make sure root mail is forwarded to an account you check _(ESSENTIAL)_
- [ ] Set up an automatic OS update schedule, or update reminders _(ESSENTIAL)_
- [ ] Check to ensure sshd service is disabled by default _(NICE)_
- [ ] Configure the screensaver to auto-lock after a period of inactivity _(NICE)_

### Considerations

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

## Best practices

What follows is a curated list of best practices that we think you should
adopt. It is most certainly non-exhaustive, but rather attempts to offer
practical advice that strikes a workable balance between security and overall
usability.

### Use Fido U2F for website 2-factor authentication

[Fido U2F][22] is a standard developed specifically to provide a mechanism for
2-factor authentication *and* combat credential phishing. Regular OTP
(one-time password) mechanisms are ineffective in the case where the attacker
is able to trick you into submitting your password and token into a malicious
site masquerading as a legitimate service. The U2F protocol will store site
authentication data on the USB token that will prevent you from accidentally
giving an attacker both your password and your one-time token if you try to
use it on anything other than the legitimate website.

See this site for a curated list of services providing Fido U2F support:

- [dongleauth.info][23]

Note, that not all browsers currently support U2F-capable hardware tokens, and
if you use sandboxes or virtualization-based isolation around your browser,
you may have to work extra hard to enable USB pass-through from the
application to your USB token.

### Password managers

#### Checklist

- [ ] Use a password manager _(ESSENTIAL)_
- [ ] Use unique, randomly generated passwords on unrelated sites _(ESSENTIAL)_
- [ ] Use a password manager that supports team sharing _(NICE)_

#### Considerations

Using strong, unique, randomly generated passwords should be a critical
requirement for every member of your team. Credential theft is happening all
the time -- either via compromised computers, stolen database dumps, remote
site exploits, or any number of other means. No credentials should be reused
across different sites, ever.

##### In-browser password manager

Every browser has a mechanism for saving passwords that is fairly secure and
can sync with vendor-maintained cloud storage while keeping the data encrypted
with a user-provided passphrase. However, this mechanism has important
disadvantages:

1. It does not work across browsers
2. It does not offer any way of sharing credentials with team members

There are several well-supported, free-or-cheap password managers that are
well-integrated into multiple browsers, work across platforms, and offer
group sharing (usually as a paid service). Solutions can be easily found via
search engines.

##### Standalone password manager

One of the major drawbacks of any password manager that comes integrated with
the browser is the fact that it's part of the application that is most likely
to be attacked by intruders. If this makes you uncomfortable (and it should),
you may choose to have two different password managers -- one for websites
that is integrated into your browser, and one that runs as a standalone
application. The latter can be used to store high-risk credentials such as
root passwords, database passwords, other shell account credentials, etc.

It may be particularly useful to have such tool for sharing superuser account
credentials with other members of your team (server root passwords, ILO
passwords, database admin passwords, bootloader passwords, etc).

A few tools can help you:

- [KeePassX][8], which improves team sharing in version 2
- [Pass][9], which uses text files and PGP and integrates with git
- [Django-Pstore][10], which uses GPG to share credentials between admins
- [Hiera-Eyaml][11], which, if you are already using Puppet for your
  infrastructure, may be a handy way to track your server/service credentials
  as part of your encrypted Hiera data store

### Securing SSH and PGP private keys

Personal encryption keys, including SSH and PGP private keys, are going to be
the most prized items on your workstation -- something the attackers will be
most interested in obtaining, as that would allow them to further attack your
infrastructure or impersonate you to other admins. You should take extra steps
to ensure that your private keys are well protected against theft.

#### Checklist

- [ ] Strong passphrases are used to protect private keys _(ESSENTIAL)_
- [ ] PGP Master key is stored on removable storage _(NICE)_
- [ ] Auth, Sign and Encrypt Subkeys are stored on a smartcard device _(NICE)_
- [ ] SSH is configured to use PGP Auth key as ssh private key _(NICE)_

#### Considerations

The best way to prevent private key theft is to use a smartcard to store your
encryption private keys and never copy them onto the workstation. There are
several manufacturers that offer OpenPGP capable devices:

- [Kernel Concepts][12], where you can purchase both the OpenPGP compatible
  smartcards and the USB readers, should you need one.
- [Yubikey][13], which offers OpenPGP smartcard functionality in addition
  to many other cool features (U2F, PIV, HOTP, etc).
- [NitroKey][21], which is based on open-source software and hardware

It is also important to make sure that the master PGP key is not stored on the
main workstation, and only subkeys are used. The master key will only be
needed when signing someone else's keys or creating new subkeys -- operations
which do not happen very frequently. You may follow [the Debian's subkeys][14]
guide to learn how to move your master key to removable storage and how to
create subkeys.

You should then configure your gnupg agent to act as ssh agent and use the
smartcard-based PGP Auth key to act as your ssh private key. We publish a
[detailed guide][15] on how to do that using either a smartcard reader or a
Yubikey NEO.

If you are not willing to go that far, at least make sure you have a strong
passphrase on both your PGP private key and your SSH private key, which will
make it harder for attackers to steal and use them.

### Hibernate or shut down, do not suspend

When a system is suspended, the RAM contents are kept on the memory chips and
can be read by an attacker (known as the [Cold Boot Attack][17]). If you are
going away from your system for an extended period of time, such as at the end
of the day, it is best to shut it down or hibernate it instead of suspending
it or leaving it on.

### SELinux on the workstation

If you are using a distribution that comes bundled with SELinux (such as
Fedora), here are some recommendation of how to make the best use of it to
maximize your workstation security.

#### Checklist

- [ ] Make sure SELinux is enforcing on your workstation _(ESSENTIAL)_
- [ ] Never `setenforce 0` _(NICE)_

#### Considerations

SELinux is a Mandatory Access Controls (MAC) extension to core POSIX
permissions functionality. It is mature, robust, and has come a long way since
its initial roll-out. Regardless, many sysadmins to this day repeat the
outdated mantra of "just turn it off."

That being said, SELinux will have limited security benefits on the
workstation, as most applications you will be running as a user are going to
be running unconfined. It does provide enough net benefit to warrant leaving
it on, as it will likely help prevent an attacker from escalating privileges
to gain root-level access via a vulnerable daemon service.

Our recommendation is to leave it on and enforcing.

##### Never `setenforce 0`

It's tempting to use `setenforce 0` to flip SELinux into permissive mode
on a temporary basis, but you should avoid doing that. This essentially turns
off SELinux for the entire system, while what you really want is to
troubleshoot a particular application or daemon.

Instead of `setenforce 0` you should be using `semanage permissive -a
[somedomain_t]` to put only that domain into permissive mode. First, find out
which domain is causing troubles by running `ausearch`:

    ausearch -ts recent -m avc

and then look for `scontext=` (source SELinux context) line, like so:

    scontext=staff_u:staff_r:gpg_pinentry_t:s0-s0:c0.c1023
                             ^^^^^^^^^^^^^^

This tells you that the domain being denied is `gpg_pinentry_t`, so if you
want to troubleshoot the application, you should add it to permissive domains:

    semanage permissive -a gpg_pinentry_t

This will allow you to use the application and collect the rest of the AVCs,
which you can then use in conjunction with `audit2allow` to write a local
policy. Once that is done and you see no new AVC denials, you can remove that
domain from permissive by running:

    semanage permissive -d gpg_pinentry_t

## Further reading

The world of IT security is a rabbit hole with no bottom. If you would like to
go deeper, or find out more about security features on your particular
distribution, please check out the following links:

- [Fedora Security Guide](https://docs-old.fedoraproject.org/en-US/Fedora/19/html/Security_Guide/index.html)
- [CESG Ubuntu Security Guide](https://www.gov.uk/government/publications/end-user-devices-security-guidance-ubuntu-1404-lts)
- [Debian Security Manual](https://www.debian.org/doc/manuals/securing-debian-howto/index.en.html)
- [Arch Linux Security Wiki](https://wiki.archlinux.org/index.php/Security)
- [Mac OSX Security](https://www.apple.com/support/security/guides/)

## License
This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][0].

[0]: http://creativecommons.org/licenses/by-sa/4.0/
[1]: https://github.com/QubesOS/qubes-antievilmaid
[2]: https://en.wikipedia.org/wiki/IEEE_1394#Security_issues
[3]: https://qubes-os.org/
[4]: https://xkcd.com/936/
[5]: https://spideroak.com/
[6]: https://code.google.com/p/chromium/wiki/LinuxSandboxing
[7]: http://www.thoughtcrime.org/software/sslstrip/
[8]: https://keepassx.org/
[9]: http://www.passwordstore.org/
[10]: https://pypi.python.org/pypi/django-pstore
[11]: https://github.com/TomPoulton/hiera-eyaml
[12]: http://shop.kernelconcepts.de/
[13]: https://www.yubico.com/products/yubikey-hardware/
[14]: https://wiki.debian.org/Subkeys
[15]: https://github.com/lfit/ssh-gpg-smartcard-config
[16]: http://www.pavelkogan.com/2014/05/23/luks-full-disk-encryption/
[17]: https://en.wikipedia.org/wiki/Cold_boot_attack
[18]: http://www.linux.com/news/featured-blogs/167-amanda-mcpherson/850607-linux-foundation-sysadmins-open-source-their-it-policies
[19]: https://firejail.wordpress.com/
[20]: https://firejail.wordpress.com/documentation-2/firefox-guide/
[21]: https://www.nitrokey.com/
[22]: https://en.wikipedia.org/wiki/Universal_2nd_Factor
[23]: http://www.dongleauth.info/
[24]: https://subgraph.com/sgos/
