---
layout: single
title: 在Laravel 5.4 內活用 Handler & Exception處理錯誤頁面（淺談）
categories: [laravel]
---

Laravel 在預設上已經幫我們把Exception處理的很好了，相關的內容可以參考[官方文件](https://laravel.com/docs/5.4/errors#the-exception-handler)，這邊會先假設文章已經稍微看過，不針對裡面內容做細數。從相關文件我們可以知道，例外的處理一律都是在`App\Exceptions\Handler`這隻檔案。對`Exception`來說，處理錯誤頁面的部份在`render`這個方法實作。

#### 實作

我們可以先看到5.4版本的render 實作方式

```php
public function render($request, Exception $exception)    
{
	return parent::render($request, $exception);
}
```

接著我們看看`parent`(ExceptionHandler)內的方法

```php
    public function render($request, Exception $e)
    {
        $e = $this->prepareException($e);
        if ($e instanceof HttpResponseException) {
            return $e->getResponse();
        } elseif ($e instanceof AuthenticationException) {
            return $this->unauthenticated($request, $e);
        } elseif ($e instanceof ValidationException) {
            return $this->convertValidationExceptionToResponse($e, $request);
        }
        return $this->prepareResponse($request, $e);
    }
```

首先我們可以看到這邊針對一些Laravel 獨有的 `Exception`去做處理。若是其他的`Exception`則會跑到`$this->prepareResponse($request, $e);`這一段來執行。

再來我們看一下在`prepareResponse`這一段程式碼。

```php
protected function prepareResponse($request, Exception $e)
{
	if ($this->isHttpException($e)) {
		return $this->toIlluminateResponse($this->renderHttpException($e), $e);
	} else {
		return $this->toIlluminateResponse($this->convertExceptionToResponse($e), $e);
	}
}
//...
protected function isHttpException(Exception $e)
{
	return $e instanceof HttpException;
}
```

所以我們知道當`Exception `是`HttpException`時，我們可以執行到`renderHttpException`這個方法。再來我們看看這個方法：

```php
protected function renderHttpException(HttpException $e)
{
	$status = $e->getStatusCode();
	view()->replaceNamespace('errors', [
		resource_path('views/errors'),
		__DIR__.'/views',
	]);
	if (view()->exists("errors::{$status}")) {
		return response()->view("errors::{$status}", ['exception' => $e], $status, $e->getHeaders());
	} else {
		return $this->convertExceptionToResponse($e);
	}
}
```

所以我們可以知道執行到這個方法後便會去找出相對應的`StatusCode`.blade去做顯示。

而若這些`Exception`不是`HttpException`時則會跑到`convertExceptionToResponse`方法。

```php
protected function convertExceptionToResponse(Exception $e)
{
	$e = FlattenException::create($e);
	$handler = new SymfonyExceptionHandler(config('app.debug', false));
	return SymfonyResponse::create($handler->getHtml($e), $e->getStatusCode(), $e->getHeaders());
}
```

當執行到這段程式碼時，錯誤訊息會在`APP_DEBUG = true`的時候做顯示。一般我們在正式環境時，會將`APP_DEBUG = false`，此時只要是拋出`Exception`，就會直接顯示`Whoops, looks like something went wrong.`避免在正式環境暴露出相關的錯誤程式代碼。

但是在有些狀況下，反而會希望使用者看到某些特定的錯誤訊息方便我們去做排錯，而不是在狀況發生後再透過相關的時間/操作上去推敲log位置，例如是內部人員使用的系統。

但是我們又不希望直接噴出相關程式碼的錯誤，指希望拋出我們指定的訊息。

所以我們只要把程式碼導向`renderHttpException`這個方法，就可以取得`blade`的頁面做客製化的需求。

假設我們現在有一個`CustomerException`

```php
//XXXService.php...
throw new CustomerException('SOMETHING YOU SHOULD KNOW');
//
```

在拋出時透過`Handler` 的`render`到`prepareResponse`。所以我們要在`App\Exceptions\Handler`內新增`prepareResponse`去覆寫：

```php
//App\Exceptions\Handler.php
protected function prepareResponse($request, Exception $e)
{
	if (!config('app.debug') and !$this->isHttpException($e)) {
	$e = new HttpException(500, $e->getMessage(), $e);
	}

	return parent::prepareResponse($request, $e);
}
```

如此一來，只要當`.env`裡面的`APP_DEBUG = false`，錯誤訊息都會被捕捉，且會以`errors.500.blade.php`這個畫面顯示(當然你要記得新增這個檔案出來)。相關的應用如`400`系列的錯誤訊息也可用類似方法撰寫。

#### 後記

因為筆者手上還有Laravel 5.4以下的專案(受限於php 5.6)，所以這部份需要自己手刻，但其實目前最新的版本[Laravel 5.7](https://github.com/laravel/framework/blob/5.7/src/Illuminate/Foundation/Exceptions/Handler.php)已經針對這個項目做過優化，若您是使用Laravel5.7可當做走馬看花看看就好。