---
layout: single
title: 常見變數命名規則(Naming convention)
categories: [laravel]
---

如何命名一個`function`/`Variable`是公認寫程式時最困難/最花時間的項目之一，這次介紹四個命名規則：`Camel Case`, `Pascal Case`, `Snake Case`, `Kebab Case`；以下會介紹這幾個規則的命名方式，及常用場景。`匈牙利命名法`因為太特殊了，筆者沒在用，如果有興趣的可以[詳閱](https://en.wikipedia.org/wiki/Hungarian_notation)。

## CamelCase(駝峰式命名法, camelCase)

中文是完全照自命上去翻譯，就像駱駝的峰一樣，

- 命名方式：通常是只第一單字小寫，其他單字大寫的情況。

- 例子：`getVariableName`，`cartItems`,`testHowToGetAName`等

- 常用場景：適用於一般變數/方法。

有人會依第一個字母的大小寫分為兩類：`Lower Camel Case`, `Upper Camel Case`，不過`Upper Camel Case`又稱為`Pascal Case`，所以比較少人這樣稱呼。

又因這個特性，所以有人會以`camelCase`稱呼，剛好第一個字小寫，第二個字大寫。 

## Pascal Case(Pascal Case)

如上面所述，有人也稱它為`Upper Camel Case`，泛指所有單字第一個自都大寫的命名方式。

- 命名方式：單字都大寫。
- 例子：`UserRepository`。
- 常用場景：` Class`名稱。

## Snake Case(Snake_Case)

如字面的意思，像蛇一樣，所以是用底線連結。

- 命名方式：在單字間加入底線。
- 例子：`ITEM_TYPE`, `created_at`, `updated_at`等。
- 常用場景：`const`變數名稱，資料庫欄位名稱。

## Kebeb Case(Kebeb-Case)

`Kebeb`本身是烤肉串的意思，變數就像烤肉串一樣串在一起。

- 命名方式：在單字間加入破折號hyphen。
- 例子：`good-to-eat`, `cart-item`。
- 常用場景：通常會用在網址。如本篇的`naming-convention`

以上是簡單介紹，至於專案內要如何活用這些變數命名方式，則需仰賴專案的負責同事一起討論規劃了了。