# Dockerfile for XRDP on Ubuntu 16.04 desktop

This Dockerfile is for image which runs XRDP on Ubuntu 16.04 desktop
including Unity, Gnome classic like, Xfce4, MATE, LXDE and KDE Plasma.

* Building image takes about 30 minutes.
* Image size is about 3GB including layer.
* Running container needs at least 300 - 500MB RAM.
* XRDP startup needs about 5 seconds.

## Usage

    $ docker build -t <image> . --build-arg <argument>

### argument

    user=<user>         # User name (doxrdp).
    password=<password> # User password (doxrdp).
    desktop=<desktop>   # Desktop environment (unity).
    xrdp=<xrdp>         # XRDP version (0.9.1-7).
    model=<model>       # Keyboard model (pc105).
    layout=<layout>     # Keyboard layout (us).
    cpu=<cpu>           # CPU arch (amd64).

### desktop

    unity   # Unity.
    classic # Gnome classic like.
    xubuntu # Xfce4.
    mate    # MATE.
    lubuntu # LXDE.
    kubuntu # KDE Plasma.

## Example

    $ docker build -t ubuntu-1604-xrdp-unity . \
       --build-arg desktop=unity --build-arg user=myname \
       --build-arg password=mypassword
    $ id=$(docker run -d ubuntu-1604-xrdp-unity)
    $ ipaddr=$(docker inspect \
       --format="{{ .NetworkSettings.IPAddress }}" "${id}")
    $ # XRDP startup needs about 5 seconds.
    $ rdesktop -g 1024x768 -u myname "${ipaddr}" # Connect to Unity.

## Known issue

* This image uses latest XRDP [1] version which will be changed with
  Ubuntu update.

  [1]: http://archive.ubuntu.com/ubuntu/pool/universe/x/xrdp

* In case of lubuntu, "No session for pid" error dialog will be
  displayed on creating new session.

* This image removed gnome-software because gnome-software will
  pressure CPU at the first start for about 5 minutes. If you need
  gnome-software on container, you need to wait till gnome-software
  initialization completed. Dockerfile should run the initialization
  of gnome-software in advance.

* "Log Out" at the top right does not work in unity and classic. Need
  to disconnect from client side or restart XRDP.

* Screen lock is enabled in kubuntu because could not find a way to
  disable it via CLI. Please disable it via GUI.
