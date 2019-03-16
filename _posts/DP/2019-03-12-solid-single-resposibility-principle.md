---
layout: single
title: SOLID 原則 - Single Resposibility Principle(SRP單一職責原則)
categories: [dp]
---
SOLID原則代表物件導向中的五種不同的開發原則，從這些原則中才衍伸出不同的Design Pattern，而Design Pattern的實踐，大多都是為了不違反SOLID原則，所以謹記SOLID原則可以讓程式碼在維護上會更有彈性。SOLID原則共包括:

1. ### Single Resposibility Principle(SRP單一職責原理)

2. ### Open-Closed Principle(OCP開放封閉原則)

3. ### Liskov Substitution Principle(里式替換原則)

4. ### Interface Segregation Principle (介面隔離原則)

5. ### Dependency Inversion Principle(依賴反轉原則)

### Single Resposibility Principle(SRP單一職責原理)

> A class should have only one reason to change
>
> 一個class 應該只負責一項工作。

一個class 只處理該class負責的事項，將不相關的責任移到另外的介面，增加class的內聚力(cohension)，減少耦合(coupling)。

以下有個簡單的例子：假設我們有一個把銷售報告輸出的物件，稱為`SalesReporter`好了，如果就比較方便的義大利麵條式寫法，我們會把全部的東西都丟到同一個class內，以輸出報告來說，她至少要需要幾個功能

- 輸入時間區間
- 到資料庫內撈資料
- 計算
- 輸出格式

```php
<?php

class SalesReporter {
    public function between($start, $end)
   	{
        $sales = $this->queryDBForSalesBetween($start, $end);
        
        return $this->format($sales);
   	}
   	
   	protected function queryDBForSalesBetween($start, $end)
   	{
        return DB::table('sales')->whereBetween('created_at', [$start, $end])->sum('charge') / 100;
   	}
   	
   	protect function format($sales)
   	{
        return "<h1>Sales: $sales</h1>";
   	}
}
```

好了，我們現在在`SalesReporter`內新增了一個`between`方法，讓客戶輸入數值，接著利用`queryDBForSalesBetween`方法去資料庫撈資料及計算，最後透過`format`輸出，所以我們成功地將所有的方法都放在同一個class內，這種方法有個比較常見的稱呼為`義大利麵條式程式(Spaghetti code)`，在維護上會不方便許多。

假設你需要更改從資料庫的取得，你要來`SalesReporter`，或是你要更改輸出的方式，也要來`SalesReporter`處理。但是這些實際上並不屬於`SalesReporter`的職責，它應該只需要知道哪些區間的資料，然後讓這些區間的資訊傳給一個物件去搜尋及計算，接著將搜尋出來的結果，再丟給某個物件去輸出成想要的格式。

所以我們首先可以把搜尋方法抽離出一個`SalesRepository`，並可以把方法解化成`between`就好。

```php
<?php

class SalesRepository{
    public function between($start, $end)
    {
    	return DB::table('sales')->whereBetween('created_at', [$start, $end])->sum('charge') / 100;   
    }
}
```

再來處理`format`的方法，在例子內我們式輸出成`html`的方式，但是難保之後不會有其他像是`json`或是`array`的格式，所以我們可以建立一個介面來定義輸出的方法，讓所有輸出的格式去實做。

建立一個`SalesOutputInterface`的介面，並含有一個`output`的方法。

```php
<?php

interface SalesOutputInterface{
    public function output($sales);
}
```

接著實作`HtmlOutput`

```php
<?php

class HtmlOutput implements SalesOutputInterface{
    public function output($sales)
    {
     	 return "<h1>Sales: $sales</h1>";   
    }
}
```

所以這樣我們就可以簡化本來的`SalesReporter`

```php
<?php

class SalesReporter {
	
	private $repo;
	
	public function __construct(SalesRepository $repo)
    {
        $this->repo = $repo;
    }
    
    public function between($start, $end, SalesOutputInterface $formatter)
   	{
        $sales = $this->repo->between($start, $end);
        
        return $formatter->output($sales);
   	}
}
```

接著我們再使用時就需要用依賴注入的方式新增

```php
$report = new SalesReporter(new SalesRepository);

$start = Carbon\Carbon::now()->subDays(10);
$end = Carbon\Carbon::now();

return $report->between($start, $end, new HtmlOutput);
```

這樣做的好處是在之後程式碼越來越多時，維護上會變得更為容易，假設之後需要增加其他的功能，我們可以根據職責把功能寫在不同的class內，降低彼此間的耦合，另一方面提高程式的可讀性，在撰寫測試案例的時候也會比較容易。