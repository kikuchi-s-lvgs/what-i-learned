phpのDockerfileについて
php-cli→cliのみ。サーバーで動かすことを求めている人はこれじゃない。
php-fpm→基本はこれ。CGI込。OSはデビアン。PHPの拡張機能、PHP以外の拡張機能(PECL)などは、やり方があるので公式を見たほうが良い。
https://hub.docker.com/_/phpPHPの拡張の場合は、docker-php-ext-installとかのコマンドが用意されているのでこれでインストール。PECLの場合はPECLコマンドが入っているはずなのでそれでInstall。



php-fpmのイメージの中は、初期だとこんな感じで圧縮されている。
/usr/src/php.tar.xzそこで、docker-php-source extractをすると、解凍される。/usr/src/php/ext/の配下に各種拡張エクステンションが入っている。

```
bcmath      curl     exif          gd       imap       libxml    odbc     pdo           pdo_odbc    posix       session    soap     standard  tokenizer  xmlwriter　bz2         date     ext_skel.php  gettext  interbase  mbstring  opcache  pdo_dblib     pdo_pgsql   pspell      shmop      sockets  sysvmsg   wddx       xsl　calendar    dba      fileinfo      gmp      intl       mysqli    openssl  pdo_firebird  pdo_sqlite  readline    simplexml  sodium   sysvsem   xml        zend_test　com_dotnet  dom      filter        hash     json       mysqlnd   pcntl    pdo_mysql     pgsql       recode      skeleton   spl      sysvshm   xmlreader  zip　ctype       enchant  ftp           iconv    ldap       oci8      pcre     pdo_oci       phar        reflection  snmp       sqlite3  tidy      xmlrpc     zlib
```

こんな感じ。これがデフォルト。ここにあれば、あとはdocker-php-ext-installでインストールできる。で、Redisの場合はデフォで配置されていないので、git cloneしてくる必要がある。というわけ。解凍した際は、docker-php-source delete　をして解凍ファイルを削除しておくこと。せっかく圧縮してイメージが小さくなっているのに意味なくなるから。
→しかもこれに関しては同じRUNの中で実行するのがBP



**このリンク先が参考になった。**

http://dqn.sakusakutto.jp/2015/07/php_extension_pecl_phpize.html



すでに公式イメージのtar.gzに含められているphpモジュールは、docker-php-ext-installでいける。あまりわからないけどこれはいわゆるcomposerでインストールするのと同じかも？？→同じだった。packagistがリポジトリのことで、そこにあるソースをcomposerを使ってインストールや管理しているだけ。つまりcomposer使わずにtarを落としてきて解凍してビルドして配置するのも同じこと。要するにソースをビルドして(.soファイルを作成して)特定のファイル(extention_dir)に置いているだけ。

一方でcで書かれているモジュールはpeclで基本的にインストールできる。
モジュール名でググってpeclかpackagistでどちらに出てくるかで判断できる。

例えばredisもメモキャッシュもpecl

つまり、公式のドッカーイメージにはある程度デフォルトでコンパイルされたツールがある。→①
さらに、/usr/src/ext配下にいくつかソースが置かれていて、そこにあるものに関しては、docker-php-ext-installでインストール可能。
→②依存するモジュールは先にyumとかで落としておくことに注意。て、c言語で書かれたモジュールは基本peclで管理されていて、それは普通にpeclコマンドでインストールできる。→③
ただし、公式ドッカーイメージに関しては、peclインストールしたあとにdocker-php-ext-enable モジュール名
と打たないと有効にならないらしい。この辺の仕組みはこのドッカーイメージ特有のものだと思われる。普通とは違うところに.soを置いている？のかも。
→`'extension=redis.so'` `>> /etc/php.ini　`これかも 

おそらく、ララベルとかfwでphpモジュールインストールしたい場合はプロジェクトルートでcomposer使うのが良いと思われる。
composer使わないときに、docker-php-ext-installを使うとかかな？

でも、peclはphpのレイヤが違うからミドルウェアの感覚で良さそう。





`git clone --depth 1`
とかを使うと容量を削減できる。
https://qiita.com/naohikowatanabe/items/97cb57333f4e32e2d46b