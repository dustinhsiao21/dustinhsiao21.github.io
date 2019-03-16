---
layout: single
title: 淺談在LARAVEL 內用MOCKERY寫測試
categories: [laravel]
---

一般在寫測試時，無論是Unit test 或是Integration test，都會很頻繁的用到Mock的功能，尤其是當測試的案例涉及到`第三方單位的API時`，應該不會有人希望真的打API過去測試才對(如果是簡訊的話真的會扣錢QQ)...。Mockery的功用要是在於可以『模仿/代替』你要執行的程式碼。舉例來說，假設要寫一個『訂單成立後開發票』的測試，程式碼可能會很像以下

```php
<?php

class OrderServiceTest extends \Tests\TestCase
{
    public function setUp()
    {
        parent::setUp();
        $this->order = app(OrderService::class);
        $this->invoice = app(InvoiceService::class);
    }
	//....
    public function testNewOrder()
    {
        //arrange
        //...
            
        //act
        $order = $this->order->newOrder($data);
        $result = $this->invoice->newInvoice($order);
        //如果這邊newInvoice沒有mock，就會把資料真的傳送給開立發票的單位。
        //assert
    }
}
```

### 實作

一般筆者的習慣會在TestCase內新增一個initMock的方法，這樣所有繼承的測試都可以直接使用這個方法。

```php
<?php

class TestCase extends IlluminateTestCase
{
	//....
    /**
     * 初始化mock物件
     *
     * @param string $class
     * @return Mockery
     */
    protected function initMock($class)
    {
        $mock = \Mockery::mock($class);
        $this->app->instance($class, $mock);

        return $mock;
    }
}
```

所以在測試檔案內就可以直接用`$this->initMock(CLASS)`去Mock要執行的類。

假設在上述的`InvoiceService`類內有個`newInvoice`的方法，我們可以先把類Mock起來。

```php
<?php

class OrderServiceTest extends \Tests\TestCase
{
    public function setUp()
    {
        parent::setUp();
        $this->order = app(OrderService::class);
        //$this->invoice = app(InvoiceService::class);這個不需要了
        $this->invoice = $this->initMock(InvoiceService::class);//Mock要往外送的類
    }
	//....
}
```

把類Mock起來後，我們可以對他做一些設定。相關的方法可以參閱vendor內的`/mockery/mockery/library/Mockery/Expectation`。這邊舉幾個筆者常用的方法：

- `shouldReceive()`:應該被呼叫的方法，假設你要呼叫InvoiceService內的`newInvoice` 方法，就可以寫成`shouldReceive('newInvoice')`

- `once()`:只呼叫一次

- `with()`:應該被傳入的參數。

  > 這邊強烈建議用`with`取代`withAnyArgs()`。可以當做是一種`assert`來確認傳入方法的參數是否正確，尤其當執行的物件要執行到Mock的方法前還有經過很多方法時，用`with()` 可以確保資料傳入的正確性。

- `andReturn()`:回傳的參數。

- `andReturnUsing(function()use(){...})`：和回傳參數一樣，只是允許傳入一個clousre，筆者有時候會直接用`factory`當做返回的內容。

- `->andThrow(new Execption('xxx'))`:拋出例外。

所以最後可能會變成這樣：

```php
<?php

class OrderServiceTest extends \Tests\TestCase
{
    public function setUp()
    {
        parent::setUp();
        $this->order = app(OrderService::class);
        $this->invoice = $this->initMock(InvoiceService::class);//Mock要往外送的類
    }
	//....
    public function testNewOrder()
    {
        //arrange
        $invoiceReturn = [
            'invoice_number' = 'xxx'
            //...
        ];
        //...
            
        //act
        $order = $this->order->newOrder($data);
        $this->mock->shouldReceive('newInvoice')
            		->once()
            		->with($order)
            		->andReturn($invoiceReturn);
        $result = $this->invoice->newInvoice($order);
        //$result 內就會是$invoiceReturn的資料，接著可以繼續往下執行。
        //assert
    }
}
```

以上就是簡單的使用Mockery的方式。

### 提醒

這邊有個常見的錯誤，如果你要Mock的物件包含在要測試的物件內，就必須讓mock比待測物件提早執行。假設我們現在要深入的測試一下`InvoiceService`這個物件，但是我們又不希望`newInvoice`真的被執行 ，所以我們要把真正執行送出功能的物件給Mock起來。假設這邊真正執行`newInvoice` 功能的是一個Invoice 的類，可能是這樣：

```php
class InvoiceService
{
	public function __construct(Invoice $invoice)
    {
        $this->invoice = $invoice
    }
    
    public function newInvoice(Order $order)
    {
        $invoiceData = $this->formatOrder($order);
        return $this->invoice->invoice($invoiceData);
    }
    
    private function formatOrder(Order $order)
    {
        //...
    }
}
```

注入的`Invoice`可能是個SDK，是真正把資訊往外送的地方。

所以我們在寫`InvoiceService` 測試時，可能真正要`Mock`的是 `Invoice`這個類。而mock 要在`依賴注入前先執行`

```php
class InvoiceServiceTest extends \Test\TestCase
{
    public function setUp()
    {
        $this->mock = $this->initMock(Invoice::class);
        $this->service = app(InvoiceService::class);
    }
}
```

這樣`Invoice`才會真正的被Mock起來。

### 結語

在寫測試時，尤其是Integration test，`務必`要隨時注意串接第三方套件的情況，否則有時候就會發生，如果你有串簡訊功能，每跑一次測試，就會寄出一封簡訊，導致點數越來越少的情況(~~筆者絕對不會承認有發生過以上事件~~)。



