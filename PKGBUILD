# Maintainer: Noah Vogt (noahvogt) <noah@noahvogt.com>
# Maintainer: Levente Polyak <anthraxx[at]archlinux[dot]org>
# Maintainer: Guillaume ALAUX <guillaume@archlinux.org>

# TODO add test, see about packaging jtreg and using it here

pkgbase=java-openjdk-xdg
pkgname=('jre-openjdk-headless-xdg' 'jre-openjdk-xdg' 'jdk-openjdk-xdg' 'openjdk-src-xdg' 'openjdk-doc-xdg')
_majorver=22
_minorver=0
_securityver=2
_updatever=36
# pkgver=${_majorver}.${_minorver}.${_securityver}.u${_updatever}
pkgver=${_majorver}.u${_updatever}
pkgrel=1
# _git_tag=jdk-${_majorver}.${_minorver}.${_securityver}+${_updatever}
_git_tag=jdk-${_majorver}+${_updatever}
arch=('x86_64')
url='https://openjdk.java.net/'
license=('LicenseRef-Java')
makedepends=('java-environment>=17' 'cpio' 'unzip' 'zip' 'libelf' 'libcups' 'libx11'
             'libxrender' 'libxtst' 'libxt' 'libxext' 'libxrandr' 'alsa-lib' 'pandoc'
             'graphviz' 'freetype2' 'libjpeg-turbo' 'giflib' 'libpng' 'lcms2'
             'libnet' 'bash' 'harfbuzz' 'gcc-libs' 'glibc')
source=(https://github.com/openjdk/jdk${_majorver}u/archive/${_git_tag}.tar.gz
        freedesktop-java.desktop
        freedesktop-jconsole.desktop
        freedesktop-jshell.desktop
        xdg-basedir-compliant-fontconfig.patch
        xdg-basedir-compliant-userPrefs.patch)
sha256sums=('19cbda061fa41860fa2251f0994e7792c06aec63c8d0ae650353c850be5a8a4c'
            '228fb453e6c652baad71abf734430cda08c287cb8df935ad3ad6d2e9346c7fdf'
            'ed9e43756f450ca01647c495070044276ee9fa7810eb90c99d7e2a29c4a61ef2'
            '93697b752739c1f233cf98f3fa3b945fc775de4d40a31dd21afccda7d0c9d01e'
            '25860396475759236e0edf66711b842143b0ddee47eed61e080da158bbc58ce9'
            '48f9e40c4ae8eb79d17fb676893a89b95ac43616827725a9d10de2b1f357642c')
provides=('jre-openjdk-headless' 'jre-openjdk' 'jdk-openjdk' 'openjdk-src' 'openjdk-doc')
conflicts=('jre-openjdk-headless' 'jre-openjdk' 'jdk-openjdk' 'openjdk-src' 'openjdk-doc')

case "${CARCH}" in
  x86_64) _JARCH='x86_64';;
  i686)   _JARCH='x86';;
esac

_jvmdir=/usr/lib/jvm/java-${_majorver}-openjdk
_jdkdir=jdk${_majorver}u-${_git_tag//+/-}
_imgdir=${_jdkdir}/build/linux-${_JARCH}-server-release/images

_nonheadless=(lib/libawt_xawt.so
              lib/libjawt.so
              lib/libjsound.so
              lib/libsplashscreen.so)

_commondeps=('java-runtime-common>=3' 'ca-certificates-utils' 'nss' 'libjpeg-turbo' 'libjpeg.so'
           'lcms2' 'liblcms2.so' 'libnet' 'freetype2' 'libfreetype.so' 'harfbuzz' 'libharfbuzz.so'
           'glibc' 'gcc-libs')

prepare() {
  cd ${_jdkdir}
  patch -p1 -i ../xdg-basedir-compliant-fontconfig.patch
  patch -p1 -i ../xdg-basedir-compliant-userPrefs.patch
}

build() {
  cd ${_jdkdir}

  NUM_PROC_OPT=''
  MAKEFLAG_J=$(echo ${MAKEFLAGS} | sed -En 's/.*-j([0-9]+).*/\1/p')
  if [ -n "${MAKEFLAG_J}" ]; then
    # http://hg.openjdk.java.net/jdk10/jdk10/file/85e6cb013b98/make/InitSupport.gmk#l105
    echo "Removing '-j${MAKEFLAG_J}' from MAKEFLAGS to prevent build fail. Passing it directly to ./configure."
    export MAKEFLAGS=${MAKEFLAGS/-j${MAKEFLAG_J}/}
    NUM_PROC_OPT="--with-num-cores=${MAKEFLAG_J}"
  fi

  # Avoid optimization of HotSpot to be lowered from O3 to O2
  local _CFLAGS="${CFLAGS//-O2/-O3} ${CPPFLAGS} -fcommon"
  local _CXXFLAGS="${CXXFLAGS//-O2/-O3} ${CPPFLAGS} -fcommon"
  local _LDFLAGS=${LDFLAGS}
  if [[ ${CARCH} = i686 ]]; then
    echo "Removing '-fno-plt' from CFLAGS and CXXFLAGS to prevent build fail with this architecture"
    _CFLAGS=${CFLAGS/-fno-plt/}
    _CXXFLAGS=${CXXFLAGS/-fno-plt/}
  fi

  # TODO: Should be rechecked for the next releases
  # compiling with -fexceptions leads to:
  # /usr/bin/ld: /build/java-openjdk/src/jdk17u-jdk-17.0.3-2/build/linux-x86_64-server-release/hotspot/variant-server/libjvm/objs/zPhysicalMemory.o: in function `ZList<ZMemory>::~ZList()':
  # /build/java-openjdk/src/jdk17u-jdk-17.0.3-2/src/hotspot/share/gc/z/zList.hpp:54: undefined reference to `ZListNode<ZMemory>::~ZListNode()'
  # collect2: error: ld returned 1 exit status
  _CFLAGS=${CFLAGS/-fexceptions/}
  _CXXFLAGS=${CXXFLAGS/-fexceptions/}

  # CFLAGS, CXXFLAGS and LDFLAGS are ignored as shown by a warning
  # in the output of ./configure unless used like such:
  #  --with-extra-cflags="${CFLAGS}"
  #  --with-extra-cxxflags="${CXXFLAGS}"
  #  --with-extra-ldflags="${LDFLAGS}"
  # See also paragraph "Configure Control Variables from "jdk${_majorver}-${_git_tag}/common/doc/building.md
  unset CFLAGS
  unset CXXFLAGS
  unset LDFLAGS

  if check_option "lto" "y"; then
    jvm_features="zgc,shenandoahgc,link-time-opt"
  else
    jvm_features="zgc,shenandoahgc"
  fi


  bash configure \
    --with-version-build="${_updatever}" \
    --with-version-pre="" \
    --with-version-opt="" \
    --with-stdc++lib=dynamic \
    --with-extra-cflags="${_CFLAGS}" \
    --with-extra-cxxflags="${_CXXFLAGS}" \
    --with-extra-ldflags="${_LDFLAGS}" \
    --with-libjpeg=system \
    --with-giflib=system \
    --with-libpng=system \
    --with-lcms=system \
    --with-zlib=system \
    --with-harfbuzz=system \
    --with-jvm-features="${jvm_features}" \
    --with-native-debug-symbols=internal \
    --enable-unlimited-crypto \
    --disable-warnings-as-errors \
    ${NUM_PROC_OPT}
    #--disable-javac-server

  make images legacy-jre-image docs

  # https://bugs.openjdk.java.net/browse/JDK-8173610
  find "${srcdir}/${_imgdir}" -iname '*.so' -exec chmod +x {} \;
}

check() {
  cd ${_jdkdir}
  # TODO package jtreg
  # make -k check
}

package_jre-openjdk-headless-xdg() {
  pkgdesc="OpenJDK Java ${_majorver} headless runtime environment - with improved Support for the XDG Base Directory Specification"
  depends=("${_commondeps[@]}")
  optdepends=('java-rhino: for some JavaScript support')
  provides=("java-runtime-headless=${_majorver}" "java-runtime-headless-openjdk=${_majorver}" "jre${_majorver}-openjdk-headless=${pkgver}-${pkgrel}")
  conflicts=("jdk-openjdk" "jre-openjdk")
  backup=(etc/${pkgbase}/logging.properties
          etc/${pkgbase}/management/jmxremote.access
          etc/${pkgbase}/management/jmxremote.password.template
          etc/${pkgbase}/management/management.properties
          etc/${pkgbase}/net.properties
          etc/${pkgbase}/security/java.policy
          etc/${pkgbase}/security/java.security
          etc/${pkgbase}/security/policy/README.txt
          etc/${pkgbase}/security/policy/limited/default_US_export.policy
          etc/${pkgbase}/security/policy/limited/default_local.policy
          etc/${pkgbase}/security/policy/limited/exempt_local.policy
          etc/${pkgbase}/security/policy/unlimited/default_US_export.policy
          etc/${pkgbase}/security/policy/unlimited/default_local.policy
          etc/${pkgbase}/sound.properties)
  install=install_jre-openjdk-headless.sh

  cd ${_imgdir}/jre

  install -dm 755 "${pkgdir}${_jvmdir}"

  cp -a bin lib \
    "${pkgdir}${_jvmdir}"

  for f in "${_nonheadless[@]}"; do
    rm "${pkgdir}${_jvmdir}/${f}"
  done

  # Conf
  install -dm 755 "${pkgdir}/etc"
  cp -r conf "${pkgdir}/etc/${pkgbase}"
  ln -s /etc/${pkgbase} "${pkgdir}/${_jvmdir}/conf"

  # Legal
  install -dm 755 "${pkgdir}/usr/share/licenses"
  cp -r legal "${pkgdir}/usr/share/licenses/${pkgbase}"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
  ln -s /usr/share/licenses/${pkgbase} "${pkgdir}/${_jvmdir}/legal"

  # Man pages
  for f in bin/*; do
    f=$(basename "${f}")
    _man=../jdk/man/man1/"${f}.1"
    test -f "${_man}" && install -Dm 644 "${_man}" "${pkgdir}/usr/share/man/man1/${f}-openjdk${_majorver}.1"
  done
  ln -s /usr/share/man "${pkgdir}/${_jvmdir}/man"

  # Link JKS keystore from ca-certificates-utils
  rm -f "${pkgdir}${_jvmdir}/lib/security/cacerts"
  ln -sf /etc/ssl/certs/java/cacerts "${pkgdir}${_jvmdir}/lib/security/cacerts"
}

package_jre-openjdk-xdg() {
  pkgdesc="OpenJDK Java ${_majorver} full runtime environment - with improved Support for the XDG Base Directory Specification"
  depends=("${_commondeps[@]}" 'giflib' 'libgif.so' 'libpng')
  optdepends=('alsa-lib: for basic sound support'
              'gtk2: for the Gtk+ 2 look and feel - desktop usage'
              'gtk3: for the Gtk+ 3 look and feel - desktop usage')
  provides=("java-runtime=${_majorver}" "java-runtime-openjdk=${_majorver}" "jre${_majorver}-openjdk=${pkgver}-${pkgrel}"
            "java-runtime-headless=${_majorver}" "java-runtime-headless-openjdk=${_majorver}" "jre${_majorver}-openjdk-headless=${pkgver}-${pkgrel}")
  conflicts=("jdk-openjdk" "jre-openjdk-headless")
  backup=(etc/${pkgbase}/logging.properties
          etc/${pkgbase}/management/jmxremote.access
          etc/${pkgbase}/management/jmxremote.password.template
          etc/${pkgbase}/management/management.properties
          etc/${pkgbase}/net.properties
          etc/${pkgbase}/security/java.policy
          etc/${pkgbase}/security/java.security
          etc/${pkgbase}/security/policy/README.txt
          etc/${pkgbase}/security/policy/limited/default_US_export.policy
          etc/${pkgbase}/security/policy/limited/default_local.policy
          etc/${pkgbase}/security/policy/limited/exempt_local.policy
          etc/${pkgbase}/security/policy/unlimited/default_US_export.policy
          etc/${pkgbase}/security/policy/unlimited/default_local.policy
          etc/${pkgbase}/sound.properties)

  install=install_jre-openjdk.sh

  cd ${_imgdir}/jre

  install -dm 755 "${pkgdir}${_jvmdir}"

   cp -a bin lib \
    "${pkgdir}${_jvmdir}"


  for f in "${_nonheadless[@]}"; do
    install -Dm 755 ${f} "${pkgdir}${_jvmdir}/${f}"
  done

  # Conf
  install -dm 755 "${pkgdir}/etc"
  cp -r conf "${pkgdir}/etc/${pkgbase}"
  ln -s /etc/${pkgbase} "${pkgdir}/${_jvmdir}/conf"

  # Legal
  install -dm 755 "${pkgdir}/usr/share/licenses"
  cp -r legal "${pkgdir}/usr/share/licenses/${pkgbase}"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
  ln -s /usr/share/licenses/${pkgbase} "${pkgdir}/${_jvmdir}/legal"

  # Man pages
  for f in bin/*; do
    f=$(basename "${f}")
    _man=../jdk/man/man1/"${f}.1"
    test -f "${_man}" && install -Dm 644 "${_man}" "${pkgdir}/usr/share/man/man1/${f}-openjdk${_majorver}.1"
  done
  ln -s /usr/share/man "${pkgdir}/${_jvmdir}/man"

  # Link JKS keystore from ca-certificates-utils
  rm -f "${pkgdir}${_jvmdir}/lib/security/cacerts"
  ln -sf /etc/ssl/certs/java/cacerts "${pkgdir}${_jvmdir}/lib/security/cacerts"

  # Desktop files
  for f in java; do
    install -Dm 644 \
      "${srcdir}/freedesktop-${f}.desktop" \
      "${pkgdir}/usr/share/applications/${f}-${pkgbase}.desktop"
  done
}

package_jdk-openjdk-xdg() {
  pkgdesc="OpenJDK Java ${_majorver} development kit - with improved Support for the XDG Base Directory Specification"
  depends=("${_commondeps[@]}" 'java-environment-common=3'
           'hicolor-icon-theme' 'libelf' 'libgif.so' 'libpng'
           'ca-certificates-utils' 'nss' 'libjpeg-turbo' 'libjpeg.so'
           'lcms2' 'liblcms2.so' 'libnet' 'freetype2' 'libfreetype.so' 'harfbuzz'
           'libharfbuzz.so')
  optdepends=('java-rhino: for some JavaScript support'
              'alsa-lib: for basic sound support'
              'gtk2: for the Gtk+ 2 look and feel - desktop usage'
              'gtk3: for the Gtk+ 3 look and feel - desktop usage')
  provides=("java-environment=${_majorver}" "java-environment-openjdk=${_majorver}" "jdk${_majorver}-openjdk=${pkgver}-${pkgrel}"
            "java-runtime=${_majorver}" "java-runtime-openjdk=${_majorver}" "jre${_majorver}-openjdk=${pkgver}-${pkgrel}"
            "java-runtime-headless=${_majorver}" "java-runtime-headless-openjdk=${_majorver}" "jre${_majorver}-openjdk-headless=${pkgver}-${pkgrel}")
  conflicts=("jre-openjdk" "jre-openjdk-headless")
  backup=(etc/${pkgbase}/logging.properties
          etc/${pkgbase}/management/jmxremote.access
          etc/${pkgbase}/management/jmxremote.password.template
          etc/${pkgbase}/management/management.properties
          etc/${pkgbase}/net.properties
          etc/${pkgbase}/security/java.policy
          etc/${pkgbase}/security/java.security
          etc/${pkgbase}/security/policy/README.txt
          etc/${pkgbase}/security/policy/limited/default_US_export.policy
          etc/${pkgbase}/security/policy/limited/default_local.policy
          etc/${pkgbase}/security/policy/limited/exempt_local.policy
          etc/${pkgbase}/security/policy/unlimited/default_US_export.policy
          etc/${pkgbase}/security/policy/unlimited/default_local.policy
          etc/${pkgbase}/sound.properties)

  install=install_jdk-openjdk.sh

  cd ${_imgdir}/jdk

  install -dm 755 "${pkgdir}${_jvmdir}"

  cp -a bin demo include jmods lib release \
    "${pkgdir}${_jvmdir}"

  rm "${pkgdir}${_jvmdir}/lib/src.zip"

  # Conf
  install -dm 755 "${pkgdir}/etc"
  cp -r conf "${pkgdir}/etc/${pkgbase}"
  ln -s /etc/${pkgbase} "${pkgdir}/${_jvmdir}/conf"

  # Legal
  install -dm 755 "${pkgdir}/usr/share/licenses"
  cp -r legal "${pkgdir}/usr/share/licenses/${pkgbase}"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
  ln -s /usr/share/licenses/${pkgbase} "${pkgdir}/${_jvmdir}/legal"

  # Man pages
  for f in bin/*; do
    f=$(basename "${f}")
     _man=../jdk/man/man1/"${f}.1"
    test -f "${_man}" && install -Dm 644 "${_man}" "${pkgdir}/usr/share/man/man1/${f}-openjdk${_majorver}.1"
  done

  ln -s /usr/share/man "${pkgdir}/${_jvmdir}/man"

  # Link JKS keystore from ca-certificates-utils
  rm -f "${pkgdir}${_jvmdir}/lib/security/cacerts"
  ln -sf /etc/ssl/certs/java/cacerts "${pkgdir}${_jvmdir}/lib/security/cacerts"

  # Icons
  for s in 16 24 32 48; do
    install -Dm 644 \
      "${srcdir}/${_jdkdir}/src/java.desktop/unix/classes/sun/awt/X11/java-icon${s}.png" \
      "${pkgdir}/usr/share/icons/hicolor/${s}x${s}/apps/${pkgbase}.png"
  done

  # Desktop files
  for f in jconsole java jshell; do
    install -Dm 644 \
      "${srcdir}/freedesktop-${f}.desktop" \
      "${pkgdir}/usr/share/applications/${f}-${pkgbase}.desktop"
  done
}

package_openjdk-src-xdg() {
  pkgdesc="OpenJDK Java ${_majorver} sources - with improved Support for the XDG Base Directory Specification"
  # Depends on JDK to get license files
  depends=("jdk${_majorver}-openjdk=${pkgver}-${pkgrel}")
  provides=("openjdk${_majorver}-src=${pkgver}-${pkgrel}")

  install -Dm 644 -t "${pkgdir}${_jvmdir}/lib" ${_imgdir}/jdk/lib/src.zip

  install -dm 755 "${pkgdir}/usr/share/licenses"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_openjdk-doc-xdg() {
  pkgdesc="OpenJDK Java ${_majorver} documentation - with improved Support for the XDG Base Directory Specification"
  # Depends on JDK to get license files
  depends=("jdk${_majorver}-openjdk=${pkgver}-${pkgrel}")
  provides=("openjdk${_majorver}-doc=${pkgver}-${pkgrel}")

  install -dm 755 "${pkgdir}/usr/share/doc"
  cp -r ${_imgdir}/docs "${pkgdir}/usr/share/doc/${pkgbase}"

  install -dm 755 "${pkgdir}/usr/share/licenses"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

# vim: ts=2 sw=2 et:
