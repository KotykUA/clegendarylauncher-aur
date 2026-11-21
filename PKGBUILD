# Maintainer: aquafish <tarasfed2016@gmail.com>
pkgname=clegendarylauncher-bin
pkgver=1.1.3
pkgrel=1
pkgdesc='CLegendaryLauncher (prebuilt Linux AppImage repackaged for pacman)'
arch=('x86_64')
url='https://github.com/WER-CORE/CL-Win-Edition--Update'
license=('GPL-3.0-only')
depends=('glibc' 'gcc-libs' 'zlib' 'icu' 'openssl' 'fontconfig' 'libx11' 'libxcb')
options=('!strip' '!debug')

_appimage='CLegendaryLauncher-x64.AppImage'
source=("${_appimage}::https://github.com/WER-CORE/CL-Win-Edition--Update/releases/download/v${pkgver}/${_appimage}" 'LICENSE')
noextract=("${_appimage}")
sha256sums=(
    'c4a47ceaac63435bc95ec35f008b1a51828b1b4760e0ceb107a0b179270a5e0a'
    'ff9eb3cd9421cc3eb2fe994b708d322a2a0cd79efff780f2ab5932f839daedde'
)

prepare() {
    cd "${srcdir}"
    chmod +x "${_appimage}"
    rm -rf -- "${srcdir}/squashfs-root"
    "${srcdir}/${_appimage}" --appimage-extract >/dev/null

    if [[ ! -f "${srcdir}/squashfs-root/usr/bin/CL.Avalonia" ]]; then
        printf 'ERROR: CL.Avalonia not found in extracted AppImage.\n' >&2
        return 1
    fi
}

package() {
    local _appdir="${pkgdir}/usr/lib/clegendarylauncher"

    # Keep all bundled Avalonia/.NET DLLs and native libraries together.
    install -dm755 "${_appdir}"
    cp -a "${srcdir}/squashfs-root/usr/bin/." "${_appdir}/"
    chmod 755 "${_appdir}/CL.Avalonia"

    # A normal executable on PATH; AppImage and FUSE aren't needed at runtime.
    install -dm755 "${pkgdir}/usr/bin"
    cat > "${pkgdir}/usr/bin/clegendarylauncher" <<'LAUNCHER'
#!/bin/sh
cd /usr/lib/clegendarylauncher || exit 1
exec ./CL.Avalonia "$@"
LAUNCHER
    chmod 755 "${pkgdir}/usr/bin/clegendarylauncher"

    # Preserve upstream desktop entry while correcting AppImage-only paths.
    install -Dm644 "${srcdir}/squashfs-root/CLegendaryLauncher.desktop" \
        "${pkgdir}/usr/share/applications/clegendarylauncher.desktop"
    sed -i \
        -e 's|^Exec=.*|Exec=clegendarylauncher|' \
        -e 's|^TryExec=.*|TryExec=clegendarylauncher|' \
        -e 's|^Icon=.*|Icon=clegendarylauncher|' \
        -e '/^Path=/d' \
        "${pkgdir}/usr/share/applications/clegendarylauncher.desktop"

    # pixmaps works for PNG icons without guessing the icon dimensions.
        install -Dm644 "${srcdir}/squashfs-root/CLegendaryLauncher.png" \
        "${pkgdir}/usr/share/pixmaps/clegendarylauncher.png"

    install -Dm644 "${srcdir}/LICENSE" \
        "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
