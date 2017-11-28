# Lua
Lua is a lite version embedded interpreter for programming lanugage.
the [Lua official web site](http://www.lua.org/about.html) describe it.

## How to embed into our code
put the source code into our code and build it as a library

### for C++
when we want to embed Lua, we need init the lua environment in our memory
> lua_State *luaState=luaL_newState();

then we can register new function or replace function for lua
> /*
> * the lua_CFunction prototype is
> * int (*) (lua_State *L);
> *
> * the sample function
> * int myPrint(lua_State *L);
> *
> */
>
> lua_pushcfunction(luaState, myPrint);
> lua_setglobal(luaState, "print")
>

set lua user data, the data can be get in our register functions
> /*
> * we have a variable with type int like:
> * int variable;
> * 
> */
> void **data = reinterpret_cast<void**>(lua_newuserdata(luaState, sizeof(void*));
> *data = reinterpret_cast<void*>(&variable);
> lua_setglobal(luaState, "__global_user_data__");

and we can use the code to run lua script file
> luaL_dofile(luaState, "xxxx.lua");

