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
