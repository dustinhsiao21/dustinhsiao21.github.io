---
layout: single
title: 從PHPUnit測試失敗了解Static
categories: [laravel]
---

PHP的web 請求往往是獨立(isolation)運行的，但是PHPUnit卻不是。一般情況下，你可以假設每個請求都會重新加載所有的類，但是PHPUnit卻是於同一個內存空間中運行所有測試，意味著`static變數`於測試的運行中持續存在。這種測試間的依賴關係將會導致測試單獨通過但是於套件中失敗的情況發生，是測試員的一個惡夢。

### 起因

近期的專案導入了[paratest](https://github.com/paratestphp/paratest)這個讓測試開多執行緒提高測試效率的套件(大概是節省以往測試的3-4倍時間)。但是前幾天遇到一個和`Observer`相關的bug，為了要找出到底是哪個環節出了問題，所以攺用了本來的`phpunit --debug`的方式。

一般測試時，如果有拋出錯誤，一般會有phpunit 會有trace讓你知道是哪個測試環節出了問題，如果有使用`Observer`時，只會知道是哪個`Observer`出錯，但是不會知道是哪個測試導致的。而`phpunit --debug`會顯示當下的單元測試名稱可以讓程式人員知道是哪個測試出了問題。

結果在跑到應該要出現錯誤的單元測試前就出現了預期之外的報錯，而這個錯誤在跑`paratest`的時候並沒有出現。後來就針對該項測試用`phpunit --filter`=XXXXXTest的方式去執行，但是卻通過了。一開始以為是假數據出狀況的問題，所以用`phpunit --repeat=100`多跑幾次，還是全部通過。後來上網查了資料一開始以為是`Mockery`忘了關閉，但是後來發現Laravel於[5.1](https://github.com/laravel/framework/blob/5.1/src/Illuminate/Foundation/Testing/TestCase.php)時就解決了這個問題。後來才發現有可能是`static`出問題。

### 關於Static

我們先來看看php官方說法[Static Keword](http://php.net/manual/en/language.oop5.static.php)

>Declaring class properties or methods as static makes them accessible without needing an instantiation of the class. A property declared as static cannot be accessed with an instantiated class object (though a static method can).
>
>### [Static methods](http://php.net/manual/en/language.oop5.static.php#language.oop5.static.methods)
>
>Because static methods are callable without an instance of the object created, the pseudo-variable $this is not available inside the method declared as static.

Static是於類(Class)第一次被加載時就創建的，所以類的外部使用不需要對象(Object)而使用類名就可以訪問到靜態成員。而靜態方法的呼叫也是直接透過類執行，而因為不是對像(object)所以無法使用`$this`方法，一般使用`self::`呼叫。

```php
<?php
class Foo {
    protected static $property;
        
    public static function aStaticMethod() {
        self::$property
    }
}

Foo::$property;
Foo::aStaticMethod();
```

`Static`有點像是一種全域的使用方式，他是直接對應到類，而非實例。例如

```php
<?
class Person {
    public static $age = 10;
    public static function addAge()
    {
        return self::$age++;
    }
}

//如果我們實例化一個人出來
$jane = new Person();
$jane->addAge() //+1
    
Person::$age; //11
//接著我們實例化另一個人出來。
$john = new Person();
$john->addAge() //+1
    
//由於會對應到同一個age，所以不會因為實例化而從新計算
Person::$age; //12
?>
```

所以可能會造成物件內屬性無法封裝的情況發生，在使用上需要格外的注意。而我們遇到的錯誤原因則是在做某個頁面時，因為會存取到固定的值，所以我們便把特定的東西放在`static`裡面，大致上如下：

```php
class Cart{
    public static $items
    public function loadItems()
    {
        //...
        if(!self::$items){
            self::$types = $this->service->getItems();
        }
        //...
    }
}
```

當初是為了處理優化的問題，不需要重複的執行載入動作，但是在測試時可能會需要新增測試資料來判斷是否有正常執行，就會導致無法再次新增，而發生頁面上錯誤的情形。

所以在使用`static`時真的要十分小心阿！

