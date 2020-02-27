---
title: lua应用
date: 2019-09-06 09:28:52
tags:
categories: 3rd
---

# 基础
全文基于lua5.1
## CAPI

> lua.h 基础函数, lua_ 开头, Lua没有定义全局变量, lua_State 动态结构保持所有状态
> lauxlib.h 辅助函数 luaL_ 开头, 基于基础函数api, 侧重解决具体业务
> lualib.h 定义了标准库, 用户可以包含这个文件, 来统一openlibs, 如 luaL_openlibs

## 栈
Lua和C语言数据交换, 有两个问题, 动态和静态类型之间区别, 自动和手动内存管理的区别
Lua设计了一个抽象的栈, 用于与其他语言数据交换, 栈中元素可以保存任何Lua类型的值

<!-- more -->

{% fold 点击显代码 %}
```
// 入栈, 常量nil, 布尔, 双精度浮点, 整数, 任意长度字符串, 零结尾字符串
// 注意lua不会有指向外部字符串的指针, 会自己生产副本
void lua_pushnil(lua_State* L); 
void lua_pushboolean(lua_State* L, int bool);
void lua_pushnumber(lua_State* L, lua_Number n);
void lua_pushinteger(lua_State* L, lua_Integer n);
void lua_pushlstring(lua_State* L, const char* s, size_t len);
void lua_push(lua_State* L, const char* s);

// 查询元素, 索引:1 入栈第一个元素, 2 入栈第二个元素; -1 栈顶元素, -2 
int lua_is(lua_State* L, int index);
//  从栈中获取一个值
int lua_toboolean(lua_State* L, int index);
lua_Number lua_tonumber(lua_State* L, int index);
lua_Integer lua_tointeger(lua_State* L, int index);
const char* lua_tolstring(lua_State* L, int index, size_t *len);
size_t lua_objlen(lua_State* L, int index);

// 类型(lua.h)
LUA_TNIL, LUA_TBOOLEAN, LUA_TNUMBER, LUA_TSTRING, LUA_TTHREAD, LUA_TUSERDATA, LUA_TFUNCTION

// 其他操作
// 返回栈中元素个数
int lua_gettop(lua_State* L); 
// 将栈顶设置到指定位置, 修改栈元素个数, 不足用nil补充, 多余的丢弃
// 特例清除栈
void lua_settop(lua_State* L, int index);
lua_settop(L, 0);
// 从栈中弹出n个元素
#define lua_pop(L, n) lua_settop(L, -(n)-1)

void lua_pushvalue(lua_State* L, int index);
void lua_remove(lua_State* L, int index);
void lua_insert(lua_State* L, int index);
void lua_replace(lua_State* L, int index);


// example
static void stackDump(lua_State* L)
{
  int top = lua_gettop(L);
  for(int i=1; i<top; ++i)
  {
    int t = lua_type(L, i);
	switch(t)
	{
	  case LUA_TSTRING:
	  {
	    printf("%s", lua_tostring(L, i));
	    break;
	  }
	  case LUA_TBOOLEAN:
	  {
	    printf(lua_toboolean(L, i)? "true" : "false");
		break;
	  }
	  case :
	  {
	  }
	  default:
	  { /*其他值*/
	    printf("%s", lua_typename(L, i));
		break;
	  }
	  printf(" ");
	}
  }
  printf("\n");
}
```
{% endfold %}

# 应用
## 调用lua文件中的变量/函数
{% fold 点击显代码 %}
```
// lua文件
width = 30
height = 80

// c应用
void load(lua_State* L, int* w, int* h)
{
  if(luaL_loadfile(L, fname) || lua_pcall(L, 0, 0, 0))
    error(L, "cannot run config file %s", lua_tostring(L, 0));
  lua_getglobal(L, "width");
  lua_getglobal(L, "height");
  
  *w = lua_tointeger(L, -2);
  *h = lua_tointeger(L, -1);
}

// table, 获取全局变量backgroud, 确认类型, get_field获取值
backgroud = {r=1, g=2, b=3}

  if(luaL_loadfile(L, fname) || lua_pcall(L, 0, 0, 0))
    error(L, "cannot run config file %s", lua_tostring(L, 0));
	
  lua_getglobal(L, backgroud);
  if(!lua_istable(L, -1))
    error(L, " is not a table");

  red = getfield(L, "r")
  ...
  
// c 调用lua函数
function f (x, y)
  return x*y;
end

void f(int x, int y)
{
  // 压入函数和参数
  lua_getglobal(L, f);
  lua_pushinteger(L, 1);
  lua_pushinteger(L, 20);
  
  // 调用lua函数
  if(lua_pcall(L, 2, 1, 0) != 0)
    error(...)
  
  // 检查结果
  if(lua_isinteger(L, -1) == 0)
	int z = lua_tointeger(L, -1);
  else
    return -1;
    
  // 清除结果
  lua_pop(L, -1);
  return z;
}

```
{% endfold %}

## Lua调用C函数
{% fold 点击显代码 %}
```
// 定义c函数
static int l_sin(lua_State* L)
{
  double d = luaL_checknumber(L, 1);
  lua_pushnumber(L, sin(d));
  return 1;
}

// lua调用
lua_pushfunction(L, l_sin);
lua_setglobal(L, "mysin");

// 获取目录路径
static int l_dir(lua_State* L)
{
  ...
  lua_newtable(L);
  int i=1;
  while((entry = readdir(dir)) != NULL)
  {
    lua_pushnumber(L, i++); // 压入KEY
	lua_pushstring(L, entry->dname); // 压入value
	lua_settable(L, -3);
  }
  ...
}

```
{% endfold %}

## Lua调用C模块
{% fold 点击显代码 %}
```
// 1. 定义C模块函数
static int l_dir(lua_State* L)
{...}

// 2. 声明数组, 包含字符串名称和函数指针
static const struct LuaL_Reg mylib[] = {
  {"dir", l_dir},
  {NULL, NULL}
}

// 3. 声明一个主函数
int luaopen_mylib()
{
  luaL_register(L, mylib);
  return 1;
}

// 4. lua调用so 或 重新编译添加到标准库列表中打开
require "mylib"

luaL_openlibs 打开标准库列表(linit.c)
```
{% endfold %}

# 编写C函数技巧
## 数组/字符串操作
{% fold 点击显代码 %}
```
// 数组操作, index 表示 table 在栈里的位置, key 表示元素在table中的位置(从1开始)
// 取值, stack[index][key]
void lua_rawgeti(lua_State* L, int index, int key);
// 赋值, 将栈顶的值赋值到 stack[index][key]
void lua_rawseti(lua_State* L, int index, int key);

-- 定义一个全局table
LanguagesTable = 
{
    "lua",
    "c",
    "c++",
    "java",
    "python",
}

//-- 定义一个打印函数
function func_printarray()
    print("\n");
    for index,value in pairs(LanguagesTable) do
        print("["..index.."] = ".. value);
    end
end

//-- c++代码
lua_State *L = lua_open();
luaL_openlibs(L);

luaL_dofile(L,"rawgetitest.lua");   // 加载执行lua文件
lua_getglobal(L,"LanguagesTable");  // 将全局表压入栈


lua_rawgeti(L, -1, 2);              // 取LanguagesTable[2]的值, -1 是栈中位置
if(lua_isnil(L, -1))
{
	printf("c++ --> [2] = nil\n");
}
else
{
	printf("c++ --> [2] = %s\n", lua_tostring(L, -1)); // 输出 c
}
lua_pop(L,1);                       // 弹出栈顶变量


lua_getglobal(L, "func_printarray");// 改变之前先调用打印函数，查看原数组
lua_pcall(L, 0, 0, 0);              // 输出 LanguagesTable 一共5个元素

lua_pushstring(L, "php");           // 将要赋值的结果压入栈
lua_rawseti(L, -2, 4);              // 赋值操作, -2是table在栈中位置

lua_pushstring(L, "swift");         // 将要赋值的结果压入栈
lua_rawseti(L, -2, 8);              // 赋值操作, -2是table在栈中位置

lua_getglobal(L, "func_printarray");// 改变之后再调用打印函数，查看改变后的结果
lua_pcall(L, 0, 0, 0);              // LanguagesTable[4] = php, LanguagesTable[8] = swift, 一共6个元素

lua_close(L);                       //关闭lua环境  


// ------------------------------
// 把s[i, j]子串传递给lua
lua_pushlstring(L, s+i, j-i+1);

```
{% endfold %}

## 在C函数中保存状态
C API提供3中方式：注册表(全局table, 多个模块共享), 环境(模块私有数据), upvalue(在特定函数中可见)

### 注册表
```
// 获取全局变量
lua_getfield(L, LUA_REGISTRINDEX, "KEY");

// 创建一个唯一的key, 取值
int r = luaL_ref(L, LUA_REGISTRINDEX);
lua_rawgeti(L, LUA_REGISTRINDEX, r);

// 释放引用和该值
luaL_unref(L, LUA_REGISTRINDEX);

// 保存字符串
static char KEY = 'key'
lua_pushlightuserdata(L, (void*)&KEY);
lua_pushstring(L, myStr);
lua_settable(L, LUA_REGISTRINDEX);

// 检索一个字符串
lua_pushlightuserdata(L, (void*)&KEY);
lua_gettable(L, LUA_REGISTRINDEX);
myStr = lua_tostring(L, -1);
```

### 环境
在本模块中可见
```
{
  lua_newtable(L);
  lua_replace(L, LUA_ENVIRONINDEX);
  luaL_register(L, "name", tab); // 创建一个name的table, 用tab去填充
}
```

### upvalue
相当于一个静态变量, 在一个特性函数中可见
```

```

# 用户自定义类型

[参考](https://www.cnblogs.com/sifenkesi/p/3897245.html)
## userdata
```
// 申请指定大小内存, 压栈,  返回内存地址
void* lua_newuserdata(lua_State* L, size_t size);
CUser* p = (CUser*)lua_touserdata(L, 1);

// 检查 cond 是否为真。如果不为真，以标准信息形式抛出一个错误
void luaL_argcheck (lua_State *L,
                    int cond,
                    int arg,
                    const char *extramsg);
```

## 元素
怎么确第一个参数就是我们想要的数组userdata, 不是其他的userdata?
用名称来标记
```
// 如果registry已经有tnme键值，则函数返回0; 否则创建一个[tname, metatable]，并放入registry，并返回1。
int luaL_newmetatable (lua_State *L, const char *tname);

// 获取registry中的tname对应的metatable，并入栈
int luaL_getmetatable (lua_State *L, const char *tname);

// 将栈顶对象的metatable设置为registry表中键tname对应的值
void luaL_setmetatable (lua_State *L, const char *tname);

// 检查栈的指定位置是否为元表，并且是否具有和指定名称相匹配的表
void* luaL_checkudata(lua_State*L,int index,const char*tname);

// 注意区分
void lua_setmetatable (lua_State *L, int index);
int lua_getmetatable (lua_State *L, int index);
```

## 轻量级
用户管理内存, 保持指针
void lua_pushlightuserdata(lua_State* L, void* p);


# 多线程
多线程的目的是为了协同程序, 挂起某些程序的执行, 并在稍后恢复。
从C API角度看, 一个线程一个栈; 
只要创建一个Lua状态, 就会自动在这个状态中创建一个线程。

```
// L1 以一个空栈进行运行, L 的栈顶就是这个新线程
lua_State* L1 = lua_newstate(L);

printf("%d\n", lua_gettop(L1));  --> 0
printf("%d\n", lua_typename(L)); --> thread

// L1 会被垃圾回收
lua_pop(L1);   

// 执行函数或在lua_yield后恢复
lua_resume(lua_State* L, int narg);
// 挂起lua调用成, 也就是C程序
lua_yield(L, nres);

// 线程之间数据交换
lua_pushstring(L2, lua_tostring(L1, 1));

```

# 内存管理
垃圾回收，原子操作














































