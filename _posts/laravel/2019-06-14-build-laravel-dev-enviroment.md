## Laravel 開發環境安裝

### MySQL

選擇安裝Mysql 5.7版，使用`HomeBrew`，可以使用`brew info mysql`檢查目前穩定的版本，截至今天(2019/4/27)為止，stable版本為`8.0.15` ，筆者這邊示範安裝5.7版，輸入`brew install mysql@5.7`時可以看到目前版本為`5.7.25(bottled)[keg-only]`。

```bash
brew install mysql@5.7
```

安裝完後可以直接開啟，若您想要之後重開在背景都會直接運作，可以用：

```bash
brew services start mysql@5.7
```

或是可以直接透過以下方始啟動

```bash
/usr/local/opt/mysql@5.7/bin/mysql.server start
```

由於`mysql@5.7`ˋ是`[keg-only]`，所以當你直接執行`mysql`時會出現`command not found`的情況。因為他不是現行版本，所以預設情況下只會放在`Cellar`內。必須要`symlink`到`/usr/local`。

```bash
brew link mysql@5.7 --force
```

這時候應該就可以執行`mysql` 了

```bash
sudo mysql -v
//mysql  Ver 14.14 Distrib 5.7.25, for osx
```

接著先在外部更改密碼：

```bash
mysqladmin -u root password 'yourpassword'
```

再來可以登入`mysql`

```bash
mysql -u root -p
```

並輸入剛剛的密碼，即可登入。

如果需要，你可以增加其他使用者，並為使用者增加權限等等。

```bash
 CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'newpassword';
 GRANT ALL PRIVILEGES ON newdatabase.* TO 'newuser'@'localhost';
 ...
 quit //登出
```

###  Sequel Pro

用GUI管理MySQL的工具，安裝方式也是使用HomeBrew

```bash
brew cask install sequel-pro
```

**備註** `brew cask`和`brew`的主要差異是`brew cask`會將檔案下載，並自動拖曳到`Application` 裡面，可以直接使用應用程式。

### Composer

可以直接參考`Composer`的[官方說明](<https://getcomposer.org/download/>)，或是參照下列方式：

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

之後應該就可以直接輸入`php composer.phar`執行，如果想要簡單一點，把Composer 移到`Globally`：

```bash
mv composer.phar /usr/local/bin/composer
```

就可以直接輸入並執行`composer`。 

若想要將composer 提供的套件做全域安裝，如等等會提到的`laravel new`, `valet`, 或是`phpcs`等，也可以把composer 放到`$PATH`內

```bash
export PATH=$PATH:~/.composer/vendor/bin

//echo $PATH 檢查
```

## 安裝Laravel

安裝laravel 可以透過兩種方式：透過 `composer`或是`laravel installer`。

1. 如果已經有`Composer`，就可以用`Composer`安裝`Laravel`

   ```bash
   composer create-project --prefer-dist laravel/laravel yourprojectname
   ```

   如果是初次安裝，可以先去泡杯咖啡。

2. 透過`laravel installer`安裝，先安裝`laravel/installer`

   ```bash
   composer global require laravel/installer
   ```

   接者只要透過指令`laravel new`即可安裝`laravel`。

   ```
   laravel new YOUR_PROJECT_NAME
   ```

安裝完後，可直接執行：

```bash
php artisan serve
//http://127.0.0.1:8000
```

就可以直接看到`laravel`的初始畫面了

### Valet

最後介紹一下`Valet`，如果不想要每次要開laravel專案內容時都執行`php artisan serve`，或是安裝一個`Web Server`像是`apache`, `nginx`，可以考慮安裝`Valet`這個套件。(only for Mac)

這個套件有幾個優點：

```php
composer global require laravel/valet
```

並且確保`~/.composer/vendor/bin`在你的`PATH`內：

這時候就可以直接安裝`valet`了，在安裝valet前，可以先確定是否有用`HomeBrew`安裝過`php`及`ngnix`，如果沒有的話他會一併幫你安裝。

```bash
valet install
```

安裝好後就可以直接執行了。可以先檢查是否安裝成功

```bash
ping foobar.test
```

接著`valet`提供了讓某個資料夾都可以使用`valet`的功能，只要到你的目錄下使用(假設我們新增一個Sites目錄)`valet:park`

```bash
mkdir Sites
cd Sites
valet park
```

如此一來，之後在這個`Sites` 目錄底下的`laravel ` 專案，就可以直接瀏覽器上觀看。例如安裝一個名稱為`blog` 的專案

```bash
composer create-project --prefer-dist laravel/laravel blog
```

接著只要到瀏覽器輸入`http:blog.test` 就可以看到網頁了。

Valet也提供幾個十分實用的功能：如`secure`

直接添加SSL

```bash
valet secure blog
```

就可以直接開啟`https://blog.test`

