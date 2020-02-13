---
layout: post
title:  "使用C或C++扩展Python(一)-从python调用C"
date: 2018-12-28 08:15:00 +0800
categories: jekyll update
---
原文：https://docs.python.org/dev/extending/extending.html

"使用C或C++扩展Python(一)-从python调用C"
===
&emsp;&emsp;如果您知道如何用C编程，那么在Python中添加新的内置模块是相当容易的。扩展模块可以做两件不能在Python中直接完成的事情：它们可以实现新的内置对象类型，也可以调用C库函数和系统调用。为了支持扩展，PythonAPI(应用程序编程器接口)定义了一组函数、宏和变量，这些函数、宏和变量提供了对Python运行时系统大部分的访问。PythonAPI是通过包含报头"Python.h"而合并到C源文件中的。"Python.h"定义的所有用户可见符号Python.h前缀为Py或PY,为了方便起见，"Python.h"包括一些标准的头文件：&lt;stdio.h&gt;, &lt;string.h&gt;, &lt;errno.h&gt;，和&lt;stdlib.h&gt;。<br>
&emsp;&emsp;让我们创建一个扩展模块，名为spam。我们想要为C库函数创建一个Python接口system() ，此函数以一个以空结尾的字符串作为参数，并返回一个整数。我们希望这个函数可以从Python中调用，如下所示：<br>
```
>>> import spam
>>> status = spam.system("ls -l")
```

完整程序清单：
```
/*
spam.c
compile:gcc -fPIC -shared -I /usr/include/python3.5m/ spam.c -o spam.so
usage:
>>> import spam
>>> spam.system("ls -l")
reference:https://docs.python.org/dev/extending/extending.html
*/
#include <Python.h>

static PyObject *SpamError;

static PyObject *
spam_system(PyObject *self, PyObject *args)
{
    const char *command;
    int sts;

    if (!PyArg_ParseTuple(args, "s", &command))
        return NULL;
    sts = system(command);
    if (sts < 0) {
        PyErr_SetString(SpamError, "System command failed");
        return NULL;
    }
    return PyLong_FromLong(sts);
}

static PyMethodDef SpamMethods[] = {
    {"system",(PyCFunction)spam_system,METH_VARARGS,"Execute a shell command"},
    {NULL,NULL,0,NULL}
};

static struct PyModuleDef spammodule = {
    PyModuleDef_HEAD_INIT,
    "spam",   /* name of module */
    "Example module that creates an extension type.", /* module documentation, may be NULL */
    -1,       /* size of per-interpreter state of the module,or -1 if the module keeps state in global variables. */
    SpamMethods
};


PyMODINIT_FUNC
PyInit_spam(void)
{
    PyObject *m;

    m = PyModule_Create(&spammodule);
    if (m == NULL)
        return NULL;

    SpamError = PyErr_NewException("spam.error", NULL, NULL);
    Py_INCREF(SpamError);
    PyModule_AddObject(m, "error", SpamError);
    return m;
}
```