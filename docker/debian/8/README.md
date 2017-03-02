# Dockerfile for XRDP on Debian 8 desktop

This Dockerfile is for image which runs XRDP on Debian 8 desktop
including GNOME3, Xfce4, MATE, LXDE and KDE Plasma.

* Building image takes about 30 minutes.
* Image size is about 3GB including layer.
* Running container needs at least 100 - 500MB RAM. kde needs 500MB,
  gnome needs 300MB and others needs 100MB.
* XRDP startup needs about 5 seconds.

## Usage

    $ docker build -t <image> . --build-arg <argument>

### argument

    user=<user>         # User name (doxrdp).
    password=<password> # User password (doxrdp).
    desktop=<desktop>   # Desktop environment (gnome).
    xrdp=<xrdp>         # XRDP version (0.9.1-4~bpo8+1).
    model=<model>       # Keyboard model (pc105).
    layout=<layout>     # Keyboard layout (us).
    variant<variant>    # Keyboard variant ("English (US)").
    cpu=<cpu>           # CPU arch (amd64).

### desktop

    gnome # GNOME3
    xfce  # Xfce4.
    mate  # MATE.
    lxde  # LXDE.
    kde   # KDE Plasma.

## Example

    $ docker build -t debian-8-xrdp_gnome . \
       --build-arg desktop=gnome --build-arg user=myname \
       --build-arg password=mypassword
    $ id=$(docker run -d debian-8-xrdp_gnome)
    $ ipaddr=$(docker inspect \
       --format="{{ .NetworkSettings.IPAddress }}" "${id}")
    $ # XRDP startup needs about 5 seconds.
    $ rdesktop -g 1024x768 -u myname "${ipaddr}" # Connect to GNOME3.

## Known issue

* This image uses latest XRDP [1] version which will be changed with
  Debian update.

  [1]: http://ftp.debian.org/debian/pool/main/x/xrdp/

* In case of lxde, "No session for pid" error dialog will be displayed
  on creating new session.
