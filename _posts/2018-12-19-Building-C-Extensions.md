---
layout: post
title:  "构建C和C++扩展"
date: 2018-12-19 08:15:00 +0800
categories: jekyll update
---
4. Building C and C++ Extensions
4.构建C和C++扩展
A C extension for CPython is a shared library (e.g. a .so file on Linux, .pyd on Windows), which exports an initialization function.
CPython的C扩展是一个共享库（例如，Linux上的.so文件，Windows上的.pyd），它导出一个初始化函数。

To be importable, the shared library must be available on PYTHONPATH, and must be named after the module name, with an appropriate extension. When using distutils, the correct filename is generated automatically.
为了便于导入，共享库必须在PYTHONPATH上可用，并且必须以模块名称命名，并具有适当的扩展。当使用distutils时，将自动生成正确的文件名。

The initialization function has the signature:
初始化函数具有签名：

PyObject* PyInit_modulename(void)
It returns either a fully-initialized module, or a PyModuleDef instance. See Initializing C modules for details.
它返回一个完全初始化的模块，或者一个PyModuleDef实例。有关详细信息，请参阅初始化C模块。

For modules with ASCII-only names, the function must be named PyInit_<modulename>, with <modulename> replaced by the name of the module. When using Multi-phase initialization, non-ASCII module names are allowed. In this case, the initialization function name is PyInitU_<modulename>, with <modulename> encoded using Python’s punycode encoding with hyphens replaced by underscores. In Python:
对于只有ASCII名称的模块，函数必须命名为PyInit_<module name>，用<module name>替换为模块的名称。当使用多阶段初始化时，允许使用非ASCII模块名称。在这种情况下，初始化函数名是PyInitU_<modulename>，其中<modulename>使用Python的punycode编码，用下划线替换连字符。在Python中：
```
def initfunc_name(name):
    try:
        suffix = b'_' + name.encode('ascii')
    except UnicodeEncodeError:
        suffix = b'U_' + name.encode('punycode').replace(b'-', b'_')
    return b'PyInit' + suffix
```
It is possible to export multiple modules from a single shared library by defining multiple initialization functions. However, importing them requires using symbolic links or a custom importer, because by default only the function corresponding to the filename is found. See the “Multiple modules in one library” section in PEP 489 for details.
通过定义多个初始化函数，可以从单个共享库导出多个模块。但是，导入它们需要使用符号链接或自定义导入程序，因为默认情况下只找到与文件名对应的函数。有关详细信息，请参阅PEP 489中的“一个库中的多个模块”部分。

4.1. Building C and C++ Extensions with distutils
4.1 用ditudil构建C和C++扩展
Extension modules can be built using distutils, which is included in Python. Since distutils also supports creation of binary packages, users don’t necessarily need a compiler and distutils to install the extension.
扩展模块可以使用包含在Python中的distutils构建。由于distutils还支持创建二进制包，所以用户不一定需要编译器和distutils来安装扩展。


A distutils package contains a driver script, setup.py. This is a plain Python file, which, in the most simple case, could look like this:
distutils包包含驱动程序脚本setup.py。这是一个普通的Python文件，在最简单的情况下，它可以如下所示：
```
from distutils.core import setup, Extension

module1 = Extension('demo',
                    sources = ['demo.c'])

setup (name = 'PackageName',
       version = '1.0',
       description = 'This is a demo package',
       ext_modules = [module1])
```       
With this setup.py, and a file demo.c, running
使用此setup.py和一个文件demo.c，运行
```
python setup.py build
```
will compile demo.c, and produce an extension module named demo in the build directory. Depending on the system, the module file will end up in a subdirectory build/lib.system, and may have a name like demo.so or demo.pyd.
将编译demo.c，并在构建目录中生成名为demo的扩展模块。根据系统的不同，模块文件将以子目录build/lib.system结束，并且可能有demo.so或demo.pyd这样的名称。

In the setup.py, all execution is performed by calling the setup function. This takes a variable number of keyword arguments, of which the example above uses only a subset. Specifically, the example specifies meta-information to build packages, and it specifies the contents of the package. Normally, a package will contain additional modules, like Python source modules, documentation, subpackages, etc. Please refer to the distutils documentation in Distributing Python Modules (Legacy version) to learn more about the features of distutils; this section explains building extension modules only.
在setup.py中，通过调用setup函数来执行所有执行。这需要可变数量的关键字参数，上面的示例只使用其中的一个子集。具体而言，该示例指定构建包的元信息，并指定包的内容。通常，包将包含其他模块，如Python源模块、文档、子包等。请参阅分发Python模块（遗留版本）中的distutils文档，以了解更多关于distutils的特性；本节仅解释构建扩展模块。

It is common to pre-compute arguments to setup(), to better structure the driver script. In the example above, the ext_modules argument to setup() is a list of extension modules, each of which is an instance of the Extension. In the example, the instance defines an extension named demo which is build by compiling a single source file, demo.c.
通常预先计算setup()的参数，以便更好地构造驱动程序脚本。在上面的示例中，setup()的ext_modules参数是扩展模块的列表，每个扩展模块都是扩展的实例。在该示例中，实例定义了名为demo的扩展，该扩展是通过编译单个源文件demo.c构建的。
In many cases, building an extension is more complex, since additional preprocessor defines and libraries may be needed. This is demonstrated in the example below.
在许多情况下，构建扩展更加复杂，因为可能需要额外的预处理器定义和库。这在下面的示例中演示。
```
from distutils.core import setup, Extension

module1 = Extension('demo',
                    define_macros = [('MAJOR_VERSION', '1'),
                                     ('MINOR_VERSION', '0')],
                    include_dirs = ['/usr/local/include'],
                    libraries = ['tcl83'],
                    library_dirs = ['/usr/local/lib'],
                    sources = ['demo.c'])

setup (name = 'PackageName',
       version = '1.0',
       description = 'This is a demo package',
       author = 'Martin v. Loewis',
       author_email = 'martin@v.loewis.de',
       url = 'https://docs.python.org/extending/building',
       long_description = '''
This is really just a demo package.
''',
       ext_modules = [module1])
```
In this example, setup() is called with additional meta-information, which is recommended when distribution packages have to be built. For the extension itself, it specifies preprocessor defines, include directories, library directories, and libraries. Depending on the compiler, distutils passes this information in different ways to the compiler. For example, on Unix, this may result in the compilation commands
在这个示例中，使用附加的元信息调用setup()，当必须构建分发包时，建议使用附加的元信息。对于扩展本身，它指定了预处理器定义，包括目录、库目录和库。根据编译器的不同，distutils以不同的方式将此信息传递给编译器。例如，在Unix上，这可能导致编译命令
```
gcc -DNDEBUG -g -O3 -Wall -Wstrict-prototypes -fPIC -DMAJOR_VERSION=1 -DMINOR_VERSION=0 -I/usr/local/include -I/usr/local/include/python2.2 -c demo.c -o build/temp.linux-i686-2.2/demo.o

gcc -shared build/temp.linux-i686-2.2/demo.o -L/usr/local/lib -ltcl83 -o build/lib.linux-i686-2.2/demo.so
These lines are for demonstration purposes only; distutils users should trust that distutils gets the invocations right.
```
4.2. Distributing your extension modules
4.2 分发扩展模块
When an extension has been successfully build, there are three ways to use it.
当一个扩展已经成功构建时，有三种方法使用它。

End-users will typically want to install the module, they do so by running
最终用户通常希望安装该模块，他们通过运行
```
python setup.py install
```
Module maintainers should produce source packages; to do so, they run
模块维护人员应该生成源包；为此，它们运行
```
python setup.py sdist
```
In some cases, additional files need to be included in a source distribution; this is done through a MANIFEST.in file; see Specifying the files to distribute for details.
在某些情况下，需要在源分发中包括其他文件；这是通过MANIFEST.in文件完成的；有关详细信息，请参阅指定要分发的文件。

If the source distribution has been build successfully, maintainers can also create binary distributions. Depending on the platform, one of the following commands can be used to do so.
如果源发行版已经成功构建，维护人员还可以创建二进制发行版。根据平台的不同，可以使用以下命令之一。
```
python setup.py bdist_wininst
python setup.py bdist_rpm
python setup.py bdist_dumb
```