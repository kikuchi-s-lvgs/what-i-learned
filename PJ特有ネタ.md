- Twigなのは以前からTwigだったから（laravelを使用する前から）
- パッケージとしてlaravel-lvmが存在していて、アプリ側はそれを使用しているに過ぎない。基本機能はlaravel-lvmが提供してくれているので、アプリ側のディレクトリの中身が薄っぺらっかたりする。（アプリ特有の機能や、View関係しか実装しないイメージ。）
- laravel-lvmではテストを回しているが、アプリ側のko,kcでは特にUnitテストなどはしていない。したがって、laravel-lvmに関係ない実装をする際は回すテストが無いので目視確認をある程度したらマージされるらしい。
  - これはなんとかしたい部分…



## Views

- sp/pc/commonあたりを主に触るはず
- fpはフィーチャーフォン、legacyはサポート終了したブラウザで表示されるディレクトリのこと。基本は無視で良いが、それらのページでも実装されている、フォームの送信関係の機能に手を加えるときは触ることがあるので注意。
- notificationはメール関係
- simpleは使用していない
- xmlはサイトマップを構成している。基本は触らない。バッチで自動生成したりしている。
- クライアント側でぬるぬるさせたいときは部分的にVueを使用している。また、apiでサーバにデータを要求するときもvueの中でやっている。twigはあくまでテンプレートエンジン。サーバーからデータを渡してそれを性的なコンテンツとして表示したいときはシンプルにtwig。
- viewの名前とコントローラ、ルーティングはセットになっている
- vueファイルは実質scriptしか書いていないのでvueの記法を使用したJSファイルと考えても差し支えない。セットになっているhtmlファイルにレイアウトが書かれている。
- なぜSFCで統一しないの？
  - デザイナーもコードを触ることが多く、JSの知識はないがhtmlやcssのクラスをいじることはある。つまり、SFCにしてしまうと、jsの記述を目にして戸惑ってしまうことがあるので、ファイルごと分けてしまっている。とのことらしい。
- なぜSPAじゃないの？
  - SEOの問題。クローラがHTMLを取得してからJSを実行するには多少の時間がかかると言われているので、できるだけクローラが取得した時点ですべての要素がレンダリングされていることが望ましいため。
  - したがってSEO対策の観点からSSRは欠かせない。で、Laravelを使用しているのでそのままLaravelでレンダリングしている。
  - 逆に言うと、DFとかならSPAにしてもよいかも、とのこと。
  - **アイデア：Nuxtを導入してLaravelはただのAPIサーバーにできない？**



## CICD関連

- featureがPrefixにつくブランチがPushされるとおそらくCircleCI連携（？）でCodeBuildに処理が移る。
- ビルドスクリプトの中で、composer installとかyarn prodとかして、きれいな環境でProd用のビルドを走らせる。その中でDockerイメージも作成してECRにプッシュしている。
- ECRにプッシュしたイメージを使用するタスク定義を作成して、クラスターに対して直接タスクの起動を指示。（おそらく）PRがクローズされるまで、タスクが動き続ける。
- PR上にECSで公開しているIPがはられて誰でも見に行ける。
- コンテナインスタンスは1台の模様。そのせいで1つのコンテナインスタンスに10個以上のタスクが走っている状況となっている。
  - これは問題ないのだろうか時になるが、検証環境のURL上での操作が問題ないのであればコスト的にも優しいのでOKそう。
- 問題はビルドがかなり遅いこと。結構待つ気がする。これはcodebuildのスクリプトを工夫するか、設定でキャッシュをOnにするとかする必要はあるが、キャッシュして変更が反映されないと意味ないし、チューニングが難しそう。ただ、ここはなんとかしたい気がする…
- ちなみにmasterブランチへのPRを作成すると、同時にstg-envというPrefixとともに新しいPRが作成される。常に、元のPRを追従しているぽい。このPRをマージすると、STGへリリースされるらしい。このへんの仕組はあまりわかってないけど、CircleCIでごりごり書いている気がする。



## テスト

やっぱUTの仕組みないとちょっと怖い…

検証環境は同じPRであれば同じIPとポートで上がってくる。

見た目の変更や、ディレクターとの合意を取りたいとき以外は基本はレビューの際に検証環境は使わなくて良いらしい。Lintが通ったらレビュー依頼してOK。



Redisとかがまさにそうらしいけど、MedicalのAWSアカウントにまだ残っているものもあるらしい。エンドポイントとか探してlmcになかったらMedicalを見ると良さそう。

