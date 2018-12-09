---
layout: single
title: 在VS Code內簡單使用Flow - Javascript靜態類型檢查工具
categories: [js]
---

## 在VS Code內簡單使用Flow - Javascript靜態類型檢查工具

這邊文章要簡單介紹`Flow`這個Javascript的靜態類型檢查工具。Javascript是一個動態資料型態的程式語言，換言之在宣告變數的時候不需要指定型別，所以很可能會出現這樣的宣告方式:

```js
//好啦這樣有點誇張...
let numberOfProducts = '123' //你以為這是數字，結果裡面放了字串...
```

`Flow`的出現就是為了解決這個問題，尤其是專案越大時，變數的可讀性/識別度就相當重要。當然要把Javascript改用成強型別的語言有其他方式，例如可以用整合性更好的`TypeScript`這個超集語言。但是由於`TypeScript`的整合成本較高，所以可以暫時先把`Flow`引入開發流程當中。可參考Vue作者雨溪大大的文章:[Vue 2.0 为什么选用 Flow 进行静态代码检查而不是直接使用 TypeScript？](https://www.zhihu.com/question/46397274)

如標題說的，`Flow`是一個JS的靜態類型`檢查工具`，所以這項工具只會在`開發`時啟用，會在撰寫程式碼時出現提醒，並再最後編譯時可以用babel工具來移除。

`Flow`的使用也十分簡單，只要再程式碼的最上面加入`// @flow`的註解，`Flow`就會直接進行檢查。

##### 安裝

首先透過全域安裝

```bash
npm install -g flow-bin
```

再來到專案底下初始化`Flow`的設定檔

```bash
 flow init
```

初始化完後會出現`.flowconfig`這隻檔案。

接下來只要在你的`JS`程式碼最上頭加上`// @ flow`後

```js
// @flow
function foo(x: number): string {//輸入應該是number 輸出應該是string
    if (x) {
      return x;
    }
    return "default string";
}
foo('123'); //輸入用string
```

在終端機輸入`flow`

```bash
flow
// result
Error -----------------------------------------------------------------------------------------------                               
Cannot return x because number [1] is incompatible with string [2]. 
 [1][2] 15| function foo(x: ?number): string { 
        16|     if (x) {        
        17|       return x;     
        18|     }               
        19|     return "default string"; 
        20| }                              
Error --------------------------------------------------------------

Cannot call foo with '123' bound to x because string [1] is incompatible with number [2].            
   
 [2] 15| function foo(x: ?number): string {           
     16|     if (x) {
     17|       return x;         
     18|     }                  
     19|     return "default string";        
     20| }                          
 [1] 21| foo('123');        
```

就會開始檢查靜態型別有無問題。

使用VS code當作開發環境，記得先到設定內取消`javascript`的檢查，案`CTRL+,`並輸入validate找到`Javascript.validate:Enable`取消勾選。

另外也可以安裝外掛程式[Flow language Support](https://marketplace.visualstudio.com/items?itemName=flowtype.flow-for-vscode)，可以在撰寫程式碼時透過VS code直接告知錯誤。

```bash
[flow] Cannot return `x` because numebr[1] is incompatible with string [2]
[flow] Cannot call `foo` with `123` bound to `x` because string [1] is incompatible with number [2] 
```

如果要在輸出時移除`flow`，要使用到`babel編譯器` ，我們可以透過命令列執行

```bash
npm install -g babel-cli //安裝全域
npm install --save-dev babel-plugin-transform-flow-strip-types //在專案內安裝移除套件

```

接著建立一個`.babelrc`設定檔，並加入以下:

```js
{
  "plugins": [
    "transform-flow-strip-types"
  ]
}
```

接著就可以使用babel編譯了，這邊需要知道要編譯到的目錄檔案，舉例來說你撰寫程式碼的地方在`src`然後要轉到`dist`這個目標目錄

```bash
babel src -d dist
```

##### 簡單應用

`Flow`提供了簡單的[測試環境](https://flow.org/try/)可以供線上使用。

提供的[基礎型別](https://flow.org/en/docs/types/primitives/)有

- Booleans
- Strings
- Numbers
- null
- undefined
- symbols

最基本的變數宣告，需在變數後面加上`:型別`。

```js
let numberOfProducts: number = 123; //Works!
let numberOfProducts: number = '123';//Error!
//Cannot assign `'123'` to `numberOfProducts` because string [1] is incompatible with number [2].References:...

//也可以用(|)or
let numberOfProducts: number|string = '123';// Works!
let numberOfProducts: number|string = 123;//Works!
```

方法(function):

```js
// @flow
function concat(a: string, b: string): string {
  return a + b;
}

concat("foo", "bar"); // Works!
// $ExpectError
concat(true, false);  // Error!
```

輸入的參數也可以用非必填(optional)，非必填的選項可以是`null`, `undefinded`。只要再型別前面加上`?`即可`。

```js
// @flow
function concat(a: string, b: ?string): string {
  return 'correct';
}

concat("foo"); // Works!
concat("foo", null); // Works!
concat("foo", '123'); // Works!
concat("foo", 123); // Errors!
```

物件:

```js
// @flow
var obj2: {
  foo: number,
  bar: boolean,
  baz: string,
} = {
  foo: 1,
  bar: true,
  baz: 'three',
};
```

如果這個物件會被重複使用，也可以直接用`type`宣告。

```js
// @flow
type A = {
  foo: number,
  bar: boolean,
  baz: string,
}
var obj2: A = {
  foo: 1,
  bar: 222, //error
  baz: 'three',
};
```

以上是比較基本的應用，其他應用可以參照[官網說明](https://flow.org/en/docs/types/)

