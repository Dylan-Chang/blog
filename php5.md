#Review PHP (PHP 5 Power Programming note)

## motto
  "The best way to be ready for the future is to invent it."

  John Sculley
  
  "A language that doesn't have everything is actually easier to program in than some that do."

  Dennis M. Ritchie
  
  "High thoughts must have a high language."

   Aristophanes

Only time will tell if the PHP 5 release will be as successful as its two predecessors (PHP 3 and PHP 4). The new features and changes aim to rid PHP of any weaknesses it may have had and make sure that it stays in the lead as the world's best web-scripting language.

只有时间会告诉我们如果PHP 5版本将作为其两位前任的成功(PHP 3和php 4)。新功能和更改旨在消除PHP的任何弱点可能有并确保它始终处于领先作为世界上最好的web脚本语言。

## Basic Language
PHP borrows a bit of its syntax from other languages such as C, shell, Perl, and even Java. It is really a hybrid language, taking the best features from other languages and creating an easy-to-use and powerful scripting language.

PHP借用有点语法从其他语言如C、shell、Perl和甚至Java。它确实是一个混合语言，以其他语文的最佳功能和创建一个易于使用的和强大的脚本语言。

## object-oriented programming (OOP)
### syntactic sugar
    When Zeev Suraski added the object-oriented syntax back in the days of PHP 3, it was added as "syntactic sugar for accessing collections." 
### public/private/protected access modifiers for methods and properties.

### Static Properties (静态属性)
   As you know by now, classes can declare properties. Each instance of the class (i.e., object) has its own copy of these properties. However, a class can also contain static properties. Unlike regular properties, these belong to the class itself and not to any instance of it. Therefore, they are often called class properties as opposed to object or instance properties. You can also think of static properties as global variables that sit inside a class but are accessible from anywhere via the class.
   你也可以认为静态属性是全局变量，它坐在里面一个类，但是可以从任何地方通过类。

### Static Methods 
  Similar to static properties, PHP supports declaring methods as static. What this means is that your static methods are part of the class and are not bound to any specific object instance and its properties. Therefore, $this isn't accessible in these methods, but the class itself is by using self to access it. Because static methods aren't bound to any specific object, you can call them without creating an object instance by using the class_name::method() syntax. You may also call them from an object instance using $this->method(), but $this won't be defined in the called method. For clarity, you should use self::method() instead of $this->method().

### Class Constants
Global constants have existed in PHP for a long time. These could be defined using the define() function. With improved encapsulation support in PHP 5, you can now define constants inside classes. Similar to static members, they belong to the class and not to instances of the class. Class constants are always case-sensitive. The declaration syntax is intuitive, and accessing constants is similar to accessing static members.

### Polymorphism (多态)
The subject of polymorphism is probably the most important in OOP. Using classes and inheritance makes it easy to describe a real-life situation as opposed to just a collection of functions and data. They also make it much easier to grow projects by reusing code mainly via inheritance. Also, to write robust and extensible code, you usually want to have as few as possible flow-control statements (such as if() statements). Polymorphism answers all these needs and more.

    Consider the following code:

    class Cat {
        function miau()
        {
            print "miau";
        }
    }

    class Dog {
        function wuff()
        {
            print "wuff";
        }
    }

    function printTheRightSound($obj)
    {
        if ($obj instanceof Cat) {
            $obj->miau();
        } else if ($obj instanceof Dog) {
            $obj->wuff();
        } else {
            print "Error: Passed wrong kind of object";
        }
        print "\n";
    }

    printTheRightSound(new Cat());
    printTheRightSound(new Dog());


    The output is
    miau
    wuff



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
