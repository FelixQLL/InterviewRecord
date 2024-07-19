# 单例模式

## 懒汉式
在第一次使用时才创建单例实例，即单例实例在第一次调用时初始化。

``` C++
namespace common {

template <class T>
class Singleton {
 protected:
  // 构造函数和析构函数设为protected，防止在类外部创建和销毁实例
  Singleton() {}
  virtual ~Singleton() {}

  // 禁用拷贝构造函数和赋值运算符，防止复制实例
  Singleton(const Singleton<T> &);
  Singleton<T> &operator=(const Singleton<T>);

 public:
  // 提供一个静态方法来访问唯一实例
  static T &Instance() {
    static T instance; // 在第一次调用时创建实例，之后返回同一个实例
    return instance;
  }
};

}
```
### 懒汉式单例模式的特点
- **延迟加载：** 实例在第一次使用时才创建，而不是在程序启动时就创建。
- **线程安全：** C++11 标准保证了局部静态变量的初始化是线程安全的，因此这个实现是线程安全的。
- **节省资源：** 只有在需要的时候才创建实例，避免了不必要的资源浪费。

### static的作用
1. 静态局部变量，保证单例对象**只被创建一次**：存储在静态存储区，程序开始时分配，程序结束时释放
2. 静态成员函数允许通过类名直接访问单例实例，而不需要实例化对象。**实现全局访问**（允许通过 DeviceDB::Instance()访问，不需要创建单例对象） -- **原因：** 静态成员函数**属于类本身**，而不是类的任何一个实例
3. **简洁且线程安全**：根据C++11标准，当局部静态变量在多线程环境中第一次被访问时，其初始化由编译器保证是线程安全的。这意味着，在同一时刻，只有一个线程能够初始化该静态变量，而其他线程会等待初始化完成。

### 为什么禁用拷贝构造函数里用的 &
防止对象在传递过程中被拷贝

## 饿汉式
在类加载时就创建单例实例，即单例实例在类加载时初始化。

``` C++
namespace common {

template <class T>
class Singleton {
 private:
  static T instance; // 在类加载时创建实例
 protected:
  // 构造函数和析构函数设为protected，防止在类外部创建和销毁实例
  Singleton() {}
  virtual ~Singleton() {}

  // 禁用拷贝构造函数和赋值运算符，防止复制实例
  Singleton(const Singleton<T> &);
  Singleton<T> &operator=(const Singleton<T>);

 public:
  // 提供一个静态方法来访问唯一实例
  static T &Instance() {
    return instance;
  }
};

}
```
