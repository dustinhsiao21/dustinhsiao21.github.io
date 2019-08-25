---
layout: single
title: PHP-regular-expression快速入門筆記
categories: [note]
---

### PHP-regular-expression快速入門筆記：

1. 前後要有`//`。
2. 最後會有`Modifiers`，常用的有：
   - `g`: 搜索全部，預設指搜索第一個，`php`內使用時預設搜全部。
   - `i`:大小寫`不敏感`，簡單來說就是`Adam != adam`，大小寫一定要完全一樣才可以。
3. 常用符號：

```
? => 1 or 0
* => 0 or more
+ => 1 or more

. => any single character
\. => real period
\w => all character
\s => a white space character
\d => a digit
[abc] => matches a single character in the given set
[^abc] => matches a single character outside the given set
(foo|bar|baz) => matches any of the alterbatives specified

[] => match inside
() => capture group, could use $n for each capture.
{N, M} => matches N to M times.

^ => matchs any at the beginning
$ => end of it.

?= => positive lookahead
?! => negative lookahead
?<= => positive lookbehide
?<! => negative lookbehide

?: => non-capturing group
```

### PHP Regexp Expressions Support

```php
preg_match($pattern, $subject, [, array &$matches]) //取得第一個相同的，並返回0or1, 若有第三個參數會將結果返回給參數。
preg_match_all($pattern, $subject, [, array &$matches]) // 取得全部相同的，並返回0or1, 若有第三個參數會將結果返回給參數。
preg_replace($pattern, $replacement, $subject) // 找到符合的並替換掉。
preg_split($pattern, $subject) // 分割字串組合成array。
preg_grep($pattern, $subject) // 找出符合正規表達式的value並返回。
```

