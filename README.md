# android_ndk_tips
##ndk学习笔记
###1.android 里面使用原生库

	LOCAL_LDLIBS :=  -lz\     //zlib
                 	-llog	//log

###2.android错误 D/dalvikvm: No JNI_OnLoad found in
##ndk移植笔记
###1.[curl and libcurl](http://curl.haxx.se/)

操作系统：Ubuntu 15.10 64位

ndk：android-ndk-r10e-linux-x86_64

gcc：(Ubuntu 5.2.1-22ubuntu2) 5.2.1 20151010

####&ensp;&ensp;第一步：配置使用环境：
	
&ensp;&ensp;``vim /etc/profile``

	#根据自己的环境进行替换
	export NDK_HOME=/home/soft/android-ndk-r10e
	export TOOLCHAIN=$NDK_HOME/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64/bin
	export PATH=$NDK_HOME:$TOOLCHAIN:$PATH

&ensp;&ensp; 使profile文件生效：

&ensp;&ensp;``source /etc/profile ``

####&ensp;&ensp;第二步配置libcurl
	#解压
	tar -zxvf curl-7.45.0.tar.gz
	cd curl-7.45.0
	#配置
	./configure --host=arm-linux --target=arm-linux CC=arm-linux-androideabi-gcc --disable-tftp --disable-sspi --disable-ipv6 --disable-ldaps --disable-ldap --disable-telnet --disable-pop3 --disable-ftp --disable-imap --disable-smtp --disable-pop3 --disable-rtsp --disable-ares --without-ca-bundle --disable-warnings --disable-manual --without-nss --enable-shared --without-zlib --without-random CFLAGS="-nostdlib" CPPFLAGS="-I$NDK_HOME/platforms/android-9/arch-arm/usr/include " LDFLAGS="-L$NDK_HOME/platforms/android-9/arch-arm/usr/lib/ -lc -ldl"

&ensp;&ensp;以上通过--disable- 去关闭一些不需要使用的协议，执行上述命令后如果你能看见以下内容说明你成功了：

	  configure: Configured to build curl/libcurl:
	  curl version:     7.45.0
	  Host setup:       arm-unknown-linux-gnu
	  Install prefix:   /usr/local
	  Compiler:         arm-linux-androideabi-gcc
	  SSL support:      no      (--with-{ssl,gnutls,nss,polarssl,cyassl,axtls,winssl,darwinssl} )
	  SSH support:      no      (--with-libssh2)
	  zlib support:     no      (--with-zlib)
	  GSS-API support:  no      (--with-gssapi)
	  TLS-SRP support:  no      (--enable-tls-srp)
	  resolver:         default (--enable-ares / --enable-threaded-resolver)
	  IPv6 support:     no      (--enable-ipv6)
	  Unix sockets support: enabled
	  IDN support:      no      (--with-{libidn,winidn})
	  Build libcurl:    Shared=yes, Static=yes
	  Built-in manual:  no      (--enable-manual)
	  --libcurl option: enabled (--disable-libcurl-option)
	  Verbose errors:   enabled (--disable-verbose)
	  SSPI support:     no      (--enable-sspi)
	  ca cert bundle:   no
	  ca cert path:     no
	  LDAP support:     no      (--enable-ldap / --with-ldap-lib / --with-lber-lib)
	  LDAPS support:    no      (--enable-ldaps)
	  RTSP support:     no      (--enable-rtsp)
	  RTMP support:     no      (--with-librtmp)
	  metalink support: no      (--with-libmetalink)
	  HTTP2 support:    disabled (--with-nghttp2)
	  Protocols:        DICT FILE GOPHER HTTP
	
	  SONAME bump:     yes - WARNING: this library will be built with the SONAME
	                   number bumped due to (a detected) ABI breakage.
	                   See lib/README.curl_off_t for details on this.

	

上述显示了对一些协议的支持，如果需要可以通过./configue 配置的时候进行打开，打开之后你的linux也要支持相应的库才可以进行编译