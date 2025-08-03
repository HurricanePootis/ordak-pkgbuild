# Maintainer: HurricanePootis <hurricanepootis@protonmail.com>
pkgname=duckstation-git
pkgver=0.1.r9439.g5c682d2
pkgrel=1
pkgdesc="Fast PlayStation 1 emulator for x86-64/AArch32/AArch64/RV64"
arch=(x86_64)
url="https://github.com/stenzek/duckstation"
license=('CC-BY-NC-ND-4.0')
depends=('glibc' 'gcc-libs' 'hicolor-icon-theme' 'qt6-base' 'libjpeg-turbo' 'zlib' 'zstd' 'freetype2' 'libwebp' 'libpng' 'systemd-libs' 'libzip' 'libbacktrace' 'curl' 'ffmpeg')
makedepends=('git' 'cmake' 'lld' 'llvm' 'extra-cmake-modules' 'qt6-tools' 'patchelf' 'ninja' 'jack' 'pipewire' 'libpulse' 'sndio' 'libdecor' 'wayland' 'python' 'vulkan-headers' 'libunwind' 'clang')
provides=("${pkgname::-4}")
conflicts=("${pkgname::-4}")
source=("$pkgname::git+$url.git"
	"1.patch::$url/commit/5ed79613905a967fa99eee77c3ec025df534fe9d.patch"
	"2.patch::$url/commit/30df16cc767297c544e1311a3de4d10da30fe00c.patch")
options=(!lto !debug)
sha256sums=('SKIP'
            'c3415372ad2053631762d82f74347d9d08f8cb786a56c9f63e1c38f9277520dd'
            '076bdafad5cd976ada63c892b1611325f000b650c573945b2aaf6259ab38effc')
#Set this to true to build a debug build
_debug=false
_buildtype=Release
[[ $_debug = "true" ]] && options+=(!strip) && _buildtype=RelWithDebInfo

pkgver(){
	cd "$srcdir/$pkgname"
	git describe --long --abbrev=7 | sed 's/\([^-]*-g\)/r\1/;s/-/./g'
}


prepare() {
	cd "$srcdir/$pkgname"
	if [[ ! -f "$srcdir/$pkgname/scripts/packaging/arch/PKGBUILD" ]]
	then
	patch -p1 -R < "$srcdir/1.patch"
	patch -p1 -R < "$srcdir/2.patch"
	fi

	if [[ $_debug = "true" ]]
	then
	sed -i "s/Release/${_buildtype}/g" "$srcdir/$pkgname/scripts/deps/build-dependencies-linux.sh"
	fi
}

build() {
	cd "$srcdir"

	if [[ ! -f "$srcdir/build-deps/lib/libsoundtouch.so" ]]
	then
	echo "Building deps"
	mkdir build-deps && cd build-deps
	"$srcdir/$pkgname"/scripts/deps/build-dependencies-linux.sh -system-freetype -system-harfbuzz -system-libjpeg \
	-system-libpng -system-libwebp -system-libzip -system-zlib -system-zstd -system-qt
	fi

	cd "$srcdir"
	cmake -B build -S "$pkgname" \
	-DCMAKE_C_COMPILER=clang \
	-DCMAKE_CXX_COMPILER=clang++ \
	-DCMAKE_C_FLAGS="$CFLAGS -flto=thin" \
	-DCMAKE_CXX_FLAGS="$CXXFLAGS -flto=thin" \
	-DCMAKE_EXE_LINKER_FLAGS_INIT="-fuse-ld=lld" \
	-DCMAKE_SHARED_LINKER_FLAGS_INIT="-fuse-ld=lld" \
	-DCMAKE_BUILD_TYPE=${_buildtype} \
	-Dcpuinfo_DIR="$srcdir/build-deps/share/cpuinfo" \
	-DDiscordRPC_DIR="$srcdir/build-deps/lib/cmake/DiscordRPC" \
	-Dplutosvg_DIR="$srcdir/build-deps/lib/cmake/plutosvg" \
	-DShaderc_DIR="$srcdir/build-deps/lib/cmake/Shaderc" \
	-Dspirv_cross_c_shared_DIR="$srcdir/build-deps/share/spirv_cross_c_shared/cmake" \
	-DSoundTouch_DIR="$srcdir/build-deps/lib/cmake/SoundTouch" \
	-DSDL3_DIR="$srcdir/build-deps/lib/cmake/SDL3" \
	-DCMAKE_INSTALL_PREFIX=/usr \
	-GNinja

	cmake --build build
}

package() {
	cd "$srcdir"
	install -Dm755 "$srcdir/build/bin/${pkgname::-4}-qt" "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	mkdir -p "$pkgdir/usr/lib/${pkgname::-4}/"
	cp -r -a build/bin/resources "$pkgdir/usr/lib/${pkgname::-4}"
	cp -r -a build/bin/translations "$pkgdir/usr/lib/${pkgname::-4}"
	install -Dm644 "$srcdir/build/bin/resources/org.duckstation.DuckStation.png" \
		"$pkgdir/usr/share/icons/hicolor/512x512/apps/org.duckstation.DuckStation.png"
	install -Dm644 "$srcdir/build/bin/resources/org.duckstation.DuckStation.desktop" \
		"$pkgdir/usr/share/applications/org.duckstation.DuckStation.desktop"

	#Find version numbers
	cd "$srcdir/build-deps/lib"
	find . -type f -name '*.so*' -exec install -Dm755 {} -t "$pkgdir/usr/lib/${pkgname::-4}/lib" \;
	_sdl3=$(find . ! -type l | grep libSDL3.so | sed 's/.\/libSDL3.so.//g')
	_pluto=$(find . ! -type l | grep libplutosvg.so | sed 's/.\/libplutosvg.so.//g')
	_soundtouch=$(find . ! -type l | grep libsoundtouch.so | sed 's/.\/libsoundtouch.so.//g')
	_cross=$(find . ! -type l | grep libspirv-cross-c-shared.so | sed 's/.\/libspirv-cross-c-shared.so.//g')

	#patch duckstation-qt
	patchelf --replace-needed libsoundtouch.so.2 /usr/lib/${pkgname::-4}/lib/libsoundtouch.so.${_soundtouch} "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	patchelf --replace-needed libplutosvg.so.0 /usr/lib/${pkgname::-4}/lib/libplutosvg.so.${_pluto} "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	patchelf --replace-needed libcpuinfo.so /usr/lib/${pkgname::-4}/lib/libcpuinfo.so "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	patchelf --replace-needed libSDL3.so.0 /usr/lib/${pkgname::-4}/lib/libSDL3.so.${_sdl3} "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	patchelf --add-needed /usr/lib/${pkgname::-4}/lib/libshaderc_ds.so "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	patchelf --add-needed /usr/lib/${pkgname::-4}/lib/libspirv-cross-c-shared.so.${_cross} "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	patchelf --add-needed /usr/lib/${pkgname::-4}/lib/libdiscord-rpc.so "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	patchelf --remove-rpath "$pkgdir/usr/lib/${pkgname::-4}/${pkgname::-4}-qt"
	patchelf --remove-rpath "$pkgdir/usr/lib/${pkgname::-4}/lib/libSDL3.so.${_sdl3}"
	#run
	install -dm755 "$pkgdir/usr/bin"
	cat >> "$pkgdir/usr/bin/${pkgname::-4}-qt" <<EOF
#!/usr/bin/env sh
cd /usr/lib/${pkgname::-4}/ || exit
./${pkgname::-4}-qt "\$@"
exit
EOF
	chmod 755 "$pkgdir/usr/bin/${pkgname::-4}-qt"
}
