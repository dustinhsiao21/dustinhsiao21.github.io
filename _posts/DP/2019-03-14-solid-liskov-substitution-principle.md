---
layout: single
title: SOLID 原則 - Liskov Substitution Principle(里式替換原則)
categories: [dp]
---
>Derived classes(*Subtypes*) must be substitutable for their base classes(base types)
>
>若程式內有使用到繼承，或是Interface的實作，則在系統中，凡base types(父類別或是interface)出現的地方，都可以被subtypes(子類別或是該interface的實作)來取代，而不會破壞程式原有的行為。

先舉個簡單的例子：

假設有一個類別`A`，裡面有的`fire`的方法，另外有個類別`B`繼承了類別`A`。

若有個`doSomething`的方法會用到類別`A`，按照`LSP`的原則，類別A可以被替換成類別`B`而不出錯。

```php
<?php
class A {
    public function fire(){}
}

class B extends A{
    public function fire(){}
}

function doSomething(A obj)
{
    
}
```

乍看之下這個原則是乎很容易理解且達成，但是我們在開發時往往會出現以下狀況：

現在假設一個`VideoPlayer`類別，包含了`play`的方法。`play`方法預計會返回一個結果。

現在有另一個`AviVideoPlayer`類別，繼承了`VideoPlayer`，也實作了`play`的方法，並加入了檔名的判斷，若非avi的檔名就會拋出錯誤例外。

```php
<?php
class VideoPlayer{
    public function play($file){
        
    }
}

class AviVideoPlayer{
    public function play($file){
    	if(pathinfo($file, PATHINFO_EXTENSION) !== 'avi')
    	{
            throw new Exception(); //violate the LSP
    	}
    }
}
```

但在這樣的情況下就違反了`LSP`原則：因為`AviVideoPlayer`的`play`方法內，有可能會拋出例外，和`VideoPlayer`的預期結果不符。所以在替換時，可能會造成系統會有錯誤的情況發生。

上面想個都是以類別當做例子，接下來我們來看看`interface`的例子：

假設一個`interface`取名為`LessonRepositoryInterface `裡面定義了`getAll()`的方法。因為是課程的Repository，所以我們可能會用`file`或是`Db`(database)的方式去取得課程資訊。

```php
<?php

interface LessonRepositoryInterface {
    public function getAll();
}

class FileLessonRepository implements LessonRepositoryInterface{
    public function getAll()
    {
        return [];
    }
}

class DbLessonRepository implements LessonRepositoryInterface{
    public function getAll()
    {
        return Lesson::all();
    }
}

function foo(LessonRepositoryInterface $lesson)
{
    $lessons = $lesson->getAll();
    
    if(is_array($lessons)){
        //...
    }elseif($lessons instanceof Collection)){
        $lessons->toArray();
    }
}
```

這也很明顯的違反了`LSP` 。在`Laravel`中，`Lesson::all()`會返回的是`Collection`，而非陣列，所以對兩者來說，他們的`輸出output`並不相同，可能會造成系統的異常。

而對於`foo`這個方法來說，他也違反了`OCP(Open-Closed Principle)`原則，在方法內來判斷輸入類別。

所以我們可以知道當破壞了`LSP`的同時，可能也代表你的程式碼在設計的過程中也違反了`OCP`。 

透過`OCP(Open-Closed Principle)`的方式，我們可以定義`interface`的方法，可是僅能限制`輸入(input)`，但是我們卻無法去限制`輸出`，在PHP7以前由於受限於`PHP`本身限制，我們無法定義輸出的`Postconditions`，所以我們僅能在`Interface`內利用`PHPDOC`的方式註明回傳的型別。來讓工程師知道該方法在實作時應該要符合相對應的輸出類別。

```php
interface LessonRepositoryInterface {
	/**
	* Fetch all records
	*
	* @return array
	*/
    public function getAll();
}

class DbLessonRepository implements LessonRepositoryInterface{
    public function getAll()
    {
        return Lesson::all()->toArray();
    }
}

function foo(LessonRepositoryInterface $lesson)
{
    $lessons = $lesson->getAll();
}
```

總結來說，在不違反`LSP`的原則下設計類別/Interface ，需遵守下列四點：

1. Signature must match.(子類別的方法必須和父類別一致)
2. Preconditions can't be greater.
3. Postconditions at least equal to. 
4. Execption types must match.