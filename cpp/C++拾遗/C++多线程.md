# C++多线程

### 1. 创建线程

```C++
std::thread th1 {可执行对象，[参数1, ...]};
```

创建线程需要传递一个可执行对象，C++中可执行对象包括三种：

+ 函数
+ 重载()运算符的类的对象
+ lambda表达式

### join, detach, joinable

+ join：等待子线程结束
+ detach: 不等待子线程结束
+ joinable：当该函数返回true时表示可以调用join或detach函数，当调用了join/detach函数之后，这个函数返回false。

### 2. 多线程构造函数

构造函数源码：

```c++
    template<typename _Callable, typename... _Args,
	     typename = _Require<__not_same<_Callable>>>
      explicit
      thread(_Callable&& __f, _Args&&... __args)
      {
	static_assert( __is_invocable<typename decay<_Callable>::type,
				      typename decay<_Args>::type...>::value,
	  "std::thread arguments must be invocable after conversion to rvalues"
	  );

#ifdef GTHR_ACTIVE_PROXY
	// Create a reference to pthread_create, not just the gthr weak symbol.
	auto __depend = reinterpret_cast<void(*)()>(&pthread_create);
#else
	auto __depend = nullptr;
#endif
        _M_start_thread(_S_make_state(
	      __make_invoker(std::forward<_Callable>(__f),
			     std::forward<_Args>(__args)...)),
	    __depend);
      }
```

首先构造函数使用了函数模板，当我们调用构造函数创建线程对象的时候，更具模板实参推导确定参数类型。不存在为重载的运算符、转换函数及构造函数显式指定模板实参的方法，因为它们的调用中并未使用函数名。

注意形参列表中的&&不是右值引用，在函数模板中，形如T&&，且T是推导类型，则&&代表的是转发引用（forward reference）。

模板函数推导原则：

一：  实参                         形参

​          左值                        左值引用

​		  右值                        右值类型

​          常量左值                常量左值引用

二：  引用折叠：

​			int& & ---> int &

​			int&& & ---> int &

​			int & && ---> int &

​			int && && ---> int&&



举例1：

```C++
tempalte<typename XX>
void test(XX&& x) {
	
}
int a = 1;
test(a);
```

根据推导规则一，T被推导为int&， 因此形参列表此时变为int&& &，根据引用折叠变为int &，因此函数的形参类型为左值引用。

举例2：

```c++
tempalte<typename XX>
void test(XX&& x) {
	
}
test(1);
```

根据推导规则一，T被推导为int，因此形参列表变为int&&，因此不需要进行引用折叠，所以函数test形参列表为右值引用。

总结以下，传递左值，形参类型变为左值引用，模板参数变为左值引用；传递右值，形参类型变为右值引用，模板参数变为对应的类型。

make_invoker --> __decayed_tuple 构建调用器的时候对参数进行了一次拷贝

_S_make_state：创建状态的时候对参数进行了一次拷贝。