# genie

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/I3I1VA18)

A quick way into a systemd "bottle" for WSL

What does that even mean?

Well, this gives you a way to run systemd as pid 1, with all the trimmings, inside WSL 2. It does this by creating a pid namespace, the eponymous poor-man's-container "bottle", starting up systemd in there, and entering it, and providing some helpful shortcuts to do so.

If you want to try it, please read this entire document first, _especially_ the BUGS section.

## NOTE: WSL 2 ONLY

Note: it is only possible to run _systemd_ (and thus _genie_ ) under WSL 2; WSL 1 does not support the system calls required to do so. If you are running inside a distro configured as WSL 1, even if your system supports WSL 2, genie will fail to operate properly.

## INSTALLATION

If there is a package available for your distribution, this is the recommended method of installing genie.

### Arch

For Arch Linux users, there are prebuilt packages available at:

https://aur.archlinux.org/packages/genie-systemd/ and

https://aur.archlinux.org/packages/genie-systemd-git/

The former of which is prebuilt and the latter of which compiles it from source. Both install all needed dependencies, save that for version 1.17 and above, you will have to install .NET Core 3.0 runtime explicitly. A PKGBUILD file for this in Arch is available here for those who wish to use it: https://github.com/arkane-systems/genie/files/3827049/dotnet-PKGBUILD.tar.gz ; you can also make your own, and for security and good practice, please always check what a PKGBUILD does before you use it.

Thanks to Arley Henostroza for providing these.

### Debian

Dependent packages on Debian are _daemonize_, _dbus_, _dotnet-runtime-3.0_, _hostess_, _policykit-1_, _systemd_, and _util-linux_ . For the most part, these are either already installed or in the distro and able to be installed automatically.

The chief exceptions are _dotnet-runtime-3.0_ , for which you will need to follow the installation instructions here:

https://dotnet.microsoft.com/download/linux-package-manager/debian9/runtime-3.0.0

And _hostess_ ( https://github.com/cbednarski/hostess ), a package for which has been placed in the wsl-translinux repo for your convenience and which should thus install automatically with genie.

To install, add the wsl-translinux repository here: https://packagecloud.io/arkane-systems/wsl-translinux , and install genie using the command:

_sudo apt install systemd-genie_

### Ubuntu (and other Debian derivatives)

Use the Debian package.

### Other

Currently, there are no installation packages available for other distributions/package-management systems. It is possible to download the genie.tar.gz file from the Releases page (essentially, the unwrapped Debian package) and manually place the files within, other than the DEBIAN folder, in the matching places and with the correct provisions. Note that this is a packaged build, rather than a local-install build, and thus expects itself and hostess to both be installed under _/usr/bin_.

If you are able to build an install package for another distribution/package-management system, please consider adding your build process to the genie makefile in the same manner as the _debian_ target, and submitting a pull request. While not able to maintain packages myself for every distro, keeping the build processes for as many as possible in this repo would be useful to future genie users.

### ...OR BUILD IT YOURSELF

It is possible to build your own version of genie and install it locally. To do so, you will require _build-essential_ and _dotnet-sdk-3.0_ in addition to the other dependencies, all of which must be installed manually. Note that since hostess is not normally installed from a package but by direct download from https://github.com/cbednarski/hostess , when using a local-install build of genie, hostess must be installed in _/usr/local/bin_ .

After cloning the repository, run

```
make install
```

This will build genie and install it under _/usr/local_ .

## USAGE

```
genie:
  Handles transitions to the "bottle" namespace for systemd under WSL.

Usage:
  genie [options] [command]

Options:
  -v, --verbose <VERBOSE>    Display verbose progress messages
  --version                  Display version information

Commands:
  -i, --initialize           Initialize the bottle (if necessary) only.
  -s, --shell                Initialize the bottle (if necessary), and run a shell in it.
  -c, --command <COMMAND>    Initialize the bottle (if necessary), and run the specified command in it.
```

So, it has three modes, all of which will set up the bottle and run systemd in it if it isn't already running for simplicity of use.

_genie -i_ will set up the bottle - including changing the WSL hostname by suffixing -wsl, to distinguish it from the Windows host -  run systemd, and then exit. This is intended for use if you want services running all the time in the background, or to preinitialize things so you needn't worry about startup time later on, and for this purpose is ideally run from Task Scheduler on logon.

_genie -s_ runs your login shell inside the bottle; basically, Windows-side, _wsl genie -s_ is your substitute for just _wsl_ to get started, or for the shortcut you get to start a shell in the distro. It follows login semantics, and as such does not preserve the current working directory.

_genie -c [command]_ runs _command_ inside the bottle, then exits. The return code is the return code of the command. It follows sudo semantics, and so does preserve the cwd.

## RECOMMENDATIONS

Once you have this up and running, I suggest disabling via systemctl the _getty@tty1_ service (since logging on and using WSL is done via ptsen, not ttys).

Further tips on usage from other genie users can be found on the wiki for this repo.

## BUGS

1. This breaks _pstree_ and other _/proc_-walking tools that count on everything being a child of pid 1, because entering the namespace with a shell or other process leaves that process with a ppid of 0. To the best of my knowledge, I can't set the ppid of a process, and if I'm wrong about that, please send edification and pull requests to be gratefully accepted.

2. It is considerably clunkier than I'd like it to be, inasmuch as you have to invoke genie every time to get inside the bottle, either manually (replacing, for example, _wsl [command]_ with _wsl genie -c [command]_), or by using your own shortcut in place of the one WSL gives you for the distro, using which will put you _outside_ the bottle. Pull requests, etc.
