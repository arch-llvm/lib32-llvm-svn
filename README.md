# lib32-llvm-svn

This is an [Arch](https://www.archlinux.org/) Linux [PKGBUILD](https://wiki.archlinux.org/index.php/Creating_packages) for the [LLVM](http://llvm.org/) compiler infrastructure, the [Clang](http://clang.llvm.org/) frontend, and the various tools associated with it. It's available in the [Arch User Repository](https://wiki.archlinux.org/index.php/Arch_User_Repository) as [lib32-llvm-svn](https://aur.archlinux.org/pkgbase/lib32-llvm-svn/).

Main development is in the master branch, while the AUR git repo is mirrored by the [aur](https://github.com/kerberizer/lib32-llvm-svn/tree/aur) branch.

## IMPORTANT INFORMATION

**PLEASE READ THIS ONE CAREFULLY**

This is a fairly complex package. The only recommended and supported method of building is in a clean chroot as [described on the Arch Wiki](https://wiki.archlinux.org/index.php/DeveloperWiki:Building_in_a_Clean_Chroot). A crude example is also provided further below. The use of [AUR helpers](https://wiki.archlinux.org/index.php/AUR_helpers) (yaourt, pacaur, etc.) in particular is discouraged; it may or may not work for you.

Also, unlike the [official package](https://www.archlinux.org/packages/?sort=&repo=Multilib&q=lib32-llvm&maintainer=&flagged=), which provides the latest **stable** release, this one builds the code straight from the SVN source repository, where development is constantly taking place. Thus, it brings all the latest bells and whistles, but also tends to bring and all the **latest bugs**. It is therefore strongly recommended to use this LLVM/Clang build only for testing. Use in production should only be reserved for cases where you do need a particular feature (or a fix for some bug), which are not yet available in the stable releases.

## Binary packages

Pre-built, binary packages are available from two unofficial repositories:

- _lordheavy_'s [mesa-git](https://wiki.archlinux.org/index.php/Unofficial_user_repositories#mesa-git), which may be particularly useful for those who need LLVM solely as a Mesa dependency. Note that the packages are built against the [testing] repos. _lordheavy_ is an Arch Linux developer and trusted user (TU).

- _kerberizer_'s [llvm-svn](https://wiki.archlinux.org/index.php/Unofficial_user_repositories#llvm-svn), which is automatically rebuilt every 6 hours from this PKGBUILD and the latest SVN code. The packages are built against the [core/extra] repos. _kerberizer_ is the current maintainer.

Both repos provide `x86_64`, `i686` and `multilib` packages. _kerberizer_'s repo is also PGP signed.

## If using LLVM as Mesa dependency

You may find helpful the topic "[mesa-git - latest videodrivers & issues](https://bbs.archlinux.org/viewtopic.php?id=212819)" on the Arch Linux forums.

## Building in a clean chroot example

If you need a more detailed and specific example on how to build this package in a clean chroot, a crude excerpt from the build script of the _kerberizer_'s binary repo is presented here. You can also check the [full script](https://github.com/kerberizer/pkg-rebuild-llvm).

It is meant to allow building `lib32-llvm-svn` too, hence why `gcc-multilib` is used. The code takes advantage of multiple cores when building and compressing; the example here is tailored to an 8-core/threads system. The user's ccache cache is utilised as well, so frequent rebuilds can be much faster. If you don't sign your packages, omit the lines mentioning `PACKAGER` and `GPGKEY`, otherwise they need to be set correctly. The chroot (`${x86_64_chroot}`) is best set up in `/tmp`, but this requires a lot of RAM (most likely at least 32 GB, since `/tmp` is by default half the size of the physical RAM detected); second best solution is on an SSD. The latter goes for `~/.ccache` as well. Note that the latest versions of systemd mount `/tmp` with the nosuid flag. You need to turn this flag off before building on `/tmp`, or else the build will fail.

```shell
cd /path/to/where/lib32-llvm-svn/is/cloned

x86_64_chroot="/chroot/x86_64"

sudo mkdir -p "${x86_64_chroot}/root"

sudo /usr/bin/mkarchroot \
    -C /usr/share/devtools/pacman-multilib.conf \
    -M /usr/share/devtools/makepkg-x86_64.conf \
    -c /var/cache/pacman/pkg \
    "${x86_64_chroot}/root" \
    base-devel ccache

sudo /usr/bin/arch-nspawn "${x86_64_chroot}/root" /bin/bash -c "yes | pacman -Sy gcc-multilib"

sudo /usr/bin/arch-nspawn "${x86_64_chroot}/root" /bin/bash -c \
    "echo -e \"CCACHE_DIR='/.ccache'\nXZ_DEFAULTS='--threads=8'\" >>/etc/environment ; \
     sed \
        -e 's/^#MAKEFLAGS=.*$/MAKEFLAGS=\"-j9\"/' \
        -e '/^BUILDENV=/s/\!ccache/ccache/' \
        -e 's/^#PACKAGER=.*$/PACKAGER=\"Some One <someone@somewhere.com>\"/' \
        -e 's/^#GPGKEY=.*$/GPGKEY=\"0x0000000000000000\"/' \
        -i /etc/makepkg.conf"

sudo /usr/bin/makechrootpkg -c -d ~/.ccache:/.ccache -r "${x86_64_chroot}"
```

It's advisable to always start this from scratch, i.e. don't reuse the old chroot, but create it anew for each build (it uses the local pacman cache, so doesn't waste bandwidth, and if located in /tmp or on an SSD, is pretty fast).
