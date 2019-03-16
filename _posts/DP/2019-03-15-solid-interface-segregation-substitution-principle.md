---
layout: single
title: SOLID 原則 - Interface Segregation Principle (介面隔離原則)
categories: [dp]
---
>Clients should not be forced to depend on methods that they do not use
>(A client should not be forced to implement an interface that it doesn’t use.)
>客戶(類別)不應該實作沒用到的方法。

ISP的實作前是需要先釐清每個類別的需要功能。舉個例子來說，現在有個工人-雇主的關係。

我們先設定雇主需要的事情，最簡單的事需要管理工人，所以在`manage`方法內注入`工人(worker)`，所以`manage`這個功能就會相依到`worker`這個類別會做的事情。

```php
<?php

class Captain {
    public function manage(Worker $worker)
    {
        
    }
}
```

我們在來假設`worker`會作兩件事情，`work及˙sleep`，而我們在管理時需要讓工人時做這兩個方法。

```php
<?php
class Worker {
    public function work()
    {
        
    }
    
    public function wleep()
    {
        
    }
}
class Captain {
    public function manage(Worker $worker)
    {
        $worker->work();
        $worker->sleep();
    }
}
```

目前到這個階段看來都算合理，如果接下來我們要雇用`Android`來工作呢?這時候我們可以先實作出一個介面出來，定義`worker`需要會哪些方法。

```php
<?php
interface WorkerInterface {
    public function work()
    public function sleep()
}
```

接著我們實作`人類工人HumanWorker`及`Android工人AndroidWroker`

```php
<?php
interface WorkerInterface {
    public function work();
    public function sleep();
}

class HumanWorker implements WorkerInterface{
    public function work()
    {
        
    }
    public function sleep()
    {
        
    }
}

class AndroidWroker implements WorkerInterface{
    public function work()
    {
        return 'Android work';
    }
    public function sleep()
    {
        return null;
    }
}
class Captain {
    public function manage(Worker $worker)
    {
        $worker->work();
        $worker->sleep();
    }
}
```

到這邊我們知道，`AndroidWorker`本身並不需要休息(機器人不需要休息)，所以就違反了`ISP`，因為實作了沒有用到的方法(`sleep`)，所以這個時候就可以把`sleep`這個方法抽離出來，另外做一個介面`SleepableInterface`，並只讓有休息功能的`HumanWorker`實做，`AndroidWroker`就只需要實作`WorkableInterface`。

```php
<?php
interface WorkableInterface {
    public function work();
}

interface SleepableInterface {
    public function sleep();
}

class HumanWorker implements WorkableInterface, SleepableInterface{
    public function work()
    {
        
    }
    public function sleep()
    {
        
    }
}

class AndroidWroker implements WorkableInterface{
    public function work()
    {
        return 'Android work';
    }
}
```

但是這樣到`manage`的功能就會報錯，因為`AndroidWroker`並沒有`sleep`的功能。而如果我們用判斷式處理，就會違反`OCP`原則，應該讓擴充容易，而不讓新增功能影響到原有的程式。

```php
class Captain {
    public function manage(WorkableInterface $worker)
    {
        $worker->work();
        if($work istanceof AndroidWroker) return; // violate Open-Closed Principle
        $worker->sleep();
    }
}
```

所以這時候我們新增一個`beManaged`的介面，讓`HumanWorker及AndroidWorker`實作他們被管理時會用到的方法。

```php
interface BeManageableInterface {
    public funtcion beManaged();
}

interface WorkableInterface {
    public function work();
}

interface SleepableInterface {
    public function sleep();
}

class HumanWorker implements WorkableInterface, SleepableInterface, BeManageableInterface{
    public function work()
    {
        
    }
    public function sleep()
    {
        
    }
    
    public function beManage()
    {
        $this->work();
        $this->sleep();
    }
}

class AndroidWroker implements WorkableInterface, BeManageableInterface{
    public function work()
    {
        return 'Android work';
    }
    
    public function beManage()
    {
        $this->work();
    }
}
```

最後，我們只要讓`Manage`這個方法注入`BeManageableInterface`介面，並使用`$worker->beManage()`方法就可以操作本來應該要有的功能。

```php
class Captain {
    public function manage(BeManageableInterface $worker)
    {
        $worker->beManage();
    }
}
```

而之後若有其他的功能擴充，如其他的`工人`，所以就可以不用來修改`manage`的內容，反而需要修改被實作`BeManageableInterface`介面的物件，如此一來符合`OCP`也符合`ISP`了