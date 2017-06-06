#Review PHP
PHP 5 Power Programming

  "The best way to be ready for the future is to invent it."

  John Sculley

Only time will tell if the PHP 5 release will be as successful as its two predecessors (PHP 3 and PHP 4). The new features and changes aim to rid PHP of any weaknesses it may have had and make sure that it stays in the lead as the world's best web-scripting language.

只有时间会告诉我们如果PHP 5版本将作为其两位前任的成功(PHP 3和php 4)。新功能和更改旨在消除PHP的任何弱点可能有并确保它始终处于领先作为世界上最好的web脚本语言。

## object－oriented 
### syntactic sugar
When Zeev Suraski added the object-oriented syntax back in the days of PHP 3, it was added as "syntactic sugar for accessing collections."


## Exception handling.

PHP 5 adds the ability for the well-known TRy/throw/catch structured exception-handling paradigm. 


## Namespace

### User Contributed Notes - SteveWa 
Thought this might help other newbies like me...

Name collisions means: 
you create a function named db_connect, and somebody elses code that you use in your file (i.e. an include) has the same function with the same name.

To get around that problem, you rename your function SteveWa_db_connect  which makes your code longer and harder to read.

Now you can use namespaces to keep your function name separate from anyone else's function name, and you won't have to make extra_long_named functions to get around the name collision problem.

So a namespace is like a pointer to a file path where you can find the source of the function you are working with

所以命名空间就像一个指针，该指针指向一个文件路径，在这里您可以找到您正在使用的功能的源

[link](http://php.net/manual/en/language.namespaces.rationale.php#102662)
