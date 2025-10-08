## 前言

最近在写C++异步的一些代码，感觉到之前的设计方法有点很无力，现在也积累了，一些遇到的处理办法。

## 1.异步内存管理

对比起`js`, `c++`的异步就会复杂很多，js异步可以使用`async`的语法糖来实现，有内存回收机制，不用管理对象的生命周期。

对于大量使用回调来实现的异步来说，对象的生命周期就变得很难判断，很容易遭成调用悬空的指针，手动管理无疑会很困难。这里使用了智能指针，`shared_ptr`或`unipue_ptr`来自动管理生命周期，让对象可以存活到最后引用的时刻。

```c++
class Obj : public std::enable_shared_from_this<Obj> { 
  
	Obj(); 
	//这里用工厂模式
	static std::shard_ptr<Obj> create() { 
		return std::make_shared<Obj>(); 
	}
}
```

## 2.回调传递上下文

在异步中会用到回调，回调一般要向异步对象传递上下文或调用对象，在c代码中就是通过传递`void*`指针，在回调中再类型转换回来，但是这在c++中肯定是不安全的设计。

在lambda之前是实现一个callback类来传递上下文和回调，现在可以用lambda这个语法糖。

```c++
Obj::handle() { 
	auto self = shared_from_this(); //获取指针
  
	onEvent( [self](result* res) { 
		self->action(); 
		self->handleEvent(); 
	}); //这样就可以在lambda中使用self了，还传递了指针给异步对象，保证了指针在回调之前还是有效的。
}
```



## 3.列表初始化（`initializer_list`）

这是我在创建`json`时发现的，在原本自己实现的`json`的`api`中，需要复杂的函数来创建一个json，但是json本身的语法就很简洁而且`nlohmann json`也提供了简洁的api设计，这里就使用了`initializer_list`

`json`: 

```json
{ 
	"name": "away", 
	"port": 8080, 
	"arr": [1, 2, 3]
}
```

`nlohmann`: 

```c++
json = { 
	{"name", "away"}, 
	{"port", 8080}, 
	{"arr", {1, 2, 3}}, 
}
```

那么肯定这是一个很好的api设计，肯定要学习。

```c++
lept_value::lept_value(std::initializer_list<lept_value> initList)
{
	bool is_an_object = std::all_of(initList.begin(), initList.end(),
		[](const lept_value& ele)
		{
			return ele.type == lept_type::array && ele.v.arr.size() == 2
				&& ele.v.arr[0].type == lept_type::string;
		});
  
	if (is_an_object)
	{
		object_t obj;
		for (auto &it : initList)
		{
			obj.emplace(it.v.arr[0].v.s, it.v.arr[1]);
		}

		this->set_object(std::move(obj));
	} else
	{
		this->set_array(array_t(initList));
	}
}
//当然nlohmann中有更加好的设计和细节，这里是我的实现
```

## 4.鸭子类型（duck type）

想象一个场景，有两个类，一个上下文或者会话类，他拥有或者依赖另一个类，但是创建依赖对象时，要有一些上下文信息，我们是传入个个细节信息还是传入上下文？

首先是传入上下文，那么依赖对象在设计之初就会依赖一个上下文类，这样就会相互依赖，这一定是不好的设计，当然可以用依赖抽象类来实现，但是先等一下。

再是传入必要信息，这样就会暴露很多基础变量，这样也不好。

这里除了抽象类，还有一种方法就是在头文件中不定义具体的类型的指针，智能指针是可以不用具体类型声明的，让后在源文件中指定具体类型，只需要实现这些几个接口就可以编译通过，不需要继承负担。

```c++
//Obj.hpp
class Obj { 
	template<typename T>
  	Obj(std::shared_ptr<T> ctx); 
}; 

//Obj.cpp
template<typename T>
Obj::Obj(std::shared_ptr<T> ctx) { 
	ctx->get(); 
  	//....
}

template Obj::Obj(std::shared_ptr<A> ctx); //指定具体的类
```

