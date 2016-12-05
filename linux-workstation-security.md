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

All of them, obviously, can be different if there is a compelling reason.

## Post-installation hardening

Post-installation security hardening will depend greatly on your distribution
of choice, so it is futile to provide detailed instructions in a general
document such as this one. However, here are some steps you should take:

### Checklist

- [ ] Check your firewalls to ensure all incoming ports are filtered _(ESSENTIAL)_
- [ ] Make sure root mail is forwarded to an account you check _(ESSENTIAL)_
- [ ] Set up an automatic OS update schedule, or update reminders _(ESSENTIAL)_

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

## Best practices

What follows is a curated list of best practices that we think you should
adopt. It is most certainly non-exhaustive, but rather attempts to offer
practical advice that strikes a workable balance between security and overall
usability.

### Securing SSH and PGP private keys

Personal encryption keys, including SSH and PGP private keys, are going to be
the most prized items on your workstation -- something the attackers will be
most interested in obtaining, as that would allow them to further attack your
infrastructure or impersonate you to other admins. You should take extra steps
to ensure that your private keys are well protected against theft.

#### Checklist

- [ ] Strong passphrases are used to protect private keys _(ESSENTIAL)_
- [ ] SSH is configured to use PGP Auth key as ssh private key _(NICE)_

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
