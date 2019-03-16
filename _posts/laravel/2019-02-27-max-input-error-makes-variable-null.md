---
layout: single
title: max_input_vars 錯誤導致資料傳輸異常
categories: [laravel]
---

`Input variables exceeded 1000`的錯誤是因為在`php.ini`內`max_input_vars`預設值為`1000`，當數量超過`1000`時`php`會拋出`E_WARNING`，`E_WARNING`的錯誤訊息並不會造成中斷(halted)，但是會導致數據在傳輸超過`1000`之外的變數為`null`，所以可能會間接的導致系統異常。

近期客人在新增訂單時，發現該筆訂單有些內容正常，但是有部分內容異常。查了程式碼半天發現都沒有問題，最後打開瀏覽器看`response`時才發現有錯誤訊息

```php
Warning:  Unknown: Input variables exceeded 1000. To increase the limit change max_input_vars in php.ini. in Unknown on line 0<br />
```

再細查才發現，這一個`POST`包含了超過1000個變數，所以第1001個變數外的內容都被設定為`null`，又很剛好的1001個變數之後的每個變數都沒有關連/null時並不會引起系統錯誤...

經上網查詢後才發現這是由`php.ini`的`Runtime Configuration`的`max_input_vars`控制。可以透過指令列查詢數值

```php
php -ini|grep max_input_vars
```



#### 解決方法

筆者目前使用到的解決方法有二：一個是直接修改`php.ini`的檔案即可，另一個是針對該網頁應用程式( web application)的`.htaccess`做修改

**備註:由於`max_input_vars`本身的`Changeable`為`PHP_INI_PERDIR`，所以無法透過`ini_set()`的方式修改，[可參考](http://www.php.net/manual/en/configuration.changes.modes.php)**

##### 1. 方法一：直接修改`php.ini`

理論上來說，找到php.ini 找到該數值之後更新，並重啟`apache`就可以了。

```bash
php --ini // 確定php.ini location
cd /PATH/TO/PHP.INI
sudo vim php.ini // 找到max_input_vars，並把[;]取消，將數值改為2000
sudo apachectl restart
```

直接修改`php.ini`是修改設定檔，會間接的導致同台主機上的其他服務都會受到影響。如果只想針對單一web application 的內容做修正的話，可以採用第二種方式:針對`.htaccess`做修改

##### 2. 方法二：修改`.htaccess`

對`laravel`的框架來說，在每個`public`資料夾內都會有`.htaccess`的檔案，用來管控整個應用程式的讀取權限。

在`.htaccess`內加上

```bash
php_value max_input_vars 2000 //輸入你要的數字
```

即可直接完成設定，且不需要重啟`apache`

##### 結語

最後我們來看一下php上的說明:[max_input_vars](http://php.net/manual/en/info.configuration.php#ini.max-input-vars)

>How many [input variables](http://php.net/manual/en/language.variables.external.php) may be accepted (limit is applied to `$_GET` `$_POST` and `$_COOKIE` superglobal separately). Use of this directive mitigates the possibility of denial of service attacks which use hash collisions. If there are more input variables than specified by this directive, an **E_WARNING** is issued, and further input variables are truncated from the request.

手冊上寫明了` Use of this directive mitigates the possibility of denial of service attacks which use hash collisions`，可以減少`DDOS`的發生可能，所以站在資安的角度來說，我們不應該去改變這個數值，應該要嘗試去改變`POST`的變數數量，例如可能把重複性高的變數放成`json`處理後再傳送。