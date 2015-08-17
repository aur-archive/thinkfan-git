# Maintainer: SSF <punx69 at gmx dot net>

pkgname=thinkfan-git
pkgver=20150410.1856
pkgrel=1
pkgdesc="A simple fan control program (includes both systemd and openrc)"
arch=('i686' 'x86_64')
url="http://thinkfan.sourceforge.net"
license=('GPL3')
provides=("thinkfan=$pkgver")
makedepends=('git' 'cmake' 'make' 'sed')
optdepends=('libatasmart: read HDD temperatures')
source=("$pkgname::git://git.code.sf.net/p/thinkfan/code#branch=master")
md5sums=('SKIP')
###wxtradeps
if (pacman -Q libatasmart >/dev/null); then
  depends+=('libatasmart')
fi
if (pacman -Q openrc-core >/dev/null); then
  depends+=('openrc-core')
else
  depends+=('systemd')
fi


pkgver() {
	#doesn't work using date instead
	#cd "$srcdir/$pkgname"
	#ver=$(git describe --long|sed -e 's/\([^-]*-g\)/r\1/;s/-/./g')
	printf "$(date -u +%Y%m%d.%H%M)"
}

build() {
	cd "$srcdir/$pkgname"
###thats weird...
	cp src/* .
	if (pacman -Q libatasmart >/dev/null); then
		msg "building against libatasmart"
		cmake -DUSE_ATASMART:BOOL=ON -DCMAKE_INSTALL_PREFIX:PATH=/usr .
	else
		msg "building without libatasmart"
		cmake -DUSE_ATASMART:BOOL=OFF -DCMAKE_INSTALL_PREFIX:PATH=/usr . 
	fi
	make
}

package() {
	cd "$srcdir"/"$pkgname"
	mkdir -p ${pkgdir}/etc/default
###don't overwrite configs
	if [ -f /etc/thinkfan.conf ]; then
		cp /etc/thinkfan.conf ${pkgdir}/etc/thinkfan.conf
	else
		cp examples/thinkfan.conf.simple ${pkgdir}/etc/thinkfan.conf
	fi
	if [ -f /etc/default/thinkfan ]; then
		cp /etc/default/thinkfan ${pkgdir}/etc/default/thinkfan
	else
		cp rcscripts/thinkfan.default ${pkgdir}/etc/default/thinkfan
	fi
	make install DESTDIR=${pkgdir}
###since sbin is a symlink...
	mkdir -p ${pkgdir}/usr/bin
	mv ${pkgdir}/usr/sbin/thinkfan ${pkgdir}/usr/bin/thinkfan
	rm -df ${pkgdir}/usr/sbin
	if (pacman -Q openrc-core >/dev/null); then
		sed -i 's|#!/sbin/runscript|#!/usr/bin/openrc-run|' rcscripts/thinkfan.gentoo
		install -Dm744 rcscripts/thinkfan.gentoo ${pkgdir}/etc/init.d/thinkfan
	else
		install -Dm644 rcscripts/thinkfan.service ${pkgdir}/usr/lib/systemd/system/thinkfan.service
	fi
	mkdir -p ${pkgdir}/usr/share/licenses/thinkfan
	mv COPYING ${pkgdir}/usr/share/licenses/thinkfan
###add some hints
	cat << EOF

############################	
# How to get this working? #
############################

#1: load the thinkpad_acpi kernel module with the fan control option eg:

modprobe thinkpad_acpi fan_control=1

#2: Edit the config file /etc/thinkfan.conf
(there is also a complex libatasmart example included in
/usr/share/doc/thinkfan)

#3: Test it by launching/starting the thinkfan service(rc-service, etc)

#4: If everything works add #1 to your /etc/modprobe.d and
enable thinkfan at boot (/etc/default/thinkfan) eg:

# printf "options thinkpad_acpi fan_control=1" > /etc/modprobe.d/thinkfan.conf
# sed -i 's#START=no#START=yes#' /etc/default/thinkfan

You also might need to add it to your init system!(rc-update, etc)

Press any key to continue...
EOF
read stopme
}
