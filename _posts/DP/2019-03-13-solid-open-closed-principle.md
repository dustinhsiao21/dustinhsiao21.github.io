---
layout: single
title: SOLID 原則 - Open Closed Principle(OCP開放封閉原則)
categories: [dp]
---
>Robert C. Martin(Uncle Bob):
>Entities(class, modules, functions, etc.) should be open for extension, but closed for modification. 
>軟體的對象中，對於擴展是開放的，但是對於修改是封閉的。

一般我們在寫程式的時候，最怕的不是程式寫不出來，而是修改程式。雖然可以從『自動化測試』方面著手，但是如果從『設計面』做最根本處理的話，系統將會變得更易於修改及擴充。

所以我們需要藉由增加新的程式碼來擴充系統的功能，而不是藉由修改原本已經存在的程式碼。(If the OCP is applied well, then further changes of that kind are achieved by adding new code, not by changing old code that already works.)

##### Example

假設現在有一個矩型物件

```php
<?php namespace Acme;

class Square {
    public $width;
    public $height;
    
    function __construct($height, $width)
    {
        $this->height = $height;
        $this->width = $width;
    }
}
```

接著想要計算矩型的面積，根據SRP原則，我們把計算的方法獨立寫一個物件`AreaCalculator`

```php
<?php namespace Acme;

class AreaCalculator {
	public function calculate($squares)
	{
        foreach($squares as $square)
        {
            $area[] = $square->width * $square->height;
        }
        return array_sum($area);
	}
}
```

透過傳入`$squares`的`Square`Array，去透過長*寬做面積計算。在透過`array_sum`取出總和。

假如現在有另外一個需求，要計算圓形面積時，可能會增加一個`CirCle`物件，並在`AreaCalculator`的`calculate`做判斷。

`Circle 物件`:

```
<?php namespace Acme;

class Circle {
    public $radius;
    
    function __construct($radius)
    {
        $this->radius = $radius;
    }
}

```

修正``AreaCalculator`，因為不是`squares`了，所以我們把名稱更改為`$shape`

```php
<?php namespace Acme;

class AreaCalculator {
	public function calculate($shapes)
	{
        foreach($shapes as $shape)
        {
        	if(is_a($shape, 'Square'))
        	{
            	$area[] = $shape->width * $shape->height;    
        	}
			elseif(is_a($shape, 'Circle'))
			{
                $area[] = $shape->radius*$shape->radius*pi();
			}
        }
        return array_sum($area);
	}
}
```

所以說日後如果有增加其他的形狀需要計算，我們就要再多一個`elseif(is_a($shape, 'Triangle')`，因此`calculate`的結構會隨著每一次的修改一直被破壞，如此一來就會違反`OCP`的原則。所以我們要做些修改。

##### Solution

Uncle Bob 曾說過

> Separate extensible behavior behind an interface, and flip the dependencies.
>
> 分離擴展的功能到Interface，反轉依賴關係。

所以我們增加一個interface

```php
<?php namespace Acme;

interface Shape {
    public function area();
}
```

所以我們要把所有的『形狀』實現(implement)出來

```php
//Square
<?php namespace Acme;

class Square implements Shape{
    public $width;
    public $height;
    
    function __construct($height, $width)
    {
        $this->height = $height;
        $this->width = $width;
    }
    
    public function area()
    {
        return  $shape->width * $shape->height;
    }
}

//Circle
<?php namespace Acme;

class Circle implements Shape{
    public $radius;
    
    function __construct($radius)
    {
        $this->radius = $radius;
    }
    public function area()
    {
        return $area[] = $shape->radius*$shape->radius*pi();
    }
}
```

所以我們就可以簡化`calculate`

```
<?php namespace Acme;

class AreaCalculator {
	public function calculate($shapes)
	{
        foreach($shapes as $shape)
        {
 			$area = $shape->area();
        }
        
        return array_sum($area);
	}
}
```

這樣如果我們要另外增加其他的『形狀』，只需在『形狀』的物件內描述`area`的計算方式，而不需要更動到`calculate`。

用結帳當做另外一個例子。現在假設我們有一個`Checkout`的物件，當結帳開始(`begin`)時，我們會藉由客戶的訂購資訊去計算客戶的付款，假設為現金付款(`acceptCash`)。

```php
<?php namespace Acme;

class Checkout {
    public function begin(Receipt $receipt)
    {
        $this->acceptCash($receipt);
    }
    
    public function acceptCash($receipt)
    {
        // accept the cash
    }
}
```

但是如果說客戶用其他的方式來付款呢?CreditCard?Paypel?

所以我們會

1. 新增一個Interface
2. 新增其他付款方式(CashPaymentMethod)
3. 注入付款方式到`begin`

```
// interface
<?php namespace Acme;

Interface PaymentMethodInterface {
    public function acceptPayment($receipt)
}

// CashPaymentMethod
<?php namespace Acme;

Interface CashPaymentMethod implements PaymentMethodInterface{
    public function acceptPayment($receipt)
    {
        // accept the cash
    }
}

// Checkout
<?php namespace Acme;

class Checkout {
    public function begin(Receipt $receipt, PaymentMethodInterface $payment)
    {
        $payment->acceptPayment($receipt);
    }
}
```

如此一來不管新增多少付款方式，只要新增`CreditCardPaymentMethod`, `PaypalPaymentMethod`，即可，不需要更動到`begin`的內容。