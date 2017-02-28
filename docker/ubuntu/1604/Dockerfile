# Dockerfile for XRDP on Ubuntu 16.04 desktop.
#
# Copyright (C) 2017 Hiroo MATSUMOTO <hiroom2.mail@gmail.com>
#

FROM ubuntu:16.04
MAINTAINER Hiroo MATSUMOTO <hiroom2.mail@gmail.com>
LABEL \
Description="XRDP on Ubuntu 16.04 desktop" \
Version="1.0"
EXPOSE 3389

ARG user=${user:-doxrdp}
ARG password=${password:-doxrdp}
ARG desktop=${desktop:-unity}
ARG xrdp=${xrdp:-0.9.1-7}
ARG model=${model:-pc105}
ARG layout=${layout:-us}
ARG cpu=${cpu:-amd64}

# Check desktop argument at an early stage.
RUN \
case "${desktop}" in \
unity) true;; \
classic) true;; \
xubuntu) true;; \
mate) true;; \
lubuntu) true;; \
kubuntu) true;; \
*) echo "Not supported desktop: ${desktop}" 1>&2; false;; \
esac

# Set locale.
RUN locale-gen en_US en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
RUN locale > /etc/default/locale

# Update Ubuntu.
RUN \
apt-get update -y && \
apt-get upgrade -y

# Install expect.
RUN apt-get install -y expect

# Install debconf at an early stage for debconf-set-selections.
RUN apt-get install -y debconf

# Set keyboard.
RUN \
echo "\
keyboard-configuration keyboard-configuration/modelcode \
string ${model}\n\
keyboard-configuration keyboard-configuration/layoutcode \
string ${layout}\n\
" | debconf-set-selections

# Create user.
RUN \
apt-get install -y sudo bash && \
sed -e 's/%sudo\(.*\)ALL$/%sudo\1NOPASSWD:ALL/g' -i /etc/sudoers && \
useradd -m ${user} -s /bin/bash && \
gpasswd -a ${user} sudo && \
printf "\
set timeout -1\n\
spawn passwd ${user}\n\
expect \"Enter new UNIX password:\" { send \"${password}\\\n\" }\n\
expect \"Retype new UNIX password:\" { send \"${password}\\\n\" }\n\
expect eof\n\
" | expect

# Install TigerVNC from upstream.
RUN \
apt-get install -y dpkg-dev git && \
mkdir tigervnc.build && \
cd tigervnc.build && \
git clone https://github.com/TigerVNC/tigervnc && \
cd tigervnc && \
git checkout ff872614b507d0aa8bfbd09ef41550390cfe658a -b branch && \
ln -s contrib/packages/deb/ubuntu-xenial/debian && \
chmod a+x debian/rules && \
pkgs=$(dpkg-checkbuilddeps 2>&1 | \
       sed -e 's/.*build dependencies://g' -e 's/([^)]*)//g') && \
apt-get install -y -o 'apt::install-recommends=true' ${pkgs} && \
fakeroot debian/rules binary && \
apt-get remove -y dpkg-dev git ${pkgs} && \
cd .. && \
(dpkg -i *.deb || (apt-get -f install -y && dpkg -i *.deb)) && \
cd .. && \
rm -rf tigervnc.build

# Install latest XRDP. And apply patch to latest XRDP for changing
# Xvnc session to be default because Xorg session's cursor has black
# background.
RUN \
mkdir xrdp && \
cd xrdp && \
apt-get install -y wget patch && \
URL=http://archive.ubuntu.com/ubuntu/pool/universe/x/xrdp && \
PKG="\
xrdp_${xrdp}_${cpu}.deb \
xorgxrdp_${xrdp}_${cpu}.deb \
" && \
for pkg in ${PKG}; do \
  wget -q ${URL}/${pkg} && \
  (dpkg -i "${pkg}" || apt-get -f install -y); \
done && \
cd .. && \
rm -rf xrdp && \
cd /etc/xrdp && \
echo "\
--- xrdp.ini.org	2017-02-23 07:14:16.614833723 +0000\n\
+++ xrdp.ini	2017-02-23 07:15:10.489078284 +0000\n\
@@ -147,15 +147,6 @@ tcutils=true\n\
 ; Session types\n\
 ;\n\
 \n\
-[Xorg]\n\
-name=Xorg\n\
-lib=libxup.so\n\
-username=ask\n\
-password=ask\n\
-ip=127.0.0.1\n\
-port=-1\n\
-code=20\n\
-\n\
 [Xvnc]\n\
 name=Xvnc\n\
 lib=libvnc.so\n\
@@ -166,6 +157,15 @@ port=-1\n\
 #xserverbpp=24\n\
 #delay_ms=2000\n\
 \n\
+[Xorg]\n\
+name=Xorg\n\
+lib=libxup.so\n\
+username=ask\n\
+password=ask\n\
+ip=127.0.0.1\n\
+port=-1\n\
+code=20\n\
+\n\
 [console]\n\
 name=console\n\
 lib=libvnc.so\n\
" | patch -p0 && \
apt-get remove -y wget patch

# Disable resolvconf postinst to /etc/resolv.conf which cannot be
# updated in container.
# https://github.com/docker/docker/issues/1297#issuecomment-115458690
RUN \
echo "resolvconf resolvconf/linkify-resolvconf boolean false" | \
debconf-set-selections

# Install desktop.
RUN \
case "${desktop}" in \
unity)\
  apt-get install -y ubuntu-desktop unity && \
  ln -s /usr/bin/unity-control-center \
        /usr/bin/gnome-control-center && \
  service dbus start && \
  su - ${user} -c "dbus-launch gsettings set \
  org.gnome.desktop.lockdown disable-lock-screen true" && \
  su - ${user} -c "dbus-launch gsettings set \
  org.gnome.desktop.session idle-delay 0" && \
  service dbus stop && \
  printf "\
/usr/lib/gnome-session/gnome-session-binary --session=ubuntu &\n\
/usr/lib/x86_64-linux-gnu/unity/unity-panel-service &\n\
/usr/lib/unity-settings-daemon/unity-settings-daemon &\n\
\n\
for indicator in /usr/lib/x86_64-linux-gnu/indicator-*; do\n\
  basename=\$(basename \${indicator})\n\
  dirname=\$(dirname \${indicator})\n\
  service=\${dirname}/\${basename}/\${basename}-service\n\
  \${service} &\n\
done\n\
\n\
unity\n\
" > /home/${user}/.xsession && \
  chown ${user}:${user} /home/${user}/.xsession;; \
classic) \
  apt-get install -y ubuntu-desktop gnome-panel metacity && \
  ln -s /usr/bin/unity-control-center \
        /usr/bin/gnome-control-center && \
  printf "\
gnome-panel &\n\
metacity &\n\
" > /home/${user}/.xsessionrc && \
  chown ${user}:${user} /home/${user}/.xsessionrc;; \
mate) \
  apt-get install -y ubuntu-mate-desktop ubuntu-mate-core \
                     mate-desktop-environment-extras && \
  service dbus start && \
  su - ${user} -c "dbus-launch gsettings set \
  org.mate.screensaver idle-activation-enabled false" && \
  su - ${user} -c "dbus-launch gsettings set \
  org.mate.screensaver lock-enabled false" && \
  service dbus stop && \
  echo "mate-session" > /home/${user}/.xsession && \
  chown ${user}:${user} /home/${user}/.xsession;; \
xubuntu) \
  apt-get install -y xubuntu-desktop && \
  echo "mode: off\n" > /home/${user}/.xscreensaver && \
  chown ${user}:${user} /home/${user}/.xscreensaver && \
  echo "xfce4-session" > /home/${user}/.xsession && \
  chown ${user}:${user} /home/${user}/.xsession;; \
lubuntu) \
  apt-get install -y lubuntu-desktop && \
  apt-get remove -y light-locker && \
  echo "lxsession -s Lubuntu -e LXDE" > /home/${user}/.xsession && \
  chown ${user}:${user} /home/${user}/.xsession;; \
kubuntu) \
  apt-get install -y kubuntu-desktop && \
  apt-get install -y libglib2.0-bin && \
  echo "startkde" > /home/${user}/.xsession && \
  chown ${user}:${user} /home/${user}/.xsession;; \
esac

# Fix incomplete language support.
RUN \
apt-get install -y language-selector-common && \
apt-get install -y language-pack-gnome-en && \
apt-get install -y $(check-language-support)

# Remove gnome-software. If you need image including gnome-software,
# you need to run gnome-software on container till gnome-software
# initialization completed. And you need to stop rsyslog, dbus and
# xrdp service. Then you need to commit container as image.
RUN apt-get remove -y gnome-software

# Remove expect.
RUN apt-get remove -y expect

# Clean cache
RUN \
apt-get autoremove -y && \
apt-get clean -y all

# Use rsyslog instead of journald.
CMD \
service rsyslog start && \
service dbus start && \
service xrdp start && \
tail -f /var/log/syslog
