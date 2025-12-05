## C++ 模版技巧与反射



## 查询函数的反射机制

```c++

#include <tuple>

template<typename NotAFunction>
struct function_traits_helper;

// 处理普通函数
template<typename Ret, typename... Args>
struct function_traits_helper<Ret(*)(Args...)> {
    using result = Ret;
    using parameters = std::tuple<Args...>;
};

// 处理const 成员函数
template<typename Ret, typename C, typename... Args>
struct function_traits_helper<Ret(C::*)(Args...) const> {
    using result = Ret;
    using parameters = std::tuple<Args...>;
};

// 处理成员函数
template<typename Ret, typename C, typename... Args>
struct function_traits_helper<Ret(C::*)(Args...)> {
    using result = Ret;
    using parameters = std::tuple<Args...>;
};

// 处理一般函数
template<typename T, typename = void>
struct function_traits: function_traits_helper<T> {};

// 特化模板，处理lambda函数和Functor
template<typename Lambda>
struct function_traits<Lambda, std::void_t<decltype(&Lambda::operator())>>: function_traits_helper<decltype(&Lambda::operator())> {};


template <typename F>
using function_arguments_t = typename function_traits<F>::parameters;

int ret_int(double a);
struct Foo {
    void bar(int a, int b);
};

auto square = [](float a, float b) {return a * b;};

struct Functor {
    int operator()(int a, float b) {return a + static_cast<int>(b);}
};

int main() {
using F1 = decltype(&ret_int);
static_assert(std::is_same_v<std::tuple<double>, function_arguments_t<F1>>);
using F2 = decltype(&Foo::bar);
static_assert(std::is_same_v<std::tuple<int, int>, function_arguments_t<F2>>);
using F3 = decltype(square);
static_assert(std::is_same_v<std::tuple<float, float>, function_arguments_t<F3>>);
using F4 = Functor;
static_assert(std::is_same_v<std::tuple<int, float>, function_arguments_t<F4>>);
}
```

同样下面有更加简洁的写法

```c++
template <typename T> // lambda 特化
struct function_traits : function_traits<decltype(&T::operator())> {};

template <typename R, typename... Args>
struct function_traits<R(*)(Args...)> {
    using args_tuple = std::tuple<Args...>;
    using function_type = std::function<void(Args...)>;
};

template <typename C, typename R, typename... Args>
struct function_traits<R(C::*)(Args...)> {
    using args_tuple = std::tuple<Args...>;
    using function_type = std::function<void(Args...)>;
};

template <typename C, typename R, typename... Args>
struct function_traits<R(C::*)(Args...) const> {
    using args_tuple = std::tuple<Args...>;
    using function_type = std::function<void(Args...)>;
};

using function_type = typename function_traits::
```

