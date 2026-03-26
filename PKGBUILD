pkgbase=kitty
pkgname=(kitty kitty-terminfo kitty-shell-integration)
pkgver=0.46.2
pkgrel=1
pkgdesc="kitty terminal (FX-4320/bdver2 optimized)"
arch=('x86_64')
url="https://github.com/kovidgoyal/kitty"
license=('GPL-3.0-only')
depends=(
    'cairo'
    'dbus'
    'freetype2'
    'fontconfig'
    'harfbuzz'
    'hicolor-icon-theme'
    'lcms2'
    'libgl'
    'libpng'
    'librsync'
    'libxkbcommon'
    'openssl'
    'python3'
    'wayland'
    'xxhash'
    'zlib'
    'libx11'
    'libxcursor'
    'libxkbcommon-x11'
    'libxi'
)
makedepends=(
    'clang'
    'lld'
    'compiler-rt'
    'go'
    'libxinerama'
    'libxrandr'
    'simde'
    'ttf-nerd-fonts-symbols-mono'
    'wayland-protocols'
)
options=("!lto")
source=("https://github.com/kovidgoyal/${pkgbase}/releases/download/v${pkgver}/${pkgbase}-${pkgver}.tar.xz"{,.sig})
b2sums=('b5e162436e92b19d36cb72a5a625efeec21be0377c7b9ce9788980a380fabc115ee07dd9917f7b4d767bfc3416dd99b34f57cdbf183eaac60ae14a9731a82a58'
        'SKIP')
validpgpkeys=('3CE1780F78DD88DF45194FD706BC317B515ACE7C')

build() {
    cd "$pkgbase-$pkgver"

    export CC="clang"
    export CXX="clang++"
    export AR="llvm-ar"
    export NM="llvm-nm"
    export RANLIB="llvm-ranlib"
    export STRIP="llvm-strip"

    export CFLAGS="-march=x86-64-v2 -mtune=bdver2 -mavx -mfma -maes -mbmi -mf16c -O3 -flto=thin -ffunction-sections -fdata-sections -fomit-frame-pointer"
    export CXXFLAGS="${CFLAGS}"
    export CPPFLAGS=""

    export LDFLAGS="-flto=thin -fuse-ld=lld -Wl,--gc-sections -Wl,--icf=safe -Wl,--as-needed -Wl,-O2 -Wl,-z,relro,-z,now"

    export KITTY_NO_LTO=1

    local _cflags_go="-march=x86-64-v2 -O3 -ffunction-sections -fdata-sections -fomit-frame-pointer"
    local _ldflags_go="-fuse-ld=lld -Wl,--gc-sections -Wl,--icf=safe -Wl,--as-needed -Wl,-O2 -Wl,-z,relro,-z,now"

    export CGO_CPPFLAGS=""
    export CGO_CFLAGS="${_cflags_go}"
    export CGO_CXXFLAGS="${_cflags_go}"
    export CGO_LDFLAGS="${_ldflags_go}"
    export GOFLAGS="-buildmode=pie -trimpath -ldflags=-linkmode=external -mod=readonly -modcacherw"
    export GOAMD64=v2

    python3 setup.py linux-package --update-check-interval=0 --verbose
}

package_kitty() {
    depends+=('kitty-terminfo' 'kitty-shell-integration')
    optdepends=('imagemagick: viewing images with icat'
                'python-pygments: syntax highlighting in kitty +kitten diff'
                'libcanberra: playing "bell" sound on terminal bell')

    cd "$srcdir/$pkgbase-$pkgver"
    cp -r linux-package "${pkgdir}"/usr

    linux-package/bin/kitten __complete__ setup bash | install -Dm644 /dev/stdin "${pkgdir}"/usr/share/bash-completion/completions/kitty
    linux-package/bin/kitten __complete__ setup fish | install -Dm644 /dev/stdin "${pkgdir}"/usr/share/fish/vendor_completions.d/kitty.fish
    linux-package/bin/kitten __complete__ setup zsh  | install -Dm644 /dev/stdin "${pkgdir}"/usr/share/zsh/site-functions/_kitty

    install -Dm 644 "${pkgdir}"/usr/share/icons/hicolor/256x256/apps/kitty.png "${pkgdir}"/usr/share/pixmaps/kitty.png

    rm -r "$pkgdir"/usr/share/terminfo
    rm -r "$pkgdir"/usr/lib/kitty/shell-integration

    install -Dm 644 docs/_build/html/_downloads/*/kitty.conf "${pkgdir}"/usr/share/doc/${pkgname}/kitty.conf
}

package_kitty-terminfo() {
    pkgdesc='Terminfo for kitty, an OpenGL-based terminal emulator'
    depends=('ncurses')

    mkdir -p "$pkgdir/usr/share/terminfo"
    tic -x -o "$pkgdir/usr/share/terminfo" $pkgbase-$pkgver/terminfo/kitty.terminfo
}

package_kitty-shell-integration() {
    pkgdesc='Shell integration scripts for kitty, an OpenGL-based terminal emulator'
    depends=('python3')
    optdepends+=('bash: Bash completions'
                 'zsh: ZSH completions'
                 'fish: Fish completions')

    mkdir -p "$pkgdir/usr/lib/kitty/"
    cp -r "$srcdir/$pkgbase-$pkgver/shell-integration" "$pkgdir/usr/lib/kitty/"
}
