# Dockerfile for XRDP on CentOS 7 desktop

This Dockerfile is for image which runs XRDP on CentOS 7 desktop
including Xfce4 and MATE.

* Building image takes about 10 minutes.
* Image size is about 2GB including layer.
* Running container needs at least 100 - 150MB RAM. xfce needs 100MB
  and mate needs 150MB.
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

## Example

    $ docker build -t doxrdp-centos-7-xfce . \
       --build-arg desktop=xfce --build-arg user=myname \
       --build-arg password=mypassword
    $ id=$(docker run -d --cap-add=SYS_ADMIN --tmpfs /tmp \
       --tmpfs /run -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
       doxrdp-centos-7-xfce)
    $ ipaddr=$(docker inspect \
       --format="{{ .NetworkSettings.IPAddress }}" "${id}")
    $ # XRDP startup needs about 5 seconds.
    $ rdesktop -g 1024x768 -u myname "${ipaddr}" # Connect to Xfce4.
