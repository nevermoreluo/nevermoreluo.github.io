---
title: 'Lua 源码中 l_likely, l_unlikey 是什么意思'
date: 2022-01-07 10:04:14
tags:  [lua, c]
---


最近在排查c++程序内部调用lua_pcall时产生`C stack overflow`异常，
研究问题时发现lua源码中存在一些likely调用,其实在其他代码中也见到过类似的调用，
那么我们今天就来探究一下它到底是什么逻辑<!-- more -->    


lua代码：

```c++

// 校验c层的nCcalls调用计数
// getCcalls函数返回lua层调用c层回调call的计数值
// LUAI_MAXCCALLS 是调用限制的宏定义 5.4中定义为200
// 即如果调用超过200时 调用luaE_checkcstack 引发异常
if (l_unlikely(getCcalls(L) >= LUAI_MAXCCALLS))
    luaE_checkcstack(L);

//////////////////////////////////////
#define l_likely(x)	luai_likely(x)
#define l_unlikely(x)	luai_unlikely(x)

/*
** macros to improve jump prediction, used mostly for error handling
** and debug facilities. (Some macros in the Lua API use these macros.
** Define LUA_NOBUILTIN if you do not want '__builtin_expect' in your
** code.)
*/
#if !defined(luai_likely)
    #if defined(__GNUC__) && !defined(LUA_NOBUILTIN)
        #define luai_likely(x)		(__builtin_expect(((x) != 0), 1))
        #define luai_unlikely(x)	(__builtin_expect(((x) != 0), 0))
    #else
        #define luai_likely(x)		(x)
        #define luai_unlikely(x)	(x)
    #endif
#endif

```

好，直到这里我们发现这是一个编译时的宏定义，甚至在某些特定情况下l_likely和l_unlikely调用了和没调用没区别  
即不论`if (l_likely(r))`还是`if (l_unlikely(r))` 在逻辑语义上是完全等价于`if (r) ` 


## 那么，什么是__builtin_expect ? 

```
Built-in Function: long __builtin_expect (long EXP, long C)
You may use `__builtin_expect' to provide the compiler with branch prediction information.
The return value is the value of EXP, which should be an integral  expression.  The value of C must be a compile-time constant.  The semantics of the built-in are that it is expected that EXP == C.

For example:
    if (__builtin_expect (x, 0))
        foo ();

would indicate that we do not expect to call `foo', since we expect `x' to be zero.  Since you are limited to integral expressions for EXP, you should use constructions such as
    if (__builtin_expect (ptr != NULL, 1))
        error ();
when testing pointer or floating-point values.
```
乍一看满头问号，预测信息？ 不对啊，只有我写代码才是玄学才对啊（不是  



其实是这样的，这个函数是gcc提供的为编译优化处理的函数，并不会对逻辑语义造成影响  
例如:
```
if (l_ulikely(<绝大多数情况都不会为真>)) {
    // do if
} else {
    // do else
}
```
上述语句中，在gcc(>=2.96)编译的情况下，编译器将会把else部分的逻辑进行编译优化，以期待减少指令跳转带来的性能损耗  
那么具体优化了啥呢？  
现在处理器都是流水线的，有些里面有多个逻辑运算单元，系统可以提前取多条指令进行并行处理，但遇到跳转时，则需要重新取指令，这相对于不用重新去指令就降低了速度。  
目的是增加条件分支预测的准确性，cpu会提前装载后面的指令，遇到条件转移指令时会提前预测并装载某个分支的指令。unlikely 表示你可以确认该条件是极少发生的，相反 likely 表示该条件多数情况下会发生。编译器会产生相应的代码来优化 cpu 执行效率。  

## 总结  
不论`if (l_likely(r))`还是`if (l_unlikely(r))` 在逻辑语义上是完全等价于`if (r) `  
我们可能会在大多数地方找到不一样名字关于likely/unlikely的宏定义，  
函数本质上是为了编译时优化逻辑跳转减少重新取指令的开销，并不会影响实际的逻辑语义  

因此，在你十分确信绝大多数情况下为真时使用likely,或者在你十分确信绝大多数情况下为假时使用unlikely.  
例如问题最开始的`if (l_unlikely(getCcalls(L) >= LUAI_MAXCCALLS))`, 因为正常情况下不可能引发`C Stack Overflow`的异常  

