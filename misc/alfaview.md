# How to install alfaview on openSUSE Leap 15.1

## Introduction

The [alfaview](https://alfaview.com) video conferencing software is available as a client for Linux. Installation on openSUSE 15.1 turned out to be problematic due to several reasons.

* Software provided as .deb package for Debian
* Dependency to glibc 2.27, openSUSE installs 2.26

## Package conversion and installation

1. Install package `alien` with yast
2. Download alfaview, alfaview_8.12.1.deb at the time fo this writing
3. Convert to rpm and install rpm, use the `--nodeps` option to accept the unresolved dependency

```sh
alien --to-rpm alfaview_8.12.1.deb
sudo rpm -i --nodeps alfaview-8.12.1-2.x86_64.rpm
```

The alfaview application is now visible in the start menu but it is not (yet) functional.

## glibc 2.27 installation

**Important** Install in a non-default location to prevent corruption of your base system.

Download glibc 2.27 sources from https://ftp.gnu.org/gnu/glibc/glibc-2.27.tar.xz

Extract to a temporary location, then compile and install:

```sh
tar -xJf https://ftp.gnu.org/gnu/glibc/glibc-2.27.tar.xz
cd glibc-2.27
mkdir glibc-build; cd glibc-build

# Important: Install in a non-default location to prevent corruption of your base system
../configure --prefix=/some/path/glibc-2.27
make
sudo make install
```

## Alfaview installation patch

Alfaview has to be started with libm 2.27, check that the following works on the command line:

```sh
export LD_PRELOAD=/some/path/glibc-2.27/lib/libm.so.6; /opt/alfaview/alfaview
```

Modify the Exec entry in `/usr/share/applications/alfaview.desktop` accordingly:

`Exec=export LD_PRELOAD=/some/path/glibc-2.27/lib/libm.so.6; env /opt/alfaview/alfaview  --url=%u`

You should now be able to
* Start alfaview from the start menu
* Start alfaview from invitation links

## Trouble shooting

### Video support

Alfaview requires gstreamer-plugins-bad for video support. Install the respective package with yast.

### Invitation links

Firefox will automatically start alfaview if the mime types are correctly registered by `alfaview.desktop`. If that doesn't work then check correctness of the local copy (if it exists), look for the `MimeType` entry:
* System-wide default: `/usr/share/applications/alfaview.desktop`
* User-local: `~/.local/share/applications/alfaview.desktop`
