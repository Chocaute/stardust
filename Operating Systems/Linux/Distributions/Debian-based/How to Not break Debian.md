---
created: 2024-03-13T17:31
updated: 2024-03-13T17:35
tags:
  - Debian
related link: https://wiki.debian.org/DontBreakDebian#Don.27t_make_a_FrankenDebian
---
> Debian is a robust and reliable system, however users with root access can make any change they want. As such, it's very easy for new users to break their systems by not doing things the Debian way. This page lists common mistakes made by new users. Some of the things listed here can be done safely, but only if you have enough experience to know how to fix your system when things go wrong.
> 
> The general theme to the advice here is that consequences are not always immediate, and can make future upgrades impossible without a complete reinstall. If upgrading without a complete reinstall is important to you, be careful not to make the mistakes outlined below.

One of the primary advantages of Debian is its central repository with many thousands of software packages. If you're coming to Debian from another operating system, you might be used to installing software that you find on random websites. 

On Debian **installing software from random websites is a [bad habit](https://wiki.debian.org/DebianSoftware#Footnotes)**. 

It's always better to use software from the official Debian repositories if at all possible. The packages in the Debian repositories are known to work well and install properly. Only using software from the Debian repositories is also much safer than installing from random websites which could bundle malware and other security risks.


## Don't make a FrankenDebian

Debian Stable should not be combined with other releases carelessly. If you're trying to install software that isn't available in the current Debian Stable release, it's not a good idea to add repositories for other Debian releases.

First of all, apt-get upgrade default behavior is to upgrade any installed package to the highest available version. If, for example, you configure the _bookworm_ archive on a _bullseye_ system, APT will try to upgrade almost all packages to _bookworm_.

This can be mitigated by configuring apt pinning to give priority to packages from _bullseye_.

However, even installing few packages from a "future" release can be risky. The problems might not happen right away, but the next time you install updates.

The reason things can break is because the software packaged for one Debian release is built to be compatible with the rest of the software for that release. For example, installing packages from _bookworm_ on a _bullseye_ system could also install newer versions of core libraries including _[libc6](https://packages.debian.org/libc6 "DebianPkg")_. This results in a system that is not _testing_ or _stable_ but a broken mix of the two.

Repositories that can create a FrankenDebian if used with Debian Stable:

-   Debian _testing_ release (currently _bookworm_)
    
-   Debian _unstable_ release (also known as _sid_)
    
-   Ubuntu, Mint or other derivative repositories are **not** compatible with Debian!
    
-   Ubuntu PPAs and other repositories created to distribute single applications

Some third-party repositories might appear safe to use as they contain only packages that have no equivalent in Debian. However, there are no guarantees that any repository will not add more packages in future, leading to breakage.

Finally, packages in official Debian releases have gone through extensive testing, often for months, and only fit packages are allowed in a release. On the other hand, packages from external sources might alter files belonging to other packages, configure the system in unexpected ways, introduce vulnerabilities, cause licensing issues.

Once packages from unofficial sources are introduced in a system it can become difficult to pinpoint the cause of breakage especially if it happens after months.

## Don't use GPU manufacturer install scripts

Debian includes free and open-source drivers that support most video cards. The free drivers provide the best integration with the rest of the Debian system and work quite well for most users.

If you absolutely must have the proprietary closed-source drivers, **do not download them directly from the manufacturer's website!** Installing drivers this way only works for the current kernel, and after the next kernel update, your video drivers will not work until they are manually reinstalled again.

Fortunately there is a Debian way to install video card drivers using packages in the repository. Installing the drivers the Debian way will make sure that the drivers continue to work after kernel updates.

-   [AtiHowTo](https://wiki.debian.org/AtiHowTo) has instructions on setting up the recommended free and open-source drivers for ATI/AMD video cards.
    
-   [NvidiaGraphicsDrivers](https://wiki.debian.org/NvidiaGraphicsDrivers) has instructions for installing the proprietary NVIDIA drivers the Debian way.

## Don't suffer from Shiny New Stuff Syndrome

The reason that Debian Stable is so reliable is because software is extensively tested and bug-fixed before being included. This means that the most recent version of software is often not available in the Stable repositories. But it doesn't mean that the software is too old to be useful!

Before attempting to install the newest version of some software from somewhere other than the Debian Stable repositories, here are some things to keep in mind:

-   Debian [backports security fixes](https://security-tracker.debian.org/tracker/) and reliability fixes. Judging software by comparing the version number of the Debian package to the upstream version number does not take this into account.
    
-   Installing software from places other than official Debian repositories are not covered by [Debian' Security team](https://www.debian.org/security/).

Please note: bugs are **found** in existing software but only new releases of a software can introduce **new** bugs and vulnerabilities.

As a release enters Debian and receives bugfixes, the number of unknown vulnerabilities and bugs will constantly decrease during the package lifetime.

## 'make install' can conflict with packages

It's quite easy to compile software from source code tarballs downloaded from the software's website, but not always so easy to remove it later. Often the instructions that come with the source code include instructions to use commands like _./configure && make && make install_.

When you install software this way, you **will not** be able to remove it with _apt-get_ or _Synaptic_. The APT packaging system can only remove software that was installed by the APT packaging system. Even worse, software installed this way can sometimes conflict with the software packaged for Debian.

Software installed this way also does not benefit from security updates the way that Debian packages do. If you want to keep your system up to date without having to manually compile and reinstall for every update, stick to the Debian packages.

The _make install_ script may make invalid assumptions about where the compiled binary and its associated files should exist in the filesystem and under what set of permissions / ownership it should run. Software installed this way could also replace important software vital to system and package maintenance, making it difficult to repair your system using standard Debian tools.

## Don't blindly follow bad advice

Unfortunately there's a lot of bad advice on the Internet. Tutorials found on blogs, forums and other sites often include instructions that will break your system in subtle ways. Don't simply follow the first advice you find, or the tutorial that seems the easiest. Spend some time reading the documentation and compare the difference between tutorials.

It's better to take the time to figure out the correct way to do something first than spending even more time fixing a broken system later. You would not let some random stranger feed your baby; do not execute commands without first understanding what they do.

Blog and forum posts don't expire. Instructions that might have been safe a couple of years ago might not be safe to follow any more. When in doubt keep researching and read your version's documentation.

## Read The Fantastic Manuals

Often reading a tutorial is only enough to get a general idea of how to install or use an application. Almost all of the software packaged for Debian has at least _some_ documentation available. Some places to look:

-   The Debian documentation homepage: [https://www.debian.org/doc/](https://www.debian.org/doc/)
    
-   The Debian Administrator's Handbook: [https://debian-handbook.info/](https://debian-handbook.info/)
    
-   The **apropos** command will help you find manual pages.
    
-   The **man** command for reading the manual pages for commands you don't understand.
    
-   Some software has a separate _<**package name**>-doc_ package containing documentation.
    
-   Every Debian package installed on your system has a directory in _/usr/share/doc_ that will often contain a _README.Debian_ file with information about differences from the upstream version of the software along with additional documentation.

## Don't blindly remove software

Sometimes when you remove a package, the package manager needs to remove other packages too. This is because the additional packages depend on the package you're trying to remove.

If this happens, the package manager will show you a list everything that will be removed and ask for your confirmation. **Make sure to read this list carefully!** If you don't know what some of the packages to be removed are for, read the descriptions for each one. When in doubt, do more research. Some resources that can help you research packages:

-   [https://www.debian.org/distrib/packages](https://www.debian.org/distrib/packages)
    
-   [apt-cache(8)](https://manpages.debian.org/man/8/apt-cache "DebianMan") commands:
    
    -   _apt-cache show <package name>_ to see information on a package
        
    -   _apt-cache policy <package name>_ to see version information for a package
        
    -   _apt-cache depends <package name>_ to dependencies of a package
        
-   [aptitude(8)](https://manpages.debian.org/man/8/aptitude "DebianMan") commands:
    
    -   _aptitude why <package name>_ to show incomplete reason why a package is installed
        
    -   _aptitude why-not <package name>_ to show reasons why a package cannot be installed
        
-   Use the _--simulate_ option with [apt-get(8)](https://manpages.debian.org/man/8/apt-get "DebianMan") and _aptitude_, which like the other commands in this list does **not** need to be run as root:
    
    -   _apt-get --simulate remove <package name>_
        

### Read package descriptions before installing

It is recommended to read the descriptions of packages before installing them. Sometimes software will have different packages available in the Debian repository, with each package configured a different way. Read the package descriptions and search for similar package names to make sure you get the one that you want.

This point can be especially important for packages that install kernel modules.

## Take notes

It's easy to forget the steps you took to do something on your computer, especially several months later when you're trying to upgrade. Sometimes when you try several different ways of solving a problem, it's easy to forget which method was successful the next day!

It's a very good idea to take notes about the software you've installed and configuration changes you've made. When editing configuration files, it's also a very good idea to include comments in the file explaining the reason for the changes and the date they were made.

## Some safer ways to install software not available in Debian Stable

Sometimes the need arises for installing software that is not packaged for Debian, or a newer version than is packaged for the _stable_ release. Below are some ways to reduce the risks described above.

### Backported packages

Newer versions of packages can often be found in the [Debian Backports](https://backports.debian.org/) archive. These packages are not tested as extensively as packages including in a Debian _stable_ release and should be installed in moderation.

More experienced users can make their own backports of the latest Debian software. self-backporting is usually safer than other approaches. When self-backporting fails it indicates that installing the software manually (with _make install_ or an installer script for example) could compromise your Debian system.

-   [SimpleBackportCreation](https://wiki.debian.org/SimpleBackportCreation)
    
-   [Alternative instructions](http://ircbots.debian.net/factoids/factoid.php?key=simple+sid+backport) are available as a factoid from [Debian IRC bot](https://wiki.debian.org/IRC/DpkgBot).
    
-   Also on IRC the _judd_ bot provides the [checkbackport](http://ircbots.debian.net/judd/#judd-builddep) command to provide some guidance as to whether backporting is possible by querying the [UltimateDebianDatabase](https://wiki.debian.org/UltimateDebianDatabase).
    

### Building from source

If you are building software from source obtained otherwise than from Debian, it's a good idea to build and run it as a normal user, within that user's home directory. If you keep sensitive, valuable, or non-replaceable data in your home directory, it might be a good idea to create another user account for this purpose.

[automake](https://packages.debian.org/automake "DebianPkg") and other build systems can install self-built software in non-standard locations. It is a bad idea to be root or use "sudo" to install self-built software into /usr/bin or the other standard locations where regular packages place files. It is almost always possible to instead install into your home directory instead. (Using _./configure --prefix=~/.local_ or similar.) If you understand how to edit Makefiles, then you can alter the makefile in such a way as to render _make install_ useful for your system or add a _prefix=~/.local_ option.

If you want to make the software available to all users, do not allow it to install itself to the _/usr_ directory hierarchy, as only Debian packages are meant to create files there. Installing software to the _/usr/local_ will make it available to all users, and will not interfere with the package manager. The [stow](https://packages.debian.org/stow "DebianPkg") package can be useful for managing software installed to _/usr/local_.

## Less safe ways to install software not available in Debian Stable

Please note: Packages in official Debian releases have gone through extensive testing, often for months, and only fit packages are allowed in a release. On the other hand, software from external sources can introduce security, reliability and legal issues. Debian does not endorse the use of software from external sources.

### Using chroot, containers, and virtual machines

Another strategy for using software not available in Debian stable is to run the software in a virtual Debian system contained in its on directory or image file. This allows software to be installed on the virtual Debian system without having any effect on the primary, or host, Debian system running your computer.

Debian includes a variety of tools that provide varying degrees of isolation from the host operating system. Some include:

-   [Schroot](https://wiki.debian.org/Schroot)
    
-   [LXC](https://wiki.debian.org/LXC)
    
-   [gnome-boxes](https://packages.debian.org/gnome-boxes "DebianPkg")
    
-   [libvirt](https://wiki.debian.org/libvirt) and [KVM](https://wiki.debian.org/KVM)
    
-   [systemd-container](https://packages.debian.org/systemd-container "DebianPkg") package for the [systemd-nspawn](https://manpages.debian.org/man/systemd-nspawn "DebianMan") and [machinectl](https://manpages.debian.org/man/machinectl "DebianMan") container commands
    
-   [Docker](https://wiki.debian.org/Docker)
    
-   flatpak - see below
-   snap - see below

#### Flatpak

Some applications and games are also available in the new [Flatpak](http://flatpak.org/) package format. Flatpaks can be installed locally by non-root users and do not interfere with the Debian package system. Flatpak applications can also run in a [sandbox](https://flatpak.org/faq/#Is_Flatpak_a_container_technology_). A [flatpak](https://packages.debian.org/flatpak "DebianPkg") package is available for Debian since _stretch_. [gnome-software](https://packages.debian.org/gnome-software "DebianPkg") can update and install Flatpak apps with the [gnome-software-plugin-flatpak](https://packages.debian.org/gnome-software-plugin-flatpak "DebianPkg") package installed. For more information see the [FlatpakHowto](https://wiki.debian.org/FlatpakHowto) wiki page.

#### Snap

Another alternative is the [Snappy](https://snapcraft.io/) system developed by Canonical, the company providing support for Ubuntu. Snaps are essentially similar to Flatpaks but currently (2018-10-26) the central snapcraft repository has more applications packaged than Flathub.

Important note: Many users are wary of Snaps. Use at your own discretion. They update on their own schedule, and install files to nonstandard locations. It may not be wise to use a Snap without first understanding the reputation/limitations of Snap.

## Get the most out of peer support resources

When looking for support it's important to remember that Debian is a volunteer project and people will be more inclined to help if you're polite and willing to put a little effort in yourself. Here are some general guidelines that will help you get help:

-   Research the issue on your own first, including reading documentation and using search engines.
-   Provide details and ask smart questions: [http://www.catb.org/~esr/faqs/smart-questions.html](http://www.catb.org/~esr/faqs/smart-questions.html).
    
-   If you get frustrated don't take it out on the volunteers trying to help, even if they seem frustrated with you.
-   Don't expect to be spoon fed, if you need to be guided through step by step it's a sign that you need to learn more on your own by reading documentation.
-   If you know how to answer a question from another user, you are encouraged to contribute!
-   On IRC especially:
    -   **Don't hit <enter> every few words** it gets hard to follow.
        
    -   Wait around for a response, people often disappear just before someone answers their question.
    -   Use [https://paste.debian.net/](https://paste.debian.net/) instead of pasting directly into the channel.
        

# See Also

-   [WhyDebian](https://wiki.debian.org/WhyDebian)
    
-   [FAQsFromDebianUser](https://wiki.debian.org/FAQsFromDebianUser)
    
-   [DebianResources](https://wiki.debian.org/DebianResources)
    
-   [DebianSoftware](https://wiki.debian.org/DebianSoftware) -- The software available to Debian Stable users
