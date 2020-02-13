---
layout: post
title:  "Odoo加载机制流程源码解析"
date: 2019-07-12 08:15:00 +0800
categories: jekyll update
---

"Odoo加载机制流程源码解析
===
本文参考主要内容参考！[]https://www.cnblogs.com/dancesir/p/7008852.html
分析的是odoo12的源码，加了一些自己的理解，特别是关于model注册到registry部分内容，查遍整个互联网也没看到详细解释，为自己研究和原创。模块加载外层就是封装一个Registry(Mapping)对象:实际是一个字典，它包含对应的db，model等映射关系，一个DB对应一个Registry。后续的操作都会围绕这个Registry进行，将相关的数据赋值给相应的属性项。modules/registry.py定义了Registry类，Registry类的__new__函数调用new()函数产生一个实例，new()调用odoo.modules.load_modules()函数装载模块（位于modules/loading.py:321)。其中的核心程序为load_modules函数。函数经过如下步骤完成了模块（module）和模型（model）的加载。

1. 初始化数据库（初次运行）
调用odoo.modules.db.initialize(cr)(modules/db.py:17)完成加载base模块下的base.sql文件并执行。
此时数据库表为(具体见base.sql)：
```
CREATE TABLE ir_actions (
...
CREATE TABLE ir_model (
...
CREATE TABLE ir_model_fields (
...
CREATE TABLE res_lang (
...
CREATE TABLE res_users (
...
create table wkf (
...
CREATE TABLE ir_module_category (
...
CREATE TABLE ir_module_module (
...
CREATE TABLE ir_module_module_dependency (
...
CREATE TABLE ir_model_data (
...
CREATE TABLE ir_model_constraint (
...
CREATE TABLE ir_model_relation (
...  
CREATE TABLE res_currency (
...
CREATE TABLE res_company (
...
CREATE TABLE res_partner (
...
```
这20张表是odoo系统级的，它是模块加载及系统运行的基础。后续模块生成的表及相关数据都可以在这20张中找到蛛丝马迹。

2. 数据库表初始化后，就可以加载模块数据（addons）到数据库了，这个也是odoo作为平台灵活的原因，所有的数据都在数据库。
找到addons-path下所有的模块,然后一个一个的加载到数据库中。
先完成如下操作odoo.modules.db.initialize()(modules/db.py:336)：
* 将模块信息写入ir_module_module表：
* 将module信息写入ir_model_data表：
一个module要写两次ir_model_data表，
* 写module的dependency表：根据依赖关系进行判断，递归更新那些需要auto_install的模块状态为“to install”。

到目前为止，模块的加载都是在数据库级别，只是将“模块文件”信息存入数据库表，但是还没有真正加载到程序中。
Odoo运行时查找object是通过Registry.get()获取的，而不是通过python自己的机制来找到相应的object，所以odoo在最初import模块时(cli/command.py:53)会把模块下包含的model全部注册到models.py的MetaModel.module_to_models类变量(注意此时仅仅登记了model的类，并未完成实例化)。
下面的步骤就是加载模块到内存（实例化）。

3. 加载base模块
graph.add_module()(:352)定义在modules/graph.py中两个类Graph,Node。Graph是一个存放Node的类，Node代表一个模块，Graph表示了模块之间的关系。其depth代表深度0，代表没有依赖模块，children代表依赖他的模块，所以Node的graph指向Graph类。
Graph的add_modules方法使得一组module和其关系信息全部导入Graph，当一个模块的所有依赖模块已经在Graph里，则简单的加入该模块（add_node），如果不满足，则加入later并且将模块放到列表最后，如此循环，直到packages列表空或者later列表里的模块已经超过等待导入的模块。注意info保存了模块所有信息的字典，通过load_information_from_description_file()导入文件__manifest__.py的内容。<br>。
之后调用module/loading.py(:360)的load_module_graph()方法加载模块，最终执行加载的方法。这个方法是odoo加载model的核心，通过 __import__方法加载模块，这个是python的机制，当import到某个继承了BaseModel类的class时，它的实例化将有别于python自身的实例化操作，后者说它根本不会通过python自身的__new__方法创建实例，所有的实例创建都是通过实例化Registry对象在执行new函数是调用load_modules(registry.py:86)实现模型的实例化。load_modules(loading.py:321) _build_model 方法及元类创建。通过这种方式实例化model就可以解决我们在xml中配置model时指定的继承，字段，约束等各种属性。

4. 标记需要加载或者更新的模块（db）
5. 加载被标记的模块（加载过程与加载base模块一致）
6. 完成及清理安装
7. 清理菜单
8. 删除卸载的模块
9. 核实model的view
10. 运行post-install测试