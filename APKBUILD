maintainer="lauren n. liberda <lauren@selfisekai.rocks>"
pkgname=code-server-oss
pkgver=1.94.2
pkgrel=1
pkgdesc="Visual Studio Server Code (OSS, with VSX)"
url="https://github.com/microsoft/vscode"
arch="aarch64 x86_64" # electron
license="MIT"
depends="ripgrep"
makedepends="
	krb5-dev
	libsecret-dev
	npm
	python3
	"
install="$pkgname.post-install"
source="$pkgname-$pkgver.tar.gz::https://github.com/microsoft/vscode/archive/refs/tags/$pkgver.tar.gz
	launcher
	ext
	"
builddir="$srcdir/vscode-$pkgver"
options="!check net" # no tests (that make sense to run..)

export ELECTRON_SKIP_BINARY_DOWNLOAD=1
export PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1
export PLATFORM=linux-alpine

prepare() {
	default_prepare

	sed -i -e "/cp\.execSync('git config pull.rebase merges');/d" \
		-e "/cp\.execSync('git config blame.ignoreRevsFile .git-blame-ignore-revs');/d" build/npm/postinstall.js
	sed -i "s/docker run --rm \${imageName}:\${nodeVersion}-alpine //g" build/gulpfile.reh.js
	sed -i -E -e "s/const useSourcemaps = (.+);/const useSourcemaps = false;/g" \
		-e "s/const useSourcemaps = (.+);/const useSourcemaps = false;/g"  build/lib/optimize.ts
	
	# Extensions
	sed -i "/builtInExtensions/,+49d" product.json && \
		sed -i 'N;s/,\n}/,\n\t"builtInExtensions": [],\n\t"extensionsGallery": {\n\t\t"serviceUrl": "https:\/\/open-vsx.org\/vscode\/gallery",\n\t\t"itemUrl": "https:\/\/open-vsx.org\/vscode\/item"\n\t}\n}/' product.json
	
	# Remove useless packages
	sed -i "/native-keymap/d" package.json && \
		sed -i -e "/import type \* as nativeKeymap from 'native-keymap';/d" \
			-e "/import \* as platform from '..\/..\/..\/base\/common\/platform.js';/d" \
			-e "/const nativeKeymapMod = await import('native-keymap');/,+11d" \
			-e "/readKeyboardLayoutData/,+4d" src/vs/platform/keyboardLayout/electron-main/keyboardLayoutMainService.ts && \
			sed -i "/native-keymap/,+7d" src/vs/platform/environment/test/node/nativeModules.integrationTest.ts

	sed -i "/native-is-elevated/d" package.json && \
		sed -i "s/(await import('native-is-elevated')).default()/false/g" src/vs/platform/native/electron-main/nativeHostMainService.ts && \
		sed -i "/native-is-elevated/,+7d" src/vs/platform/environment/test/node/nativeModules.integrationTest.ts

	npm ci --ignore-scripts
}

build() {
	npm run gulp vscode-reh-web-$PLATFORM-min && \
    	npm run gulp vscode-reh-web-$PLATFORM-min-ci
}

package() {
	mkdir -p "$pkgdir"/usr/lib/code-server-oss
	cp -a ../vscode-reh-web-linux-alpine/node_modules "$pkgdir"/usr/lib/code-server-oss/
	cp -a ../vscode-reh-web-linux-alpine/resources "$pkgdir"/usr/lib/code-server-oss/
	cp -a ../vscode-reh-web-linux-alpine/extensions "$pkgdir"/usr/lib/code-server-oss/
	cp -a ../vscode-reh-web-linux-alpine/package.json "$pkgdir"/usr/lib/code-server-oss/
	mkdir -p "$pkgdir"/usr/lib/code-server-oss/resources/app
	cp -a ../vscode-reh-web-linux-alpine/out "$pkgdir"/usr/lib/code-server-oss/resources/app/
	cp -a ../vscode-reh-web-linux-alpine/product.json "$pkgdir"/usr/lib/code-server-oss/resources/app/

	install -Dm755 "$srcdir"/launcher "$pkgdir"/usr/bin/code-server-oss
	install -Dm755 "$srcdir"/ext "$pkgdir"/usr/bin/ext
}

sha512sums="
48bba7074b72ea1ea04f6e53b5a3faafd476190e84bf5ee572745eac8a335dfe229f148fe8ba00d8add2d0fea29c8c34a8f8075f1cb1452e9821343dbd675ab0  code-server-oss-1.94.2.tar.gz
349ccd4cdfae47224daec4fb93e91e7c1f7469b83353aa10d7776c3859d16e3220c75d5b0d88f68557587cebcefa7b95ca428713c49d0d49ec662c6ee83971da  launcher
a3ef7e39579b85188c93316d364da5887b0f5de9030383707197238da9633bc5d0273eb441dbd94002053cc1f7b33c8c64a5325a6e5137ee0809d4a8fcb91329  ext
"
