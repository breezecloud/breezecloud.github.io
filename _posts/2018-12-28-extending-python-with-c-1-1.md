---
layout: post
title:  "使用C或C++扩展Python(一)-从python调用C(续)"
date: 2018-12-28 09:15:00 +0800
categories: jekyll update
---
原文：https://docs.python.org/dev/extending/extending.html

"使用C或C++扩展Python(一)-从python调用C(续)"
===
如果您知道如何用C编程，那么在Python中添加新的内置模块是相当容易的。扩展模块可以做两件不能在Python中直接完成的事情：它们可以实现新的内置对象类型，也可以调用C库函数和系统调用。
为了支持扩展，PythonAPI(应用程序编程器接口)定义了一组函数、宏和变量，这些函数、宏和变量提供了对Python运行时系统大部分的访问。PythonAPI是通过包含报头"Python.h"而合并到C源文件中的。
```
/*
keywdarg.c
compile:gcc -fPIC -shared -I /usr/include/python3.5m/ keywdarg.c -o keywdarg.so
usage:
>>> import keywds
>>> keywds.parrot(voltage=1)
reference:https://docs.python.org/dev/extending/extending.html
*/
#include <Python.h>

static PyObject *
keywdarg_parrot(PyObject *self, PyObject *args, PyObject *keywds)
{
    int voltage;
    const char *state = "a stiff";
    const char *action = "voom";
    const char *type = "Norwegian Blue";

    static char *kwlist[] = {"voltage", "state", "action", "type", NULL};

    if (!PyArg_ParseTupleAndKeywords(args, keywds, "i|sss", kwlist,
                                     &voltage, &state, &action, &type))
        return NULL;

    printf("-- This parrot wouldn't %s if you put %i Volts through it.\n",
           action, voltage);
    printf("-- Lovely plumage, the %s -- It's %s!\n", type, state);

    Py_RETURN_NONE;
}

static PyMethodDef keywdarg_methods[] = {
    /* The cast of the function is necessary since PyCFunction values
     * only take two PyObject* parameters, and keywdarg_parrot() takes
     * three.
     */
    {"parrot", (PyCFunction)keywdarg_parrot, METH_VARARGS | METH_KEYWORDS,
     "Print a lovely skit to standard output."},
    {NULL, NULL, 0, NULL}   /* sentinel */
};

static struct PyModuleDef keywdargmodule = {
    PyModuleDef_HEAD_INIT,
    "keywdarg",
    NULL,
    -1,
    keywdarg_methods
};

PyMODINIT_FUNC
PyInit_keywdarg(void)
{
    return PyModule_Create(&keywdargmodule);
}
```