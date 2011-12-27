**この翻訳はまだまだ完全ではないので気をつけてください**

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

####Model.set()

`Model.set()` allows us to pass attributes into an instance of our model. Attributes can either be set during initialization or later on.
`Model.set()`を用いることでModelのインスタンスに値を与えることができる。属性は初期化時とその後にも設定することが可能である。

```javascript
Photo = Backbone.Model.extend({
    initialize: function(){
        console.log('this model has been initialized');
    }
});
 
/*Setting the value of attributes via instantiation*/
var myPhoto = new Photo({ title: 'My awesome photo', location: 'Boston' });
 
var myPhoto2 = new Photo();

/*Setting the value of attributes through Model.set()*/
myPhoto2.set({ title:'Vacation in Florida', location: 'Florida' });
```

**Default values**
**デフォルト値**

There are times when you want your model to have a set of default values (e.g. in a scenario where a complete set of data isn't provided by the user).  This can be set using a property called `defaults` in your model.
Modelにデフォルト値をセットしたい場合も出てくる。(例えば、ユーザからのデータが完璧に設定されない場合) その場合は、Model内の`defaults`プロパティを用いることでそれを設定することができる。

```javascript
Photo = Backbone.Model.extend({
    defaults:{
        title: 'Another photo!',
        tags:  ['untagged'],
        location: 'home',
        src: 'placeholder.jpg'
    },
    initialize: function(){
    }
});
 
var myPhoto = new Photo({ location: "Boston", 
                          tags:['the big game', 'vacation']}),
    title   = myPhoto.get("title"), //Another photo!
    location = myPhoto.get("location"), //Boston
    tags = myPhoto.get("tags"), // ['the big game','vacation']
    photoSrc = myPhoto.get("src"); //placeholder.jpg
```

**モデルへの変更を監視する**

Any and all of the attributes in a Backbone model can have listeners bound to them which detect when their values change.
This can be easily added to the `initialize()` function as follows:
BackboneのModelのすべての属性に対してその値が変化した際のイベントにバインドすることができる。これは`initialize()`で以下のように追加することができる。

```javascript
this.bind('change', function(){
    console.log('values for this model have changed');
});
```

In the following example, we can also log a message whenever a specific attribute (the title of our Photo model) is altered.
以下の例では、特定の属性(Photo Modelのtitle)が変化した際にメッセージをログしている。

```javascript
Photo = Backbone.Model.extend({
    defaults:{
        title: 'Another photo!',
        tags:  ['untagged'],
        location: 'home',
        src: 'placeholder.jpg'
    },
    initialize: function(){
        console.log('this model has been initialized');
        this.bind("change:title", function(){
            var title = this.get("title");
            console.log("My title has been changed to.." + title);
        });
    },
    
    setTitle: function(newTitle){
        this.set({ title: newTitle });
    }
});
 
var myPhoto = new Photo({ title:"Fishing at the lake", src:"fishing.jpg")});
myPhoto.setTitle('Fishing at sea'); 
//logs my title has been changed to.. Fishing at sea
```

**バリデーション**

Backbone supports model validation through `Model.validate()`, which allows checking the attribute values for a model prior to them being set.
Backboneでは`Model.validate()`を用いてバリデーションをかけることができ、モデルに値が設定される前にその属性値の値をチェックすることができる。

It supports including as complex or terse validation rules against attributes and is quite straight-forward to use.
If the attributes provided are valid, nothing should be returned from `.validate()`, however if they are invalid a custom error can be returned instead.
属性に対し、その複雑さに関わらずバリデーションを与えることができ、その使い方も簡単である。
もし、設定された属性が有効な場合は、`.validate()`からは何も返されないが、もし有効でない場合はカスタム可能なエラーを返してくる。

A basic example for validation can be seen below:
基本的な例は以下の通りである:

```javascript
Photo = Backbone.Model.extend({
    validate: function(attribs){
        if(attribs.src == "placeholder.jpg"){
            return "Remember to set a source for your image!";
        }
    },
    
    initialize: function(){
        console.log('this model has been initialized');
        this.bind("error", function(model, error){
            console.log(error);
        });
    }
});
 
var myPhoto = new Photo();
myPhoto.set({ title: "On the beach" });
```


###Views

Views in Backbone don't contain the markup for your application, but rather they are there to support models by defining how they should be visually represented to the user. 
This is usually achieved using JavaScript templating (e.g. Mustache, jQuery tmpl etc). When a model updates, rather than the entire page needing to be refreshed, we can simply bind a view's `render()` function to a model's `change()` event, allowing the view to always be up to date.
BackboneのViewはアプリケーションのマークアップは含んでおらず、ユーザに対してどのように視覚的に表現されるか定義することでmodelをサポートする役目を担っている。
これはJavaScriptテンプレーティング(Mustache, jQuery tmplなど)を用いることで達成できる。

####新しいViewを作る

Similar to the previous sections, creating a new view is relatively straight-forward. We simply extend `Backbone.View`.
Here's an example of a possible implementation of this, which I'll explain shortly:
前の節と同様に、新しいviewを作ることも比較的簡単である。`Backbone.View`をextendするだけである。
ここでは、これに考えられうる実装例を見てみよう, 説明は後ほど: 

```javascript
var PhotoSearch = Backbone.View.extend({
    el: $('#results'),
    render: function( event ){
        var compiled_template = _.template( $("#results-template").html() );
        this.el.html( compiled_template(this.model.toJSON()) );
        return this; //recommended as this enables calls to be chained.
    },
    events: {
        "submit #searchForm":  "search",
        "click .reset": "reset",
        "click .advanced": "switchContext"
    },
    search: function( event ){
        //executed when a form '#searchForm' has been submitted
    },
    reset: function( event ){
        //executed when an element with class "reset" has been clicked.
    },
    //etc
});
```

####What is `el`?
####`el`ってなに?

`el` is basically a reference to a DOM element and all views must have one, however Backbone allows you to specify this in four different ways. 
You can either directly use an `id`, a `tagName`, `className` or if you don't state anything `el` will simply default to a plain div element without any id or class.
Here are some quick examples of how these may be used:
`el`とは基本的にはDOMエレメントへの参照で、すべてのviewが持たなくてはならない。しかしBackboneはこれの設定方法について4つの方法を提供している。
`id`や`tagName`, `className`を用いるか、もし何も宣言しない場合は`el`は普通のidやclassを持たないただのdivとなる。
どの用にそれらが使われるか簡単な例を見てみよう:

```javascript
el: $('#results')  //select based on an ID or other valid jQuery selector.
tagName: 'li' //select based on a specific tag. Here el itself will be derived from the tagName
className: 'items' //similar to the above, this will also result in el being derived from it
el: '' //defaults to a div without an id, name or class.
```

**Note:** A combination of these methods can also be used to define `el`. For example:
**注意:** これらのメソッドのコンビネーションで`el`を定義することもできる。例えば:

```javascript
tagName: "li",
className: "container"
```
will use specific tags with a particular `className`.
これは、指定したタグと特定の`className`を使う。


**`render()`を理解する**

`render()` is an optional function to define how you would like a template to be rendered.
Backbone allows you to use any JavaScript templating solution of your choice for this but for the purposes of this book, we'll opt for Underscore's micro-templating.
`render()`はオプショナルな関数でテンプレートがどのようにレンダされるか指定するものである。
Backboneではどのテンプレーティングエンジンを使っても構わないが、この本の中では、Underscoreのテンプレーティングエンジンを使用したいと思う。

The `_.template` method in underscore compiles JavaScript templates into functions which can be evaluated for rendering. In the above view, 
I'm passing the markup from a template with id `results-template` to be compiled.
Next, I set the html for `el` (our DOM element for this view) the output of processing a JSON version of the model associated with the view through the compiled template.
Underscoreの`_.template`メソッドはJavaScriptテンプレートを関数にして、レンダリングの際に使える形にする。上のviewでは、id=`result-template`のマークアップをコンパイルされるように渡している。
次に、`el`(このViewのDOMエレメント)のhtmlを

Presto! This populates the template, giving you a data-complete set of markup in just a few short lines of code.
このように数行のコードでデータにバインドされたマークアップを作ることができる。

**The `events` attribute**
**`events`属性**

The Backbone `events` attribute allows us to attach event listeners to either custom selectors, or `el` if no selector is provided.
An event takes the form `{"eventName selector": "callbackFunction"}` and a number of event-types are supported, including 'click', 'submit', 'mouseover', 'dblclick' and more.
Backboneの`events`属性は指定したセレクタ、もしくは`el`そのものにイベントリスナーを付ける。イベントは`{"eventName selector": "callbackFunction"}`のフォーマットで指定され、
'click', 'submit', 'mouseover', 'dblclick' などのイベントタイプがサポートされている。

What isn't instantly obvious is that under the bonnet, Backbone uses jQuery's `.delegate()` to provide instant support for event delegation but goes a little further, 
extending it so that `this` always refers to the current view object. The only thing to really keep in mind is 
that any string callback supplied to the events attribute must have a corresponding function with the same name within the scope of your view
otherwise you may incur exceptions.
Backboneは基本的にはjQueryの`.delegate()`を使ってイベントデリゲーションをしているが、それをさらに拡張して`this`が常に現在のviewオブジェクトとなるように実装されている。
気をつけなくてはいけないのは、`events`属性で設定された文字列のコールバックはそのviewのスコープ内で同じ名前で宣言されていなければならない。そのコールバックが存在しない場合はExceptionが投げられる。

###Collections

Collections are basically sets of models and can be easily created by extending `Backbone.Collection`.
Collectionは基本的にはModelのセットで、`Backbone.Collection`をextendすることで作ることができる。

Normally, when creating a collection you'll also want to pass through a property specifying the model that your collection will contain
as well as any instance properties required.
Collectionを作るときには、Collectionの持つModelとその他のプロパティを指定したものを渡す。

In the following example, we're creating a PhotoCollection containing the Photo models we previously defined.
次の例では、前回定義した`Photo`Modelを格納する`PhotoCollection`を定義している。

```javascript
PhotoCollection = Backbone.Collection.extend({
    model: Photo
});
```

**Getters and Setters**

There are a few different options for retrieving a model from a collection. The most straight-forward is using `Collection.get()` which accepts a single id as follows:
CollectionからModelを取り出す方法はいくつか用意されている。もっともわかりやすいものは`Collection.get()`であり、以下のように一つのidを受け取る:

```javascript
var skiingEpicness = PhotoCollection.get(2);
```

Sometimes you may also want to get a model based on something called the client id. This is an id that is internally assigned automatically
when creating models that have not yet been saved, should you need to reference them.
You can find out what a model's client id is by accessing its `.cid` property.
client idと呼ばれるidでmodelを取り出したいときもあるかもしれない。client idとはまだsaveされていないmodelが, 生成時に自動的に割り当てられるidで、それを使って参照することも可能である。
modelのclient idは`.cid`プロパティにアクセスすることができる。

```javascript
var mySkiingCrash = PhotoCollection.getByCid(456);
```

Backbone Collections don't have setters as such, but do support adding new models via `.add()` and removing models via `.remove()`.
BackboneのCollectionはsetterというものを持たないが、新しいmodelを追加する場合は`.add()`を用い、削除する際は`.remove()`を用いる。

```javascript
var a = new Backbone.Model({ title: 'my vacation'}),
    b = new Backbone.Model({ title: 'my holiday'});

var photoCollection = new PhotoCollection([a,b]);
photoCollection.remove([a,b]);
```

**イベントを監視する**

As collections represent a group of items, we're also able to listen for `add` and `remove` events for when new models are added or removed from the collection.  Here's an example:
Collectionはアイテムのまとまりであるため、modelが追加もしくは削除されたときに呼ばれる`add`や`remove`イベントを監視することができる。例はこちら:

```javascript
var PhotoCollection = new Backbone.Collection;
PhotoCollection.bind("add", function(photo) {
  console.log("I liked " + photo.get("title") + ' its this one, right? '  + photo.src);
});
 
PhotoCollection.add([
  {title: "My trip to Bali", src: "bali-trip.jpg"},
  {title: "The flight home", src: "long-flight-oofta.jpg"},
  {title: "Uploading pix", src: "too-many-pics.jpg"}
]);
```

In addition, we're able to bind a `change` event to listen for changes to models in the collection.
さらに、collection内のモデルが変更された際に出される`change`イベントも同様である。

```javascript
PhotoCollection.bind("change:title", function(){
    console.log('there have been updates made to this collections titles');    
});
```

**Fetching models from the server**
**サーバからModelを読み込む**

`Collections.fetch()` provides you with a simple way to fetch a default set of models from the server in the form of a JSON array.
When this data returns, the current collection will refresh.
`Collections.fetch()`によって、JSONの配列の形でサーバからデフォルトのアイテムセットを受け取ることができる。このデータを受け取ると、現在のCollectionはrefreshされる。

```javascript
var PhotoCollection = new Backbone.Collection;
PhotoCollection.url = '/photos';
PhotoCollection.fetch();
```

Under the covers, `Backbone.sync` is the function called every time Backbone tries to read (or save) models to the server.
It uses jQuery or Zepto's ajax implementations to make these RESTful requests, however this can be overridden as per your needs.
`Backbone.sync`はBackboneがサーバから読み込んだり、セーブするときに呼ばれる関数であり、jQueryやZeptoのajaxの実装を用いてRESTfulなリクエストを行うが、 しかし、それはオーバーライドすることも可能である。

In the above fetch example if we wish to log an event when `.sync()` gets called, we can simply achieve this as follows:
上の読み込みの例で、もし`.sync()`が呼ばれる際にそのイベントをlogしたい場合は、以下のようにすることでそれが可能になる。

```javascript
Backbone.sync = function(method, model) {
  console.log("I've been passed " + method + " with " + JSON.stringify(model));
};
```

**Resetting/Refreshing Collections**
**Collectionのリセットとリフレッシュ**
Rather than adding or removing models individually, you occasionally wish to update an entire collection at once.
`Collection.reset()` allows us to replace an entire collection with new models as follows:
一つ一つ追加したり、削除するよりも、一度にCollection全体を更新する機会の方が多いと思う。
`Collection.reset()`を使うことで、以下のようにコレクション全体を置き換えることができる:

```javascript
PhotoCollection.reset([
  {title: "My trip to Scotland", src: "scotland-trip.jpg"},
  {title: "The flight from Scotland", src: "long-flight.jpg"},
  {title: "Latest snap of lock-ness", src: "lockness.jpg"}]);
```

###Underscoreのユーティリティ関数

As Backbone requires Underscore as a hard dependency,
we're able to use many of the utilities it has to offer to aid with our application development.
Here's an example of how Underscore's `sortBy()` method can be used to sort a collection of photos based on a particular attribute.
BackboneはUnderscoreと強い依存関係を持つため,アプリケーション開発を援助してくれる多くのユーティリティを使うことができる。
以下の例では、Underscoreの`sortBy()`メソッドを用いて特定の属性を元にPhotoCollectionをソートする例を示す:

```javascript
var sortedByAlphabet = PhotoCollection.sortBy(function(photo)){
    return photo.get("title").toLowerCase();
});
```
The complete list of what it can do is beyond the scope of this guide, but can be found in the official docs.
Underscoreのユーティリティをすべてカバーすることはこの本の範囲外であるため、公式のドキュメンテーションを参照してもらいたい。

###Routers

In Backbone, routers are used to handle routing for your application.
This is achieved using hash-tags with URL fragments which you can read more about if you wish. Some examples of valid routes may be seen below:
BackboneではRoutersはアプリケーションのルーティングに用いられる。これはハッシュタグを用いたURLフラグメントで達成される。以下にrouteの例を示す:

```javascript
http://unicorns.com/#/whatsup
http://unicorns.com/#/search/seasonal-horns/page2
```

Note: A router will usually have at least one URL route defined as well as a function that maps what happens when you reach that particular route.  This type of key/value pair may resemble:
注意: Routerは少なくとも一つのURLとそのURLにアクセスされた際に呼ばれる関数が定義されている必要がある。このタイプのkey/valueペアは似ている

```javascript
"/route" : "mappedFunction"
```

Let us now define our first controller by extending `Backbone.Router`.
For the purposes of this guide, we're going to continue pretending we're creating a photo gallery application that requires a GalleryRouter.
`Backbone.Router`をextendして最初のコントローラを定義してみよう。
フォトギャラリーアプリケーションを作っていると仮定して、GalleryRouterを定義をする。


Note the inline comments in the code example below as they continue the rest of the lesson on routers.
注意:以下の例のコードのインラインコメントは???

```javascript
GalleryRouter = Backbone.Router.extend({
    /* define the route and function maps for this router */
    routes:{
        "/about" : "showAbout",
        /*Sample usage: http://unicorns.com/#/about"*/
        
        "/photos/:id" : "getPhoto",
        /*This is an example of using a ":param" variable which allows us to match 
        any of the components between two URL slashes*/
        /*Sample usage: http://unicorns.com/#/photos/5*/
        
        "/search/:query" : "searchPhotos"
        /*We can also define multiple routes that are bound to the same map function,
        in this case searchPhotos(). Note below how we're optionally passing in a 
        reference to a page number if one is supplied*/
        /*Sample usage: http://unicorns.com/#/search/lolcats*/
         
        "/search/:query/p:page" : "searchPhotos",
        /*As we can see, URLs may contain as many ":param"s as we wish*/
        /*Sample usage: http://unicorns.com/#/search/lolcats/p1*/
        
        "/photos/:id/download/*imagePath" : "downloadPhoto",
        /*This is an example of using a *splat. splats are able to match any number of 
        URL components and can be combined with ":param"s*/
        /*Sample usage: http://unicorns.com/#/photos/5/download/files/lolcat-car.jpg*/
        
        /*If you wish to use splats for anything beyond default routing, it's probably a good 
        idea to leave them at the end of a URL otherwise you may need to apply regular
        expression parsing on your fragment*/
         
        "*other"    : "defaultRoute"
        //This is a default route with that also uses a *splat. Consider the
        //default route a wildcard for URLs that are either not matched or where
        //the user has incorrectly typed in a route path manually
        /*Sample usage: http://unicorns.com/#/anything*/
 
    },
    
    showAbout: function(){
    },
    
    getPhoto: function(id){
        /* 
        in this case, the id matched in the above route will be passed through
        to our function getPhoto and we can then use this as we please.
        */
        console.log("You are trying to reach photo " + id);
    },
    
    searchPhotos: function(query, page){
        console.log("Page number: " + page + " of the results for " + query);
    },
    
    downloadPhoto: function(id, path){
    },
    
    defaultRoute(other){
        console.log("Invalid. You attempted to reach:" + other);
    }
});
 
/* Now that we have a router setup, remember to instantiate it*/
 
var myGalleryRouter = new GalleryRouter;
```

Note: In Backbone 0.5+, it's possible to opt-in for HTML5 pushState support via `window.history.pushState`. 
This effectively permits non-hashtag routes such as http://www.scriptjunkie.com/just/an/example to be supported with automatic degradation should your browser not support it.
For the purposes of this tutorial, we won't be relying on this newer functionality as there have been reports about issues with it under iOS/Mobile Safari.
Backbone's hash-based routes should however suffice for our needs.
注意: Backbone 0.5以上では、HTML5の`window.history.pushState`をサポートしている。
これによってhttp://www.scriptjunkie.com/just/an/exampleのようなハッシュタグを用いないルーティングをサポートでき、ブラウザがpushStateをサポートしていない場合は自動的にハッシュタグのルーティングに変更される。
だたiOS/Mobile Safariではこの新しい機能において問題が報告されているため、このチュートリアルではこれを使わないことにする。
今のところBackboneのハッシュタグベースのルートは我々のニーズを満たしている。 

####Backbone.history

Next, we need to initialize `Backbone.history` as it handles `hashchange` events in our application.
This will automatically handle routes that have been defined and trigger callbacks when they've been accessed.
次に、`hashchange`イベントを処理するために`Backbone.history`を初期化する必要がある。 これは定義されたルートを処理し、そのURLにアクセスされた際に自動的にコールバックを呼ぶものである。

The `Backbone.history.start()` method will simply tell Backbone that it's OK to begin monitoring all `hashchange` events as follows:
`Backbone.history.start()`は単にBackboneに`hashchange`を監視するように伝えるメソッドである:

```javascript
Backbone.history.start();
Controller.saveLocation();
```

As an aside, if you would like to save application state to the URL at a particular point you can use the `.saveLocation()` method to achieve this.
It simply updates your URL fragment without the need to trigger the `hashchange` event.
もし特定のポイントでアプリケーションの状態を保存したいときは、`.saveLocation()`メソッドを使うことでそれを達成することができる。
それは単純に`hashchange`イベントを発信せずに、URLフラグメントを更新するものである。


```javascript
/*Lets imagine we would like a specific fragment for when a user zooms into a photo*/
zoomPhoto: function(factor){
    this.zoom(factor); //imagine this zooms into the image
    this.saveLocation("zoom/" + factor); //updates the fragment for us
}
```


###名前空間

When learning how to use Backbone, an important and commonly overlooked area by tutorials is namespacing.
If you already have experience with namespacing in JavaScript,
the following section will provide some advice on how to specifically apply concepts you know to Backbone,
however I will also be covering explanations for beginners to ensure everyone is on the same page.
Backboneの使い方を学ぶ際に、多くのチュートリアルでよく見過ごされているのは名前空間である。
もしJavaScriptにおける名前空間を使った経験がある場合は、次の節でその概念をBackboneに応用する方法を説明したいと思うが、初心者の方々も同じ理解度が得られるように基本的な説明も加えたいと思う。

####名前空間とは何か？

The basic idea around namespacing is to avoid collisions with other objects or variables in the global namespace.
They're important as it's best to safeguard your code from breaking in the event of another script on the page using the same variable names as you are.
As a good 'citizen' of the global namespace, it's also imperative that you do your best to similarly not prevent other developer's scripts executing due to the same issues.
名前空間の基本的な目的は、グローバル空間での他のオブジェクトや変数との衝突を防ぐことにある。
こうすることで同じページ内に読み込まれた他のスクリプトが同じ変数名を持っていた場合でもコードを停止させること無く実行させることができる。
同様に、他のデベロッパのスクリプトを停止することを防ぐためにもこの概念が非常に重要となる。


JavaScript doesn't really have built-in support for namespaces like other languages, however it does have closures which can be used to achieve a similar effect.
JavaScriptは他の言語のように、名前空間をネイティブにサポートしていないが、同様の効果を得るためにclosureを使うことができる。

In this section we'll be taking a look shortly at some examples of how you can namespace your models, views, routers and other components specifically. 
The patterns we'll be examining are:
この節では、Model, View, Routerやその他のコンポーネントの名前空間を定義する方法を説明していく。このパターンでは以下のことを検証???

* 一つのグローバル変数
* オブジェクトリテラル
* ネストされた名前空間

**一つのグローバル変数**

One popular pattern for namespacing in JavaScript is opting for a single global variable as your primary object of reference.
A skeleton implementation of this where we return an object with functions and properties can be found below:
JavaScriptでよく用いられる名前空間パターンは主要なオブジェクトとして一つのグローバル変数を選ぶ方法である。
関数とプロパティを持つオブジェクトを返すスケルトン実装を以下に示す。

```javascript
var myApplication =  (function(){ 
        function(){
            ...
        },
        return{
            ...
        }
})();
```

which you're likely to have seen before. A Backbone-specific example which may be more useful is:
これは今までに見たことあると思う。Backboneに応用した例は以下のようになる:

```javascript
var myViews = (function(){
    return {
        PhotoView: Backbone.View.extend({ .. }),
        GalleryView: Backbone.View.extend({ .. }),
        AboutView: Backbone.View.extend({ .. });
        //etc.
    };
})();
```

Here we can return a set of views or even an entire collection of models, views and routers depending on how you decide to structure your application.
Although this works for certain situations, the biggest challenge with the single global variable pattern is ensuring that no one else has used the same global variable name as you have in the page.
これはある特定の状況ではうまくいくかもしれないが、このパターンの最も大きな課題は他の誰もが同じグローバル変数名を使っていないことを保証しなくてはいけないことである。

One solution to this problem, as mentioned by Peter Michaux, is to use prefix namespacing.
It's a simple concept at heart, but the idea is you select a basic prefix namespace you wish to use (in this example, `myApplication_`) and then define any methods,
variables or other objects after the prefix.
この問題に対する一つの対応策として、Peter Michauxが提案した、接頭辞をつけた名前空間を使う方法である。
これは非常にシンプルな概念であるが、基本的な接頭辞(この例では`myApplication_`)を選び、そしてメソッド、変数そしてその他のオブジェクトそその接頭辞の後に宣言する。

```javascript
var myApplication_photoView = Backbone.View.extend({}),
myApplication_galleryView = Backbone.View.extend({});
```

This is effective from the perspective of trying to lower the chances of a particular variable existing in the global scope,
but remember that a uniquely named object can have the same effect.
This aside, the biggest issue with the pattern is that it can result in a large number of global objects once your application starts to grow.
これは特定の変数がグローバル空間に存在する確率を低くする点では効果的であるが、これもまたユニークに宣言されたオブジェクトそのものと同様の効果でしかない。

For more on Peter's views about the single global variable pattern, read his [excellent post on them]().
一つのグローバル変数パターンにおけるPeter氏の考察をより良く知りたい方は、彼のすばらしい[記事](http://michaux.ca/articles/javascript-namespacing)を読んでいただきたい。

Note: There are several other variations on the single global variable pattern out in the wild, however having reviewed quite a few, I felt these applied best to Backbone.
注意: 他にもこのシングルグローバル変数パターンの種類が示されているが、いくつかのものをレビューした結果これが一番Backboneに合っていると考えた。

**オブジェクトリテラル**
Object Literals have the advantage of not polluting the global namespace but assist in organizing code and parameters logically. 
They're beneficial if you wish to create easily-readable structures that can be expanded to support deep nesting. 
Unlike simple global variables, Object Literals often also take into account tests
for the existence of a variable by the same name so the chances of collision occurring are significantly reduced.
オブジェクトリテラルはグローバル空間を散らかさずに維持できるという利点に加え、論理的にコードとパラメータを整理するのを助けてくれる。
もし深くネスティングでき、読みやすい構造を作りたい場合は、このパターンは適している。
シンプルなグローバル変数とは違い、オブジェクトリテラルパターンは変数名の衝突を防ぐために、その変数名が存在しないかどうかのテストも考慮に入れる必要がある。

The code at the very top of the next sample demonstrates the different ways in which you can check to see if a namespace already exists before defining it.  I commonly use Option 3.
次のサンプルの一番上のものは、定義する前に名前空間が既に存在していないかどうかのチェックのバリエーションを示している。個人的にはOption3をよく使っている。


```javascript
/*Doesn't check for existence of myApplication*/
var myApplication = {};
 
/*
Does check for existence. If already defined, we use that instance.
Option 1:   if(!MyApplication) MyApplication = {};
Option 2:   var myApplication = myApplication = myApplication || {}
Option 3:   var myApplication = myApplication || {};
We can then populate our object literal to support models, views and collections (or any data, really):
*/
 
var myApplication = {
    models : {},
    views : {
        pages : {}
    },
    collections : {}
};
```

One can also opt for adding properties directly to the namespace (such as your views, in the following example):
名前空間に直接プロパティを追加することも可能である:

```javascript
var myGalleryViews = myGalleryViews || {};
myGalleryViews.photoView = Backbone.View.extend({});
myGalleryViews.galleryView = Backbone.View.extend({});
```

The benefit of this pattern is that you're able to easily encapsulate all of your models, views, routers etc. in a way 
that clearly separates them and provides a solid foundation for extending your code.
このパターンの便利な点は、model, views, routersなどすべてをカプセル化することができるため、明確にそれらを分けることができコードを拡張する際の基礎を提供してくれる点である。

This pattern has a number of useful applications.
It's often of benefit to decouple the default configuration for your application into a single area that can be easily modified
without the need to search through your entire codebase just to alter them - Object Literals work great for this purpose.
Here's an example of a hypothetical Object Literal for configuration:
このパターンはいくつかの便利な使い方がある。デフォルトの構成を???
オブジェクトリテラルはこの目的に最適である。仮定のオブジェクトリテラルの構成はこのように示すことができる:

```javascript
var myConfig = {
    language: 'english',
    defaults: {
        enableGeolocation: true,
        enableSharing: false,
        maxPhotos: 20
    },
    theme: {
        skin: 'a',
        toolbars: {
            index: 'ui-navigation-toolbar',
            pages: 'ui-custom-toolbar'    
        }
    }
}
```

Note that there are really only minor syntactical differences between the Object Literal pattern and a standard JSON data set.
If for any reason you wish to use JSON for storing your configurations instead (e.g. for simpler storage when sending to the back-end), feel free to.
オブジェクトリテラルと標準のJSONデータセットの書き方上の唯一の小さな違いがいくつか存在するが、もし構成を保存するため(例えばバックエンドに送るためのシンプルなストレージ)にJSONを使いたい場合は、そちらを使っても構わない。

For more on the Object Literal pattern, I recommend reading Rebecca Murphey's [excellent article on the topic](http://blog.rebeccamurphey.com/2009/10/15/using-objects-to-organize-your-code).
オブジェクトリテラルパターンにおいて、もっと知りたい場合はRebecca Murpheyの[このトピックにおける記事](http://blog.rebeccamurphey.com/2009/10/15/using-objects-to-organize-your-code)を読むことをお勧めする.


**ネストされた名前空間**
An extension of the Object Literal pattern is nested namespacing.
It's another common pattern used that offers a lower risk of collision due to the fact that even if a namespace already exists, it's unlikely the same nested children do.
オブジェクトリテラルの拡張版としてのパターンはネストされた名前空間のパターンである。
これは、もし名前空間が既に存在していたとしても同じネストされた子要素が存在することはあまり無いため、衝突の可能性をより低くすることができる。

Does this look familiar?
このパターンは見たこと無いだろうか？

```javascript
YAHOO.util.Dom.getElementsByClassName('test');
```

Yahoo's YUI uses the nested object namespacing pattern regularly and even DocumentCloud (the creators of Backbone) use the nested namespacing pattern in their main applications. A sample implementation of nested namespacing with Backbone may look like this:
YahooのYUIはこのネストされたオブジェクトの名前空間パターンを適用しており、DocumentCloud(Backboneの生みの親)もこのネストされた名前空間パターンを彼らのメインのアプリケーションに用いている。Backboneでのネストされた名前空間の実装サンプルは次のようである:

```javascript
var galleryApp =  galleryApp || {};
 
/*perform similar check for nested children*/
galleryApp.routers = galleryApp.routers || {};
galleryApp.model = galleryApp.model || {};
galleryApp.model.special = galleryApp.model.special || {};
 
/*routers*/
galleryApp.routers.Workspace   = Backbone.Router.extend({}); 
galleryApp.routers.PhotoSearch = Backbone.Router.extend({}); 
 
/*models*/
galleryApp.model.Photo      = Backbone.Model.extend({}); 
galleryApp.model.Comment = Backbone.Model.extend({}); 
 
/*special models*/
galleryApp.model.special.Admin = Backbone.Model.extend({});
```

This is both readable, organized and is a relatively safe way of namespacing your Backbone application in a similar fashion to what you may be used to in other languages.
これは、読みやすいと同時に整理されており、比較的安全にBackboneアプリケーションの名前空間を他の言語と同様の方法で定義することができる。

The only real caveat however is that it requires your browser's JavaScript engine first locating the galleryApp object and 
then digging down until it gets to the function you actually wish to use.
このパターンの唯一の欠点はブラウザのJavaScriptエンジンが最初にgalleryAppオブジェクトを見つけ、実際に使いたい関数にたどり着くまで深くたどっていくことで、検索するのに仕事量を増やしてしまうことである。

This can mean an increased amount of work to perform lookups, however developers such as Juriy Zaytsev (kangax) have previously tested and found the performance differences between single object namespacing vs the 'nested' approach to be quite negligible.
しかしJuriy Zaytsev(kangax)氏などのデベロッパはかつてテストを行い、ネストされていない名前空間と、ネストされたもののパフォーマンスの差は無視できるほど小さなものだと検証した。


**推奨**

Reviewing the namespace patterns above, the option that I would personally use with Backbone is nested object namespacing with the object literal pattern.
上の名前空間パターンをレビューしてみると、個人的に私がBackboneでよく使うのは、オブジェクトリテラルパターンとオブジェクトネストされた名前空間の組み合わせのオプションである。

Single global variables may work fine for applications that are relatively trivial, however,
larger codebases requiring both namespaces and deep sub-namespaces require a succinct solution that promotes readability and scales.
I feel this pattern achieves all of these objectives well and is a perfect companion for Backbone development.
シングルグローバル変数は小規模なアプリケーションには機能するかもしれないが、大規模なコードベースのアプリケーションには名前空間と深いサブ名前空間の二つを同時に使うことが
可読性と拡張性を維持するためには必要となってくる。
私はこのパターンはこれらの目的を達成するのに十分であり、かつBackboneを用いての開発の際にも完璧な組み合わせであると考えている。

###Additional Tips
### 他のヒント

####Automated Backbone Scaffolding
#### Backboneの自動Scaffolding
Scaffolding can assist in expediting how quickly you can begin a new application by creating the basic files required for a project automatically.
If you enjoy the idea of automated MVC scaffolding using Backbone, I'm happy to recommend checking out a tool called Brunch.
Scaffoldingはプロジェクトに必要な基本的なファイル群を自動的に準備することで新しいアプリケーションを迅速に始めることをアシストしてくれる。
Backboneを用いて自動的なMVCのScaffoldingをしたい場合は、Brunchと呼ばれるツールを一目見ていただきたい。

It works very well with Backbone, Underscore, jQuery and CoffeeScript and is even used by companies such as Red Bull and Jim Bean.
You may have to update any third party dependencies (e.g. latest jQuery or Zepto) when using it, but other than that it should be fairly stable to use right out of the box.
それは、Backbone, Underscore, jQueryとCoffeeScriptなどと非常に相性がよく、Red BullやJim Beanなどの会社でも使われている。 サードパーティ(jQueryやZepto)の依存関係などはアップデートが必要かもしれないが、それを除けば非常に安定的に使うことができる。


####Clarifications on Backbone's MVC

As Thomas Davis has previously noted, Backbone.js's MVC is a loose interpretation of traditional MVC, something common to many client-side MVC solutions. Backbone's views are what could be considered a wrapper for templating solutions such as the Mustache.js and `Backbone.View` is the equivalent of a controller in traditional MVC. `Backbone.Model` is however the same as a classical 'model'.
Thomas Davis氏が以前に指摘しているように、Backbone.jsのMVCは、多くのクライアントサイドMVCのソリューションに共通する従来のMVCの緩い解釈にちかいものだと言える。BackboneのViewは、Mustache.jsなどのテンプレーティングのラッパーとして捉えられ、従来のMVCのコントローラに相当するものである。 また一方で、`Backbone.Model`は従来の"モデル"と同じだといえる。

Whilst Backbone is not the only client-side MVC solution that could use some improvements in it's naming conventions, `Backbone.Controller` was probably the most central source of some confusion but has been renamed a router in more recent versions.
This won't prevent you from using Backbone effectively, however this is being pointed out just to help avoid any confusion if for any reason you opt to use an older version of the framework.
Backboneが命名規則にいくつかの改善点を用いることができる唯一のクライアントサイドMVCのソリューションでは無く、`Backbone.Controller`は中でも混乱をもたらす原因となっていたが最近のバージョンではそれがRouterと名前を変更された。
だからといってBackboneを??? しかしこれはもし何らかの形で古いバージョンのBackboneを使う場合のために、混乱を避けるために指摘されます。???

The official Backbone docs do attempt to clarify that their routers aren't really the C in MVC,
but it's important to understand where these fit rather than considering client-side MVC a 1:1 equivalent to the pattern you've probably seen in server-side development.
公式のBackboneのドキュメンテーションは明確にRouterがMVCのCにはあたらないと説明していますが、それはクライアントサイドのMVCとサーバサイド開発のそのパターンが1:1で対応しているものではないということを理解しなくてはいけない。

####Is there a limit to the number of routers I should be using?
#### ルータの数に制限はあるか？
Andrew de Andrade has pointed out that DocumentCloud themselves usually only use a single controller in most of their applications.
You're very likely to not require more than one or two routers in your own projects
as the majority of your application routing can be kept organized in a single controller without it getting unwieldy.
Andrade de Andrade氏はDocumentCloud自身が多くの彼らのアプリケーション一つのコントローラしか使わないことを指摘している。 多くのアプリケーションのRouterは1つか2つ以上のRouterを必要とする場合は非常に少ないため一つのコントローラで整理されている???



####Is Backbone too small for my application's needs?

If you find yourself unsure of whether or not your application is too large to use Backbone, I recommend reading my post on building large-scale jQuery & JavaScript applications or reviewing my slides on client-side MVC architecture options.
In both, I cover alternative solutions and my thoughts on the suitability of current MVC solutions for scaled application development.
Backboneを使うにはこのアプリケーションが多きすぎるかどうか不安な場合は、私のlarge-scale jQuery & JavaScript applicationかクライアントサイドのMVCアーキテクチャのスライドを読んでもらうことをお勧めする。 どちらも、代替のソリューションや現在のスケールされたアプリケーション開発のMVCソリューションの適応性についての私の考えも述べている。

Backbone can be used for building both trivial and complex applications as demonstrated by the many examples Ashkenas has been referencing in the Backbone documentation. 
As with any MVC framework however, it's important to dedicate time towards planning out what models and views your application really needs.
Diving straight into development without doing this can result in either spaghetti code or a large refactor later on and it's best to avoid this where possible.
Ashkenasがドキュメンテーションで参照しているようにBackboneを用いることで、小規模と大規模の両方のアプリケーションを構築することができる。
どのMVCフレームワークでも言えることだが、どのようなmodelとviewがこのアプリケーションに必要かというプランニングは非常に重要である。
無計画ですぐに開発を始めてしまうと、スパゲッティコードになったり、後ほど大規模なリファクタリングが必要になってしまうなど問題が生じるため、この可能性を回避することは賢明だといえる。

At the end of the day, the key to building large applications is not to build large applications in the first place. If you however find Backbone doesn't cut it for your requirements I strongly recommend checking out JavaScriptMVC or SproutCore as these both offer a little more than Backbone out of the box. 
Dojo and Dojo Mobile may also be of interest as these have also been used to build significantly complex apps by other developers.
最終的には、大規模なアプリケーションを構築する上で大切なことは、最初から大規模なものを作ろうとしないことである。
もしBackboneがあなたの用件に合わなかった場合は、JavaScriptMVCやSproutCoreをチェックすることをお勧めする。両方のフレームワークはともにBackboneより多くの機能を初期段階から提供してくれる。
DojoとDojo Mobileも、他のデベロッパによって多くの複雑なアプリケーションを構築するのに使用されているため、選択肢としておいておくのもいいかもしれない。


##Modular JavaScript

When we say an application is modular, we generally mean it's composed of a set of highly decoupled, distinct pieces of functionality stored in modules. As you probably know, loose coupling facilitates easier maintainability of apps by removing dependencies where possible. When this is implemented efficiently, its quite easy to see how changes to one part of a system may affect another.
アプリケーションがモジュラーであると言うとき、我々は一般的にそれが機能が高度に分離され、異なる要素のセットで構成されていてモジュールとしてまとめられている状態を意味する。ご存知のように、ルーズカップリングは可能な限り依存関係を無くすことで、アプリケーションの管理を容易にしてくれる。これが効率的に実装されている場合、システム一部への変更がシステムの別の部分にどんな影響を与えるかを一目で見ることができる。

Unlike some more traditional programming languages however, the current iteration of JavaScript (ECMA-262) doesn't provide developers with the means to import such modules of code in a clean, organized manner. 
It's one of the concerns with specifications that haven't required great thought until more recent years where the need for more organized JavaScript applications became apparent.
しかし、いくつかのより伝統的なプログラミング言語とは異なり、現行のJavaScript(ECMA - 262)には、きちんと整理された形でコードモジュールをインポートする方法はまだ存在していない。
現在、より組織化されたJavaScriptアプリケーションの必要性が明らかになった以上、これに関わるJavaScriptの仕様は懸念事項の一つとなっている。

Instead, developers at present are left to fall back on variations of the module or object literal patterns.
With many of these, module scripts are strung together in the DOM with namespaces being described by a single global object where it's still possible to incur naming collisions in your architecture. There's also no clean way to handle dependency management without some manual effort or third party tools.
その代わりに、モジュールまたはオブジェクトリテラルパターンのバリエーションの対処方法は現在のデベロッパに委ねられている。 これらの多くで、モジュールのスクリプトは、アーキテクチャ内で名前の衝突の可能性を未だに持っているシングルグローバルオブジェクトによって作られた名前空間とDOMでつなぎ合わされている。 サードパーティのツールや何かしら手も加えずにかの依存関係の管理を処理する簡単な方法はまだありません。

Whilst native solutions to these problems will be arriving in ES Harmony, the good news is that writing modular JavaScript has never been easier and you can start doing it today.
ES Harmonyにはこのような課題に対するソリューションが提供される一方で、モジュラーなJavaScriptを書くことは簡単に今日からすぐに始めることができる。

In this next part of the book, we're going to look at how to use AMD modules and RequireJS for cleanly wrapping units of code in your application into managable modules.
次のパートでは、AMDモジュールの使い方やRequireJSを使ってどのようにアプリケーションを管理しやすいモジュールにまとめるかを解説していきたいとおもう。


##RequireJSとAMDとのモジュールを整理する

In case you haven't used it before, RequireJS is a popular script loader written by James Burke - a developer who has been quite instrumental in helping shape the AMD module format, which we'll discuss more shortly.
Some of RequireJS's capabilities include helping to load multiple script files, helping define modules with or without dependencies and loading in non-script dependencies such as text files.
RequireJSを使ったことの無い方の為に少し説明すると、RequireJSとはAMDモジュールフォーマットの設計に重要な役割を果たしているデベロッパであるJames Burkeによって書かれたよく使われているスクリプトローダである。
複数のスクリプトファイルを読み込んだりする機能や、依存関係の有無でモジュールを定義するのを援助する機能や、テキストのようなスクリプト以外の依存ファイルを読み込んだりできる機能を提供する。

So, why use RequireJS with Backbone? Although Backbone is excellent when it comes to providing a sanitary structure to your applications, there are a few key areas where some additional help could be used:
では、なぜBackboneとRequireJSを使うのか? Backboneはアプリケーション構造を提供してくれる点ではすばらしいが、いくつかの点で改善が期待される:

1) Backbone doesn't endorse a particular approach to modular-development. Although this means it's quite open-ended for developers to opt for classical patterns like the module-pattern or Object Literals for structuring their apps (which both work fine), it also means developers aren't sure of what works best when other concerns come into play, such as dependency management.
1) Backboneはモジュラー開発においての特定のアプローチを持っている訳ではない。つまりアプリケーションを構造パターンとしてモジュールパターンやオブジェクトリテラルなどの典型的なものを選ぶ自由をデベロッパに与えている訳ではあるが、同時にデベロッパは依存関係管理などにおいて???

RequireJS is compatible with the AMD (Asynchronous Module Definition) format, a format which was born from a desire to write something better than the 'write lots of script tags with implicit dependencies and manage them manually' approach to development.
In addition to allowing you to clearly declare dependencies, AMD works well in the browser,
supports string IDs for dependencies,
declaring multiple modules in the same file and gives you easy-to-use tools to avoid polluting the global namespace.
RequireJSは、AMD（非同期モジュール定義）形式と互換性があり、そのAMD形式は、"たくさんのスクリプトタグを書いて、暗黙の依存関係を一つ一つ管理する方法"よりも、ベターな書き方をしたいという願望から生まれた形式である。
明確に依存関係を宣言できるようになるだけでなく、AMDはブラウザでも機能し、依存関係のための文字列IDをサポートし、一つのファイルで複数のモジュールを宣言することができ、グローバル名前空間を散らかさないためのツールも提供してくれる。

2) Let's discuss dependency management a little more as it can actually be quite challenging to get right if you're doing it by hand.
When we write modules in JavaScript, we ideally want to be able to handle the reuse of code units intelligently and sometimes this will mean pulling in other modules at run-time 
whilst at other times you may want to do this dynamically to avoid a large pay-load when the user first hits your application.
2) 正しく手で書いていくのはとても難しくなってくるため、依存関係の管理についてもう少し話してみよう。
JavaScriptでモジュールを書く際に、理想的には賢くコード単位を再利用できるような処理をしたいし、時にはランタイムで他のモジュールを読み込みたいときもあるかもしれないし、またあるときは最初のページのロード量を減らすために動的にこれを行いたいときもあるかもしれない。

Think about the GMail web-client for a moment. When users initially load up the page on their first visit, Google can simply hide widgets such as the chat module until a user has indicated (by clicking 'expand') that they wish to use it. Through dynamic dependency loading, Google could load up the chat module only then, rather than forcing all users to load it when the page first initializes.
This can improve performance and load times and can definitely prove useful when building larger applications.
Gmailのwebクライアントを少し考えてみよう。初めてユーザがページにアクセスする際、例えば、ユーザがチャットを使いたいという意思を示す("開く"をクリックするなど)まで、そのチャットモジュールは隠されています。全ユーザに強制的にすべてのスクリプトを読み込ませるのではなく、ダイナミックな読み込みによって、Googleはその意思が示されたときにだけチャットモジュールを読み込む。
これによってパフォーマンスもロード時間も改善することができ、大規模なアプリケーションを構築する際にとても便利になってくる。
I've previously written [a detailed article](http://addyosmani.com/writing-modular-js) covering both AMD and other module formats and script loaders in case you'd like to explore this topic further. 
もしこのトピックについて深く知りたい場合は、AMDやその他のモジュールフォーマットについてやその他のスクリプトローダなどについて書いている[こちらの記事](http://addyosmani.com/writing-modular-js)を読んでいただくと良い。
The takeaway is that although it's perfectly fine to develop applications without a script loader or clean module format in place, it can be of significant benefit to consider using these tools in your application development.
スクリプトローダや明確なモジュールフォーマットを持たずにアプリケーションを開発するのは一向に構わないが、これらのツールを使うことはアプリケーション開発を有利に進めることができる。


###Writing AMD modules with RequireJS

As discussed above, the overall goal for the AMD format is to provide a solution for modular JavaScript that developers can use today. The two key concepts you need to be aware of when using it with a script-loader are a `define()` method for facilitating module definition and a `require()` method for handling dependency loading. `define()` is used to define named or unnamed modules based on the proposal using the following signature:
上述のように、AMDのフォーマットのための総合的な目標は、開発者が今から使用できるモジュラーJavaScriptのソリューションを提供することです。スクリプトローダーを使用する際に注意しなくてはならない2つの重要な概念は、モジュールの定義のための`define()`メソッドと依存関係のロードを処理するための`require()`メソッドの二つです。`define()`は以下のような書き方に基づいて、名前付きまたは名前なしのモジュールを定義するために使用されます。

```javascript
define(
    module_id /*optional*/, 
    [dependencies] /*optional*/, 
    definition function /*function for instantiating the module or object*/
);
```

As you can tell by the inline comments, the `module_id` is an optional argument which is typically only required when non-AMD concatenation tools are being used (there may be some other edge cases where it's useful too). When this argument is left out, we call the module 'anonymous'. When working with anonymous modules, the idea of a module's identity is DRY, making it trivial to avoid duplication of filenames and code.
インラインコメントによってお分かりのように、`module_idは`は通常、非AMD連結ツールが使用されているとき(もしくはその他の便利などのエッジケース)に必要とされるオプションの引数です。この引数が省略された場合、このモジュールは"anonymous(匿名)"であると言います。匿名のモジュールを用いる場合、モジュールのIDのアイデアは、ファイル名とコードの重複を避けるために小さなものとするためにDRYであると言える。???

Back to the define signature, the dependencies argument represents an array of dependencies which are required by the module you are defining and the third argument ('definition function') is a function that's executed to instantiate your module. A barebone module (compatible with RequireJS) could be defined using `define()` as follows:
`define`の書き方に戻ると、依存関係の引数は、あなたが定義しているモジュールに必要な依存関係の配列を受け取ります。そして3つめの引数("定義関数")は、モジュールをインスタンス化するために実行される関数です。モジュール(RequireJSと互換性がある)の骨組みは`define()`を用いて次の通りに定義することができます。

```javascript
// A module ID has been omitted here to make the module anonymous

define(['foo', 'bar'], 
    // module definition function
    // dependencies (foo and bar) are mapped to function parameters
    function ( foo, bar ) {
        // return a value that defines the module export
        // (i.e the functionality we want to expose for consumption)
    
        // create your module here
        var myModule = {
            doStuff:function(){
                console.log('Yay! Stuff');
            }
        }

        return myModule;
});

####もう一つの書き方
There is also a [sugared version](http://requirejs.org/docs/whyamd.html#sugar) of `define()` available that allows you to declare your dependencies as local variables using `require()`. This will feel familiar to anyone who's used node, and can be easier to add or remove dependencies.
[シュガーシンタックスバージョン](http://requirejs.org/docs/whyamd.html#sugar)の`define()`の書き方も存在し、その場合、依存関係を`require()`を用いてローカル変数として宣言することもできます。これは[node](http://nodejs.org)を使ったことがある人であれば馴染みかもしれませんし、依存関係の削除も追加も容易です。
Here is the previous snippet using the alternate syntax:
これが、そのもう一つの書き方です:


```javascript
// A module ID has been omitted here to make the module anonymous

define(function(require){
        // module definition function
    // dependencies (foo and bar) are defined as local vars
    var foo = require('foo'),
        bar = require('bar');
        
        // return a value that defines the module export
        // (i.e the functionality we want to expose for consumption)
    
        // create your module here
        var myModule = {
            doStuff:function(){
                console.log('Yay! Stuff');
            }
        }

        return myModule;
});
```

The `require()` method is typically used to load code in a top-level JavaScript file or within a module should you wish to dynamically fetch dependencies. An example of its usage is:
あなたが動的に依存関係を読み取りたい場合、メソッドは、通常、トップレベルのJavaScriptファイルまたはモジュール内のコードを読み込むために使用されます。その使い方の例は、次のとおりです。

```javascript
// Consider 'foo' and 'bar' are two external modules
// In this example, the 'exports' from the two modules loaded are passed as
// function arguments to the callback (foo and bar)
// so that they can similarly be accessed

require(['foo', 'bar'], function ( foo, bar ) {
        // rest of your code here
        foo.doSomething();
});
```


**Wrapping modules, views and other components with AMD**
***モジュール・View・その他のコンポーネントをAMDでラップする***

Now that we've taken a look at how to define AMD modules, let's review how to go about wrapping components like views and collections so that they can also be easily loaded as dependencies for any parts of your application that require them. At it's simplest, a Backbone model may just require Backbone and Underscore.js. These are considered it's dependencies and so, to write an AMD model module, we would simply do this:
さてAMDのモジュールを定義する方法を見てきたので、次は、ViewやCollectionなどのコンポーネントを、それらを必要とするアプリケーションの依存関係として読み込めるように、ラップする方法を検討してみよう。最も単純に捉えると、BackboneのModelは、単にBackboneとUnderscore.jsのみが必要となってきますので、これらはその依存関係と見なされます。AMD Modelのモジュールを書き方は単純に以下の通りです:

```javascript
define(['underscore', 'backbone'], function(_, Backbone) {
  var myModel = Backbone.Model.extend({

    // Default attributes 
    defaults: {
      content: "hello world",
    },

    // A dummy initialization method
    initialize: function() {
      if (!this.get("content")) {
        this.set({"content": this.defaults.content});
      }
    },

    clear: function() {
      this.destroy();
      this.view.remove();
    }

  });
  return myModel;
});
```

Note how we alias Underscore.js's instance to `_` and Backbone to just `Backbone`, making it very trivial to convert non-AMD code over to using this module format. For a view which might require other dependencies such as jQuery, this can similarly be done as follows:
Underscore.jsのインスタンスを`_`へ、Backboneを`Backbone`というようにエイリアスを作って、この非AMDコードをモジュールフォーマットに変換??? jQueryなどのような他の依存関係を要するViewも次のようにかけます:

```javascript
define([
  'jquery',
  'underscore', 
  'backbone',
  'collections/mycollection',
  'views/myview'
  ], function($, _, Backbone, myCollection, myView){

  var AppView = Backbone.View.extend({
  ...
```

Aliasing to the dollar-sign (`$`), once again makes it very easy to encapsulate any part of an application you wish using AMD.
`$`にエイリアスを当てることで、アプリケーションのどの要素もこのAMDを使うことでカプセル化することが容易になります。


##External [Underscore/Handlebars/Mustache] templates using RequireJS
##外部テンプレート[Underscore/Handlebars/Mustache]をRequireJSと使う場合

Moving your [Underscore/Mustache/Handlebars] templates to external files is actually quite straight-forward. As this application makes use of RequireJS, I'll discuss how to implement external templates using this specific script loader.
[Underscore/Mustache/Handlebars]などのテンプレートファイルを外部に移すのも非常に簡単です。このアプリケーションはRequireJSを利用するので、この特定のスクリプトローダを使用して外部テンプレートを実装する方法について説明します。

RequireJS has a special plugin called text.js which is used to load in text file dependencies. To use the text plugin, simply follow these simple steps:
RequireJSには、テキストファイルの依存関係をロードするために使用できるtext.jsと呼ばれる特別なプラグインがあります。テキストプラグインを使用するには、以下の簡単なステップに従ってください。

1. Download the plugin from http://requirejs.org/docs/download.html#text and place it in either the same directory as your application's main JS file or a suitable sub-directory.
1. [text.js]( http://requirejs.org/docs/download.html#text )プラグインをダウンロードして、アプリケーションのメインのJSファイルまたは適切なサブディレクトリと同じディレクトリに配置してください。

2. Next, include the text.js plugin in your initial RequireJS configuration options. In the code snippet below, we assume that RequireJS is being included in our page prior to this code snippet being executed. Any of the other scripts being loaded are just there for the sake of example.
2. RequireJSの設定オプションにtext.jsプラグインを追加します。下のコードのように、RequireJSがこの設定オプションより上で読み込まれていることを前提としています。以下のコードでは例のために他のスクリプトも読み込んであります。

 
```javascript
require.config( {
    paths: {
        'backbone':         'libs/AMDbackbone-0.5.3',
        'underscore':       'libs/underscore-1.2.2',
        'text':             'libs/require/text',
        'jquery':           'libs/jQuery-1.7.1',
        'json2':            'libs/json2',
        'datepicker':       'libs/jQuery.ui.datepicker',
        'datepickermobile': 'libs/jquery.ui.datepicker.mobile',
        'jquerymobile':     'libs/jquery.mobile-1.0'
    },
    baseUrl: 'app'
} );
```

3. When the `text!` prefix is used for a dependency, RequireJS will automatically load the text plugin and treat the dependency as a text resource. A typical example of this in action may look like..
3. `text!`接頭辞が依存ファイルに使用された場合、RequireJSは自動的にテキストのプラグインをロードし、テキストリソースとして依存関係を扱います。この典型的な例は次のようになります:

```javascript
require(['js/app', 'text!templates/mainView.html'],
    function(app, mainView){
        // the contents of the mainView file will be
        // loaded into mainView for usage.
    }
);
```

4. Finally we can use the text resource that's been loaded for templating purposes. You're probably used to storing your HTML templates inline using a script with a specific identifier.
4. そしてテンプレート用にロードされたテキストのリソースを使用することができるようになります。以前まではおそらく、HTMLに特定のidで埋め込んでいたと思います。

With Underscore.js's micro-templating (and jQuery) this would typically be:
Underscore.jsのテンプレートエンジンとjQueryでは以下のようになります:

HTML:

```html
<script type="text/template" id="mainViewTemplate">
    <% _.each( person, function( person_item ){ %>
        <li><%= person_item.get("name") %></li>  
    <% }); %>
</script>
```

JS:

```javascript
var compiled_template = _.template( $('#mainViewTemplate').html() );
```

With RequireJS and the text plugin however, it's as simple as saving your template into an external text file (say, `mainView.html`) and doing the following:
RequireJSとtextプラグインで、複数のテンプレートファイルを一つのファイル(例えば`mainView.html`)にまとめることも可能です:

```javascript
require(['js/app', 'text!templates/mainView.html'],
    function(app, mainView){
        
        var compiled_template = _.template( mainView );
    }
);
```

That's it!. You can then go applying your template to a view in Backbone doing something like:
これだけです! これが終わったので、次は以下のようにBackboneのViewにテンプレートをあてるだけです:

```javascript
collection.someview.el.html( compiled_template( { results: collection.models } ) );
```


All templating solutions will have their own custom methods for handling template compilation, but if you understand the above, substituting Underscore's micro-templating for any other solution should be fairly trivial.
すべてのテンプレートエンジンはそのコンパイルを処理するために独自のカスタムメソッドを持っていますが、上記をきちんと理解していれば、ほかのテンプレーティングエンジンとの置き換えは非常に簡単なはずです。

**Note:** You may also be interested in looking at [Require.js tpl](https://github.com/ZeeAgency/requirejs-tpl). It's an AMD-compatible version of the Underscore templating system that also includes support for optimization (pre-compiled templates) which can lead to better performance and no evals. I have yet to use it myself, but it comes as a recommended resource.
**ノート** もしかすると[Require.js tpl](https://github.com/ZeeAgency/requirejs-tpl)もいい選択肢かもしれないです。これはAMDと互換性があるUnderscoreのテンプレーティングシステムでevalフリーでかつより高いパフォーマンスに最適化できるようにもなっています。まだ私自身は使ったことはありませんが、推奨できるオプションだと思います。


##Optimizing Backbone apps for production with the RequireJS Optimizer

As experienced developers may know, an essential final step when writing both small and large JavaScript web applications is the build process.  The majority of non-trivial apps are likely to consist of more than one or two scripts and so optimizing, minimizing and concatenating your scripts prior to pushing them to production will require your users to download a reduced number (if not just one) script file.
経験豊かなデベロッパであれば知っているかもしれませんが、小規模および大規模なJavaScript Webアプリケーションを構築する上での大切な最後のステップは、そのビルドプロセスです。大規模アプリケーションは多くの場合、複数のスクリプトを含んでいるため、プロダクションにプッシュする前に、ページが読み込まなくてはならないスクリプトの数を減らすため、最適化、最小化やファイルの連結が必要になってきます。

Note: If you haven't looked at build processes before and this is your first time hearing about them, you might find [my post and screencast on this topic](http://addyosmani.com/blog/client-side-build-process/) useful.
ノート: これまでにビルドプロセスを無いという方には、またはそれらの話を聞くのが初めての方々にはこのトピックに関連して私が以前書いた[記事とスクリーンキャスト]( http://addyosmani.com/blog/client-side-build )が役に立つかもしれません。

With some other structural JavaScript frameworks, my recommendation would normally be to implicitly use YUI Compressor or Google's closure compiler tools, but we have a slightly more elegant when it comes to Backbone if you're using RequireJS. RequireJS has a command line optimization tool called r.js which has a number of capabilities, including:
私は大抵YUI CompressorかGoogleのClosureコンパイラツールを進めますが、RequireJSとBackboneを使っている場合は、より良いオプションが存在します。RequireJSには`r.js`と呼ばれるコマンドラインの最適化ツールがあります。それには以下の利点が含まれます:

* Concatenating specific scripts and minifiying them using external tools such as UglifyJS (which is used by default) or Google's Closure Compiler for optimal browser delivery, whilst preserving the ability to dynamically load modules
* スクリプトの読み込み時間を最適化するために、UglifyJS(デフォルト)やGoogleのClosureコンパイラなどを使って特定のファイルを連結/縮小化することができますが、一方で動的にモジュールを読み込むことも可能です。
* Optimizing CSS and stylesheets by inlining CSS files imported using @import, stripping out comments etc.
* `@import`で読み込まれているCSSを一つのファイルにまとめたりコメントを省いたりしてくれます
* The ability to run AMD projects in both Node and Rhino (more on this later)
* AMDプロジェクトをNodeとRhinoで動かすこともできます。(後ほど詳しく)

You'll notice that I mentioned the word 'specific' in the first bullet point. The RequireJS optimizer only concatenates module scripts that have been specified in arrays of string literals passed to top-level (i.e non-local) require and define calls. As clarified by the [optimizer docs](http://requirejs.org/docs/optimization.html) this means that Backbone modules defined like this:
最初の箇条書きで'特定の'と書いたことにお気づきだと思いますが、RequireJSの最適化ツールはトップレベルで渡された配列で指定されたモジュールしか最適化を行いません。[最適化ツールのドキュメンテーション](http://requirejs.org/docs/optimization.html)で書かれているように、Backboneのモジュールは以下のように定義できます:

```javascript
define(['jquery','backbone','underscore', collections/sample','views/test'], 
    function($,Backbone, _, Sample, Test){
        //...
    });
```

will combine fine, however inline dependencies such as:
これらはきちんと統合されますが、以下のようにインラインで書かれている依存関係の場合:

```javascript
var models = someCondition ? ['models/ab','models/ac'] : ['models/ba','models/bc'];
```

will be ignored. This is by design as it ensures that dynamic dependency/module loading can still take place even after optimization. 
これらは最適化からは取り除かれます。なぜならこのような動的に読み込まれた依存関係・モジュールは最適化の後でもその通り読み込まれないといけないからである。

Although the RequireJS optimizer works fine in both Node and Java environments, it's strongly recommended to run it under Node as it executes significantly faster there. In my experience, it's a piece of cake to get setup with either environment, so go for whichever you feel most comfortable with. 
RequireJSオプティマイザはNodeとJavaのどちらの環境でも実行することは可能ですが、Nodeの方がはるかに早く実行されるのでそちらの方が推奨されています。私の経験では、走らせること自体はどちらの環境でも簡単なので、好みの環境で行っていただいたらいいと思います。

To get started with r.js, grab it from the [RequireJS download page](http://requirejs.org/docs/download.html#rjs) or [through NPM](http://requirejs.org/docs/optimization.html#download). Now, the RequireJS optimizer works absolutely fine for single script and CSS files, but for most cases you'll want to actually optimize an entire Backbone project. You *could* do this completely from the command-line, but a cleaner option is using build profiles.

`r.js`を始めるには、[RequireJSのダウンロードページ](http://requirejs.org/docs/download.html#rjs) もしくは[npm](http://requirejs.org/docs/optimization.html#download)から落としてきてください。今のところRequireJSオプティマイザはスクリプトファイルとCSSファイルに対してはきちんと動作していますが、あなたの場合はBackboneのプロジェクト全体を最適化したいと考えていると思います。コマンドラインからすべて行うことも可能ではありますが、すっきりしたやり方はビルドプロファイルを作る方法です。

Below is an example of a build file taken from the modular jQuery Mobile app referenced later in this book. A **build profile** (commonly named `app.build.js`) informs RequireJS to copy all of the content of `appDir` to a directory defined by `dir` (in this case `../release`). This will apply all of the necessary optimizations inside the release folder. The `baseUrl` is used to resolve the paths for your modules. It should ideally be relative to `appDir`.
本書の後半で作っていくモジュール型jQueryのモバイルアプリの場合のビルドファイルの例です。**ビルドプロファイル**(一般的なファイル名は`app.build.js`)で定義されたディレクトリ`dir`(この場合は`../release`)に`appDir`の内容のすべてをコピーするようにRequireJSに伝えます。これで、リリースのフォルダ内にこのプロジェクトに必要なすべてのファイルに最適化を適用します。`baseUrl`でモジュールへのパスを指定するのですが、理想的には`appDir`に相対的なパスであることが望ましいです。

Near the bottom of this sample file, you'll see an array called `modules`. This is where you specify the module names you wish to have optimized. In this case we're optimizing the main application called 'app', which maps to `appDir/app.js`. If we had set the `baseUrl` to 'scripts', it would be mapped to `appDir/scripts/app.js`.
[ この ]サンプルファイルの下の方で、`modules`と呼ばれる配列を見かけると思います。これが最適化したいモジュール名を指定するところとなります。このケースでは`appDir/app.js`にマッピングされた`app`と呼ばれるメインアプリケーションを最適化しています。もし`baseUrl`を`scripts`に指定していたら、`app`は`appDir/scripts/app.js`にマップされます。

```javascript
({
    appDir: "./",
    baseUrl: "./",
    dir: "../release",
    paths: {
       'backbone':          'libs/AMDbackbone-0.5.3',
        'underscore':       'libs/underscore-1.2.2',
        'jquery':           'libs/jQuery-1.7.1',
        'json2':            'libs/json2',
        'datepicker':       'libs/jQuery.ui.datepicker',
        'datepickermobile': 'libs/jquery.ui.datepicker.mobile',
        'jquerymobile':     'libs/jquery.mobile-1.0'
    },
    optimize: "uglify",
    modules: [
        {
            name: "app",
            exclude: [
                // If you prefer not to include certain libs exclude them here
            ]
        }
    ]
})
```

The way the build system in r.js works is that it traverses app.js (whatever modules you've passed) and resolved dependencies, concatenating them into the final `release`(dir) folder. CSS is treated the same way.
`r.js`がどのように動いているのかというと、まずapp.js(その他にあなたが指定したモジュール)を読み込み、依存関係を明確にして、それらを連結して最終的に`release`ディレクトリに配置されます。CSSも同様です。

The build profile is usually placed inside the 'scripts' or 'js' directory of your project. As per the docs, this file can however exist anywhere you wish, but you'll need to edit the contents of your build profile accordingly. 
このビルドプロファイルは、通常プロジェクトディレクトリの"script"や"js"に配置されます。ドキュメントに従えば、このファイル好みにあわせて任意の場所に配置できますが、それに応じてビルドプロファイルの内容を編集する必要があります。

Finally, to run the build, execute the following command once insice your `appDir` or `appDir/scripts` directory:
最終的に、`appDir`内もしくは`appDir/scripts`内で以下のコマンドを打つことで、ビルドを実行します:

```javascript
node ../../r.js -o app.build.js
```

That's it. As long as you have UglifyJS/Closure tools setup correctly, r.js should be able to easily optimize your entire Backbone project in just a few key-strokes. If you would like to learn more about build profiles, James Burke has a [heavily commented sample file](https://github.com/jrburke/r.js/blob/master/build/example.build.js) with all the possible options available.
これでおしまいです。UglifyJS/Closureツールが正しくセットアップされている限り、r.jsはほんの少しのタイピングでBackboneプロジェクト全体を最適化することができます。ビルドプロファイルについてもっと知りたい方は、James Burkeがすべてのオプションを含めて[細かくコメントが加えられたサンプルファイル](https://github.com/jrburke/r.js/blob/master/build/example.build.js)を公開しているのでそちらを参照していただきたい。

