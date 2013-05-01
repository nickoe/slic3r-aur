# Contributor: swiftgeek
# Contributor: Eric Anderson <ejona86@gmail.com>
# Maintainer: Nick Østergaard <oe.nick at gmail dot com>

pkgname=slic3r
pkgver=0.9.9
pkgrel=3
pkgdesc="An STL-to-GCODE translator for RepRap 3D printers, aiming to be a modern and fast alternative to Skeinforge"
arch=('any')
url="http://slic3r.org/"
license=('GPL')
depends=('perl' 'perl-moo' 'perl-boost-geometry-utils=0.06' 'perl-math-clipper>=1.17'
         'perl-math-convexhull' 'perl-math-geometry-voronoi'
         'perl-math-planepath' 'perl-math-convexhull-monotonechain'
         'perl-io-stringy' 'perl-encode-locale')
optdepends=('perl-wx: GUI support'
            'perl-growl-gntp: notifications support via growl'
            'perl-net-dbus: notifications support via any dbus-based notifier'
            'perl-xml-sax-expatxs: make AMF parsing faster'
            'perl-xml-sax: Additive Manufacturing File Format (AMF) support')
source=(
    "$pkgname-$pkgver.tar.gz::https://github.com/alexrj/Slic3r/archive/$pkgver.tar.gz"
    'slic3r.desktop'
    'slic3r')
md5sums=('c8142c3a9d9ccbe4808136abbf75537b'
         'cf0130330574a13b4372beb8f241d71e'
         'a30a96504f11c95956dd8ce645b77504')

build() {
  cd "$srcdir/Slic3r-$pkgver"

  export PERL_MM_USE_DEFAULT=0 PERL_AUTOINSTALL="--skipdeps" \
      PERL_MM_OPT="INSTALLDIRS=vendor DESTDIR='$pkgdir'" \
      PERL_MB_OPT="--installdirs vendor --destdir '$pkgdir'" \
      MODULEBUILDRC=/dev/null SLIC3R_NO_AUTO=1

  # Build stage
  /usr/bin/perl Build.PL
  ./Build
  ./Build test
}

package () {
	cd "$srcdir/Slic3r-$pkgver"
	./Build install

  install -d $pkgdir/usr/bin/vendor_perl/var
  install -m 644 var/* $pkgdir/usr/bin/vendor_perl/var/

  install -d $pkgdir/usr/share/applications
  install -m 644 $srcdir/slic3r.desktop $pkgdir/usr/share/applications/

  # Just to have a more sane bin name also, and automagically fix perl LANG
  # problems
  install -m 755 $srcdir/slic3r $pkgdir/usr/bin/
}
