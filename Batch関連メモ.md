Batchメモ

- app/Console/Commands配下に、Commandクラスを継承した形でクラスを作り、ある程度決まった形でクラスを作成 [参照①](https://se-tomo.com/2018/10/13/laravel-%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89/) [参照②](https://readouble.com/laravel/6.x/ja/artisan.html)
  - $signatureに実際に叩くコマンドを代入
    - `php artisan $signature`

- app/Console/Kernel.phpを見ると使用しているコマンドファイルがわかる
  - ここをみるに、Project側のみだけでなく、共通の機能はlvm-laravelの方で定義されている事がわかる。つまり、バッチ処理の内容を探していてProject-Hogeにない場合はLVMに飛ぶと存在する。



KC

```
php artisan master_cache:refresh
```

これはRedis関係ないっぽい？reolveしたあとのmaster_facade->makeで何をしているのか次第。以下のファイルに記載されている。

`project-data/project-KO/laravel/vendor/lvgs/laravel-lvm/src/Commands/MasterCacheRefresh.php`



以下のコマンドはRedisに保存している。コード見た感じファットなのでかなり時間がかかる。リファクタのやりがいがありそうなコードだった…ひたすらforで回して6層ネストしてる…バッチにするかデータの持ち方変えるともっと早くなるとは思うが…

```
php artisan search_available:register
```

`project-data/project-KC/laravel/app/Console/Commands/SearchAvailableRegisterCommand.php`

こちらのコードで使用しているRepositoryTrait.phpでなにやら複雑な設定ファイルを書いていて分かりづらいが、repositoryとして最終的にcache-configのbatch項目で設定された場所に配置するっぽい。（今回で言えばRedis）KOはわかりやすく書いているのにこの違いはなんだろう…



predisを使ったら成功したが、phpredisはだめ…なにこれ。しかもDevと他の環境はOSが違うからここに時間を掛ける必要もない気がする。検証環境のinstallにphp-redisを追加してとりあえずやって見れば良いのでは…という気持ち。どうせDevで使わないので…