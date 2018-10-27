---
title: Extending Python with C or C++
categories:
  - Geek
tags:
  - python
  - c
date: 2018-10-20 20:03:29
---

探索在linux环境下，如何用c/c++语言扩展部分python逻辑。

<!--more-->

开发环境：
- ubuntu 16.04
- python3
- clion

[简单的例子](https://en.wikibooks.org/wiki/Python_Programming/Extending_with_C)

[FindPythonLibs](https://cmake.org/cmake/help/latest/module/FindPythonLibs.html)
在这里可以看到cmake如何添加python.h支持，只需要在原有的cmakeLists.txt添加如下指令即可：
```
find_package(PythonLibs 3 REQUIRED)
include_directories(${PYTHON_INCLUDE_DIRS})
...
target_link_libraries(${project} ${PYTHON_LIBRARIES})
```
如果对python版本没有需求，find_package第二个参数3可以省略。

假设我们现在用c编写了一个基础函数，并返回一个c类型的结构体，对应的main.h和main.c内容如下:

main.h:
```
#ifndef CBOX_MAIN_H
#define CBOX_MAIN_H
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
typedef struct executeResult {
    int val1;
    long val2;
    char *str;
} c_result;
int func(int x, int y, const char *s, c_result *result);
#endif //CBOX_MAIN_H
```
main.c:
```
#include "main.h"
int func(int x, int y, const char *s, c_result *result) {
    result->val1 = x;
    result->val2 = y;
    result->str = malloc(strlen(s) + 1);
    stpcpy(result->str, s);
    return x + y;
}
int main() {
    printf("Hello, World!\n");
    return 0;
}
```
为了在python中使用这个c函数，需要利用python.h头文件对原有的函数进行包装，最终我们想达成如下效果(上述函数最后一个参数为结果结构体的引用):
```python
>>> import cbox
>>> code, result = cbox.func(1, 2, "string here")
>>> code
3
>>> result.val1
1
>>> result.str
string here
```
在python.h文件当中，最重要的一个结构体是PyObject，这个结构体可以和python当中直接使用的任何对象进行转化。

一般情况下，包装函数结构如下:
```cpp
static PyObject *
wrapper_func(PyObject *self, PyObject *args) {
	// parse args
    // using args call orgin func
    // return values
}
```
固定的包含两个参数self和args，self指向模块对象和模块级别的函数，对于一个方法它指向对象的实例。args指向一个python元组对象，对应着python方法中的位置参数。

使用PyArg_ParseTuple函数来从args中解析参数:
```
int a, b;
const char *str;
if (!PyArg_ParseTuple(args, "iis", &a, &b, &str)) 
    return NULL;
```
"iis"代表解析两个integer和一个string，更详细的解析格式可以看[官方文档](https://docs.python.org/2.0/ext/parseTuple.html)

对于包装函数，如果返回NULL，在python中即为抛出异常，具体的异常信息在后面会说明。

PyArg_ParseTuple如果解析失败会返回0值，并设置恰当的提示信息，否则返回1值并将参数解析到相应的c类型数据中。

如果为void函数，需要返回一个None值：
```cpp
Py_INCREF(Py_None);
return Py_None;
```

对于异常处理，需要在文件开头声明一个PyObject指针:
```cpp
static PyObject *CboxError;
```
然后在模块的初始化函数中初始化这个异常对象:
```cpp
PyMODINIT_FUNC
PyInit_cbox(void)
{
    PyObject *m;

    m = PyModule_Create(&cboxmodule);
    if (m == NULL)
        return NULL;

    CboxError = PyErr_NewException("cbox.error", NULL, NULL);
    Py_INCREF(CboxError);
    PyModule_AddObject(m, "error", CboxError);
    return m;
}
```
对于异常处理，最常使用的应该是PyErr_SetString函数，在初始化之后，可以在包装函数中如下使用:
```cpp
if (error_occurred) {
	PyErr_SetString(CboxError, "some error occurred");
    return NULL;
}
```
现在再回到一开始的包装函数，如果只返回一个ret code，可以如下编写：

```cpp
static PyObject *
wrapper_func(PyObject *self, PyObject *args) {
    int a, b;
    const char *str;
    if (!PyArg_ParseTuple(args, "iis", &a, &b, &str))
        return NULL;
    c_result result;
    int code = func(a, b, str, &result);
    return PyLong_FromLong(code);
}
```

现在可以开始处理模块的方法表和初始化函数：
```cpp
static PyMethodDef cBoxMethods[] = {
		...
        {"func",  wrapper_func, METH_VARARGS,
                    "a func in c box."},
        ...
        {NULL, NULL, 0, NULL}        /* Sentinel */
};
```
每一个元素为一个四元组(函数名，函数，参数类型，方法介绍)，第三个参数类型一般为METH_VARARGS或者METH_VARARGS | METH_KEYWORDS，分别代表裸位置参数或者位置参数+关键字参数的组合，对应着使用PyArg_ParseTuple()或者PyArg_ParseTupleAndKeywords()函数来解析。

然后定义这个python模块：
```cpp
static struct PyModuleDef cboxmodule = {
        PyModuleDef_HEAD_INIT,
        "cbox",     /* name of module */
        NULL,       /* module documentation, may be NULL */
        -1,         /* size of per-interpreter state of the module,
                    or -1 if the module keeps state in global variables. */
        cBoxMethods
};
```
编写对应的初始化函数，注意这个函数的命名，必须是PyInit_{packge_name}形式：
```
PyMODINIT_FUNC
PyInit_cbox(void)
{
    PyObject *m;

    m = PyModule_Create(&cboxmodule);
    if (m == NULL)
        return NULL;

    CboxError = PyErr_NewException("cbox.error", NULL, NULL);
    Py_INCREF(CboxError);
    PyModule_AddObject(m, "error", CboxError);
    return m;
}
```

为了方便，在同级目录下创建setup.py文件：
```python
from distutils.core import setup, Extension

cbox_module = Extension('cbox', sources=['cbox.c', 'main.c'])

setup(name='cbox',
      version='1.1',
      description='this is a demo',
      ext_modules=[cbox_module])
```
注意使用虚拟的python环境以避免破坏机器的全局python环境，pip install .进行安装

进入python终端：
```python
>>> from cbox import func
>>> func(1, 2, "ddd")
3
>>> func("q", 2, "ddd")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: an integer is required (got type str)
>>> 
```

再回到一开始的包装函数，修改返回值，利用Py_BuildValue函数即可：

```
static PyObject *
wrapper_func(PyObject *self, PyObject *args) {
    int a, b;
    const char *str;
    if (!PyArg_ParseTuple(args, "iis", &a, &b, &str))
        return NULL;
    c_result result;
    int code = func(a, b, str, &result);
    return Py_BuildValue("(i, {s:i,s:i,s:s})", 
            code, "val1", result.val1, 
            "val2", result.val2, 
            "str", result.str);
};
```
重新安装后执行如下：
```
>>> from cbox import func
>>> a, b = func(1, 2, "ddd")
>>> a
3
>>> b
{'val2': 2, 'val1': 1, 'str': 'ddd'}
>>> 
```