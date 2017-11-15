# Maintainer: Guillaume ALAUX <guillaume@archlinux.org>
# Contributor: Boyan Ding <stu_dby@126.com>

# TODO
# once icedtea:
#   pulse
#   add policytool desktop files

# Package 'openjfx' must be updated when this one is
pkgname=('jre8-openjdk-headless' 'jre8-openjdk' 'jdk8-openjdk' 'openjdk8-src' 'openjdk8-doc')
pkgbase=java8-openjdk
_java_ver=8
# Found @ http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
_jdk_update=144
_jdk_build=01
pkgver=${_java_ver}.u${_jdk_update}
_repo_ver=jdk${_java_ver}u${_jdk_update}-b${_jdk_build}
pkgrel=1
arch=('x86_64')
url='http://openjdk.java.net/'
license=('custom')
makedepends=('jdk7-openjdk' 'ccache' 'cpio' 'unzip' 'zip'
             'libxrender' 'libxtst' 'fontconfig' 'libcups' 'alsa-lib')
_url_src=http://hg.openjdk.java.net/jdk8u/jdk8u
source=(jdk8u-${_repo_ver}.tar.gz::${_url_src}/archive/${_repo_ver}.tar.gz
        corba-${_repo_ver}.tar.gz::${_url_src}/corba/archive/${_repo_ver}.tar.gz
        hotspot-${_repo_ver}.tar.gz::${_url_src}/hotspot/archive/${_repo_ver}.tar.gz
        jdk-${_repo_ver}.tar.gz::${_url_src}/jdk/archive/${_repo_ver}.tar.gz
        jaxws-${_repo_ver}.tar.gz::${_url_src}/jaxws/archive/${_repo_ver}.tar.gz
        jaxp-${_repo_ver}.tar.gz::${_url_src}/jaxp/archive/${_repo_ver}.tar.gz
        langtools-${_repo_ver}.tar.gz::${_url_src}/langtools/archive/${_repo_ver}.tar.gz
        nashorn-${_repo_ver}.tar.gz::${_url_src}/nashorn/archive/${_repo_ver}.tar.gz
        build_with_gcc6.patch)

sha256sums=('85a8a8c55cda2d3bae833578d6ad6c8bf531550ac051776d47946d17858a6721'
            'f0a153807e08c79c5ce229be6eafe7951854a7d7eb496cea7b24a0a5f305ccfe'
            '18d9fc6b9a6830cb8083e366b9ef31f8c8ebb80c816d7f516de8285181fb6bc1'
            'e8442fd2fcce332abdb1e34bb33655e43e6d88bb551fc67883b8cab043aba53f'
            '5146841822b49e1300c6c78a88be617fd72a80b76e8e2bb8d6ed526edfe1a929'
            'd5479dde109483c81f94abb91d93d11b1a899bb5d5685e7dfdd96ba8e35adf93'
            '7a3a7d2f2f6f47de876532d9ad9e3d04f0235a8d848ea958d6371dbb2535f83c'
            '5d14d0e910fec1cc34e02bf25173b25619083f2afda8e73d0bb5ba456c12b89c'
            'dcf5f495620231068b0c925a33ba7246bbeb85e9ae822b30ab77a66839c2d3b6')

case "${CARCH}" in
  'x86_64') _JARCH=amd64 ; _DOC_ARCH=x86_64 ;;
  'i686'  ) _JARCH=i386  ; _DOC_ARCH=x86    ;;
esac

_jdkname=openjdk8
_jvmdir=/usr/lib/jvm/java-8-openjdk
_prefix="jdk8u-${_repo_ver}/image"
_imgdir="${_prefix}/jvm/openjdk-1.8.0_$(printf '%.2d' ${_jdk_update})"
_nonheadless=(bin/policytool
              lib/${_JARCH}/libjsound.so
              lib/${_JARCH}/libjsoundalsa.so
              lib/${_JARCH}/libsplashscreen.so)

prepare() {
  cd "${srcdir}/jdk8u-${_repo_ver}"

  for subrepo in corba hotspot jdk jaxws jaxp langtools nashorn
  do
    ln -s ../${subrepo}-${_repo_ver} ${subrepo}
  done

  patch -p1 < ../build_with_gcc6.patch
}

build() {
  cd "${srcdir}/jdk8u-${_repo_ver}"

  unset JAVA_HOME
  # http://icedtea.classpath.org/bugzilla/show_bug.cgi?id=1346
  export MAKEFLAGS=${MAKEFLAGS/-j*}
  # https://hydra.nixos.org/build/41230444/log
  export CFLAGS="-Wno-error=deprecated-declarations"

  install -d -m 755 "${srcdir}/${_prefix}/"
  sh configure \
    --prefix="${srcdir}/${_prefix}" \
    --with-update-version="${_jdk_update}" \
    --with-build-number="b${_jdk_build}" \
    --with-milestone="fcs" \
    --enable-unlimited-crypto \
    --with-zlib=system

    # TODO OpenJDK does not want last version of giflib (add 'giflib' as dependency once fixed)
    #--with-giflib=system \

  # Without 'DEBUG_BINARIES', i686 won't build
  # http://mail.openjdk.java.net/pipermail/core-libs-dev/2013-July/019203.html
  make \
    DEBUG_BINARIES=true
  # These help to debug builds:
  #LOG=trace HOTSPOT_BUILD_JOBS=1

  make docs

  # FIXME sadly 'DESTDIR' is not used here!
  make install

  cd "${srcdir}/${_imgdir}"

  # A lot of build stuff were directly taken from
  # http://pkgs.fedoraproject.org/cgit/java-1.8.0-openjdk.git/tree/java-1.8.0-openjdk.spec

  # http://icedtea.classpath.org/bugzilla/show_bug.cgi?id=1437
  find . -iname '*.jar' -exec chmod ugo+r {} \;
  chmod ugo+r lib/ct.sym

  # remove redundant *diz and *debuginfo files
  find . -iname '*.diz' -exec rm {} \;
  find . -iname '*.debuginfo' -exec rm {} \;
}

#check() {
#  cd "${srcdir}/${pkgname}-${pkgver}"
#  make -k check
#}

package_jre8-openjdk-headless() {
  pkgdesc='OpenJDK Java 8 headless runtime environment'
  depends=('java-runtime-common' 'ca-certificates-utils' 'nss')
  optdepends=('java-rhino: for some JavaScript support')
  provides=('java-runtime-headless=8' 'java-runtime-headless-openjdk=8')
  # Upstream config files that should go to etc and get backup
  _backup_etc=(etc/java-8-openjdk/${_JARCH}/jvm.cfg
               etc/java-8-openjdk/calendars.properties
               etc/java-8-openjdk/content-types.properties
               etc/java-8-openjdk/flavormap.properties
               etc/java-8-openjdk/images/cursors/cursors.properties
               etc/java-8-openjdk/logging.properties
               etc/java-8-openjdk/management/jmxremote.access
               etc/java-8-openjdk/management/jmxremote.password
               etc/java-8-openjdk/management/management.properties
               etc/java-8-openjdk/management/snmp.acl
               etc/java-8-openjdk/net.properties
               etc/java-8-openjdk/psfont.properties.ja
               etc/java-8-openjdk/psfontj2d.properties
               etc/java-8-openjdk/security/java.policy
               etc/java-8-openjdk/security/java.security
               etc/java-8-openjdk/sound.properties)
  replaces=('jre8-openjdk-headless-wm')
  backup=(${_backup_etc[@]})
  install=install_jre8-openjdk-headless.sh

  cd "${srcdir}/${_imgdir}/jre"

  install -d -m 755 "${pkgdir}${_jvmdir}/jre/"
  cp -a bin lib "${pkgdir}${_jvmdir}/jre"

  # Set config files
  mv "${pkgdir}${_jvmdir}"/jre/lib/management/jmxremote.password{.template,}
  mv "${pkgdir}${_jvmdir}"/jre/lib/management/snmp.acl{.template,}

  # Remove 'non-headless' lib files
  for f in ${_nonheadless[@]}; do
    rm "${pkgdir}${_jvmdir}/jre/${f}"
  done

  # Man pages
  pushd "${pkgdir}${_jvmdir}/jre/bin"
  install -d -m 755 "${pkgdir}"/usr/share/man/{,ja/}man1/
  for file in *; do
    install -m 644 "${srcdir}/${_imgdir}/man/man1/${file}.1" \
      "${pkgdir}/usr/share/man/man1/${file}-${_jdkname}.1"
    install -m 644 "${srcdir}/${_imgdir}/man/ja/man1/${file}.1" \
      "${pkgdir}/usr/share/man/ja/man1/${file}-${_jdkname}.1"
  done
  popd

  # Link JKS keystore from ca-certificates-utils
  rm -f "${pkgdir}${_jvmdir}/jre/lib/security/cacerts"
  ln -sf /etc/ssl/certs/java/cacerts "${pkgdir}${_jvmdir}/jre/lib/security/cacerts"

  # Install license
  install -d -m 755 "${pkgdir}/usr/share/licenses/${pkgbase}/"
  install -m 644 ASSEMBLY_EXCEPTION LICENSE THIRD_PARTY_README \
                 "${pkgdir}/usr/share/licenses/${pkgbase}"
  ln -sf /usr/share/licenses/${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"

  # Move config files that were set in _backup_etc from ./lib to /etc
  for file in ${_backup_etc[@]}; do
    _filepkgpath=${_jvmdir}/jre/lib/${file#etc/java-8-openjdk/}
    install -D -m 644 "${pkgdir}${_filepkgpath}" "${pkgdir}/${file}"
    ln -sf /${file} "${pkgdir}${_filepkgpath}"
  done
}

package_jre8-openjdk() {
  pkgdesc='OpenJDK Java 8 full runtime environment'
  depends=("jre8-openjdk-headless=${pkgver}-${pkgrel}" 'xdg-utils' 'hicolor-icon-theme')
  optdepends=('icedtea-web: web browser plugin + Java Web Start'
              'alsa-lib: for basic sound support'
              'gtk2: for the Gtk+ look and feel - desktop usage'
              'java-openjfx: for JavaFX GUI components support')
  # TODO when adding IcedTea: 'giflib: for gif format support'
  # TODO when adding IcedTea: 'libpulse: for advanced sound support'
  provides=('java-runtime=8' 'java-runtime-openjdk=8')
  install=install_jre8-openjdk.sh
  replaces=('jre8-openjdk-wm')

  cd "${srcdir}/${_imgdir}/jre"

  # TODO? Should /usr/lib/jvm/java-8-openjdk/jre/lib/sound.properties belong to jre?
  for f in ${_nonheadless[@]}; do
    install -D ${f} "${pkgdir}${_jvmdir}/jre/${f}"
  done

  # Man pages
  pushd "${pkgdir}${_jvmdir}/jre/bin"
  install -d -m 755 "${pkgdir}"/usr/share/man/{,ja/}man1/
  for file in *; do
    install -m 644 "${srcdir}/${_imgdir}/man/man1/${file}.1" \
      "${pkgdir}/usr/share/man/man1/${file}-${_jdkname}.1"
    install -m 644 "${srcdir}/${_imgdir}/man/ja/man1/${file}.1" \
      "${pkgdir}/usr/share/man/ja/man1/${file}-${_jdkname}.1"
  done
  popd

  # Desktop files
  # TODO add these when switching to IcedTea
  #install -D -m 644 "${srcdir}/icedtea-${_icedtea_ver}/policytool.desktop" \
  #                  "${pkgdir}/usr/share/applications/policytool.desktop"

  # Install license
  install -d -m 755 "${pkgdir}/usr/share/licenses/${pkgbase}/"
  ln -sf /usr/share/licenses/${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_jdk8-openjdk() {
  pkgdesc='OpenJDK Java 8 development kit'
  depends=('java-environment-common' "jre8-openjdk=${pkgver}-${pkgrel}")
  provides=('java-environment=8' 'java-environment-openjdk=8')
  replaces=('jdk8-openjdk-wm')
  install=install_jdk8-openjdk.sh

  cd "${srcdir}/${_imgdir}"

  # Main files
  install -d -m 755 "${pkgdir}${_jvmdir}"

  cp -a include lib "${pkgdir}${_jvmdir}"

  # 'bin' files
  pushd bin

  # 'java-rmi.cgi' will be handled separately as it should not be in the PATH and has no man page
  for b in $(ls | grep -v java-rmi.cgi); do
    if [ -e ../jre/bin/${b} ]; then
      # Provide a link of the jre binary in the jdk/bin/ directory
      ln -s ../jre/bin/${b} "${pkgdir}${_jvmdir}/bin/${b}"
    else
      # Copy binary to jdk/bin/
      install -D -m 755 ${b} "${pkgdir}${_jvmdir}/bin/${b}"
      # Copy man page
      install -D -m 644 ../man/man1/${b}.1 "${pkgdir}/usr/share/man/man1/${b}-${_jdkname}.1"
      install -D -m 644 ../man/ja/man1/${b}.1 "${pkgdir}/usr/share/man/ja/man1/${b}-${_jdkname}.1"
    fi
  done
  popd

  # Handling 'java-rmi.cgi' separately
  install -D -m 755 bin/java-rmi.cgi "${pkgdir}${_jvmdir}/bin/java-rmi.cgi"

  # Desktop files.
  # TODO add these when switching to IcedTea
  #install -m 644 "${srcdir}/icedtea-${_icedtea_ver}/jconsole.desktop" \
  #  "${pkgdir}/usr/share/applications"

  # link license
  install -d -m 755 "${pkgdir}/usr/share/licenses/"
  ln -sf /usr/share/licenses/${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_openjdk8-src() {
  pkgdesc='OpenJDK Java 8 sources'

  install -D "${srcdir}/${_imgdir}/src.zip" "${pkgdir}${_jvmdir}/src.zip"
}

package_openjdk8-doc() {
  pkgdesc='OpenJDK Java 8 documentation'

  install -d -m 755 "${pkgdir}/usr/share/doc/${pkgbase}/"
  cp -r "${srcdir}"/jdk8u-${_repo_ver}/build/linux-${_DOC_ARCH}-normal-server-release/docs/* \
    "${pkgdir}/usr/share/doc/${pkgbase}/"
}
