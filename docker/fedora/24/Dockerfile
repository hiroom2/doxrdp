# Dockerfile for XRDP on Fedora 24 desktop.
#
# Copyright (C) 2017 Hiroo MATSUMOTO <hiroom2.mail@gmail.com>
#

FROM fedora:24
MAINTAINER Hiroo MATSUMOTO <hiroom2.mail@gmail.com>
LABEL \
Description="XRDP on Fedora 24 desktop" \
Version="1.0"
EXPOSE 3389
STOPSIGNAL SIGRTMIN+3
ENV container docker

ARG user=${user:-doxrdp}
ARG password=${password:-doxrdp}
ARG desktop=${desktop:-xfce}

# Check desktop argument at an early stage.
RUN \
case "${desktop}" in \
xfce) true;; \
mate) true;; \
lxde) true;; \
*) echo "Not supported desktop: ${desktop}" 1>&2; false;; \
esac

# Update Fedora.
RUN dnf update -y

# Install xrdp and tigervnc-server.
RUN \
dnf install -y xrdp tigervnc-server && \
ln -s /usr/lib/systemd/system/xrdp.service \
/etc/systemd/system/multi-user.target.wants/xrdp.service

# Create user.
RUN \
dnf install -y expect sudo passwd && \
sed -e 's/%wheel\(.*\)/#%wheel\1/g' \
    -e 's/#.*%wheel\(.*\)NOPASSWD\(.*\)/%wheel\1NOPASSWD\2/g' \
    -i /etc/sudoers && \
useradd -m ${user} -s /bin/bash && \
gpasswd -a ${user} wheel && \
printf "\
set timeout -1\n\
spawn passwd ${user}\n\
expect \"New password:\" { send \"${password}\\\n\" }\n\
expect \"Retype new password:\" { send \"${password}\\\n\" }\n\
expect eof\n\
" | expect && \
dnf remove -y expect

# Install desktop.
RUN \
case "${desktop}" in \
xfce) \
  dnf groupinstall -y "Xfce Desktop" && \
  dnf install -y firefox thunderbird && \
  mkdir -p /home/${user}/.config/xfce4 && \
  printf "\
MailReader=thunderbird\n\
WebBrowser=firefox\n\
" > /home/${user}/.config/xfce4/helpers.rc && \
  chown ${user}:${user} -R /home/${user}/.config && \
  rm -f /etc/xdg/autostart/xfce-polkit.desktop && \
  echo "mode: off\n" > /home/${user}/.xscreensaver && \
  chown ${user}:${user} /home/${user}/.xscreensaver && \
  echo "xfce4-session" > /home/${user}/.Xclients && \
  chmod a+x /home/${user}/.Xclients && \
  chown ${user}:${user} /home/${user}/.Xclients;; \
mate) \
  dnf groupinstall -y "MATE Desktop" && \
  dnf remove -y mate-screensaver && \
  echo "mate-session" > /home/${user}/.Xclients && \
  chmod a+x /home/${user}/.Xclients && \
  chown ${user}:${user} /home/${user}/.Xclients;; \
lxde) \
  dnf groupinstall -y "LXDE Desktop" && \
  echo "mode: off\n" > /home/${user}/.xscreensaver && \
  chown ${user}:${user} /home/${user}/.xscreensaver && \
  echo "lxsession -s LXDE -e LXDE" > /home/${user}/.Xclients && \
  chmod a+x /home/${user}/.Xclients && \
  chown ${user}:${user} /home/${user}/.Xclients;; \
esac

# Remove NetworkManager. It is unneeded but takes time for boot up.
RUN \
dnf remove -y NetworkManager

# Clean cache.
RUN \
dnf autoremove -y && \
dnf clean all && \
/usr/sbin/ldconfig

# Run systemd.
CMD /sbin/init
