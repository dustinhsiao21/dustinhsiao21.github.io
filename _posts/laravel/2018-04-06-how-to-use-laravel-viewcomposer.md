---
layout: single
title: 淺談Laravel 的 ViewComposer應用
categories: [laravel]
---

## 時機

在Laravel內的ViewComposer可以用來處理不同畫面內用到相同參數的情況。舉例來說，現在有一個購物網站，網站內的某個店家可能會有許多頁面需要重複用到店家的基本資料，像是[關於我], [店家商品], [店家聯絡資料], [店家...], 一般我們會對每一個頁面(Controller)傳送固定的資料。舉例如下

```php
class StoreController extends controller
{
    public function index()
    {
    	$info = Store::find(1);
        //...get some data
        return view('index', compact('info', ...));
    }
    
    public function order()
    {
        $info = Store::find(1);
        //...get some order
        return view('order', compact('info',...));
    }
    
    public funtcion products()
    {
    	$info = Store::find(1);
        //...get some products
        return view('products', compact('info', ...))
    }
}
```

因為在三個頁面內都需要用到`info`內的資料，所以需要重複寫三次，但是若之後要維護的時候就要一次改三個地方，這種重複的Code通常就是重構的第一目標，這時候就非常適合用ViewComposer處理。



## 方法

1. 首先要先新增一個你要使用的`Composer`。你可以新增在`app/Http/ViewComposers`內。(沒有的話就自行新增吧!)

```php
   <?php

   namespace App\Http\ViewComposers;

   use Illuminate\View\View
   use App\Repositories\StoreRepository

   class StoreComposer
   {
        /**
        * The store repository implementation.
        *
        * @var StoreRepository
        */
       protected $stores;

       /**
        * Create a new store composer.
        *
        * @param  StoreRepository  $stores
        * @return void
        */
       public function __construct(StoreRepository $stores)
       {
           // Dependencies automatically resolved by service container...
           $this->stores = $stores;
       }

       /**
        * Bind data to the view.
        *
        * @param  View  $view
        * @return void
        */
       public function compose(View $view)
       {
           $view->with('info', $this->stores->find(1));
       }
   }
```

   若以上述例子為例，我們綁定了一個參數名稱為`info`的資料，接下來要處理那些頁面要包含這些資料。

2. 利用 `ServiceProvider`註冊。我們可以透過新增`ViewComposerServiceProvider`來統一管理`ViewComposer`的內容。

```php
   <?php

   namespace App\Providers;

   use Illuminate\Support\Facades\View;
   use Illuminate\Support\ServiceProvider;

   class ViewComposerServiceProvider extends ServiceProvider
   {
       /**
        * Register bindings in the container.
        *
        * @return void
        */
       public function boot()
       {
           // Using class based composers...
           View::composer(
               'index', 'App\Http\ViewComposers\StoreComposer'
           );

           // Using Closure based composers...
           View::composer('dashboard', function ($view) {
               //
           });
       }

       /**
        * Register the service provider.
        *
        * @return void
        */
       public function register()
       {
           //
       }
   }
```

   可以透過View::composer的方式綁定。第一個參數是`綁定blade的名稱`， 第二個是綁定的內容來源，可以透過`class`or`Closure`的方式。

   當然也可以綁定到多個view:

   ```php
   View::composer(['index', 'orders', 'products'], 'App\Http\ViewComposers\StoreComposer')
   ```

3. 最後就是要在`config/app.php`內加入到`providers`的`array`內

```php
   'providers' =>[
       //...
    	App\Providers\ViewComposerServiceProvider::class,
   ];
```

如以一來就可以在多個View內綁定`info`這個參數了，所以你可以在`index.blade.php`, `orders.blade.php`, `products.blade.php`內使用`{% raw %}{{ $info->xxx }}{% endraw %}`來取得商店資料內容了。

**註：**其實這樣會有一個壞處，就是在多人合作時如果同事沒有用過這種方式，他會找不到資料的來源到底是哪裡，因為在Controller明明就沒有傳值過去...所以若要在專案內使用此方法，記得要在View內註解一下囉。