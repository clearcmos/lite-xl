pkgname=lite-xl-custom
pkgdesc='A lightweight text editor written in Lua (fork with fractional scaling)'
pkgver=2.1.8.r4.e880edfb
pkgrel=1
arch=('x86_64')
url='https://github.com/clearcmos/lite-xl'
license=('MIT')
depends=(freetype2
         glibc
         pcre2
         sdl3)
makedepends=(git
             meson)
provides=('lite-xl')
conflicts=('lite-xl' 'lite')
source=("$pkgname::git+https://github.com/clearcmos/lite-xl.git")
sha256sums=('SKIP')

pkgver() {
    cd "$pkgname"
    printf "2.1.8.r%s.%s" "$(git rev-list v2.1.8..HEAD --count 2>/dev/null || echo 0)" "$(git rev-parse --short HEAD)"
}

build() {
    arch-meson "$pkgname" build \
        -Drenderer=true
    meson compile -C build
}

package() {
    meson install -C build --destdir "$pkgdir" --skip-subprojects
    install -Dm0644 -t "$pkgdir/usr/share/licenses/$pkgname/" "$pkgname/LICENSE"
}
