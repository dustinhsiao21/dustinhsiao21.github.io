---
layout: single
title: 利用Jquery的on 及off 開關套件Bootstrap-toggle
categories: [js]
---

相信大家都用過很多以`Boostrap`為基礎的Plugin，而`Bootstrap`又相依到`Jquery`這個`Javascript`的Library， 所以很多Plugin都是以`Jquery`驅動的，如這次要提到的例子[Bootstrap-toggle](http://www.bootstraptoggle.com/)，或是可參照[Bootstrap Library](https://startbootstrap.com/bootstrap-resources/)。

### 簡介Bootstrap-toggle

`Bootstrap-toggle`是一個相當常見且好用的`checkbox on-off`套件。[可參照](http://www.bootstraptoggle.com/)安裝使用(這邊先跳過安裝過程)。

筆者在使用的時候主要是用API的方式：

```html
<input type="checkbox" id="toggle-test" checked>
<script>
$(function(){
    $('#toggle-test').bootstrapToggle();
})
</script>
```

而在`bootstrapToggle()`內可以填入option，以下只舉個筆者常用的屬性。

- on：當checkbox為on時顯示的text，預設為`on`
- off：當checkbox為off時顯示的text，預設為`off`
- size：按鈕的大小，分為`large, normal,small, mini `預設為`normal`
- onstyle：按鈕的外觀樣式，分為`default, primary, success, info, warning`，就是Bootstrap內常見的樣式，預設為`default`。

所以以上例子可以改寫成

```html
<script>
$('#toggle-test').bootstrapToggle({
    on:'開啟', 
    off:'關閉',
    size:'small', 
    onstyle:'success'
});
</script>
```

接著你可以透過`Event`去處理開關之後要做的事情，譬如你有一個開關的套件，要開啟後執行一些事情，例如向後送API:

```js
$('#toggle-test').bootstrapToggle({
    on:'開啟', 
    off:'關閉',
    size:'small', 
    onstyle:'success'
}).change(function(){
    $.post('YOUR/API/URL', function(response){
        console.log(response)
    }).fail(function(response){
        console.info(response)
    });
});
```

`change`是指當按鈕被改變時的事件。

假設現在有個情境，當`post`為`fail`時，要讓按鈕回復到`off`的樣式。若直接用直覺的方式處理:

```js
//...
.change(function(){
	var $this = $(this);
    $.post('YOUR/API/URL', function(response){
        console.log(response)
    }).fail(function(response){
        $this.bootstrapToggle('off');
    });
});
```

因為觸動`bootstrapToggle(toggle)`後也會一併的觸發`change`事件，所以會再一次的執行`post`，嚴重點可能會出現`無限callback`。

那不然我們設置一個flag當作開關好了：

```js
var postError = false;
$('#toggle-test').bootstrapToggle({
    on:'開啟', 
    off:'關閉',
    size:'small', 
    onstyle:'success'
}).change(function(){
    var $this = $(this);
    if(postError){
        return true;
    }
    $.post('YOUR/API/URL', function(response){
        console.log(response)
    }).fail(function(response){
        postError = true;
        $this.bootstrapToggle('off');
        postError = false;
    });
});
```

這樣城市是可以正常運作的，不過就...阿...醜了點，所以接下來就會提到這次的主角`Jquery`的`on.off`事件。

### Jquery的on及off

參考Jquery的document：[on](http://api.jquery.com/on/)及[off](http://api.jquery.com/off/)，可以發現這兩個項目可以直接當作是件的開關。而在我們的例子裡，這是要暫時的關閉`change`這個事件。所以上述的程式碼可以簡潔如下：

```js
function notify(){
    var $this = $(this);
    $.post('YOUR/API/URL', function(response){
        //...
    }).fail(function(response){
        $this.off('change'); //先關閉Change事件。
        $this.bootstrapToggle('off');
        $this.on('change', notify);  //執行完後打開Change 事件，並執行handler
    });
}

$('#toggle-test').bootstrapToggle({
    on:'開啟', 
    off:'關閉',
    size:'small', 
    onstyle:'success'
}).change(notify);
```

這樣是不是清楚又明瞭多了呢!