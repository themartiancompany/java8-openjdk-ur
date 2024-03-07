# SPDX-License-Identifier: AGPL-3.0
#
# Maintainer:  Levente Polyak <anthraxx[at]archlinux[dot]org>
# Maintainer:  Frederik Schwan <freswa at archlinux dot org>
# Maintainer:  Guillaume ALAUX <guillaume@archlinux.org>
# Maintainer:  Pellegrino Prevete (tallero) <pellegrinoprevete@gmail.com>
# Maintainer:  Truocolo <truocolo@aol.com>
# Contributor: Boyan Ding <stu_dby@126.com>

_lang="java"
_majver=8
_minver=402
_updatever=06
pkgver=${_majver}.${_minver}.u${_updatever}
_tagver="${_minver}-b${_updatever}"
_sdk="jdk"
_runtime="jre"
_proj="open${_sdk}"
_Proj="OpenJDK Java ${_majver}"
_pkg="${_sdk}${_majver}u"
_pkgname="${_proj}${_majver}"
_pkgbase="${_lang}-${_majver}-${_proj}"
pkgbase="${_lang}${_majver}-${_proj}"
pkgname=(
  "${_runtime}${_majver}-${_proj}-headless"
  "${_runtime}${_majver}-${_proj}"
  "${_sdk}${_majver}-${_proj}"
  "${_pkgname}-src"
  "${_pkgname}-doc"
)
pkgrel=1
arch=(
  'x86_64'
  'arm'
  'armv7l'
  'aarch64'
  'i686'
)
url="https://${_proj}.lang.net"
_gh="https://github.com"
_ns="${_proj}"
_url="${_gh}/${_ns}/${_pkg}"
license=(
  'custom'
)
makedepends=(
  'alsa-lib'
  'bash'
  'ccache'
  'cpio'
  'fontconfig'
  'giflib'
  'git'
  "${_lang}-environment=${_majver}"
  'libcups'
  'libxrender'
  'libxtst'
  'unzip'
  'zip'
)
options=(
  !lto
)
source=(
  "${_url}/archive/refs/tags/${_pkg}${_tagver}.tar.gz"
  gcc11.patch
)
b2sums=(
  'dee05e214756da4d1dcce0f923a0c10b9e385b5945689039c370ae8ac60f3e1324c629c24d9194f63471430b3c94680f0dcb2c3bdfd13d1e2034673cf9123cae'
  '9679e4dfb6027a87376081489c09810812d6849573afac4ea96abe3a3e00ca5b6af7d0ffb010c43b93cfa913f9e97fbb9f11e19fcc86a89b4548442671c32da1'
)

case "${CARCH}" in
  'x86_64') \
    _JARCH=amd64 ; \
    _DOC_ARCH=x86_64 ;;
  'i686'  ) \
    _JARCH=i386  ; \
    _DOC_ARCH=x86 ;;
  'arm') \
    _JARCH=armv7l ; \
    _DOC_ARCH=armv7l ;;
esac
_jvmdir="/usr/lib/jvm/${_pkgbase}"
_prefix="${_pkg}/image"
_imgdir="${_prefix}/jvm/${_proj}-1.8.0_$( \
  printf \
    '%.2d' \
    "${_minver}")"
_nonheadless=(
  bin/policytool
  lib/${_JARCH}/libjsound.so
  lib/${_JARCH}/libjsoundalsa.so
  lib/${_JARCH}/libsplashscreen.so
)
prepare() {
  cd \
    "${_pkg}-${_pkg}${_tagver}"
  # Fix build with C++17 (Fedora)
  patch \
    -Np1 \
    -i \
    "${srcdir}"/gcc11.patch
}

build() {
  local \
    _configure_opts=() \
    _cflags=() \
    _cxxflags=()
  cd \
    "${_pkg}-${_pkg}${_tagver}"
  unset \
    JAVA_HOME
  # http://icedtea.classpath.org/bugzilla/show_bug.cgi?id=1346
  export \
    MAKEFLAGS=${MAKEFLAGS/-j*}
  # Avoid optimization of HotSpot being lowered from O3 to O2
  # -fno-exceptions for FS#73134
  _cflags=(
    "${CFLAGS//-O2/-O3}"
    -Wno-error=nonnull
    -Wno-error=deprecated-declarations
    -Wno-error=stringop-overflow=
    -Wno-error=return-type
    -Wno-error=cpp
    -fno-lifetime-dse
    -fno-delete-null-pointer-checks
    -fcommon
    -fno-exceptions
    -Wno-error=format-overflow=
  )
  _cxxflags=(
    "${CXXFLAGS}"
    -fcommon
    -fno-exceptions
  )
  export \
    CFLAGS="${_cflags[*]}" \
    CXXFLAGS="${_cxxflags[*]}"
  _configure_opts=(
    --prefix="${srcdir}/${_prefix}"
    --with-update-version="${_minver}"
    --with-build-number="b${_updatever}"
    --with-milestone="fcs"
    --enable-unlimited-crypto
    --with-giflib=system
    --with-zlib=system
    --with-extra-cflags="${_cflags[*]}"
    --with-extra-cxxflags="${_cxxflags[*]}"
    --with-extra-ldflags="${LDFLAGS}"
  )
  install \
    -dm 755 \
    "${srcdir}/${_prefix}"
  bash \
    configure \
    "${_configure_opts[@]}"
  # These help to debug builds: 
  # LOG=trace HOTSPOT_BUILD_JOBS=1
  # Without 'DEBUG_BINARIES', i686 won't build: 
  # http://mail.openjdk.java.net/pipermail/core-libs-dev/2013-July/019203.html
  make
  make \
    docs
  # FIXME sadly 'DESTDIR' is not used here!
  make \
    install
  cd \
    ../${_imgdir}
  # A lot of build stuff were directly taken from
  # http://pkgs.fedoraproject.org/cgit/java-1.8.0-openjdk.git/tree/java-1.8.0-openjdk.spec
  # http://icedtea.classpath.org/bugzilla/show_bug.cgi?id=1437
  find \
    . \
    -iname \
      '*.jar' \
    -exec \
      chmod \
        ugo+r \
        {} \;
  chmod \
    ugo+r \
    lib/ct.sym
  # remove redundant *diz and *debuginfo files
  find \
    . \
    -iname \
      '*.diz' \
    -exec \
      rm \
        {} \;
  find \
    . \
    -iname \
      '*.debuginfo' \
    -exec \
      rm \
        {} \;
}

check() {
  cd \
    "${_pkg}-${_pkg}${_tagver}"
  # make \
  #   -k \
  #   test
}

package_jre8-openjdk-headless() {
  local \
    _f \
    _file
  pkgdesc="${_Proj} headless runtime environment"
  depends=(
    "${_lang}-runtime-common"
    'ca-certificates-utils'
    'nss'
  )
  optdepends=(
    "${_lang}-rhino: for some JavaScript support"
  )
  provides=(
    "${_lang}-runtime-headless=${_majver}"
    "${_lang}-runtime-headless-${_proj}=${_majver}"
  )
  # Upstream config files that should go to etc and get backup
  _backup_etc=(
    etc/${_pkgbase}/${_JARCH}/jvm.cfg
    etc/${_pkgbase}/calendars.properties
    etc/${_pkgbase}/content-types.properties
    etc/${_pkgbase}/flavormap.properties
    etc/${_pkgbase}/images/cursors/cursors.properties
    etc/${_pkgbase}/logging.properties
    etc/${_pkgbase}/management/jmxremote.access
    etc/${_pkgbase}/management/jmxremote.password
    etc/${_pkgbase}/management/management.properties
    etc/${_pkgbase}/management/snmp.acl
    etc/${_pkgbase}/net.properties
    etc/${_pkgbase}/psfont.properties.ja
    etc/${_pkgbase}/psfontj2d.properties
    etc/${_pkgbase}/security/java.policy
    etc/${_pkgbase}/security/java.security
    etc/${_pkgbase}/sound.properties)
  replaces=(
    "jre${_majver}-${_proj}-headless-wm"
  )
  backup=(
    ${_backup_etc[@]}
  )
  install="install_jre${_majver}-${_proj}-headless.sh"
  cd \
    ${_imgdir}/jre
  install \
    -dm \
      755 \
    "${pkgdir}${_jvmdir}/jre/"
  cp \
    -a \
    bin \
    lib \
    "${pkgdir}${_jvmdir}/jre"
  # FS#52692
  cp \
    ../release \
    "${pkgdir}${_jvmdir}"
  # Set config files
  mv \
    "${pkgdir}${_jvmdir}"/jre/lib/management/jmxremote.password{.template,}
  mv \
    "${pkgdir}${_jvmdir}"/jre/lib/management/snmp.acl{.template,}
  # Remove 'non-headless' lib files
  for _f in "${_nonheadless[@]}"; do
    rm \
      "${pkgdir}${_jvmdir}/jre/${_f}"
  done
  # Man pages
  pushd "${pkgdir}${_jvmdir}/jre/bin"
  install -d -m 755 "${pkgdir}"/usr/share/man/{,ja/}man1/
  for _file in *; do
    if [ -f "${srcdir}/${_imgdir}/man/man1/${_file}.1" ]; then
      install \
        -m 644 \
        "${srcdir}/${_imgdir}/man/man1/${_file}.1" \
        "${pkgdir}/usr/share/man/man1/${_file}-${_jdkname}.1"
    fi
    if [ -f "${srcdir}/${_imgdir}/man/ja/man1/${_file}.1" ]; then
      install \
        -m 644 \
        "${srcdir}/${_imgdir}/man/ja/man1/${_file}.1" \
        "${pkgdir}/usr/share/man/ja/man1/${_file}-${_jdkname}.1"
    fi
  done
  popd
  # Link JKS keystore from ca-certificates-utils
  rm \
    -f \
    "${pkgdir}${_jvmdir}/jre/lib/security/cacerts"
  ln \
    -sf \
    /etc/ssl/certs/java/cacerts \
    "${pkgdir}${_jvmdir}/jre/lib/security/cacerts"
  # Install license
  install \
    -dm \
      755 \
    "${pkgdir}/usr/share/licenses/${pkgbase}/"
  install \
    -m 644 \
    ASSEMBLY_EXCEPTION LICENSE THIRD_PARTY_README \
    "${pkgdir}/usr/share/licenses/${pkgbase}"
  ln \
    -sf \
    "/usr/share/licenses/${pkgbase}" \
    "${pkgdir}/usr/share/licenses/${pkgname}"
  # Move config files that were set 
  # in _backup_etc from ./lib to /etc
  for _file in "${_backup_etc[@]}"; do
    _filepkgpath=${_jvmdir}/jre/lib/${_file#etc/java-8-openjdk/}
    install \
      -Dm \
        644 \
      "${pkgdir}${_filepkgpath}" \
      "${pkgdir}/${_file}"
    ln \
      -sf \
      "/${_file}" \
      "${pkgdir}${_filepkgpath}"
  done
}

package_jre8-openjdk() {
  local \
    _f \
    _file
  pkgdesc="${_Proj} full runtime environment"
  depends=(
    "jre${_majver}-${_proj}-headless=${pkgver}-${pkgrel}"
    'xdg-utils'
    'hicolor-icon-theme'
    'giflib'
  )
  optdepends=(
    'icedtea-web: web browser plugin + Java Web Start'
    'alsa-lib: for basic sound support'
    'gtk2: for the Gtk+ look and feel - desktop usage'
    "${_lang}${_majver}-openjfx: for JavaFX GUI components support"
  )
  provides=(
    "${_lang}-runtime=${_majver}"
    "${_lang}-runtime-${_proj}=${_majver}"
  )
  install="install_jre${_majver}-${_proj}.sh"
  replaces=(
    "jre${_majver}-${_proj}-wm"
  )
  cd \
    "${_imgdir}/jre"
  for _f in "${_nonheadless[@]}"; do
    install \
      -D ${_f} "${pkgdir}${_jvmdir}/jre/${_f}"
  done
  # Man pages
  pushd \
    "${pkgdir}${_jvmdir}/jre/bin"
  install \
    -dm 755 \
    "${pkgdir}"/usr/share/man/{,ja/}man1/
  for _file in *; do
    install \
      -m 644 \
      "${srcdir}/${_imgdir}/man/man1/${_file}.1" \
      "${pkgdir}/usr/share/man/man1/${_file}-${_jdkname}.1"
    install \
      -m 644 \
      "${srcdir}/${_imgdir}/man/ja/man1/${_file}.1" \
      "${pkgdir}/usr/share/man/ja/man1/${_file}-${_jdkname}.1"
  done
  popd
  # Install license
  install \
    -dm 755 \
    "${pkgdir}/usr/share/licenses/${pkgbase}/"
  ln \
    -sf \
    /usr/share/licenses/${pkgbase} \
    "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_jdk8-openjdk() {
  local \
    _b
  pkgdesc="${_Proj} development kit"
  depends=(
    "${_lang}-environment-common"
    "jre${_majver}-${_proj}=${pkgver}-${pkgrel}"
  )
  provides=(
    "${_lang}-environment=${_majver}"
    "${_lang}-environment-${_proj}=${_majver}"
  )
  replaces=(
    "${_sdk}${_majver}-${proj}-wm"
  )
  install="install_${_sdk}${_majver}-${_proj}.sh"
  cd \
    ${_imgdir}
  # Main files
  install \
    -dm 755 \
    "${pkgdir}${_jvmdir}"
  cp \
    -a \
    include \
    lib \
    "${pkgdir}${_jvmdir}"
  # 'bin' files
  pushd \
    bin
  # 'java-rmi.cgi' will be handled separately
  # as it should not be in the PATH and has no man page
  for _b \
    in $( \
      ls | \
        grep \
          -v \
          java-rmi.cgi); do
    if [ -e ../jre/bin/${_b} ]; then
      # Provide a link of the jre binary in the jdk/bin/ directory
      ln \
        -s \
        ../jre/bin/${_b} \
        "${pkgdir}${_jvmdir}/bin/${_b}"
    else
      # Copy binary to jdk/bin/
      install \
        -Dm \
          755 \
          "${_b}" \
          "${pkgdir}${_jvmdir}/bin/${_b}"
      # Copy man page
      if [ -f ../man/man1/${_b}.1 ]; then
        install \
          -Dm 644 \
          "../man/man1/${_b}.1" \
          "${pkgdir}/usr/share/man/man1/${_b}-${_jdkname}.1"
      fi
      if [ -f ../man/ja/man1/${_b}.1 ]; then
        install \
          -Dm 644 \
          "../man/ja/man1/${_b}.1" \
          "${pkgdir}/usr/share/man/ja/man1/${_b}-${_jdkname}.1"
      fi
    fi
  done
  popd
  # Handling 'java-rmi.cgi' separately
  install \
    -Dm 755 \
    bin/java-rmi.cgi \
    "${pkgdir}${_jvmdir}/bin/java-rmi.cgi"
  # link license
  install \
    -dm 755 \
    "${pkgdir}/usr/share/licenses/"
  ln \
    -sf \
    "/usr/share/licenses/${pkgbase}" \
    "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_openjdk8-src() {
  pkgdesc="${_Proj} sources"
  install \
    -D \
    "${srcdir}/${_imgdir}/src.zip" \
    "${pkgdir}${_jvmdir}/src.zip"
}

package_openjdk8-doc() {
  pkgdesc="${_Proj} documentation"
  install \
    -dm 755 \
    "${pkgdir}/usr/share/doc/${pkgbase}"
  cp \
    -r \
    "${_pkg}-${_pkg}${_tagver}/build/linux-${_DOC_ARCH}-normal-server-release/docs/"* \
    "${pkgdir}/usr/share/doc/${pkgbase}"
}

# vim:set sw=2 sts=-1 et:
