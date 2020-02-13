---
layout: post
title:  "使用C或C++扩展Python(二)-从C调用python"
date: 2019-01-02 08:15:00 +0800
categories: jekyll update
---
原文：https://docs.python.org/dev/extending/extending.html

"使用C或C++扩展Python(二)-从C调用python"
===

```
/*
spamcall.c
compile:gcc -fPIC -shared -I /usr/include/python3.5m/ spamcall.c -o spamcall.so
usage:
>>> import spamcall
>>> def func(a):
        print(a)
>>> spamcall.set_calllback(func)
>>> spamcall.call_it(123)
123
reference:https://docs.python.org/dev/extending/extending.html
*/
#include <Python.h>

static PyObject *my_callback = NULL;    // def func(a):

static PyObject *
my_set_callback(PyObject *dummy, PyObject *args)
{
    PyObject *result = NULL;
    PyObject *temp;

    if (PyArg_ParseTuple(args, "O:set_callback", &temp)) {
        if (!PyCallable_Check(temp)) {
            PyErr_SetString(PyExc_TypeError, "parameter must be callable");
            return NULL;
        }
        Py_XINCREF(temp);
        Py_XDECREF(my_callback);
        my_callback = temp;
        Py_INCREF(Py_None);
        result = Py_None;
    }
    return result;
}

static PyObject *
my_call_it(PyObject *dummy, PyObject *args)
{
    int arg;
    PyObject *arglist;
    PyObject *result;


    arg = 123;
    if (!PyArg_ParseTuple(args, "i", &arg))
        return NULL;
    arglist = Py_BuildValue("(i)", arg);
    result  = PyObject_CallObject(my_callback, arglist);
    Py_DECREF(arglist);
    if (result == NULL)
        return NULL;
    Py_DECREF(result);

    Py_RETURN_NONE;    // Py_INCREF(Py_None); return Py_None;
}

static PyMethodDef SpamMethods[] = {
    {"set_callback", my_set_callback, METH_VARARGS, "set a callback"},
    {"call_it", my_call_it, METH_VARARGS, "call it"},
    {NULL, NULL, 0, NULL}
};

static struct PyModuleDef spammodule = {
    PyModuleDef_HEAD_INIT,
    "spamcall",   /* name of module */
    "Example module that creates an extension type.", /* module documentation, may be NULL */
    -1,       /* size of per-interpreter state of the module,or -1 if the module keeps state in global variables. */
    SpamMethods
};

PyMODINIT_FUNC
PyInit_spamcall(void)
{
    PyObject *m;

    m = PyModule_Create(&spammodule);
    if (m == NULL)
        return NULL;
    return m;
}
```