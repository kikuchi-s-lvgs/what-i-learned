Prod/Stgで適用されているのは

```
METHOD_CACHE_DRIVER
BATCH_CACHE_DRIVER
SESSION_DRIVER
```

の項目がRedisとなっている。他の環境はこの部分がFileになっている。Fileの場合は、Local環境の、storage配下に配置されるはず。

`SESSION_DRIVER` をRedisにする場合はSESSION_REDISの設定も必要。ただし、使い分けていない場合は、普通のRedisのHostやPassなどとおなじになるはず。

このかんきょうへんすうは、config/database.phpにある。クラスター構成でdefault/sessionで一応向き先を分けている。が、実際は同じREDIS。



SessionDriverに関しては、config/session.phpに記載。

Batchキャッシュやメソッドキャッシュについてはconfig/cache/phpに記載。



### Redisの接続情報とClientの設定

https://readouble.com/laravel/6.x/ja/redis.html

**バッチだとしてもRedisの基本的な設定(Clientやhost, port, passなど)は、config/database.phpに記載するので注意。**

# Batch

カスタムのartisanコマンドは`app/Console/Commands`配下に置くと使用できる。

バッチ処理に関しても、ここにバッチ処理のコマンドを作成して、 artisanコマンドでバッチを走らせているぽい。

KOとKCで仕組みが違うので注意。KOでは、わかりやすく、Cacheクラスのstore関数を'batch'引数で実行、みたいな感じ。Configで設定しているbatchの内容でデータを保存。独自のキャッシュを用意していたりするが、基本はこのやり方なので追えばわかりそう。KCはちょっと複雑そうだった…Repositoryのtraitを使用していたり？ぽい。ただ、設定値は基本的には同じになっているはずなので共通でOK。

Cacheは大本がInterfaceとして定義されていて、get/set/delete/clearなどが存在。これを実装しつつ、中身はfileだったり、redisだったりで使い方が異なるのだと思われる。Redisを使うにしても、database.phpの設定ファイルに書いた方は、クライアントとしてphpredisを使ったりしていて、Driverがそもそも違うので使い方は大きく異なるはず。cacheの方は使用用途が限られるので（基本はKVSとしての使用のみ）、Laravel側ですでにそれっぽいのを実装しているイメージ。



## 料金試算

| サービス    | vCPU | メモリ | 月額料金 |
| ----------- | ---- | ------ | -------- |
| Elasticache | 2    | 1.37   | 4,039    |
| Fargate     | 1    | 2      | 4,795    |
| FargateSpot | 1    | 2      | 1,785    |
| EC2         | 2    | 1      | 1,058    |
| EC2(Spot)   | 2    | 1      | 317      |



### 非機能要件

- 今回は検証用環境なのでフェイルオーバーなどは不要
- 勤務時間さえ稼働できればOK（土日は不要）
- 可用性はあまり求めないものの、揮発したあとに再度バッチを簡単に走らせる仕組みがほしい

### Spotを使用する際は

- 中断2分前に警告が来るのでそれを拾って違うAZやファミリーのスポットインスタンスを起動するか、とりあえずオンデマンドを起動させて、あとからスポットの入札価格を変更する。
- ポイントは、
  - 中断されたことを検知できるか
  - バッチを必要なときに簡単に走らせられる仕組みがあるか



以下のDockerfileをビルドして中に入ると

```dockerfile
FROM        ubuntu:18.04
RUN         apt-get update && apt-get install -y redis-server
EXPOSE      6379
ENTRYPOINT  ["/usr/bin/redis-server"]
```

/etc/redis配下にredis.confが置かれていた。権限はredis:redisになっている。620の権限。defautlのまま、saveすると、ルート(/)直下にdump.rdbが作成されていた。



ちなみに公式DockerfileはDebianの模様。ディレクトリ構造などが異なる。



redisは起動時に第一引数としてパスを渡すとそれが設定ファイルとなる。



設定ファイルを読み解く

ベースはこれhttps://qiita.com/uryyyyyyy/items/9ccadcccf7f7060d544a

```
スナップショットの頻度のデフォルト値。1つ以上のキーが更新されたとき、900秒後にRDBへスナップショット。今回はバッチなので一気に大量の処理が書かれるはずかつ、処理に時間かかるのでこのデフォルトでも良いかと。
save 900 1
save 300 10
save 60 10000
```



変更箇所



memo 

Spotでt3microで起動したらメモリ不足でしぬ…しかもSpotだから停止できない。。

**オンデマンドでt3smallで起動してみる。smallのspotで起動することを確認済。**

TODO: この起動済のspot のスナップショットを取得して、もう一つのスポットをそこから作成し、Redisを載せてデータがリストアされるか確認。

→できた。意図せずだけど、t2.smallでも行けた。

念のためmicroでやってみる…無理だった。ECSで動かすにはmicroでは足りないっぽい。（alpineだったらいいかもしれんが…）

ここまでできればあとは外部からAPI立たけるようにすることと、自動で復旧させるあたりの仕組み。それをCDKにすればOK.そう。

他のホストからRedisのAPI（CLI）叩いたらエラーになったから設定値いじる必要ありそう。エラーはこれ。アドレスをBindするか、protected modeをnoに設定しろとのこと。

```
(error) DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In th
is mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) J
ust disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, ho
wever MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the prot
ected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just
 for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in o
rder for the server to start accepting connections from the outside.
```



スポットインスタンスはMax値だけ指定して入札する。それを超えたらアウト。値段は随時変わる。その値段が請求される。事実上、Max値をオンデマンドと同じにしておけば死なないしそれまでは安くなるからそれで良い気もする…

> 上限料金を指定しない場合、デフォルトの上限料金はオンデマンド価格となります｡

Maxをオンデマンドにしておけば中断はめったに起きないので中断したら自動でインスタンス建てるほどの仕組みは不要かも。通知くらいはほしいけど。なので、とりあえずはRedisのバックアップ兼EBSのバックアップ取得までできていれば良い気がする。できればAMIか起動テンプレートまであるとよいか…？くらい。

CWEでそういうルールがすでにあるっぽいので、それをLamdbaとかに飛ばしてSlackすれば通知にはなる。あるいはLambdaでオンデマンドを作ることはできそうだが、最新のスナップショットを取得して起動するのは面倒そう。



必要なもの

- ECR
- ECSクラスター
- サービス（サービスディスカバリ含む）
- タスク定義
- 起動テンプレート(初回からこれを使って、中断したらスナップショットのIDだけ変えて更新とか…？)
- スナップショットのライフサイクルマネージャー
- CWEによる監視、通知
- 