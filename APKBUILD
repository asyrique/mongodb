# Maintainer: Filipp Andronov <filipp.andronov@gmail.com>
# Contributor: Marc Vertes <marc.vertes@ugrid.net>
pkgname=mongodb
pkgver=3.4.1
pkgrel=1
pkgdesc='A high-performance, open source, schema-free document-oriented database'
url='http://www.mongodb.org'
arch='x86_64'
license='AGPL3'
pkgusers="mongodb"
depends=
makedepends="scons paxmark libressl-dev pcre-dev snappy-dev boost-dev asio-dev
	libpcap-dev wiredtiger-dev zlib-dev cyrus-sasl-dev yaml-cpp-dev"
install="$pkgname.pre-install"
source="http://downloads.mongodb.org/src/mongodb-src-r${pkgver}.tar.gz
	20-fix-libc-version.patch
	40-fix-elf-native-class.patch
	backtrace.patch
	fix-asio-strerror_r.patch
	set-default-stacksize.patch
	wiredtiger-strtouq.patch
	boost160.patch
	boost162.patch

        mongodb.confd
        mongodb.initd
        mongodb.logrotate
        mongos.confd
        mongos.initd
	"
#
# 1. Force 64bits because of 40-fix-elf-native-class.patch
# 2. Use system allocator as tc-malloc doesn't build
# 2. Use as much system libs as possible, boost doesn't compile foe example
#
# TODO: proper elf-netive-class fix
# TODO: check if there are more libs could be replaced by system counterparts (see libpcap does't use, for example)
# TODO: proper fix for heap usage
# Right now code patched to always return 0 for heap usage statistics. That is necessary because musl malloc info
# isn't compatible with glibc and breaks build. It is _possible_ to patch code to return 0
# because on 64bit platform statistics is broken and returns wrong numbers, see SERVER-9168 mongo bug.
#
# There is a way to return propper value when tc-malloc used, but it doesn't compile for libmusl. Work is in progress,
# contribution strongly welcome :D
#
_builddir="$srcdir"/$pkgname-src-r$pkgver
_buildopts="
		--allocator=system \
		--disable-warnings-as-errors \
		--use-system-boost \
		--use-system-pcre \
		--use-system-wiredtiger \
		--use-system-snappy \
		--use-system-zlib \
		--use-system-yaml \
		--use-sasl-client \
		--ssl \
	"

prepare() {
        cd "$_builddir"

        local i
        for i in $source; do
                case $i in
                *.patch) msg $i; patch -p1 -i "$srcdir"/$i || return 1;;
                esac
        done
}

build() {
        cd "$_builddir"

	export SCONSFLAGS="$MAKEFLAGS"
	scons $_buildopts --prefix=$pkgdir/usr core
}

package() {
        cd "$_builddir"

	# install mongo targets
	export SCONSFLAGS="$MAKEFLAGS"
	scons $_buildopts --prefix=$pkgdir/usr install

	# java jit requires paxmark
	paxmark -m "$pkgdir"/usr/bin/mongo*

	# install alpine specific files
	install -dm700 "$pkgdir/var/lib/mongodb"
	install -dm755 "$pkgdir/var/log/mongodb"
	chown mongodb:mongodb "$pkgdir/var/lib/mongodb" "$pkgdir/var/log/mongodb"

	install -Dm755 "$srcdir/mongodb.initd" "$pkgdir/etc/init.d/mongodb"
	install -Dm644 "$srcdir/mongodb.confd" "$pkgdir/etc/conf.d/mongodb"
	install -Dm644 "$srcdir/mongodb.logrotate" "$pkgdir/etc/logrotate.d/mongodb"

	install -Dm755 "$srcdir/mongos.initd" "$pkgdir/etc/init.d/mongos"
	install -Dm644 "$srcdir/mongos.confd" "$pkgdir/etc/conf.d/mongos"
}

md5sums="87c59198dbe3aca7832378725e05fd91  mongodb-src-r3.2.10.tar.gz
7806ba5e8af9b6f99e8d88edd31e4b49  20-fix-libc-version.patch
04a348397be8ca7471d404056d8a1490  40-fix-elf-native-class.patch
86a988b5d4402227d177b8a1167008e8  backtrace.patch
cd0833592e3a23e729ebd71eb756318c  fix-asio-strerror_r.patch
2d4de6cff2918ad610efead9f7c85f4f  set-default-stacksize.patch
e6016216b02afd44164e17ae31e3ce7d  wiredtiger-strtouq.patch
1df24dc2aa6b8f4c6da22f097a921ebb  boost160.patch
79cfc7e792d1e8e30cc203a8c63ac65b  boost162.patch
7d2f94bed7bfacd32fcd52dfd931f077  mongodb.confd
792a0b53b3e843cf14176c5beb8cdfe1  mongodb.initd
49df78833de4cb6e2b9b1ab9da52c3ac  mongodb.logrotate
33b23ee722f6e5d15eb6d9c2723a346f  mongos.confd
28611e6796d71c21661dd3d7ee8f23b8  mongos.initd"
sha256sums="54f475e553827733fb351ee4b03b470297f0d08e0434fbf7e6661705124da97b  mongodb-src-r3.2.10.tar.gz
74bf7d84584e038076581d2a7f7c1aedcd80bcdf25247ce905e9c36907272b62  20-fix-libc-version.patch
3a863660113d29728d7a852b3fba73926697b496848f8ccc4d8e73e6614cfdfc  40-fix-elf-native-class.patch
300d9f6b819730de54018d4b418eb7f921ba9c83fad4958a040544b076160848  backtrace.patch
ec6d404221f02706ef2e52e00e414e98666dcc3606e78c9b3d33dfbd42a1eae7  fix-asio-strerror_r.patch
85f2e5c5de90e43446ad3a13d661c211deb64cfce9e639ba1c2fa2a4573414ae  set-default-stacksize.patch
20c465ae169e5e3642ca0c8486b2165a412dabe171e8e249c134e0a4688342a4  wiredtiger-strtouq.patch
0e9da35f4303e53daf51e78961c517895f2d12f4fa49298f01e1665e15246d73  boost160.patch
bf373d1514b7947dc4747e11babf87a4bd8e7d581781c6771a844e69c1c4d273  boost162.patch
a4ca29c577428c02cd0b0a8b46756df5f53a05519c9d13c270533cf99b9b819d  mongodb.confd
7e39fbd4dc18dba21c8767931683f4795e58e0e91b9f9a5842539923ded453c9  mongodb.initd
76994c32d999def5c925bd7be3f96687b3406f1d67b89aa6a4df8053025b1e01  mongodb.logrotate
2afd582564623da0e928ca667d37bef467334c82d08b49301f1f6c16ba177767  mongos.confd
1c48327b02bb9e1aaacaa39a4c0e9976a765a0ab0992d4e27cc4c36a33fefe2d  mongos.initd"
sha512sums="48400f00ed84922b1e734ad915c376a567af2cd32e9cdcc40819fdfbc0a5c2444e4f325b1a541fc21cf87f4d95f9bdcc64bd59eab9d25e75b28732978feda031  mongodb-src-r3.2.10.tar.gz
b9fdacb273d5a4e1e60735846b262287f84ca6cbded9393d182f69158d3162a50cae5d834f76860dcb7e965a04f5fb510d32f0757d5343168eb15f6610f05e59  20-fix-libc-version.patch
56db8f43afc94713ac65b174189e2dd903b5e1eff0b65f1bdac159e52ad4de6606c449865d73bd47bffad6a8fc339777e2b11395596e9a476867d94a219c7925  40-fix-elf-native-class.patch
7d097f497cb910c9cb81086cd8c222e43456d1a6de4adfe3e97a4d99add454279350fdeb7305dab84b3deca04afd824036d4065ee0fb8cdd8c03e96d98ee86af  backtrace.patch
f829b1ad542db3ee776d444243b8b47ab4e48a7386d9b199d7b1adafd31556cf173a5683b78ee735d6a69999ad9af5ad152fde955bbe8865f7910718991ce97c  fix-asio-strerror_r.patch
1492137b0e3456d90a79617c1cd5ead5c71b1cfaae9ee41c75d56cd25f404ec73a690f95ce0d8c064c0a14206daca8070aa769b7cdfa904a338a425b52c293fa  set-default-stacksize.patch
bbb323d428d59584703e8692bf4df7fe0d37c0324c23822bade2edd1ca78759191f778230b7107502a9d2f7f22afc84d4ea350139fc1d751ceb2fff219b9ddf8  wiredtiger-strtouq.patch
385c82875174caae433a3b381eb10f98a6fed0c8943788ddefff1de80a898e480bdbbf094a7783285cf2ae11ce3fc6878e57d58183d05be2f0837b206aaa4da6  boost160.patch
79edfd1a6eaba597b31a82e54722dccab288d8b8840a53f79140b5fca221b5acd9fbc770d99e46ea9fa0da502cdf18dd35d982c95a4aa341806c3d8b61fc732f  boost162.patch
9bcd870742c31bf25f34188ddc3c414de1103e9860dea9f54eee276b89bc2cf1226abab1749c5cda6a6fb0880e541373754e5e83d63cc7189d4b9c274fd555c3  mongodb.confd
2cb295ac0eb44acb4533c86d6d0988d5f760361a43cff7845713f3e2cf0e37023d23790a2926c613bf2b0668060c0b68d59000db52daaacd8af10e701a8b4192  mongodb.initd
8c089b1a11f494e4148fb4646265964c925bf937633a65e395ee1361d42facf837871dd493a9a2e0f480ae0e0829dbd3ed60794c5334e2716332e131fc5c2c51  mongodb.logrotate
61d8734cef644187eeadc821c89e63a3fbf61860fe2db6e74557b1c6760fe83ba7549cb04f9e3aacea4d8e7e4d81a3b1bc0d5e29715eca33c4761adb17ea9ab7  mongos.confd
13aad7247b848ac58b2bc0b40a0d30d909e950380abd0c83fab0e2a394a42dc268a66dac53cf9feec6971739977470082cc4339cca827f41044cfe43803ef3f7  mongos.initd"
