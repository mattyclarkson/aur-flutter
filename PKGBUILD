# Maintainer: Fredy García <frealgagu at gmail dot com>
# Contributor: Philip Goto <philip.goto@gmail.com>
# Contributor: Matt Clarkson <mattyclarkson@gmail.com>

channel=stable
pkgname=flutter
pkgver=1.9.1+hotfix.2
pkgrel=2
pkgdesc="A new mobile app SDK to help developers and designers build modern mobile apps for iOS and Android."
arch=("x86_64")
url="https://flutter.io"
license=("custom" "BSD" "CCPL")
depends=("dart" "git")
optdepends=(
  "android-sdk"
  "chromium"
  "dart"
  "intellij-idea-community-edition"
  "clang"
  "make"
)
makedepends=("patch" "dart" "git")
checkdepends=("shellcheck")
conflicts=("flutter-beta" "flutter-dev")
options=("!emptydirs")
source=(
  "${pkgname}-${pkgver}.tar.xz::https://storage.googleapis.com/flutter_infra/releases/${channel}/linux/flutter_linux_v${pkgver}-${channel}.tar.xz"
  "flutter.patch"
  "flutter-tools-executable.patch"
)
sha256sums=(
  "f82875a865c8dbebd10b7a69ffc4cb19d9c916054f3bbcda5a66395f30477d91"
  "5b59ac73256c2e8b788da412eae971cf8913fbaaa81d3221a3be258d8ff8fce2"
  "76b92bacee718be0e84c58e6e8a0123c88908dc81d32ad499b453bfcab151ff4"
)

build() {
  cd "${srcdir}/flutter"

  pkg_dart_version=$("${srcdir}/flutter/bin/cache/dart-sdk/bin/dart" --version)
  opt_dart_version=$("/opt/dart-sdk/bin/dart" --version)

  if [ "${pkg_dart_version}" != "" ]; then
    >&2 echo "Invalid dart versions:\n${pkg_dart_version}\n${opt_dart_version}"
    exit 1
  fi

  rm -rf "${srcdir}/flutter/bin/cache/dart-sdk"
  ln -s /opt/dart-sdk "${srcdir}/flutter/bin/cache/dart-sdk"

  "${srcdir}/flutter/bin/flutter" config --no-analytics
  "${srcdir}/flutter/bin/flutter" update-packages
  "${srcdir}/flutter/bin/flutter" precache

  patch -p1 -i "${srcdir}/flutter.patch"
  patch -p1 -i "${srcdir}/flutter-tools-executable.patch"

  dart \
    --packages="${srcdir}/flutter/packages/flutter_tools/.packages" \
    --snapshot-kind=app-jit \
    --snapshot="${srcdir}/flutter/bin/cache/flutter_tools.snapshot" \
    "${srcdir}/flutter/packages/flutter_tools/bin/flutter_tools.dart" \
    --version

  for artifact in "${srcdir}/flutter"/bin/cache/artifacts/*; do
    if ! ls -1qA "${artifact}" | grep -q .; then
      touch "${artifact}/.must-keep-to-prevent-cache-updates"
    fi
  done

  find "${srcdir}/flutter" -type f -name .packages -exec rm -r {} \;

  rm -rf "${srcdir}/flutter/.github"
  rm -rf "${srcdir}/flutter/.idea"
  rm -rf "${srcdir}/flutter/.pub-cache"
  rm -rf "${srcdir}/flutter/examples"
  rm -rf "${srcdir}/flutter/dev"

  git gc --aggressive --prune=now
}

check() {
  shellcheck "${srcdir}/flutter/bin/flutter"
}

package() {
  install -Dm644 "${srcdir}/flutter/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
  install -dm755 "${pkgdir}/opt/${pkgname}"
  install -dm755 "${pkgdir}/usr/bin"
  cp -ra "${srcdir}/flutter" "${pkgdir}/opt/"
  ln -s "/opt/${pkgname}/bin/flutter" "${pkgdir}/usr/bin/flutter"
  chmod a+rw "${pkgdir}/opt/${pkgname}/bin/cache/lockfile" "${pkgdir}/opt/${pkgname}/version"
}
