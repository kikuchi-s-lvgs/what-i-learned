# laravel関係のメモ

## 環境構築

Laravelの環境構築の際、php-fpm, Nginx, mysqlなどが最低限必要になるけど、それを用意するやり方がいくつかある。
公式でサポートしているのが

- Homestead→Vagrantを利用。基本構成の他にもRedisとかメモキャッシュとかモンゴDBとかも用意されていてかなりファットな印象。→ただの開発環境を簡単に作りたい人はこれでも良いかもだけど、今更Vagrantとか触りたくないってのが個人的な意見。
- Composer→npmみたいなもの。phpの環境と、Devサーバーの機能がインストールされるが、mysqlなどのミドルは自分でローカルに用意する必要がある。ただ、本番運用を考えるとこれが良さそう。
- Laradock→公式じゃないけど一般的に広まっているのに、laradockがある。これは名前にLaraがついているけど、Laravel用ではなく、すべてのPHPフレームワークのためのDocker環境。らしい。docker-composeで必要なツールを立てていくスタイル。homesteadのdocker版。ただし、Laravel専用ではないので機能がこちらもファットになっていて本筋とは異なるところで時間を食う可能性がある。Dockerなどの知識がある人はこれでも良いかもしれないが、もはや自分でDockerfile書いても良さそう。どうせ本番ではlaradock使うわけではないだろうし。



## サービス

サービスコンテナ→インスタンスの管理を担当。

- インスタンスの生成方法をあらかじめ定義しておく
- 使う側はメソッドを呼ぶだけでインスタンスを生成したり既存のインスタンスを取得したりできる。



## Frontend

### Vueとの統合

Laravelでは、VueやReactとの統合の仕組みは整っていて、すぐに使用することができる。
SPAではない場合、基本はbladeのテンプレートを使用するがその中にid=appをもつタグとコンポーネントを仕込んでおくことでVueのコンポーネントを表示することができる。つまり、ルーティングはLaravelに任せつつ、HTML（Blade）を表示。その中で使用したいVueのコンポーネントを組み込んで置くことでVueを使用することができる。おそらく普通は、blade側で書くことは少なく、がわだけ作って後はVueで表示やメソッド実行をしていると思われる。Bladeの中にVueを埋め込んでいるイメージ。Vueについては、resource/js/components配下でリソースを管理。resource/js/api.jsでコンポーネント名と、どのVueファイルをマッピングするかを設定しているポイ。そのあたりを見れば把握できそう。SPAの場合は、一枚だけHTMLとしてBladeをルーティングで表示するが、中身はid=appでVueを表示する。vueの中でaxiosでLaravelのAPIを叩き、データを貰うイメージ。Laravel側では、web.phpではなく、api.phpにルーティングを書くとRESTAPIのルーティングが実装できる。本来の意味でSPAするならもはやLaravelはAPIだけ開放するだけでも良さそう。フロントはS3とかで。でも必ずしもそうではなさそうだった。



## Facade

BFFみたいなイメージ。前に立って、使用者は中を意識することなく呼び出せる。実際は、定義しておいたエイリアスなどを元に、Facadeクラスを呼び出して登録しておいたコンテナを返す。

__callStatic($method, $args)

で定義されていない関数が呼ばれた場合を定義。アクセス不能なメソッドに対して静的に実行しようとしたタイミングで呼ばれる。取得してきたコンテナをもとにこの引数のmethodとargsを使って実行する。

これでFacadeわかった気がする↓

https://reffect.co.jp/laravel/laravel-facade-understanding



サービスコンテナ、プロバイダーのあたりの知識はあると良さそう。。。後はConfig系かな



### .envやConfigを更新したときは…

```bash
php artisan cache:clear
php artisan config:cache
```

これで反映されるはず。それでもだめなら↓

```
 rm -f bootstrap/cache/config.php
```