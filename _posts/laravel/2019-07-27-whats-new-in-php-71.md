---
layout: single
title: What's news in PHP7.1
categories: [laravel]
---
## Symmetric Array Destructuring

可以用來取代以往`list` 功能，例如

```php
<?php

//php 7.0 以前
list($name, $phone) = ['John', '0900123123'];
var_dump($name, $phone); //John, 0900123123

//php 7.1
[$name, $phone] = ['John', '0900123123'];
var_dump($name, $phone); //John, 0900123123
```

如果`index` 為字串的情況下也可以使用，不過就要加上名稱，如

```php
<?php

$array = ['name' => 'john', 'phone' => '0900123123'];

['name' => $name, 'phone' => $phone] = $array;

var_dump($name, $phone) //John, 0900123123
```

## Nullable and Voids Type

在php7.0 之後加了一個很重要的功能，就是你可以加入Typehint 及return type，會讓程式及IDE可以幫你除錯。在7.1的時候更加入了 `nullble`及`void`兩個 type。例如

```php
<?php
class User {
    protected $age;
    
    protected function __construct($age)
    {
        $this->age = $age;
    }
    public function getAge() : number
    {
        return $this->age;  
    }
}

var_dump((new User(21))->getAge()); //21
```

若是沒有輸入數字，則會是`null`，可以用nullable的型態表示，或是用 `？`。

```php
<?php
class User {
    protected $age;

    public function getAge() : ?number
    {
  	    return $this->age;  
    }
}

var_dump((new User())->getAge()); //null
```

而`void` 則是用在沒有回傳的情況，償傭在儲存方法，如

```php
<?php

use App\Models\User;

class UserRepository {
	public function save(User $user, array $array) : void
	{
		$user->fill($array);
    $user->save();
	}
}
```

## Multi-Catch Exception Handling

當使用try & catch時，若要對不同的`exception`做相同操作，則可以使用。例如

```php
<?php

class ChargeRejected extends Exception {}
class NotEnoughBalance extends Exception {}

class Opal {
    private $balance;
    public function pay()
    {
        if($balance < 2){
            throw new NotEnoughFunds;
        }
    }
}

try{
    (new Opal())->pay() 
} catch(ChargeRejected | NotEnoughBalance $e) {
    var_dump('you need to top up your opal card');
}
```

## Iterables

新增了一個`interable` 的`pseudo-type`。一般我們在迭代時，往往都會用`array`，如果用到laravel，可能多個`collection`，如果一個方法可以同時接受這兩個迭代的類型，則可以使用`interable`

```php
<?php
function dumpAll(interable $items){
    foreach($items as $item){
        var_dump($item);
    }
}

class NewCollection implements InteratorAggregate
{
    protected $items;

    public function __construct($items)
    {
        $this->items = $items;
    }

    public function getInterator()
    {
        return new ArrayIterator($this->items);
    }
}
$array = ['one', 'two', 'three'];
$collection  = new NewCollcation($array);

dumpAll($array); // 'one', 'two', 'three'
dumpAll($collection); // 'one', 'two', 'three'
```

Let's All~