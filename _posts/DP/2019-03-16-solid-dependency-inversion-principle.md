---
layout: single
title: SOLID 原則 - Dependency Inversion Principle(依賴反轉原則)
categories: [dp]
---
>Depend on abstractions, not on concretions
>物件的相依關係應該要依賴於抽象類別，而非抽象出來的實例(concretions)。

舉一個最簡單的例子：

我們家裡也許都有檯燈，電視或是音響等需要電力的電力設備，而這些設備要接電時，並不是直接對著房子的電線接，而是對著插頭，這個插頭就可以當作抽象類別(abstractions)，房子並不需要知道電力設備的相關細節，只需要知道他需要供電給這些插頭；而這些電力設備需要符合這些抽象類別的實作及細節。

>High level code isn't as concerned with details
>
>Low level code is more concerned with details and specifics

`High level code`及`low level code`的關係是相對的，比較的依據則是誰擁有誰(owner)。在上述例子來說房子就是`High level code`，是整個環境的`owner`，而它裡面所包含的物件內容，包含插頭(`abstractions`, `interface`...)及電力設備(`low level code`)，所以房子並不依賴於電力設備，電力設備也不依賴房子，而是他們兩個都依賴插頭這個介面。

>High-level modules should not depend on low-level modules. Both should depend on abstractions.
>
>Abstractions should not depend on details. Details should depend on abstractions.

接下來我們來看看程式碼的例子。

假設我們有一個密碼提醒的`PasswordReminder`類別，因為需要去資料庫撈資料，目前先假設我們想用`MySQL`資料庫，所以透過`Dependency Injection`的方式注入一個叫做``MySQLConnection` `的方法。

```php
<?php

class PasswordReminder
{
    private $dbConnection;
    
    public function __construct(MySQLConnection $dbConnection)
    {
		$this->dbConnection = $dbConnection;        
    }
}
```

但是如果我們需要改用其他資料庫的話，如`MongoDBConnection`，不就要撤換掉`MySQLConnection`?

所以這邊就違反了`DIP`原則，我們不應該讓`PasswordReminder`這個高階模組去直接依賴`MySQLConnection`這個低階模組，而是要讓兩個依賴抽象，我們這邊新增一個`ConnectionInterface`，並包含一個`connect`方法

```php
interface ConnectionInterface
{
    public function connect();
}
```

接著讓低模組(`MongoDBConnection`及`MySQLConnection`)實作抽象的細節。

```php
<?php
interface ConnectionInterface
{
    public function connect();
}

class MySQLConnection implements ConnectionInterface
{
    public function connect()
   	{
        //...
   	}
}

class MongoDBConnection implements ConnectionInterface
{
    public function connect()
   	{
        //...
   	}
}
```

而最後，讓`PasswordReminder`相依於`ConnectionInterface`這個抽象，而非低模組。

```php
class PasswordReminder
{
    private $dbConnection;
    
    public function __construct(ConnectionInterface $dbConnection)
    {
		$this->dbConnection = $dbConnection;        
    }
}
```

如此一來，對`PasswordReminder`這個高階模組來說，他不需要知道抽象的細節，只需要知道它提供了那些功能，並使用那些功能即可。實作的細節應該讓低階模組透過抽象去實踐，如此一來便可增加程式的擴充型及靈活性，也會一併符`OCP`原則(open for extension, but closed for modification)，當需要新功能時用plug-in的方式處理。也會符合`LSP`原則(Subtypes must be substitutable for their base types)，以確保低模組間的切換並不會讓系統異常。

最後最重要的也是最難懂的就是上述的全部SOLID原則，都會回歸到一個本質

>program to an interface,  not an implementation

All of this is about decoupling code!

最終全部的程式碼如下

```php
<?php
interface ConnectionInterface
{
    public function connect();
}

class MySQLConnection implements ConnectionInterface
{
    public function connect()
   	{
        //...
   	}
}

class MongoDBConnection implements ConnectionInterface
{
    public function connect()
   	{
        //...
   	}
}

class PasswordReminder
{
    private $dbConnection;
    
    public function __construct(ConnectionInterface $dbConnection)
    {
		$this->dbConnection = $dbConnection;        
    }
}
```

