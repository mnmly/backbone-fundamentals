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


