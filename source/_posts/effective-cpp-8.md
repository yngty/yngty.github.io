---
title: Effective C++ 8：析构函数不要抛出异常
date: 2021-10-25 13:29:41
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 8: Prevent exceptions from leaving destructors.

`C++` 本身不阻止在析构函数抛出异常，但在析构函数中抛出的异常往往会难以捕获，引发程序非正常退出或未定义行为。例如：

```c++
class Widget {
public:
    ...
    ~Widget() { ... } //假设这里可能抛出异常
};

void doSomething(){
  std::vector<Widget> v;  // v 这里被自动析构
}
```

当v被调用析构函数，它包含的所有Widget对象也都会被调用析构函数。又因为v是一个容器，如果在释放第一个元素时触发了异常，它也只能继续释放别的元素，否则会导致其它元素的资源泄露。如果在释放第二个元素的时候又触发了异常，那么程序同样会导致崩溃。

不仅仅是std::vector，所有STL容器的类甚至包括数组也都会像这样因为析构函数抛出异常而崩溃程序，所以在 `C++` 中，不要让析构函数抛出异常！

但是如果析构函数所使用的代码可能无法避免抛出异常呢？

```c++
class DBConnection{                   //某用来建立数据库连接的类
  public:
    ...
    static DBConnection create();     //建立一个连接
    void close();                     //关闭一个连接，假设可以抛出异常
};

class DBConn{                         //创建一个资源管理类来提供更好的用户接口
  public:
    ....
    ~DBConn{ db.close(); }            //终止时自动调用关闭连接的方法
  private:
    DBConnection db;
};


...{                                 
  DBConn dbc(DBConnection::create()); //创建一个DBConn类的对象
  ...                                 //使用这个对象
}                                     //对象dbc被释放资源
          
```

析构函数所调用的 `close()` 方法可能会抛出异常，那么有什么方法来解决呢？

**吞掉异常**

```c++
DBConn::~DBConn(){
  try{ 
    db.close();
  }catch(...){
    //记录访问历史
  }
}

```

**主动关闭程序**

```c++
DBConn::~DBConn(){
  try{ 
    db.close();
  }catch(...){
    //记录访问历史
    std::abort();
  }
}
```

**把可能抛出异常的代码移出析构函数**

客户在需要关闭的时候主动调用 `close()` 函数

```c++
class DBConn{
  public:
    ...
    ~DBConn();
    void close();        //当要关闭连接时，手动调用此函数
  private:
    ...
    closed = false;      //显示连接是否被手动关闭
};

void DBConn::close(){    //当需要关闭连接，手动调用此函数
  db.close();
  closed = true;
}

DBConn::~DBcon(){
  if(!closed)            //析构函数还是要留有备用，但不用每次都承担风险了
    try{
      db.close();
    }catch(...){
      //记录访问历史
      //消化异常或者主动关闭
    }
}
```


- 析构函数绝对不要抛出异常。如果一个被析构函数调用的函数可能抛出异常，析
构函数应该捕捉任何异常，然后吞下它们(不传播)或结束程序。
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么 `class` 应该提
供一个普通函数(而非在析构函数中)执行该操作。