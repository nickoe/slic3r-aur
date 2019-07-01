# vim: filetype=sh
# Maintainer: Nick Østergaard <oe.nick at gmail dot com>
# Contributor: Swift Geek <swifgeek ɐ google m č0m>
# Contributor: olasd

pkgname=slic3r
pkgver=1.3.0
pkgrel=18
pkgdesc="Slic3r is an STL-to-GCODE translator for RepRap 3D printers, aiming to be a modern and fast alternative to Skeinforge."
arch=('i686' 'x86_64' 'armv6' 'armv6h' 'armv7h')
url="http://slic3r.org/"
license=('GPL')
depends=('perl'
         'perl-moo' 'perl-sub-quote' 'perl-math-clipper' 'perl-math-convexhull' 'perl-math-geometry-voronoi' 'perl-math-planepath' 'perl-math-convexhull-monotonechain' 'perl-io-stringy' 'perl-encode-locale' 'perl-extutils-makemaker-aur>=6.82' 'perl-threads-aur>=1.96' 'perl-extutils-parsexs>=3.22' 'boost' 'perl-libwww')
makedepends=('git' 'perl-module-build-withxspp' 'perl-module-build' 'perl-devel-checklib')
optdepends=('perl-wx: GUI support'
            'perl-net-dbus: notifications support via any dbus-based notifier'
            'perl-xml-sax-expatxs: make AMF parsing faster'
            'perl-xml-sax: Additive Manufacturing File Format (AMF) support'
            'perl-wx-glcanvas: support for opengl preview'
            'perl-opengl: support for opengl preview'
            'perl-net-bonjour: support for autodiscovery of printers on network (octoprint)'
            'perl-class-xsaccessor: creating faster accessor methods' )
conflicts=('slic3r-git' 'slic3r-xs' 'slic3r-xs-git')
#Consider uncommenting line below in case of false negative test results.
#BUILDENV+=('!check')
source=("https://github.com/alexrj/Slic3r/archive/$pkgver.tar.gz" 'slic3r.desktop' 'slic3r.pl')
sha256sums=('4fb351c1ae25c213cc6f17022eb62e2f0ce9678c0439916cb536d5cb3503da31'
            'd543870c4807b2a0012e80453216d56c907e92fa9b09bb6cc75a03c86e771d9d'
            'faa4ce4b94533201cd74a48ae2c16eae156b0dae95a8fe4778a7c6b03b67b348')

prepare() {
  cd "$srcdir/Slic3r-$pkgver"
  # Nasty fix for useless Growl dependency ... please post in comments/upstream real fix, if u know one ;)
  sed -i '/Growl/d' Build.PL

  # Nasty fix for useless warning
  sed -i '/^warn \"Running Slic3r under Perl/,+1 s/^/\#/' ./lib/Slic3r.pm

  # Nasty fix for local::lib use
  find . -iregex '.*\.\(pl\|pm\|t\)' -print0 |  xargs -0 -l sed -i -e '/use local::lib/d'

  # GCC8 90f108ae8e7a4315f82e317f2141733418d86a68
  grep -q 'boost/core/noncopyable\.hpp' ./xs/src/libslic3r/GCodeSender.hpp ||
	  sed -i '/#ifdef BOOST_LIBS/a #include <boost\/core\/noncopyable.hpp>' ./xs/src/libslic3r/GCodeSender.hpp
}

build() {
  # Setting these env variables overwrites any command-line-options we don't want...
  export PERL_MM_USE_DEFAULT=1 PERL_AUTOINSTALL=--skipdeps \
    PERL_MM_OPT="INSTALLDIRS=vendor DESTDIR='$pkgdir'" \
    PERL_MB_OPT="--installdirs vendor --destdir '$pkgdir'" \
    MODULEBUILDRC=/dev/null
  export SLIC3R_NO_AUTO="true"
  cd "$srcdir/Slic3r-$pkgver/xs"
  # Dependency check - intended of package maintainer only, for now
  #TODO: make sure that this if actually works when !check inside makepkg.conf and check inside pkgbuild... find last check and check if it has ! in front?. Is check default?
  if [[ " ${BUILDENV[*]} " != *" !check "* ]] || [[ " ${BUILDENV[*]} " == *" !check"*" check "* ]]; then
    msg2 "Checking prerequisites"
    /usr/bin/perl Build.PL --gui || true #TODO: make enough seds so true is not needed
  fi

  warning " ⚠  DO NOT respond to any question with 'yes'. Report a bug in comment instead.\n"
  # Cuz cpan will install fixes to $HOME ... which is not the point of this package

  #warning "Running Slic3r under Perl = 5.16 is not supported nor recommended\nIn case of related to this issues please use ARM repository to get older perl package\n"
  #↑ detect perl 5.16? Sound's commit 9cb6dc768fe187b0324927b5ec787307f36477cd states that only that version is affected. Cannot be done with pkg-config

  # slic3r-xs Build stage
  msg2 "Building Slic3r::XS (1/3)"
  /usr/bin/perl Build.PL
  ./Build
}

check () {
  cd "$srcdir/Slic3r-$pkgver"

  msg2 "Testing Slic3r::XS - (2/3)"
  prove -Ixs/blib/arch -Ixs/blib/lib/ xs/t/

  msg2 "Testing Slic3r (3/3)"
  prove -Ixs/blib/arch -Ixs/blib/lib/ t/
}

package () {
  cd "$srcdir/Slic3r-$pkgver"
  install -d $pkgdir/usr/share/perl5/vendor_perl/
  cp -R "$srcdir/Slic3r-$pkgver/lib/"* $pkgdir/usr/share/perl5/vendor_perl/

  install -d $pkgdir/usr/bin/vendor_perl/
  install -m 755 "$srcdir/Slic3r-$pkgver/slic3r.pl" $pkgdir/usr/bin/vendor_perl/

  #TODO : Do something about utils !
  #install -m 755 $srcdir/$_gitname/utils/*.pl $pkgdir/usr/bin/
  #install -m 755 $srcdir/$_gitname/utils/post-processing/*.pl $pkgdir/usr/bin/

  # ZSH autocompletion
  install -d "${pkgdir}/usr/share/zsh/site-functions"
  install -m 0644 "$srcdir/Slic3r-$pkgver/utils/zsh/functions/_slic3r" "$pkgdir/usr/share/zsh/site-functions/_slic3r.zsh"

  # Icons " current Build.PL is not really geared for installation "
  install -d $pkgdir/usr/bin/vendor_perl/var/solarized
  install -m 644 "$srcdir/Slic3r-$pkgver/var"/*.*  $pkgdir/usr/bin/vendor_perl/var/
  if [ -d "$srcdir/Slic3r-$pkgver/var/solarized" ]; then
    install -m 644 "$srcdir/Slic3r-$pkgver/var/solarized/*.*"  $pkgdir/usr/bin/vendor_perl/var/solarized
  fi

  # Desktop icon
  install -d $pkgdir/usr/share/applications
  install -m 644 $srcdir/slic3r.desktop $pkgdir/usr/share/applications/

  # Welcome ultimate ugly - u² hack TODO: This may be not needed anymore!
  install -m 755 $srcdir/slic3r.pl $pkgdir/usr/bin/slic3r.pl

  # SLIC3R-XS
  cd "$srcdir/Slic3r-$pkgver/xs"
  ./Build install
}

