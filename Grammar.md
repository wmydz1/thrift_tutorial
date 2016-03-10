### Thrift 使用方法

第一篇blog，欢迎大家批评指正。

> 　一 前言

　　Thrift是facebook技术核心框架之一，不同开发语言开发的服务可以通过该框架实现通信。Thrift通过接口定义语言 (interface definition language，IDL) 来定义数据类型和服务，Thrift接口定义文件由Thrift代码编译器生成thrift目标语言的代码（目前支持C++,Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk和OCaml），并由生成的代码负责RPC协议层和传输层的实现。

　　简而言之，开发者只需准备一份thrift脚本，通过thrift code generator（像gcc那样输入一个命令）就能生成所要求的开发语言代码。不支持windows。

　　Thrift侧重点是构建跨语言的可伸缩的服务，特点就是支持的语言多，同时提供了完整的RPC service framework，可以很方便的直接构建服务，不需要做太多其他的工作。服务端可以根据需要编译成simple | thread-pool | threaded | nonblocking等方式；

　　本文档参考：Thrift types, Thrift IDL， Thrift:The Missing Guide.
　　
二 语法参考

____

> 　2.1 类型

　　Thrift类型系统包括预定义基本类型，用户自定义结构体，容器类型，异常和服务定义。

- 　2.1.1 基本类型

bool: 布尔值 (true or false), one byte

byte: 有符号字节

i16: 16位有符号整型

i32: 32位有符号整型

i64: 64位有符号整型

double: 64位浮点型

string: Encoding agnostic text or binary string

Note that： Thrift不支持无符号整型，因为Thrift目标语言没有无符号整型，无法转换。

___

> 2.1.2 容器（Containers）

　　Thrift容器与流行编程语言的容器类型相对应，采用Java泛型风格。它有3种可用容器类型：

list<t1>: 元素类型为t1的有序表，容许元素重复。（有序表ordered list不知道如何理解？排序的？c++的vector不排序）

set<t1>:元素类型为t1的无序表，不容许元素重复。

map<t1,t2>: 键类型为t1，值类型为t2的kv对，键不容许重复。

　　容器中元素类型可以是除了service外的任何合法Thrift类型（包括结构体和异常）。
　　
___

> 　2.1.3 结构体和异常（Structs and Exceptions）

　　Thrift结构体在概念上类似于（similar to）C语言结构体类型--将相关属性封装在一起的简便方式。Thrift结构体将会被转换成面向对象语言的类。

　　异常在语法和功能上类似于（equivalent to）结构体，差别是异常使用关键字exception而不是struct声明。但它在语义上不同于结构体：当定义一个RPC服务时，开发者可能需要声明一个远程方法抛出一个异常。

　　结构体和异常的声明将在下一节介绍。
___

> 2.1.4 服务（Services）

　　服务的定义方法在语义(semantically)上等同于面向对象语言中的接口。Thrift编译器会产生执行这些接口的client和server stub。具体参见下一节 
___


> 2.2 类型定义（Typedef)

　　Thrift支持C/C++类型定义。
　　　
```
　　 typedef i32 MyInteger // a
 　　typedef T ReT // b　　
```
  
说明：a.  末尾没有逗号。b.   struct也可以使用typedef。

____

> 2.3 枚举（Enums）

　　很多语言都有枚举，意义都一样。比如，当定义一个消息类型时，它只能是预定义的值列表中的一个，可以用枚举实现。

```
enum TweetType {
    TWEET,       // (1)
 　　RETWEET = 2, // (2)
    DM = 0xa,    // (3)
 　　REPLY
}                // (4)

struct Tweet {
    1: required i32 userId;
    2: required string userName;
    3: required string text;
    4: optional Location loc;
    5: optional TweetType tweetType = TweetType.TWEET; // (5)
    16: optional string language = "english"
}

```

　　说明：

　　(1).  编译器默认从0开始赋值

　　(2).  可以赋予某个常量某个整数

　　(3).  允许常量是十六进制整数

　　(4).  末尾没有分号

　　(5).  给常量赋缺省值时，使用常量的全称

　　注意，不同于protocal buffer，thrift不支持枚举类嵌套，枚举常量必须是32位的正整数

>　2.4 注释（Comment）

　　Thrift支持shell风格, C多行风格和Java/C++单行风格。
　　

This is a valid comment.

/*
 * This is a multi-line comment.
 * Just like in C.
 */

// C++/Java style single-line comments work just as well.


> 2.5 名字空间（Namespace）

　Thrift中的命名空间类似于C++中的namespace和java中的package，它们提供了一种组织（隔离）代码的简便方式。名字空间也可以用于解决类型定义中的名字冲突。

　　由于每种语言均有自己的命名空间定义方式（如python中有module）, thrift允许开发者针对特定语言定义namespace：　　

``` 
namespace cpp com.example.project  // (1)
namespace java com.example.project // (2)
namespace php com.example.project  
```
　　(1)． 转化成namespace com { namespace example { namespace project {

　　(2)．  转换成package com.example.project

>　2.6 Includes

　　便于管理、重用和提高模块性/组织性，我们常常分割Thrift定义在不同的文件中。包含文件搜索方式与c++一样。Thrift允许文件包含其它thrift文件，用户需要使用thrift文件名作为前缀访问被包含的对象，如： 

``` 

include "tweet.thrift"           // （1）

struct TweetSearchResult {
    1: tweet.Tweet tweet; // （2）
}

``` 

　　说明：

　　（1）．  thrift文件名要用双引号包含，末尾没有逗号或者分号

　　（2）．  注意tweet前缀


> 2.7 常量（Constant)

　　Thrift允许定义跨语言使用的常量，复杂的类型和结构体可使用JSON形式表示。 

```
const i32 INT_CONST = 1234;    // （1）

```
　　说明：

　　（1） 分号可有可无。支持16进制。



>　2.8 结构体定义（Defining Struct）

　　struct是Thrift IDL中的基本组成块，由域组成，每个域有唯一整数标识符，类型，名字和可选的缺省参数组成。如定义一个类似于Twitter服务：


```
struct Tweet {
    1: required i32 userId;                  // (1)
    2: required string userName;             // (2)
    3: required string text;
    4: optional Location loc;                // (3)
    16: optional string language = "english" // (4)
}

struct Location {                            // (5)
    1: required double latitude;
    2: required double longitude;
}

```


 (1) 每个域有一个唯一的正整数标识符；

 (2) 每个域可标识为required或optional；

 (3) 结构体可以包含其它结构体

 (4) 域可有默认值，与required或optional无关。

 (5) Thrift文件可以定义多个结构体，并在同一文件中引用，也可加入文件限定词在其它Thrift文件中引用。

　　如上所见，消息定义中的每个域都有一个唯一数字标签，这些数字标签在传输时用来确定域，一旦使用消息类型，标签不可改变。（随着项目的进展，可以要变更Thrift文件，最好不要改变原有的数字标签）

　　规范的struct定义中的每个域均会使用required或者optional关键字进行标识。如果required标识的域没有赋值，Thrift将给予提示；如果optional标识的域没有赋值，该域将不会被序列化传输；如果某个optional标识域有缺省值而用户没有重新赋值，则该域的值一直为缺省值；如果某个optional标识域有缺省值或者用户已经重新赋值，而不设置它的__isset为true，也不会被序列化传输。（不被序列化传输的后果是什么？为空为零？还是默认值，下次试试）

　　与services不同，结构体不支持继承。
　　
> 2.9 服务定义（Defining Services）

　　在流行的序列化/反序列化框架（如protocal buffer）中，Thrift是少有的提供多语言间RPC服务的框架。这是Thrift的一大特色。

　　Thrift编译器会根据选择的目标语言为server产生服务接口代码，为client产生stubs。 
　　
　　　　
___ 

```

service Twitter {
    // A method definition looks like C code. It has a return type, arguments,
    // and optionally a list of exceptions that it may throw. Note that argument
    // lists and exception list are specified using the exact same syntax as
    // field lists in structs.
    void ping(),                                    // (1)
    bool postTweet(1:Tweet tweet);                  // (2)
    TweetSearchResult searchTweets(1:string query); // (3)
 // The 'oneway' modifier indicates that the client only makes a request and
    // does not wait for any response at all. Oneway methods MUST be void.
    oneway void zip()                               // (4)
}

```


　(1) 有点乱，接口支持以逗号和分号结束；
　(2) 参数可以是基本类型和结构体；（参数是cosnt的，转换为c++语言是const&）
　(3) 返回值同参数一样；
　(4) 返回值是void，注意oneway；
Note that:参数列表的定义与结构体一样。服务支持继承。

