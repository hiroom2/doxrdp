# Dockerfile for XRDP on Debian 8 desktop.
#
# Copyright (C) 2017 Hiroo MATSUMOTO <hiroom2.mail@gmail.com>
#

FROM debian:8
MAINTAINER Hiroo MATSUMOTO <hiroom2.mail@gmail.com>
LABEL \
Description="XRDP on Debian 8 desktop" \
Version="1.0"
EXPOSE 3389

ARG user=${user:-doxrdp}
ARG password=${password:-doxrdp}
ARG desktop=${desktop:-gnome}
ARG xrdp=${xrdp:-0.9.1-4~bpo8+1}
ARG model=${model:-pc105}
ARG layout=${layout:-us}
ARG variant=${variant:-"English (US)"}
ARG cpu=${cpu:-amd64}

# Check desktop argument at an early stage.
RUN \
case "${desktop}" in \
gnome) true;; \
xfce) true;; \
mate) true;; \
lxde) true;; \
kde) true;; \
*) echo "Not supported desktop: ${desktop}" 1>&2; false;; \
esac

# Update Debian.
RUN \
apt-get update -y && \
apt-get upgrade -y

# Set locale.
RUN apt-get install -y locales
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
RUN \
sed -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' \
-i /etc/locale.gen && \
locale-gen en_US en_US.UTF-8 && \
locale > /etc/default/locale

# Install expect.
RUN apt-get install -y expect

# Install debconf at an early stage for debconf-set-selections.
RUN apt-get install -y debconf

# Set keyboard. Debian requires keyboard-configuration/variant
# compared to Ubuntu.
RUN \
echo "\
keyboard-configuration keyboard-configuration/modelcode \
string ${model}\n\
keyboard-configuration keyboard-configuration/layoutcode \
string ${layout}\n\
keyboard-configuration keyboard-configuration/variant \
string ${variant}\n\
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

# Install package for FLTK, TigerVNC and XRDP.
RUN apt-get install -y wget dpkg-dev git patch

# Install FLTK for TigerVNC.
RUN \
mkdir fltk.build && \
cd fltk.build && \
URL=http://archive.ubuntu.com/ubuntu/pool/universe/f/ && \
wget -q ${URL}/fltk1.3/fltk1.3_1.3.3.orig.tar.gz && \
wget -q ${URL}/fltk1.3/fltk1.3_1.3.3-8.dsc && \
wget -q ${URL}/fltk1.3/fltk1.3_1.3.3-8.debian.tar.xz && \
tar zxvf fltk1.3_1.3.3.orig.tar.gz && \
cd fltk-1.3.3/ && \
tar xvf ../fltk1.3_1.3.3-8.debian.tar.xz && \
pkgs="$(dpkg-checkbuilddeps 2>&1 | \
       sed -e 's/.*build dependencies://g' -e 's/|/ /g' \
           -e 's/libglu-dev//g' -e 's/libgl-dev//g' \
           -e 's/libz-dev//g' -e 's/([^)]*)//g')" && \
apt-get install -y -o 'apt::install-recommends=true' ${pkgs} && \
dpkg-buildpackage -us -uc && \
apt-get remove -y ${pkgs} && \
cd .. && \
dpkg -i *.deb || (apt-get -f install -y ; dpkg -i *.deb)  && \
cd .. && \
rm -rf fltk.build

# Install TigerVNC from upstream.
RUN \
mkdir tigervnc.build && \
cd tigervnc.build && \
git clone https://github.com/TigerVNC/tigervnc && \
cd tigervnc && \
git checkout ff872614b507d0aa8bfbd09ef41550390cfe658a -b branch && \
ln -s contrib/packages/deb/ubuntu-xenial/debian && \
chmod a+x debian/rules && \
sed -e 's/libjpeg-turbo8/libjpeg62-turbo/g' \
    -e 's/libgnutls30/libgnutls-deb0-28/g' \
    -e 's/libgnutls-dev/libgnutls28-dev/g' -i debian/control && \
pkgs="$(dpkg-checkbuilddeps 2>&1 | \
       sed -e 's/.*build dependencies://g' -e 's/([^)]*)//g')" && \
apt-get install -y -o 'apt::install-recommends=true' ${pkgs} && \
rm debian/xorg-source-patches/xserver118-patch.patch && \
touch debian/xorg-source-patches/xserver118-patch.patch && \
fakeroot debian/rules binary && \
apt-get remove -y ${pkgs} && \
cd .. && \
dpkg -i *.deb || (apt-get -f install -y ; dpkg -i *.deb) && \
cd .. && \
rm -rf tigervnc.build

# Install latest XRDP. XRDP 0.9.1-4~bpo8+1 can be installed without
# updating other package while XRDP 0.9.1-7 cannot. And apply patch to
# latest XRDP for changing Xvnc session to be default because Xorg
# session's cursor has black background.
RUN \
mkdir xrdp && \
cd xrdp && \
URL=http://ftp.debian.org/debian/pool/main/x/xrdp && \
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
" | patch -p0

# Install package for fltk, TigerVNC and XRDP.
# Remove package for building fltk and TigerVNC.
RUN apt-get remove -y wget dpkg-dev git patch

# Disable resolvconf postinst to /etc/resolv.conf which cannot be
# updated in container.
# https://github.com/docker/docker/issues/1297#issuecomment-115458690
RUN \
echo "resolvconf resolvconf/linkify-resolvconf boolean false" | \
debconf-set-selections

# Install desktop.
RUN case "${desktop}" in \
gnome) \
  apt-get install -y task-gnome-desktop && \
  service dbus start && \
  su - ${user} -c "dbus-launch gsettings set \
  org.gnome.desktop.lockdown disable-lock-screen true" && \
  su - ${user} -c "dbus-launch gsettings set \
  org.gnome.desktop.session idle-delay 0" && \
  service dbus stop;; \
xfce) \
  apt-get install -y task-xfce-desktop && \
  echo "mode: off\n" > /home/${user}/.xscreensaver && \
  chown ${user}:${user} /home/${user}/.xscreensaver;; \
mate) \
  apt-get install -y task-mate-desktop && \
  service dbus start && \
  su - ${user} -c "dbus-launch gsettings set \
  org.mate.screensaver idle-activation-enabled false" && \
  su - ${user} -c "dbus-launch gsettings set \
  org.mate.screensaver lock-enabled false" && \
  service dbus stop;; \
lxde) \
  # Disable wicd-daemon dialog.
  echo "\
Package: wicd-daemon\n\
Pin: version *\n\
Pin-Priority: -1\n\
" > /etc/apt/preferences.d/wicd-daemon && \
  apt-mark hold wicd && \
  apt-get install -y task-lxde-desktop && \
  echo "mode: off\n" > /home/${user}/.xscreensaver && \
  chown ${user}:${user} /home/${user}/.xscreensaver;; \
kde) \
  apt-get install -y task-kde-desktop;; \
esac

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
