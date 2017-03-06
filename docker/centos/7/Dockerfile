# Dockerfile for XRDP on CentOS 7 desktop.
#
# Copyright (C) 2017 Hiroo MATSUMOTO <hiroom2.mail@gmail.com>
#

FROM centos:7
MAINTAINER Hiroo MATSUMOTO <hiroom2.mail@gmail.com>
LABEL \
Description="XRDP on CentOS 7 desktop" \
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
*) echo "Not supported desktop: ${desktop}" 1>&2; false;; \
esac

# Update CentOS.
RUN yum update -y

# Install xrdp and tigervnc-server.
RUN \
yum install -y epel-release && \
yum install -y xrdp tigervnc-server && \
ln -s /usr/lib/systemd/system/xrdp.service \
/etc/systemd/system/multi-user.target.wants/xrdp.service

# Create user.
RUN \
yum install -y expect sudo bash && \
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
yum remove -y expect

# Install desktop.
RUN \
case "${desktop}" in \
xfce) \
  yum groupinstall -y "Xfce" && \
  yum install -y firefox thunderbird && \
  mkdir -p /home/${user}/.config/xfce4 && \
  printf "\
MailReader=thunderbird\n\
WebBrowser=firefox\n\
" > /home/${user}/.config/xfce4/helpers.rc && \
  rm -f /etc/xdg/autostart/xfce-polkit.desktop && \
  chown ${user}:${user} -R /home/${user}/.config && \
  echo "xfce4-session" > /home/${user}/.Xclients && \
  chmod a+x /home/${user}/.Xclients && \
  chown ${user}:${user} /home/${user}/.Xclients;; \
mate) \
  yum groupinstall -y "MATE Desktop" && \
  yum remove -y mate-screensaver && \
  echo "mate-session" > /home/${user}/.Xclients && \
  chmod a+x /home/${user}/.Xclients && \
  chown ${user}:${user} /home/${user}/.Xclients;; \
esac

# Remove NetworkManager. It is unneeded but takes time for boot up.
RUN \
yum remove -y NetworkManager

# Clean cache.
RUN \
yum autoremove -y && \
yum clean all

# Run systemd.
CMD /sbin/init
