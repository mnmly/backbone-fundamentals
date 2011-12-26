#Backbone.jsの基礎：
デスクトップとモバイルのためのモジュール型のJavaScriptアプリケーションを書く方法

[CC](http://creativecommons.org/licenses/by-nc-sa/3.0/)ライセンスで無料でリリースされた[Addy Osmani](http://twitter.com/addyosmani)による執筆中の解説である。
微調整などの貢献をしてくれているコミュニティの[メンバー](https://github.com/addyosmani/backbone-fundamentals/contributors)へ感謝を示す。

##インデックス

* 導入

* ####基本
* Model
* View
* Collection
* ルーター
* 名前空間
* その他のヒント

* ####上級
* モジュラーのJavaScript
* RequireJSとAMDによるモジュールの整理
* RequireJSのテキストプラグインとテンプレートは、外部の状態に保つ
* RequireJS OptimizerによるProductionのためBackboneアプリケーションの最適化
* 実用：AMD＆RequireJSとモジュール型Backboneアプリケーションの構築
* MediatorおよびファサードパターンでBackboneのデカップリング
* Backbone＆jQueryのモバイル
* 実用：Backbone＆モバイルjQueryを使ってモジュール型携帯アプリの構築

* ####テスト（TODO）
* Jasmineのの基礎
* Model
* View
* Collection
* ルーター

* ####Resources


In this mini-book, I'll be covering a complete run-down of Backbone.js; including models, views, collections and routers. I'll also be taking you through advanced topics like modular development with Backbone.js and AMD (with RequireJS), 
how to solve the routing problems with Backbone and jQuery Mobile, tips about scaffolding tools that can save time setting up your initial application and more.
このちっちゃい本では、Model, View, Collection, Routerなど、Backbone.jsの詳細な説明をしていきたいと思う。また、Backbone.jsとRequireJSを用いたAMDでのモジュール開発や、BackboneとjQuery Mobileでのルーティングの解決法や、初期のアプリケーションの設定を円滑にしてくれるScafoldingツールについての説明など、高度なトピックも扱う予定にしている。

Backbone is maintained by a number of contributors, most notably: Jeremy Ashkenas, creator of CoffeeScript, Docco and Underscore.js. As Jeremy is a believer in detailed documentation, 
there's a level of comfort in knowing you're unlikely to run into issues which are either not explained in the official docs or which can't be nailed down with some assistance from the #documentcloud IRC channel.
I strongly recommend using the latter if you find yourself getting stuck.
Backboneは CoffeeScript, Docco, Underscore.jsの作者であるJeremy Ashkenasを筆頭として多くのコントリビュータによって開発されている。Jeremyは詳細なドキュメンテーションの支持者であるため、
公式のドキュメンテーションで説明されていないことや#documentcloudのIRCチャンネルでもわからないなどといった問題には陥らないと思う。
もし本当につまづいてしまった時には、IRCを用いてみることを強くお勧めする。

###Backbone.jsを考慮に入れる理由とは？
Backboneの主な利点とは、ターゲットとしているプラットフォームやデバイスに関係なく、以下のことを支援してくれる

* アプリケーションの構造の整理
* サーバサイドのパーシスタンスの簡素化
* DOMとデータのデカップリング
* Model, View, Routerの簡素な仕組み
* DOMとModel, Collectionの同期化

最大の疑問は、なぜJavaScriptプロジェクトにMVCパターンを導入する必要があるのかということであるが、一言で言えば構造の簡素化である。

もし大規模なアプリケーションをつくるためにjQueryやzepto、ほかのquerySelectorAllベースのセレクタライブラリを使う場合、 ものすごく簡単に、扱いにくい量のコードになってしまうことは目に見えています。つまり、どのようにアプリケーションを構造化して整理するかというプランは非常に重要だということだ。 その懸念をModel, View, Controller(もしくは Router)に分離することで、この問題を解決することができる。

MVVM(Model-View ViewModel)パターンやモジュールパターンやその他の方法でアプリケーションを構築した経験がある場合、 同時にそれらが有効であるだけでなく、あなた自身が何をしているかを把握している必要があるということを覚えておいて頂きたい。 多くの1ページのアプリケーションにとって、MVCパターンが最も適していると考えており、Backboneは我々のニーズを満たしてくれる。


