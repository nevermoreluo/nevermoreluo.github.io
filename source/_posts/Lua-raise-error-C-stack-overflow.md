---
title: 'Lua raise error: C stack overflow'
date: 2023-09-15 16:47:43
categories:
  - Code
tags: [lua, cpp]
---


## C stack overflow
在cpp中调用lua虚拟机，出现异常。
 
其实本质上是lua层代码调用了超过指定数量 `LUAI_MAXCCALLS`（默认200次）的cfunc导致的bug  

 
### 1. require死循环
通常来说，大概率发生在require死循环中，排查require相互引用改为惰性引用或者修改程序结构可以解决  

当然如果在c++中直接调用luaState运行，并且嵌入了很多c结构体的，可以在源码lstate.c中直接在C stack overflow处加入断点，即对具体业务分析对象在何处死循环  


### 2. lua调用cfunc异常
```cpp
int luaCall(lua_State* L) {
    try {
        // 在这个业务逻辑里面存在更深层次的c层userdata调用引发了未处理的异常
        lua_pcall(L, 0, r, 0); 
    } catch (std::expection& ex) {
        // handler error
    }
}
```
  
底层原理跟还是调用次数超过了`LUAI_MAXCCALLS`  
但可能是由于我们业务中存在很多c++层userdata对象的调用引发了异常导致，lua_pcall无法正常处理nCcalls计数导致  

例如  
当C++层TimerUpdat时会调用上述`luaCall`，L->nCcalls计数加1    
luaCall调用的函数，在lua层对c++层数据库对象进行操作时，引发了未捕获异常 此时L->nCcalls计数再次加1
由于异常退出了，导致nCcalls计数未能正常处理，从而导致计数一直增长  
  
在处理数据库对象引发异常时捕获并处理抛出luaL_error



额外提供一个实用的打印堆栈的函数  
```cpp
void luaStackTrace(lua_State *L)
{
    int i;
    int top = lua_gettop(L);
    printf("---- Begin Stack ----\n");
    printf("Stack size: %i\n\n", top);
    for (i = top; i >= 1; i--)
    {
        int t = lua_type(L, i);
        switch (t)
        {
            case LUA_TSTRING:
                printf("%i -- (%i) ----string: `%s'", i, i - (top + 1), lua_tostring(L, i));
                break;

            case LUA_TBOOLEAN:
                printf("%i -- (%i) ----bool: %s", i, i - (top + 1), lua_toboolean(L, i) ? "true" : "false");
                break;

            case LUA_TNUMBER:
                printf("%i -- (%i) ----number: %g", i, i - (top + 1), lua_tonumber(L, i));
                break;

            case LUA_TTABLE:
            {
                printf("%i -- (%i) ---- %s\n", i, i - (top + 1), lua_typename(L, t));
                lua_pushnil(L);
                while (lua_next(L, i) != 0)
                {
                    switch (lua_type(L, -2))
                    {
                        case LUA_TSTRING:
                            printf("| key ----string: `%s'", lua_tostring(L, -2));
                            break;

                        case LUA_TBOOLEAN:
                            printf("| key ----bool: %s", lua_toboolean(L, -2) ? "true" : "false");
                            break;

                        case LUA_TNUMBER:
                            printf("| key ----number: %g", lua_tonumber(L, -2));
                            break;
                        default:
                            printf("| key ---- %s", lua_typename(L, -2));
                            break;
                    }
                    printf("\n");
                    switch (lua_type(L, -1))
                    {
                        case LUA_TSTRING:
                            printf("| value ----string: `%s'", lua_tostring(L, -1));
                            break;

                        case LUA_TBOOLEAN:
                            printf("| value ----bool: %s", lua_toboolean(L, -1) ? "true" : "false");
                            break;

                        case LUA_TNUMBER:
                            printf("| value ----number: %g", lua_tonumber(L, -1));
                            break;
                        default:
                            printf("| value ---- %s", lua_typename(L, -1));
                            break;
                    }

                    printf("\n");
                    lua_pop(L, 1);
                }
                printf("%i -- (%i) table end", i, i - (top + 1));
                break;
            }
            default:
                printf("%i -- (%i) ---- %s", i, i - (top + 1), lua_typename(L, t));
                break;
        }
        printf("\n");
    }
    printf("---- End Stack ----\n");
    printf("\n");
}
```