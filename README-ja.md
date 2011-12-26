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

## 基本
In this section, you'll learn the essentials about Backbone's models, views, collections and routers. 
Whilst this isn't meant as a replacement for the official documentation, 
it will help you understand many of the core concepts behind Backbone before we build mobile applications with it. I will also be covering some tips on effective namespacing.
この節では、BackboneのModel, View, CollectionとRouterについて学んでもらいたい。
公式のドキュメンテーションの代替になる訳ではないが、Backboneをつかってモバイルアプリケーションを作る前にその核の概念を知ることに手助けになると思う。
ここでは効果的な名前空間についても少し触れてみようと思う。

* Model
* Collections
* Routers
* View
* Namespacing


###Models

Backbone models contain interactive data for an application as well as the logic around this data. 
For example, we can use a model to represent the concept of a photo object including its attributes like tags, titles and a location.
BackboneのModelはアプリケーションのためのインタラクティブなデータだけでなく、そのデータに関わるロジックも含んでいる。
例えば、タグ・タイトル・ロケーションなどの属性を含んだ写真オブジェクトをモデルとして表現することができる。

Models are quite straight-forward to create and can be constructed by extending `Backbone.Model` as follows:
Modelの生成は簡単で、`Backbone.Model`をextendすることで以下のように作ることができる:

```javascript
Photo = Backbone.Model.extend({
    defaults: {
        src: 'placeholder.jpg',
        title: 'an image placeholder',
        coordinates: [0,0]
    },
    initialize: function(){
        this.bind("change:src", function(){
            var src = this.get("src"); 
            console.log('Image source updated to ' + src);
        });
    },
    changeSrc: function( source ){
        this.set({ src: source });
    }
});
 
var somePhoto = new Photo({ src: "test.jpg", title:"testing"});
somePhoto.changeSrc("magic.jpg"); // which triggers "change:src" and logs an update message to the console.

```


####初期化

The `initialize()` method is called when creating a new instance of a model. It's use is optional, however we'll be reviewing some reasons you may want to use it shortly.
`initialize()`メソッドはModelから新しいインスタンスを作る際に呼ばれる。このメソッドの使用はオプショナルだが、後ほどなぜそれを使った方がいいのか説明していきたいと思う。

```javascript
Photo = Backbone.Model.extend({
    initialize: function(){
        console.log('this model has been initialized');
    }
});
 
/*We can then create our own instance of a photo as follows:*/
var myPhoto = new Photo();
```


####Getters & Setters

**Model.get()**

`Model.get()` provides easy access to a model's attributes. Attributes which are passed through to the model on instantiation are instantly available for retrieval.
`Model.get()`はModelの属性へのアクセスを提供してくれる。属性はModelの初期化の際に渡され、すぐに使用可能となる。

```javascript
var myPhoto = new Photo({ title: "My awesome photo", 
                          src:"boston.jpg", 
                          location: "Boston", 
                          tags:['the big game', 'vacation']}),
                          
    title = myPhoto.get("title"), //my awesome photo
    location = myPhoto.get("location"), //Boston
    tags = myPhoto.get("tags"), // ['the big game','vacation']
    photoSrc = myPhoto.get("src"); //boston.jpg
```

Alternatively, if you wish to directly access all of the attributes in an model's instance directly, you can achieve this as follows:
また、modelのインスタンスのすべての属性に直接アクセスしたい場合は、以下のようにすることができる:

```javascript
var myAttributes = myPhoto.attributes;
console.log(myAttributes);
```

It is best practice to use `Model.set()` or direct instantiation to set the values of a model's attributes.
Modelの属性の値の設定は`Model.set()`を使うか、初期化の時にすることをお勧めする。

Accessing `Model.attributes` directly is generally discouraged. Instead, should you need to read or clone data, `Model.toJSON()` is recommended for this purpose. 
If you would like to access or copy a model's attributes for purposes such as JSON stringification (e.g. for serialization prior to being passed to a view), 
this can be achieved using `Model.toJSON()`:
`Model.attributes`へ直接アクセスすることは一般的に推奨されておらず、その代わりにデータを読んだりクローンしたい場合は、`Model.toJSON()`を用いることが好ましい。
JSONの文字列化(例えば, viewへ渡される前にシリアリゼーションをしたい場合)などを目的にモデルの属性にアクセスする場合は、`Model.toJSON()`を以下の用にして用いることができる。


```javascript
var myAttributes = myPhoto.toJSON();
console.log(myAttributes);
/* this returns { title: "My awesome photo", 
             src:"boston.jpg", 
             location: "Boston", 
             tags:['the big game', 'vacation']}*/
```
