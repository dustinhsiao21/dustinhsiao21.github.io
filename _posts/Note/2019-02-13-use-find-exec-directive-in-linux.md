---
layout: single
title: 淺談Linux 的find 指令之exec應用
categories: [note]
---

`find` 是很常用的Linux  指令，但是我們在查詢之餘並不會僅是看看而已，有時候會包含一些操作及簡單的排序功能。這時候就需要用到`exec`這個指令。

#### 起因

前陣子在重構專案內過大的檔案，在處理完一部分後，還需要查詢有哪些太大的檔案可以拿來重構，發現對Linux 的指令真是一整個不熟(大概僅限於ls/cd系列)，所以順便藉此記錄一下筆記。

先列出最後結果的指令

```bash
find .  -type f -exec du -h {} + | sort -rh | head -n 5
```

#### Find指令

用於查找檔案。[可參考](https://www.computerhope.com/unix/ufind.htm)

常用參數如下：

- `.`:代表當前目錄檔案。可一需求替換`~, /home`䓁。

- `-type`:指令檔案的類型，常見的有：

  - `d`:目錄
  - `f`:一般檔案
  - `p`:具名的`pipe`

- `-exec`：將搜尋出來的結果處理在使用其他指令處理。花括弧(`{}`)代表前面`find`查找出來的文件名。中止則用`+`或是`\;`。

  

#### du指令

用於顯示目錄或文件的大小，[可參考](https://www.computerhope.com/unix/udu.htm)

常用參數如下：

  - `-h`:代表`--human-readable`， 顯示人類可讀的形式，以`K, M, G`為單位。（Print sizes in human readable format, rounding values and using abbreviations. For example, "1K", "234M", "2G", etc.）
  - `-a/-all`:顯示目錄中個別文件大小。（Write counts for all files, not just [directories](https://www.computerhope.com/jargon/d/director.htm).）
  - `-c/--total `:顯示目錄中個別文件大小，同時也顯示總和。（Display a grand total.）
  - `--exclude=<目錄或文件>`：略過指令的目錄及文件。（Exclude files that match PATTERN.）
  - `--max-depth=<目錄層數>`:超過指令層數的目錄後忽略。（Print the total for a directory (or file, with **--all**) only if it is N or fewer levels below the command line argument; **--max-depth=0** is the same as **--summarize**.）

#### sort指令

用來排序。[可參考](https://www.computerhope.com/unix/usort.htm)

常用參數如下：

- `-r`:透過反向方式排序。(Reverse the result of comparisons.)
- `-h`:透過人類可讀的單位的數值做排序。(Compare human readable numbers)
- `-n`:透過數字排序。(Compare according to [string](https://www.computerhope.com/jargon/s/string.htm) numerical value.)
- `-R`:隨機排序(Sort by random hash of keys)

#### head指令

輸出頭幾個檔案。[可參考](https://www.computerhope.com/unix/uhead.htm)

常用參數如下：

- `-n/--lines=[-]num`:用來取前幾個檔案。預設是`10筆`

#### Tail指令

輸出倒數幾個檔案。[可參考](https://www.computerhope.com/unix/uhead.htm)

常用參數如下：

- `-n/--lines=[-]num`:用來取前幾個檔案。預設是`10筆`

所以就最上面的例子來說

```bash
find .  -type f -exec du -h {} + | sort -rh | head -n 5
```

這段指令的意思為：

搜尋(`find`)當前目錄底下(`.`)的一般檔案(`-type f`)，取出後再依大小排列(`-exec du -h {}`)，排列的結果再由檔案大小做反向排序(sort -rh)，並取前5個項目列出(`head -n 5`)。

