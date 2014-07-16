## 投影片播放方式

在目錄頁下，使用`python -m SimpleHTTPServer`，並且在瀏覽器中開啟template.html頁面即可。

# Web Component

## 前言

2013年時，Eric Bidelman就曾經在Google I/O迅速地以 [Web Components: A Tectonic Shift for Web Development](http://webcomponents.org/presentations/web-components-a-tectonic-shift-for-web-development-at-google-io/)介紹Web Component的特色與功用，這是HTML新世代的濫觴，也是接下來發展的新趨勢。

在國內，已有許多優秀的先進將之簡單的介紹一番，包括以下：

* Hinablue [Web Components 初探](http://blog.hinablue.me/entry/web-components-first-look)
* OThree [Web Component](https://blog.othree.net/log/2013/11/27/web-component)

在本repository中，我們會簡單帶過四個最基本的功用，並引領至目前(2014年)該領域最熱門的library - Polymer。
首先，該repository僅僅需要你15~30分鐘的時間來了解什麼是Web Component，因此並不會如各參考資料般的詳細，
如果對該技術燃起了興趣，煩請參閱以下的資料：

* [w3c spec](http://www.w3.org/TR/components-intro/)
* [<web>components</web>](http://html5-demos.appspot.com/static/webcomponents/index.html)
* [Polymer & Web Components](http://polymer-change.appspot.com/)

Web Component目前較為熱門的特性是HTML Template、Custom Element、Shadow DOM及HTML Import，
雖然W3C列出了Decorator (用以裝飾部分區段 by CSS)，但是目前狀況則是尚未spec化，故各家瀏覽器沒有實作。
以下會以這四個主題一一介紹一遍：

* HTML Template
* Custom Element
* Shadow DOM
* HTML Import

## HTML Template
HTML Template的主要功能是提供一種方式將element包裝起來，使得各個html element不會被渲染，直到使用者將它掛載、複製至document下的element中。

Basic Usage:

```
    <template id="commentTemplate">
      <div>
          <img src="">
          <div class="comment-text"></div>
      </div>
    </template>
```
在W3C spec中提到，template在整份document渲染時並不會直接顯示於其中，更妙的是，script並不會被執行、image也不會先載入，但是template tag本身是會被剖析(parse)的。主要的用處在於它的property - content，這個content其實是在`document fragment`當中(簡單說明，document fragment不會被掛載在DOM Tree中，直到被`appendChild()`到某個element下。詳情可見[this](https://developer.mozilla.org/en-US/docs/Web/API/document.createDocumentFragment))，如果需要使用到template的內容，僅需要將template的資料挖出來就可。

Example:

```
    <script>
        $(document).ready(function () {

            $('#useme').on('click', function () {
                var content = document.querySelector('#temp').content;

                // 更新 template DOM 中的内容。
                var span = content.querySelector('span');
                span.textContent = parseInt(span.textContent) + 1;
                $('#container').html('Template used: <span>' + span.textContent + '</span>');
            });
        });

    </script>
```


```
    <button id="useme">Use me</button>
    <div id="container"></div>
    <template id="temp">
      <div>Template used: <span>0</span></div>
    </template>
```
此範例說明了當template element被掛載在document fragment下時，並不會直接被渲染在瀏覽器當中。但是透過`querySelector`，我們可以抓出並修改值、顯示出來。
或者，我們也可以利用`cloneNode()`/`importNode()`，操作template element。

由於擁有`display:none`的特性，可以把各種資料藏於template中，日後再抓取出來，相當方便。
另外，`Polymer`大量使用template作為元件化HTML的基本單元，好比說把google-calendar用template包裝起來，
日後可以利用HTML import匯入template，並以custom element定義在DOM Tree中。

### Before HTML Template

在此，參考[HTML's New Template Tag](http://www.html5rocks.com/zh/tutorials/webcomponents/template/)提到的內容，重新省思一下過去曾經被採用的幾種類似方法。

* DIV with hidden property

```
<div id="mytemplate" hidden>
  <img src="logo.png">
  <div class="comment"></div>
</div>
```

其中一種方法是將div element標上hidden，這種方法很容易用javascript進行操作且不會直接顯示於網頁中，但是會對server發出image request的請求，再者，如果需要對內容用CSS修飾，容易被全域的CSS設定所影響。

```
<script id="mytemplate" type="text/x-handlebars-template">
  <img src="logo.png">
  <div class="comment"></div>
</script>
```

另外一種方法則是載入script，並且利用其內容做操作。問題在於實際操作內容是用`innerHTML`的方式，容易有XSS的可能性。

### HTML Template - 搭配

使用HTML Template的最佳方法是搭配CSS以及javascript，做成預先定義好的範本，這裡提供一個案例[Embedded Spotify](https://github.com/zenorocha/embed-spotify)，即使是Spotify player 也能夠做成web component，更甭提其他的各種功能、app了。

## Custom Element

第二個重要的議題是自定義的元素。這算是Web Component基礎中的基礎了，使用Custom Element並搭配Shadow DOM、HTML Template就可以做出一個完整的Web App出來。曾幾何時，Internet已經被某些定義好的tag所展現出內容所主導，現在，程式設計師也可以定義自己的element，甚至是自己的event, method, property。

Custom Element賦予了很彈性的設計，不僅底層的event仍然保留功能 (`addEventListener`函式, `Keyboard event`函式)，瀏覽器的`dev tool`也可以偵測這些element，簡化開發的難度。可惜的是，各家瀏覽器很少有原生支援，不過目前很多功能都由`Polymer`向前支援了 (nice!

最後一個優點是目前也有蠻多的開發者投入，因此有一些套件可以參考[Custom Element](http://customelements.io)

Usage

```
<element name="x-foo" constructor="XFoo">
  <section> Foo bar test</section>
  <script>
    var section = this.querySelector('section');
    this.register({
      prototype: {
        readyCallback: function() {
          this.textContent = section.textContent;
        },
        foo: function() {
          alert('foo() is called');
        }
      }
    });
  </script>
</element>
```

原本的使用方式如上，先定義一個element tag，並且在此element內分別定義script及要渲染的html tag，script主要用來描述此html tag的狀態。

在使用上呢，則是用以下的Code作為範例：

```
<x-foo></x-foo><!-- names must contains dash -->
/* or use in DOM */
var elem = document.createElement('x-foo');
elem.addEventListener('click', function(e) {
e.target.foo(); // alert 'foo() called'.
});
```
首先，在tag之中最好要包含一個`-` dash，在`Polymer`當中則是一定要包含，否則會有問題出現。
接著在javascript中利用`createElement()`創造一個element，並賦予他一個event handler，至此就完成建立自己的element。

## Shadow DOM

Shadow DOM在W3C上介紹的相當複雜，包含light DOM、shadow DOM。
主要可以把shadow DOM視為隱藏在DOM中的一些element，我們可以把element封裝於shadow DOM中，使得封裝在shadow DOM的CSS樣式表不會影響其他的element。

[shadow DOM in html5rocks](http://www.html5rocks.com/zh/tutorials/webcomponents/shadowdom/)中有提到一個很有趣的例子。

```
<button id="buttonTag">Hello, world!</button>
<script type='text/javascript'>
  var host = document.querySelector('button');
  var root = host.createShadowRoot();
  root.textContent = 'こんにちは、影の世界!';
</script>
```

雖然顯示的是`こんにちは、影の世界!`，但是如果使用`root.textContent`則會得到`Hello, world!`，這是因為日文字已經被封裝在shadow DOM中，沒辦法直接使用javascript存取。

我們可以參考[ShadowDOM Visualizer](http://html5-demos.appspot.com/shadowdom-visualizer) made by Google。這個網頁可以釐清一些概念。

div tag的`id`為host是整個shadow DOM的插入點 (insert point)，header、section及footer分別代表三個將顯示的內容，而select決定到這三個element所匹配的shadow DOM。

理論上是可以用iteration的方式做成一串shadow DOM，但是這樣有何意義呢？ (笑

參考repository中名片卡的範例 (card.html)
當使用此code時，你可以發現只有`nameTag`是唯一會顯示於瀏覽器中的內容，其他的元素僅僅能被javascript所操控。
將shadow root掛於shadow host稱為一個insert point (`<content></content>`)，這也是讓整個shadow DOM暴露於DOM Tree的一個方法。

基本上，把element封裝於shadow DOM當中，javascript沒辦法存取到值的這種方法非常高端，但也讓HTML有更多的變化性，我們不再僅僅是創造一份充滿html element的垃圾，而是切成一塊塊的DOM TREE，進行個別調整。

## HTML Import

Web Component中，繫緊各元件重要的一環 - 匯入。將html文件匯入至另一份html文件，以便利用它的功能；此外，這樣也方便整理各元件化後的html相依。
好比說script tag的src一般，HTML則是利用`<link rel="">`匯入，值得一提的是，各瀏覽器支援度不高，我們可以利用property 檢查看是否有支援。

儘管瀏覽器很少完整支援HTML Import，但是`Polymer`有強大的polyfill，可以很方便的匯入文件。


```
function isSupport() {
  return 'import' in document.createElement('link');
}
```

## 總結

如果你只需要現成的library且不想了解Web Component的原理，則可以到[Custom Element](http://customelements.io)尋找其他已完成的kit使用；對於developer而言，不僅javascript可以做成套件，HTML也可以套件化，大大地改變前端工程師的開發方法。

有了DOM scoping的思維，元件化後的html更加好維護。但以上所有優點都必須要以高compatibility為優先，原生的Web Component其實並沒有那麼容易做feature detection。


## Google Polymer

Google Polymer則是一個非常強大的library，不管是功能面或是相容性。在2012以後，Google I/O每次都會拿出來討論一次，今年的2014 session中則是把`material design`加入`Polymer`中，作為其UI的特色。

`Polymer`主要分成兩大群組的element，core-element及paper-element，core-element提供了一般javascript library的基本功能，好比說core-collapse的摺疊效果，core-icon提供icon font、core-iconset可以定義自己的icon，簡化製造格式化icon的繁雜手段、core-overlay提供對話視窗效果、core-ajax則是發出ajax的request。

基於`Polymer`仍然在開發階段，內建的component其實沒有像jQuery般的如此強大，但Web Component的好處在於元件化後的element可以用自己的javascript加強，想要再套入underscorejs、jQuery都是可以的。

### Polymer - Use

使用`Polymer`的方法是新建一個html頁面，並且在最前頭加上`polymer.html`
```
<link rel="import" href="../components/polymer/polymer.html">
```
並且開始定義自己的element，好比說
```
<polymer-element name="">

</polymer-element>
```

### Polymer - data binding

data binding結合attribute與property的相關聯，Polymer可以在創建element時，利用`ready()`提供初始化時要做的步驟。
一個簡單的範例：

```
<polymer-element name="polymer-cool">
  <template>
    <style>
      .cool {
        color: red;
        font-size: 18px;
      }
    </style>
    <div class="cool">You are {{praise}} <content></content>!</div>
  </template>
  <script>
    Polymer('polymer-cool', {
      praise: 'cool'
    });
  </script>
</polymer-element>
```

可以直接將`praise`綁到`template`中的`{{praise}}`裡。

### Polymer - Example

需要inject多重值時，可以在`template`上加入一個repeat的屬性，好比以下的範例：

```
<polymer-element name="binding-example">
    <template>
        <h3>Use Iterator In Template</h3>
        <template repeat="{{s in salutations}}">
            <ul>
                <li>{{s.what}}: {{s.who}}</li>
            </ul>
        </template>
    </template>
    <script type='text/javascript'>
        Polymer('binding-example', {
            ready: function () {
                this.salutations = [
                  {what: 'Hello', who: 'World'},
                  {what: 'GoodBye', who: 'DOM APIs'},
                  {what: 'Hello', who: 'Declarative'},
                  {what: 'GoodBye', who: 'Imperative'}
                ];
            }
        });
    </script>
</polymer-element>
```

`repeat`的值設定為`{{s in salutations}}`可以使`this.salutations`進行迭代運算，植入`template`當中。

另外，`Polymer`還有個有趣的功能-`Observer`，設定`observe`來監聽template的element各種狀態，一旦某個屬性改變時，就會觸發事件，angularJS、backboneJS都有類似的實作案例，待其他前輩補充。

以下這個範例是當`#first`element及`#second`element的值改變時，會觸發`cal()`函式，進而重新計算`#final`element要顯示的值。
```
<polymer-element name="x-cal">
    <template>
        <div>
            <input id="first" value="123" type="text"/>
            <input id="second" value="456" type="text"/>
            <input id="final" value="0" class="final" type="text"/>
        </div>
    </template>
    <script type='text/javascript'>
        Polymer('x-cal', {
            firstVal: document.querySelector('#first'),
            secondVal: document.querySelector('#second'),
            observe: {
                '$.first.value': 'cal',
                '$.second.value': 'cal'
            },
            ready: function () {
            },
            cal: function (oldVal, newVal) {
                var a = document.querySelector('#final');
                this.$.final.value = Number(this.$.first.value) + Number(this.$.second.value);
            }
        });
    </script>
</polymer-element>
```
ps: 在投影片中，我們簡單展示如何用observer實作一個加法的功能，可以在短短幾分鐘內完成單純的計算機。


## Polymer - Core Element

在眾多的core-element當中，我們挑選core-ajax當做範例，在`Polymer`的教學中提及可以使用它發出request取得不同種類的資料，
```
<core-ajax auto url="http://gdata.youtube.com/feeds/api/videos/"
    params='{"alt":"json", "q":"chrome"}' handleAs="json" >
</core-ajax>
```

`auto`如果設為`true`，代表當`url`或是`params`改變，就會重新發送一次request；`params`則是發送的參數內容；`handleAs`可以設定為不同種類的回傳值，text代表的是`responseText`、xml代表的是`responseXML`、json代表的是`responseText`(但是會先幫你剖析成json，不用再進行`JSON.parse()`)；當然想要設定`method`為`POST`或是`GET`也行，詳細內容請參閱[spec](http://www.polymer-project.org/docs/elements/core-elements.html#core-ajax)。

## Polymer - Paper Element

這是2014年Google I/O的其中一項重要產品，對於一般使用者來說僅需載入相對應的HTML文件就可以使用。值得一提的是FOUC (Flash of unstyled content)，在element被註冊(`registerElement()`)之前，custom element會被定義為`HTMLUnknownElement`，這代表custom element無法被樣式表所套用，造成會有閃爍的感覺。[參考文件](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/#upgrades)

一般的解決方法是加上`unresolved`屬性，在element被升為一般的`HTMLElement`之前，會一直保持hidden。

## Compatibility - 相容性

原生的Web Component在瀏覽器內支援度相當差，建議直接使用`Polymer`，如果對google提出的toolkit有興趣也可以找找這個[連結](http://googlewebcomponents.github.io)，裡頭有很多有趣的Web Component範例。

## Polymer v.s. AngularJS

最後，簡單介紹一下兩者差異，兩方作者其實對於對方的專案都持樂觀的態度。雙方的內容有些許重疊，但實際上可以把AngularJS使用在每個Component中，所以並不需要太多的取捨。`Polymer`有shadow DOM的支援，在encapsulation有相當大的進步。而`Polymer`主要是提供些高階的API (效果、UI)並消除各瀏覽器不支援Web Component的特性。

剩下來的Polymer進階功能，請參照先前提的參考資料。

