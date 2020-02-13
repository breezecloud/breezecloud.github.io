---
layout: post
title:  "使用C或C++扩展Python(三)-定义扩展类(基础篇)"
date: 2019-01-02 10:15:00 +0800
categories: jekyll update
---
原文：https://docs.python.org/3/extending/newtypes_tutorial.html

"使用C或C++扩展Python(三)-定义扩展类(基础篇)"
===

```
/*
custom.c
compile:gcc -fPIC -shared -I /usr/include/python3.5m/ custom.c -o custom.so
usage:
>>> import custom
>>> mycustom = custom.Custom()
reference:https://docs.python.org/3/extending/newtypes_tutorial.html
*/
#include <Python.h>

typedef struct {
    PyObject_HEAD
    /* Type-specific fields go here. */
} CustomObject;

static PyTypeObject CustomType = {
    PyVarObject_HEAD_INIT(NULL, 0)
    .tp_name = "custom.Custom",
    .tp_doc = "Custom objects",
    .tp_basicsize = sizeof(CustomObject),
    .tp_itemsize = 0,
    .tp_flags = Py_TPFLAGS_DEFAULT,
    .tp_new = PyType_GenericNew,
};

static PyModuleDef custommodule = {
    PyModuleDef_HEAD_INIT,
    .m_name = "custom",
    .m_doc = "Example module that creates an extension type.",
    .m_size = -1,
};

PyMODINIT_FUNC
PyInit_custom(void)
{
    PyObject *m;
    if (PyType_Ready(&CustomType) < 0)
        return NULL;

    m = PyModule_Create(&custommodule);
    if (m == NULL)
        return NULL;

    Py_INCREF(&CustomType);
    PyModule_AddObject(m, "Custom", (PyObject *) &CustomType);
    return m;
}
```