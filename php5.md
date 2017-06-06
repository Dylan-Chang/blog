object －　oriented 

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
