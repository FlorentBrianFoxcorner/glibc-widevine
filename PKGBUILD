# Maintainer:  Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc
# NOTE: valgrind requires rebuilt with each major glibc version

# ALARM: Kevin Mihelich <kevin@archlinuxarm.org>
#  - Specify our build host type
#  - Disabled distcc
#  - Strip out Arch x86 multilib
#  - Don't --enable-static-pie, broken on ARM
#  - Don't --enable-cet, x86 only

noautobuild=1

pkgname=glibc-widevine
provides=("glibc=2.33")
conflicts=("glibc")
pkgver=2.33
pkgrel=5
arch=('armv6h' 'armv7h' 'aarch64')
url='https://www.gnu.org/software/libc'
license=(GPL LGPL)
makedepends=(bison git gd python)
optdepends=('perl: for mtrace')
options=(!strip staticlibs !distcc)
#_commit=3de512be7ea6053255afed6154db9ee31d4e557a
#source=(git+https://sourceware.org/git/glibc.git#commit=$_commit
source=(https://ftp.gnu.org/gnu/glibc/glibc-$pkgver.tar.xz{,.sig}
        locale.gen.txt
        locale-gen
        sdt.h sdt-config.h
        bz27343.patch
        0001-nptl_db-Support-different-libpthread-ld.so-load-orde.patch
        0002-nptl-Check-for-compatible-GDB-in-nptl-tst-pthread-gd.patch
        0003-nptl-Do-not-build-nptl-tst-pthread-gdb-attach-as-PIE.patch
	glibc-tls-libwidevinecdm.so-since-4.10.2252.0-has-TLS-with.patch
        glibc-add-support-for-SHT_RELR-sections.patch)
validpgpkeys=(7273542B39962DF7B299931416792B4EA25340F8 # Carlos O'Donell
              BC7C7372637EC10C57D7AA6579C43DFBF1CF2187 # Siddhesh Poyarekar
              16792B4EA25340F8)
md5sums=('390bbd889c7e8e8a7041564cb6b27cca'
         'SKIP'
         '07ac979b6ab5eeb778d55f041529d623'
         '476e9113489f93b348b21e144b6a8fcf'
         '91fec3b7e75510ae2ac42533aa2e695e'
         '680df504c683640b02ed4a805797c0b2'
         'cfe57018d06bf748b8ca1779980fef33'
         '78f041fc66fee4ee372f13b00a99ff72'
         '9e418efa189c20053e887398df2253cf'
         '7a09f1693613897add1791e7aead19c9'
         'fb479854d8f481302edd8f4d3a3b6179'
         '3360d652ea425a60eae248da7d4c230d')

prepare() {
  mkdir -p glibc-build

  [[ -d glibc-$pkgver ]] && ln -s glibc-$pkgver glibc 
  cd glibc

  # commit c3479fb7939898ec22c655c383454d6e8b982a67
  patch -p1 -i "$srcdir"/bz27343.patch

  # nptl_db: Support different libpthread/ld.so load orders (bug 27744)
  patch -p1 -i "$srcdir"/0001-nptl_db-Support-different-libpthread-ld.so-load-orde.patch

  # nptl: Check for compatible GDB in nptl/tst-pthread-gdb-attach
  patch -p1 -i "$srcdir"/0002-nptl-Check-for-compatible-GDB-in-nptl-tst-pthread-gd.patch

  # nptl: Do not build nptl/tst-pthread-gdb-attach as PIE
  patch -p1 -i "$srcdir"/0003-nptl-Do-not-build-nptl-tst-pthread-gdb-attach-as-PIE.patch

  # sys-libs: Add support for SHT_RELR sections
  patch -p1 -i "$srcdir"/glibc-add-support-for-SHT_RELR-sections.patch

  # dl-tls: libwidevinecdm 64Byte alignment
  patch -p1 -i "$srcdir"/glibc-tls-libwidevinecdm.so-since-4.10.2252.0-has-TLS-with.patch

}

build() {
  local _configure_flags=(
      --prefix=/usr
      --with-headers=/usr/include
      --with-bugurl=https://github.com/archlinuxarm/PKGBUILDs/issues
      --enable-add-ons
      --enable-bind-now
      --enable-lock-elision
      --disable-multi-arch
      --enable-stack-protector=strong
      --enable-stackguard-randomization
      --enable-systemtap
      --disable-profile
      --disable-werror
  )

  cd "$srcdir/glibc-build"

  # ALARM: Specify build host types
  [[ $CARCH == "arm" ]] && CONFIGFLAG="--host=armv5tel-unknown-linux-gnueabi --build=armv5tel-unknown-linux-gnueabi"
  [[ $CARCH == "armv6h" ]] && CONFIGFLAG="--host=armv6l-unknown-linux-gnueabihf --build=armv6l-unknown-linux-gnueabihf"
  [[ $CARCH == "armv7h" ]] && CONFIGFLAG="--host=armv7l-unknown-linux-gnueabihf --build=armv7l-unknown-linux-gnueabihf"
  [[ $CARCH == "aarch64" ]] && CONFIGFLAG="--host=aarch64-unknown-linux-gnu --build=aarch64-unknown-linux-gnu"

  echo "slibdir=/usr/lib" >> configparms
  echo "rtlddir=/usr/lib" >> configparms
  echo "sbindir=/usr/bin" >> configparms
  echo "rootsbindir=/usr/bin" >> configparms

  # remove fortify for building libraries
  CPPFLAGS=${CPPFLAGS/-D_FORTIFY_SOURCE=2/}

  #
  CFLAGS=${CFLAGS/-fno-plt/}
  CXXFLAGS=${CXXFLAGS/-fno-plt/}
  LDFLAGS=${LDFLAGS/,-z,now/}

  CFLAGS=${CFLAGS/-Wp,-D_FORTIFY_SOURCE=2/}
  CXXFLAGS=${CXXFLAGS/-Wp,-D_FORTIFY_SOURCE=2/}

  "$srcdir/glibc/configure" \
      --libdir=/usr/lib \
      --libexecdir=/usr/lib \
      ${_configure_flags[@]} \
      $CONFIGFLAG

  # build libraries with fortify disabled
  echo "build-programs=no" >> configparms
  make

  # re-enable fortify for programs
  sed -i "/build-programs=/s#no#yes#" configparms

  echo "CC += -D_FORTIFY_SOURCE=2" >> configparms
  echo "CXX += -D_FORTIFY_SOURCE=2" >> configparms
  make

  # build info pages manually for reprducibility
  make info
}

check() {
  cd glibc-build

  # remove fortify in preparation to run test-suite
  sed -i '/FORTIFY/d' configparms

  # some failures are "expected"
  make check || true
}

package() {
  pkgdesc='GNU C Library'
  depends=('linux-api-headers>=4.10' tzdata filesystem)
  optdepends=('gd: for memusagestat')
  install=glibc.install
  backup=(etc/gai.conf
          etc/locale.gen
          etc/nscd.conf)

  install -dm755 "$pkgdir/etc"
  touch "$pkgdir/etc/ld.so.conf"

  make -C glibc-build install_root="$pkgdir" install
  rm -f "$pkgdir"/etc/ld.so.{cache,conf}

  # Shipped in tzdata
  rm -f "$pkgdir"/usr/bin/{tzselect,zdump,zic}

  cd glibc

  install -dm755 "$pkgdir"/usr/lib/{locale,systemd/system,tmpfiles.d}
  install -m644 nscd/nscd.conf "$pkgdir/etc/nscd.conf"
  install -m644 nscd/nscd.service "$pkgdir/usr/lib/systemd/system"
  install -m644 nscd/nscd.tmpfiles "$pkgdir/usr/lib/tmpfiles.d/nscd.conf"
  install -dm755 "$pkgdir/var/db/nscd"

  install -m644 posix/gai.conf "$pkgdir"/etc/gai.conf

  install -m755 "$srcdir/locale-gen" "$pkgdir/usr/bin"

  # Create /etc/locale.gen
  install -m644 "$srcdir/locale.gen.txt" "$pkgdir/etc/locale.gen"
  sed -e '1,3d' -e 's|/| |g' -e 's|\\| |g' -e 's|^|#|g' \
    "$srcdir/glibc/localedata/SUPPORTED" >> "$pkgdir/etc/locale.gen"

  # ALARM: symlink ld-linux.so.3 for hard-float
  [[ $CARCH == "armv6h" || $CARCH == "armv7h" ]] && ln -s /lib/ld-${pkgver}.so ${pkgdir}/usr/lib/ld-linux.so.3

  if check_option 'debug' n; then
    find "$pkgdir"/usr/bin -type f -executable -exec strip $STRIP_BINARIES {} + 2> /dev/null || true
    find "$pkgdir"/usr/lib -name '*.a' -type f -exec strip $STRIP_STATIC {} + 2> /dev/null || true

    # Do not strip these for gdb and valgrind functionality, but strip the rest
    find "$pkgdir"/usr/lib \
      -not -name 'ld-*.so' \
      -not -name 'libc-*.so' \
      -not -name 'libpthread-*.so' \
      -not -name 'libthread_db-*.so' \
      -name '*-*.so' -type f -exec strip $STRIP_SHARED {} + 2> /dev/null || true
  fi

  # Provide tracing probes to libstdc++ for exceptions, possibly for other
  # libraries too. Useful for gdb's catch command.
  install -Dm644 "$srcdir/sdt.h" "$pkgdir/usr/include/sys/sdt.h"
  install -Dm644 "$srcdir/sdt-config.h" "$pkgdir/usr/include/sys/sdt-config.h"

  # Provided by libxcrypt; keep the old shared library for backwards compatibility
  rm -f "$pkgdir"/usr/include/crypt.h "$pkgdir"/usr/lib/libcrypt.{a,so}
}
