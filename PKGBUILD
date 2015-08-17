# Maintainer: Daniel Wallace <danielwallace at gtmanfred dot com>
# Contributor: Sergej Pupykin <pupykin.s+arch@gmail.com>
# Contributor: Bartłomiej Piotrowski <barthalion@gmal.com>
# Contributor: Miroslaw Szot <mss@czlug.icis.pcz.pl>

_cfgdir=/etc/nginx
_tmpdir=/var/lib/nginx

pkgname=nginx-socket-hg
_pkgname=nginx
pkgver=5311.ae3fd1ca62e0
pkgver() {
      cd $srcdir/nginx
    echo $(hg identify -n).$(hg identify -i)
}
pkgrel=1
pkgdesc="lightweight HTTP server and IMAP/POP3 proxy server"
arch=('i686' 'x86_64')
depends=('pcre' 'zlib' 'openssl')
makedepends=('mercurial' 'git' 'passenger')
url="http://nginx.org"
license=('custom')
install=nginx.install
provides=(nginx)
backup=(${_cfgdir:1}/fastcgi.conf
		${_cfgdir:1}/fastcgi_params
		${_cfgdir:1}/koi-win
		${_cfgdir:1}/koi-utf
		${_cfgdir:1}/mime.types
		${_cfgdir:1}/nginx.conf
		${_cfgdir:1}/scgi_params
		${_cfgdir:1}/uwsgi_params
		${_cfgdir:1}/win-utf
		etc/logrotate.d/nginx)
conflicts=('nginx')

source=($_pkgname::hg+http://hg.nginx.org/nginx
		nginx.logrotate
        socket
        service
        "socket.patch"
        git://github.com/yaoweibin/nginx_syslog_patch.git
        )
md5sums=('SKIP'
         'b38744739022876554a0444d92e6603b'
         '5981d95bb07ca3e1f3640118199c219d'
         '34b083111a00ac7003b267d843b9f704'
         '574c0352c6d159c6b43895136cc26f60'
         'SKIP')
#prepare() {
#}




build() {
	cd "$srcdir"/$_pkgname

    mv auto/configure .
    mv docs/man .
    mv docs/html .
    patch -Np1 -i "$srcdir/socket.patch"
	#cd "$srcdir"/$_pkgname
	./configure \
        --with-systemd \
        --prefix=$_cfgdir \
        --conf-path=$_cfgdir/nginx.conf \
        --sbin-path=/usr/bin/nginx \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/lock/nginx.lock \
        --user=http --group=http \
        --http-log-path=/var/log/nginx/access.log \
        --error-log-path=/var/log/nginx/error.log \
        --http-client-body-temp-path=$_tmpdir/client-body \
        --http-proxy-temp-path=$_tmpdir/proxy \
        --http-uwsgi-temp-path=$_tmpdir/uwsgi \
        --with-ipv6 --with-pcre-jit \
        --with-file-aio \
        --with-http_dav_module \
        --with-http_gzip_static_module \
        --with-http_realip_module \
        --with-http_ssl_module \
        --with-http_stub_status_module
        #--add-module=/usr/lib/passenger/ext/nginx \
        #--with-imap --with-imap_ssl_module \
        #--http-fastcgi-temp-path=$_tmpdir/fastcgi \
        #--http-scgi-temp-path=$_tmpdir/scgi \
        #--add-module="$srcdir/nginx_syslog_patch/"
        #--with-http_mp4_module \
        #--with-http_realip_module \
        #--with-http_addition_module \
        #--with-http_xslt_module \
        #--with-http_image_filter_module \
        #--with-http_geoip_module \
        #--with-http_sub_module \
        #--with-http_flv_module \
        #--with-http_random_index_module \
        #--with-http_secure_link_module \
        #--with-http_degradation_module \
        #--with-http_perl_module \

	make
}

package() {
	cd "$srcdir"/$_pkgname
	make DESTDIR="$pkgdir" install

	install -d "$pkgdir"/etc/logrotate.d
	install -m644 "$srcdir"/nginx.logrotate "$pkgdir"/etc/logrotate.d/nginx

	sed -e 's|\<user\s\+\w\+;|user html;|g' \
		-e '44s|html|/usr/share/nginx/html|' \
		-e '54s|html|/usr/share/nginx/html|' \
		-i "$pkgdir"/etc/nginx/nginx.conf
	rm "$pkgdir"/etc/nginx/*.default

	install -d "$pkgdir"/$_tmpdir

	install -d "$pkgdir"/usr/share/nginx
	mv "$pkgdir"/etc/nginx/html/ "$pkgdir"/usr/share/nginx

	install -D -m644 "$srcdir/$_pkgname/docs/text/LICENSE" "$pkgdir"/usr/share/licenses/nginx/LICENSE

    #service/socket files
	install -D -m644 "$srcdir/${source[2]}" "$pkgdir/usr/lib/systemd/system/$_pkgname.socket"
	install -D -m644 "$srcdir/${source[3]}" "$pkgdir/usr/lib/systemd/system/$_pkgname.service"
	rm -rf "$pkgdir"/var/run
}
