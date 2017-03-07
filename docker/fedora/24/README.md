# Dockerfile for XRDP on Fedora 24 desktop

This Dockerfile is for image which runs XRDP on Fedora 24 desktop
including Xfce4, MATE and LXDE.

* Building image takes about 20 minutes.
* Image size is about 3GB including layer.
* Running container needs at least 300MB RAM.
* XRDP startup needs about 5 seconds.

This image uses systemd which needs the following option.

    $ docker run --cap-add=SYS_ADMIN --tmpfs /tmp --tmpfs /run \
    -v /sys/fs/cgroup:/sys/fs/cgroup:ro <image>

## Usage

    $ docker build -t <image> . --build-arg <argument>

### argument

    user=<user>         # User name (doxrdp).
    password=<password> # User password (doxrdp).
    desktop=<desktop>   # Desktop environment (xfce).

### desktop

    xfce  # Xfce4.
    mate  # MATE.
    lxde  # LXDE.

## Example

    $ docker build -t doxrdp-fedora-24-xfce . \
       --build-arg desktop=xfce --build-arg user=myname \
       --build-arg password=mypassword
    $ id=$(docker run -d --cap-add=SYS_ADMIN --tmpfs /tmp \
       --tmpfs /run -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
       doxrdp-fedora-24-xfce)
    $ ipaddr=$(docker inspect \
       --format="{{ .NetworkSettings.IPAddress }}" "${id}")
    $ # XRDP startup needs about 5 seconds.
    $ rdesktop -g 1024x768 -u myname "${ipaddr}" # Connect to Xfce4.

## Known issue

* This image cannot display login prompt thuough all systemd service
  is done. But this image can run XRDP. So if you need to operate
  console, please use "docker exec -it container bash".

* In case of lxde, "No session for pid" error dialog will be displayed
  on creating new session.
