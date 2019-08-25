---
layout: single
title: Use MySQL REGEXP in Laravel
categories: [laravel]
---
### Quick note to use MySQL REGEXP in Laravel

You can use these syntaxes in Laravel when you are searching data in a database column with JSON data type (`MySQL > 5.7.8`) or text data type stored JSON data(`MySQL < 5.7`).

```php
$query = USER::all();
//laravel < 5.6
$query->where('COLUMN', 'REGEXP', '...');
$query->whereRaw("COLUMN REGEXP 'YOUR_REGULAR_EXPRESSIN'");
//laravel > 5.6
$query->whereJsonContains('COLUMN', 'VALUE');
```

MySQL supports another type of pattern matching operation based on regular expressions. If you are familiar with regular expressions, here are some tips you could step in easier.

Same as normal:

| Pattern | What the pattern matches                             |
| ------- | ---------------------------------------------------- |
| ^       | Beginning of string                                  |
| $       | End of String                                        |
| .       | any single character                                 |
| *       | 0 or more instances of preceding element             |
| +       | 1 pr more                                            |
| ?       | 0 or 1                                               |
| [...]   | Any character listed between the square brackets     |
| [^...]  | Any character NOT listed between the square brackets |
| {n}     | n instances of preceding element                     |
| {m,n}   | m through n instances of preceding element           |
| (...)   | Capturing Groups                                     |

Some different:

using [POSIX class](https://www.regular-expressions.info/posixbrackets.html#class) instead of [Shorthand Character Classes](https://www.regular-expressions.info/shorthand.html). For example:

| shorthand | POSIX     | What the pattern matches                           | ASCII         |
| --------- | --------- | -------------------------------------------------- | ------------- |
| \w        | [:word:]  | Word characters (letters, numbers and underscores) | [A-Za-z0-9_]  |
| \d        | [:digit:] | Digits                                             | [0-9]         |
| \s        | [:space:] | All whitespace characters, including line breaks   | [ \t\r\n\v\f] |

[more detail](https://www.regular-expressions.info/posixbrackets.html#class)

for example:

if we have a permissions column with json type below:

```json
{"backend.booking":true,"backend.promotion":true,"backend.booking":ture}
```

and we want to select all values in the JSON data type are true:

```php
$query->where('permissions', 'REGEXP', '[{]("backend\.[[:alpha:]\.\_]+":true[,]?)+[}]');

{"backend.booking":true,"backend.promotion":true,"backend.booking":ture} //pass
{"backend.booking":false,"backend.promotion":true,"backend.booking":ture} //fail
```

