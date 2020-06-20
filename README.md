# Lua C API Cheat Sheet

### 스택 변화 표기

lua_XXX() 함수가 처리되는 동안, Lua 스택이 어떻게 변화되는 지를 표현한다. 스택에 논리적 인자를 push 하고, lua_XXX() 함수를 호출한다. 그 함수는 논리적 리턴 값을 스택에 넣어두었으므로, caller 코드가 스택에서 값을 뽑아서 확인하는 형태이다. 따라서 각 함수가 스택을 어떻게 변화시키는지를 반드시 확인해서 코딩해야 한다.

`[-o, +p, x]`

 - 첫번째 항목 o 는 스택 최상단에서 몇개의 값이 빠지는가.
   - lua_XXX() 함수는 논리적 인자 o 개를 스택에서 뽑는다.
 - 두번째 항목 p 는 스택 최상단에 몇개의 값이 들어가는가.
   - lua_XXX() 함수는 리턴하기 전에 그 논리적 리턴값 p 개를 스택에 push 한다.
 - 세번째 항목 x 는 에러 발생 여부가 어떻게 되는가.
   - `-` 이면 lua_XXX() 함수는 에러를 발생시키지 않는다.
   - `m` 이면 메모리 부족 에러가 발생할 수 있다.
   - `e` 이면 임의의 모든 에러가 발생할 수 있다.
   - `v` 이면 명시적 목적을 가지고 일부러 에러를 발생할 수 있다.

## 스택 조작

|이름|설명|스택 변화|
|:------|:---|---|
|lua_pushnil(L)|nil 넣기|[-0, +1, -]|
|lua_pushboolean(L, bool)|bool 넣기|[-0, +1, -]|
|lua_pushnumber(L, n)|숫자 넣기|[-0, +1, -]|
|lua_pushinteger(L, n)|숫자 넣기|[-0, +1, -]|
|lua_pushunsigned(L, n)|숫자 넣기|[-0, +1, -]|
|lua_pushstring(L, s)|문자열 넣기|[-0, +1, m]|
|lua_pushlstring(L, s, len)|부분 문자열 넣기|[-0, +1, m]|
|lua_pushfstring(L, fmt, ...)|sprintf 스타일로 문자열 만들어 넣기. 형식지정자는 sdfpc% 만 가능|[-0, +1, e]|
|lua_pushlightuserdata(L, p)|C 언어의 포인터 p 를 스택에 넣기|[-0, +1, -]|
|lua_pushcclosure(L, fn, n)|fn 함수와 n 개의 upvalue 를 가지는 C 클로져를 만들어 스택 최상단에 추가. n개의 upvalue 초기값이 스택에 들어있어야 함|[-n, +1, m]|
|lua_tonumber(L, index)|index 스택 위치에서 숫자 꺼내기|[-0, +0, -]|
|lua_tonumberx(L, index, isnum)|index 스택 위치에서 숫자 꺼내기|[-0, +0, -]|
|lua_tointeger(L, index)|index 스택 위치에서 숫자 꺼내기|[-0, +0, -]|
|lua_remove(L, index)|index 스택 위치의 항목 삭제. 그 위 항목들은 당겨짐|[-1, +0, -]|
|lua_replace(L, index)|스택 최상단 값을 index 스택 위치에 덮어쓰고 최상단 항목은 pop|[-1, +0, -]|

## 전역 변수 조작

|이름|설명|스택 변화|
|:------|:---|---|
|lua_setglobal(L, name)|스택 최상단에 있는 값을 name 이름의 전역변수로 만들기|[-1, +0, e]|
|lua_getglobal(L, name)|name 이라는 전역변수의 값을 찾아서 스택 최상단에 넣기|[-0, +1, e]|


## Table 조작

|이름|설명|스택 변화|
|:------|:---|---|
|lua_newtable(L)|테이블을 생성하여 스택 최상단에 넣음| |
|lua_gettable(L, index)|스택 최상단에 임의 타입의 키가 있을때, index 위치의 테이블에서 값 조회. 메타 테이블 있으면 사용됨|[-1, +1, e]|
|lua_getfield(L, index, k)|k 문자열 키를 가지고, index 위치의 테이블에서 값 조회. 메타 테이블 있으면 사용됨|[-0, +1, e]|
|lua_settable(L, index)|스택 최상단에 값, 그 바로 아래에 임의 타입의 키가 있을 때, index 위치의 테이블에 키와 값을 저장|[-2, +0, e]|
|lua_setfield(L, index, k)|스택 최상단의 값을, index 위치의 테이블에 k 를 키로하여 저장|[-1, +0, e]|
|lua_rawget(L, index)|스택 최상단에 키가 있을때, index 위치의 테이블에서 값 조회. 메타테이블 무시함|[-1, +1, -]|
|lua_rawgeti(L, index, key)|index 위치의 테이블에서 숫자 key 를 키값 으로 하여 값 조회. 메타테이블 무시함|[-0, +1, -]|
|lua_rawgetp(L, index, p)|index 위치의 테이블에서 포인터 p 를 키값 으로 하여 값 조회. 메타테이블 무시함|[-0, +1, -]|
|lua_rawsetp(L, index, p)|스택 최상단에 값이 있을 때, index 위치의 테이블에서 포인터 p를 키로해서 값 저장. 메타테이블 무시함|[-1, +0, m]|

## 함수 호출

|이름|설명|스택 변화|
|:------|:---|---|
|lua_call(L, nargs, nresults)|보호 되지 않는 함수 호출. 오류가 발생하면 전파됨|[-(nargs+1), +nresults, e]|
|lua_pcall(L, nargs, nresults, msgh)|보호 되는 함수 호출. 에러나면, 스택에 에러 object를 push 하고 오류 코드를 직접 반환한다|[-(nargs+1), +(nresults\|1), -]|

## 기타

|이름|설명|스택 변화|
|:------|:---|---|
|lua_len(L, index)| #연산(길이 구하기) 연산의 결과를 스택에 반환. 메타메소드 존중됨|[-0, +1, e]|
|lua_concat(L, n)|스택 최상단의 n개 문자열을 꺼내서 이어 붙인 결과를 스택 최상단에 추가|[-n, +1, e]|
|lua_isnone(L, index)|유효한 index 이면 1, 아니면 0을 직접 반환|[-0, +0, -]|


## luaL_ 보조함수

### 인자값 체크

|이름|설명|스택 변화|
|:------|:---|---|
|luaL_argcheck(L, cond, arg, extramsg)|cond 값이 참인지 체크. 아니면 표준 인자 에러 발생|[-0, +0, v]|
|luaL_checktype(L, arg, t)|arg 스택 위치에 인자가 t 타입인지 확인. 아니면 에러 발생|[-0, +0, v]|
|luaL_checkany(L, arg)|arg 스택 위치에 인자가 nil 포함해서 뭐라도 있는지 체크. 없으면 에러 발생|[-0, +0, v]|
|luaL_checknumber(L, arg)|arg 스택 위치에서 숫자 인자 꺼내기. 숫자 타입이 아니면 에러 발생|[-0, +0, v]|
|luaL_checkstring(L, arg)|arg 스택 위치에서 문자열 꺼내기. 문자열 타입이 아니면 에러 발생|[-0, +0, v]|
|luaL_optinteger(L, arg, d)|arg 스택 위치에서 숫자 인자 꺼내기. nil 혹은 없으면 기본값 d. 변환 불가 값이면 에러 발생|[-0, +0, v]|

### 버퍼(luaL_Buffer) 사용

|이름|설명|스택 변화|
|:------|:---|---|
|luaL_buffinit(L, B)|크기 지정 없이 버퍼 초기화|[-0, +0, -]|
|luaL_buffinitsize(L, B, size)|size 크기만큼 버퍼 초기화|[-?, +?, m]|
|luaL_addvalue(B)|스택 최상단의 문자열을 버퍼에 추가|[-1, +?, m]|
|luaL_addstring(B, s)|s 문자열을 버퍼에 추가|[-?, +?, m]|
|luaL_addlstring(B, s, len)|s 문자열의 앞 len 길이 만큼 버퍼에 추가|[-?, +?, m]|
|luaL_addchar(B, c)|c 문자를 버퍼에 추가|[-?, +?, m]|
|luaL_pushresult(B)|버퍼에 있는 내용을 스택 최상단에 추가|[-?, +1, m]|
|luaL_pushresultsize(B, len)|버퍼에 있는 내용을 스택 최상단에 추가|[-?, +1, m]|

### 참조

|이름|설명|스택 변화|
|:------|:---|---|
|luaL_ref(L, LUA_REGISTRYINDEX)|스택 최상단 값을 꺼내서 레지스트리에 저장하고 그 참조 값을 직접 반환|[-1, +0, m]|
|luaL_unref(L, LUA_REGISTRYINDEX, ref)|레지스트리에서 ref 에 대한 참조를 삭제|[-0, +0, -]|


### 기타

|이름|설명|스택 변화|
|:------|:---|---|
|luaL_len(L, index)|#연산(길이 구하기)의 결과를 int 로 직접 반환. 메타메소드 무시됨|[-0, +0, e]|
|luaL_newlib(L, libs)|libs 배열에 지정된 데로 C라이브러리 만들어 스택에 넣음|[-0, +1, m]|



# (WIP)
