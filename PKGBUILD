# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/mariadb
# 						Maintainer: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# 						Maintainer: Christian Hesse <mail@eworm.de>

pkgbase=mariadb
pkgname=('libmariadbclient' 'mariadb-clients' 'mytop' 'mariadb')
pkgver=10.1.25
pkgrel=2
arch=(x86_64)
license=(GPL)
url='http://mariadb.org/'
makedepends=('cmake' 'zlib' 'libaio' 'libxml2' 'openssl' 'pcre' 'jemalloc'
             'lz4' 'boost' 'libevent')
source=("https://ftp.heanet.ie/mirrors/mariadb/mariadb-$pkgver/source/mariadb-$pkgver.tar.gz"
        '0001-openssl-1-1-0.patch'
        'mariadb-sysusers.conf'
        'mariadb-tmpfile.conf')
sha256sums=('7205ecaa6f1bc16335ad88faa5c46be52b3ac628a5e795cb942a3f4335b2a0d6'
            '41d80d3ff78251bd3e05ed5c40b63e4ac71153832a00c86443be5a111daad354'
            'e1a22777c65854041f16fc0a2db3218d17b4d7e7ec7ab7a77cf49c71277c1515'
            '2af318c52ae0fe5428e8a9245d1b0fc3bc5ce153842d1563329ceb1edfa83ddd')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

prepare() {
  cd $pkgbase-$pkgver

  # openssl 1.1.0
  patch -Np1 < "${srcdir}"/0001-openssl-1-1-0.patch
}

build() {
  mkdir build
  cd build

  cmake ../$pkgbase-$pkgver \
    -DCMAKE_AR=/usr/bin/gcc-ar \
    -DCMAKE_RANLIB=/usr/bin/gcc-ranlib \
    -DBUILD_CONFIG=mysql_release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DMYSQL_DATADIR=/var/lib/mysql \
    -DMYSQL_UNIX_ADDR=/run/mysqld/mysqld.sock \
    -DDEFAULT_CHARSET=utf8mb4 \
    -DDEFAULT_COLLATION=utf8mb4_unicode_ci \
    -DENABLED_LOCAL_INFILE=ON \
    -DINSTALL_DOCDIR=share/doc/mariadb \
    -DINSTALL_DOCREADMEDIR=share/doc/mariadb \
    -DINSTALL_MANDIR=share/man \
    -DINSTALL_PLUGINDIR=lib/mysql/plugin \
    -DINSTALL_SCRIPTDIR=bin \
    -DINSTALL_SYSCONFDIR=/etc/mysql \
    -DINSTALL_SYSCONF2DIR=/etc/mysql \
    -DINSTALL_INCLUDEDIR=include/mysql \
    -DINSTALL_SUPPORTFILESDIR=share/mysql \
    -DINSTALL_MYSQLSHAREDIR=share/mysql \
    -DINSTALL_SHAREDIR=share/mysql \
    -DWITH_SYSTEMD=no \
    -DWITH_READLINE=ON \
    -DWITH_ZLIB=system \
    -DWITH_SSL=system \
    -DWITH_PCRE=system \
    -DWITH_LIBWRAP=OFF \
    -DWITH_JEMALLOC=ON \
    -DWITH_EXTRA_CHARSETS=complex \
    -DWITH_EMBEDDED_SERVER=ON \
    -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
    -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
    -DWITH_INNOBASE_STORAGE_ENGINE=1 \
    -DWITH_PARTITION_STORAGE_ENGINE=1 \
    -DWITH_TOKUDB_STORAGE_ENGINE=1 \
    -DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 \
    -DWITHOUT_FEDERATED_STORAGE_ENGINE=1 \
    -DWITHOUT_PBXT_STORAGE_ENGINE=1 \
    -DCMAKE_EXE_LINKER_FLAGS='-ljemalloc' \
    -DCMAKE_C_FLAGS="-fPIC $CFLAGS -fno-strict-aliasing -DBIG_JOINS=1 -fomit-frame-pointer -fno-delete-null-pointer-checks" \
    -DCMAKE_CXX_FLAGS="-fPIC $CXXFLAGS -fno-strict-aliasing -DBIG_JOINS=1 -felide-constructors -fno-rtti -fno-delete-null-pointer-checks" \
    -DWITH_MYSQLD_LDFLAGS="-pie ${LDFLAGS},-z,now"

  make
}

package_libmariadbclient() {
  pkgdesc='MariaDB client libraries'
  depends=('openssl' 'libaio' 'zlib' 'pcre' 'lz4' 'lzo' 'xz')
  conflicts=('libmysqlclient')

  cd build

  for dir in libmysql libmysqld libservices include; do
    make -C $dir DESTDIR="$pkgdir" install
  done

  install -D -m0755 scripts/mysql_config "$pkgdir"/usr/bin/mysql_config
  install -D -m0644 "$srcdir"/$pkgbase-$pkgver/man/mysql_config.1 "$pkgdir"/usr/share/man/man1/mysql_config.1

  install -D -m0644 support-files/mariadb.pc "$pkgdir"/usr/share/pkgconfig/mariadb.pc
  install -D -m0644 "$srcdir"/$pkgbase-$pkgver/support-files/mysql.m4 "$pkgdir"/usr/share/aclocal/mysql.m4

  # remove static libraries
  rm "$pkgdir"/usr/lib/*.a
}

package_mariadb-clients() {
  pkgdesc='MariaDB client tools'
  depends=("libmariadbclient=${pkgver}" 'zlib' 'openssl' 'jemalloc')
  conflicts=('mysql-clients')
  provides=("mysql-clients=$pkgver")

  cd build

  make -C client DESTDIR="$pkgdir" install

  # install man pages
  for man in mysql mysql_plugin mysql_upgrade mysqladmin mysqlbinlog mysqlcheck mysqldump mysqlimport mysqlshow mysqlslap mysqltest; do
    install -D -m0644 "$srcdir"/$pkgbase-$pkgver/man/$man.1 "$pkgdir"/usr/share/man/man1/$man.1
  done
}

package_mytop() {
  pkgdesc='Top clone for MariaDB'
  depends=('perl' 'perl-dbd-mysql' 'perl-term-readkey')

  cd build

  install -Dm0755 scripts/mytop "$pkgdir"/usr/bin/mytop
}

package_mariadb() {
  pkgdesc='Fast SQL database server, drop-in replacement for MySQL'
  backup=('etc/mysql/my.cnf'
          'etc/mysql/wsrep.cnf')
  install=mariadb.install
  depends=("mariadb-clients=${pkgver}" 'inetutils' 'libaio' 'libxml2' 'pcre' 'jemalloc'
           'lz4' 'boost-libs' 'lzo' 'libevent')
  optdepends=('galera: for MariaDB cluster with Galera WSREP'
              'perl-dbd-mysql: for mysqlhotcopy, mysql_convert_table_format and mysql_setpermission')
  conflicts=('mysql')
  provides=("mysql=$pkgver")
  options=('emptydirs')

  cd build

  make DESTDIR="$pkgdir" install

  cd "$pkgdir"

  # We specified INSTALL_SYSCONFDIR and INSTALL_SYSCONF2DIR to have proper paths
  # in binaries and support file. But we want our own files...
  # TOOD: Change to upstream file layout with version 10.2.x?
  rm -r etc/
  install -Dm0644 usr/share/mysql/my-medium.cnf etc/mysql/my.cnf
  install -Dm0644 usr/share/mysql/wsrep.cnf etc/mysql/wsrep.cnf

  install -Dm0644 "${srcdir}"/mariadb-tmpfile.conf usr/lib/tmpfiles.d/mariadb.conf
  install -Dm0644 "${srcdir}"/mariadb-sysusers.conf usr/lib/sysusers.d/mariadb.conf

  install -dm0700 var/lib/mysql
  chown -R 89:89 var/lib/mysql &>/dev/null

  # move to proper licenses directories
  install -d usr/share/licenses/mariadb
  mv usr/share/doc/mariadb/COPYING* usr/share/licenses/mariadb/

  # already installed to real systemd unit directory
  #rm -r usr/share/mysql/systemd/

  # provided by libmariadbclient
  rm usr/bin/mysql_config
  rm usr/lib/libmysql*
  rm usr/share/man/man1/mysql_config.1
  rm -r usr/include/
  rm -r usr/share/mysql/{aclocal,pkgconfig}

  # provided by mariadb-clients
  rm usr/bin/{mysql,mysql_plugin,mysql_upgrade,mysqladmin,mysqlbinlog,mysqlcheck,mysqldump,mysqlimport,mysqlshow,mysqlslap,mysqltest}
  rm usr/share/man/man1/{mysql,mysql_plugin,mysql_upgrade,mysqladmin,mysqlbinlog,mysqlcheck,mysqldump,mysqlimport,mysqlshow,mysqlslap,mysqltest}.1

  # provided by mytop
  rm usr/bin/mytop

  # not needed
  rm -r usr/{data,mysql-test,sql-bench}
  rm usr/share/man/man1/mysql-test-run.pl.1
}
