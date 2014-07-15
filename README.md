# Web Component
====================

## 前言

2013年時，ERIC BIDELMAN 就曾經在Google I/O迅速地以 [Web Components: A Tectonic Shift for Web Development](http://webcomponents.org/presentations/web-components-a-tectonic-shift-for-web-development-at-google-io/)介紹Web Component的特色與功用，這是HTML新世代的濫觴，也是接下來發展的新趨勢。

在國內，已有許多優秀的先進將之簡單的介紹一番包括以下：

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

* DIV in hidden

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
        foo: function() {alert('foo() is called');}
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
首先，在tag之中最好要包含一個`-` dash，在polymer當中則是一定要包含，否則會有問題出現。
接著在javascript中利用`createElement()`創造一個element，並賦予他一個event handler，至此就完成建立自己的element。

## Shadow DOM

Shadow DOM在W3C上介紹的相當複雜，包含light DOM、shadow DOM。
主要可以把shadow DOM視為隱藏在DOM中的一些element，我們可以把element封裝於shadow DOM中，使得封裝在shadow DOM的CSS樣式表不會影響其他的element。

[shadow DOM in html5rocks](http://www.html5rocks.com/zh/tutorials/webcomponents/shadowdom/)中有提到一個很有趣的例子，雖然顯示的是`こんにちは、影の世界!`，但是如果使用`root.textContent`則會得到`Hello, world!`，這是因為日文字已經被封裝在shadow DOM中，沒辦法直接使用javascript存取。

我們可以參考[ShadowDOM Visualizer](http://html5-demos.appspot.com/shadowdom-visualizer) made by Google。這個網頁可以釐清一些概念。

div tag的`id`為host是整個shadow DOM的插入點 (insert point)，header、section及footer分別代表三個將顯示的內容，而select決定到這三個element所匹配的shadow DOM。

理論上是可以用iteration的方式做成一串shadow DOM，但是這樣有何意義呢？ (笑

參考repository中名片卡的範例 (card.html)
當使用此code時，你可以發現只有`nameTag`是唯一會顯示於瀏覽器中的內容，其他的元素僅僅能被javascript所操控。
將shadow root掛於shadow host稱為一個insert point (`<content></content>`)，這也是讓整個shadow DOM暴露於DOM Tree的一個方法。

基本上，把element封裝於shadow DOM當中，javascript沒辦法存取到值的這種方法非常高端，但也讓HTML有更多的變化性，我們不再僅僅是創造一份充滿html element的垃圾，而是切成一塊塊的DOM TREE，進行個別調整。

