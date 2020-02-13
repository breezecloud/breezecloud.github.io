---
layout: post
title:  "odoo-code-example"
date: 2019-05-13 08:15:00 +0800
categories: jekyll update
---
* [(一)数据模型](#1)
   * [创建数据模型](#2)
   * [测试](#3)
   * [向模型添加字段](#4)
* [(二)视图](#100)
   * [添加菜单](#101)
   * [创建窗体视图](#102)
* [(三)业务逻辑](#200)
* [(四)权限](#300)
   * [访问控制安全](#301)
   * [Row-level访问规则](#302)


<span id="1">(一)数据模型</span>
=========
部分表名称
```
res_users 用户
res_groups 用户组(角色)
res_lang 语言
res_partner 供应商/客户/联系人
res_font 字体
res_company 公司
res_bank 银行
res_country 国家
res_country_state 州/省
res_currency 货币
res_currency_rate 汇率
ir_ui_menu 菜单 
ir_act_window 菜单动作
ir_act_window_view 菜单动作与视图的对应关系
ir_ui_view 视图
ir_cron 计划的动作
wkf 工作流
wkf_activity 活动
wkf_transition 迁移
sale_order 报价单/销售订单
sale_order_line 销售订单明细行
purchase_order 询价单/采购订单
purchase_order_line 采购订单明细行
product_category 产品分类
product_product 产品
product_template 产品信息
product_uom 计量单位
product_pricelist 价格表
product_pricelist_version 价格表版本
product_price_type 计价类型
stock_warehouse 仓库
stock_location 库位
stock_picking 分拣单(出库/入库/内部移动)
stock_move 库存移动(移库单)
stock_quant 库存分析
stock_inventory 库存盘点
stock_inventory_line 库存盘点明细
stock_production_lot 序列号(产品批次)
stock_warehouse_orderpoint 再订货规则
procurement_rule 补货规则(拉式流)
procurement_order 补货单
stock_location_route 路线
account_asset_category 固定资产类别
account_asset_asset 固定资产
account_account 会计科目
account_account_type 科目类型
account_tax 税
account_fiscalyear 会计年度
account_period 会计期间
account_invoice 发票
account_invoice_line 发票明细
account_move 会计凭证(会计分录)
account_move_line 会计凭证明细
account_voucher 记账凭证(收付款凭证)
account_voucher_line 记账明细
account_journal 凭证类型
```
<span id="2">创建数据模型</span>
=========
```
  # -*- coding: utf-8 -*- 
  from odoo import models, fields 
  class TodoTask(models.Model): 
   _name = 'todo.task' 
   _description = 'To-do Task'
   name = fields.Char('Description', required=True) 
   is_done = fields.Boolean('Done?') 
   active = fields.Boolean('Active?', default=True) 
```
<span id="3">测试</span>
=========
```
# -*- coding: utf-8 -*-
from odoo.tests.common import TransactionCase
class TestTodo(TransactionCase):
def test_create(self):
    "Create a simple Todo"
    Todo = self.env['todo.task']
    task = Todo.create({'name': 'Test Task'})
    self.assertEqual(task.is_done, False)
```
执行 $ ./odoo-bin -d todo -i todo_app --test-enable进行测试

下面动作的测试函数
```
   # def test_create(self):
        # ...
        # Test Toggle Done
        task.do_toggle_done()
        self.assertTrue(task.is_done)
        # Test Clear Done
        Todo.do_clear_done()
        self.assertFalse(task.active)
```
测试安全访问权限
```
  ＃class TestTodo（TransactionCase):
        def setUp（self，* args，** kwargs）：
            result = super（TestTodo，self）.setUp（* args，\
            ** kwargs）
            user_demo = self.env.ref（'base.user_demo'）
            self.env = self.env（user = user_demo）
            return result
```

<span id="4">向模型添加字段</span>
=========
类继承
```
# -*- coding: utf-8 -*-
  from odoo import models, fields, api
  class TodoTask(models.Model):
  _inherit = 'todo.task'
  user_id = fields.Many2one('res.users', 'Responsible')
  date_deadline = fields.Date('Deadline')
```
接下来在model文件夹里面还需要创建一个__init__.py文件，代码如下：
```
#-*- coding:utf-8 -*-
#!/usr/bin/env python
from . import todo_task
```
通过继承方式也可以对父类的字段进行属性修改，它是通过向子类添加和父类具有相同名称的字段来完成的，但是只能对字段进行属性的设置。 举例,如果我们要给当前例子的父类的name field添加一个帮助信息,我们可以在子类的todo_task.py文件里面类的成员添加如下代码: ```name = fields.Char(help="What needs to be done?")``` 注意：这个name是父类的属性<br>

继承也在业务逻辑级别起作用。添加新方法很简单：只需在继承类中声明新的函数。<br>
要扩展或更改现有逻辑，可以通过声明具有完全相同名称的方法来覆盖相应的方法。新方法将替换前一个方法，它也可以只是扩展继承类的代码，使用Python的super（）方法来调用父方法。然后，可以在调用super（）方法之前和之后，在原有逻辑周围添加新逻辑。
```
 from odoo.exceptions import ValidationError  
 # ...
 # class TodoTask(models.Model):
 # ...
 @api.one
 def do_toggle_done(self):
   if self.user_id != self.env.user:
     raise Exception('Only the responsible can do this!')
   else:
     return super(TodoTask, self).do_toggle_done()
```

原型继承复制特征
```
from odoo import models 
class TodoTask(models.Model): 
_name = 'todo.task' 
_inherit = 'mail.thread'
```
使用委托继承嵌入模型
```
from odoo import models, fields 
class User(models.Model): 
_name = 'res.users' 
_inherits = {'res.partner': 'partner_id'} 
partner_id = fields.Many2one('res.partner')
```
请注意，使用委托继承，字段是继承的，但方法不是。<br>


<span id="100">(一)视图</span>
=========

<span id="101">添加菜单</span>
=========
```
<?xml version="1.0"?> 
<odoo> 
 <act_window id="action_todo_task" 
   name="To-do Task" 
   res_model="todo.task" 
   view_mode="tree,form" /> 
 <menuitem id="menu_todo_task" 
   name="Todos" 
   action="action_todo_task" /> 
</odoo> 
```
修改菜单和操作记录
```
   < ！ — — 修改菜单项-->
   <record id="todo_app.menu_todo_task" model="ir.ui.menu">
       <field name="name">My To-Do</field>
   </record>
   <record model="ir.actions.act_window"
    id="todo_app.action_todo_task">
       <field name="context">
          {'search_default_filter_my_tasks': True}
       </field>
   </record>   
```
<span id="102">(一)视图</span>
=========
```
<?xml version="1.0"?> 
<odoo> 
 <record id="view_form_todo_task" model="ir.ui.view"> 
   <field name="name">To-do Task Form</field> 
   <field name="model">todo.task</field> 
   <field name="arch" type="xml"> 
   <form>
       <header>
           <button name="do_toggle_done" type="object" string="Toggle Done" class="oe_highlight" />
           <button name="do_clear_done" type="object" string="Clear All Done" />
       </header>
       <sheet>
           <group name="group_top">
               <group name="group_left">
                   <field name="name"/>
               </group>
               <group name="group_right">
                   <field name="is_done"/>
                   <field name="active" readonly="1" />
               </group>
           </group>
       </sheet>
   </form>
   </field> 
 </record> 
</odoo>
```
列表视图（tree）
```
<record id="view_tree_todo_task" model="ir.ui.view"> 
 <field name="name">To-do Task Tree</field> 
 <field name="model">todo.task</field> 
 <field name="arch" type="xml"> 
   <tree decoration-muted="is_done==True"> 
     <field name="name"/> 
     <field name="is_done"/> 
   </tree> 
 </field> 
</record>
```
搜索视图
```
<record id="view_filter_todo_task" model="ir.ui.view"> 
 <field name="name">To-do Task Filter</field> 
 <field name="model">todo.task</field> 
 <field name="arch" type="xml"> 
   <search> 
     <field name="name"/> 
     <filter string="Not Done" 
       domain="[('is_done','=',False)]"/> 
     <filter string="Done" 
       domain="[('is_done','!=',False)]"/> 
   </search> 
 </field> 
</record> 
```
扩展视图
```
   <record id="view_form_todo_task_inherited" model="ir.ui.view">
       <field name="name">Todo Task form - User extension</field>
       <field name="model">todo.task</field>
       <field name="inherit_id" ref="todo_app.view_form_todo_task"/>
       <field name="arch" type="xml">
           <field name="name" position="after">
               <field name="user_id" />
           </field>
           <field name="is_done" position="before">
               <field name="date_deadline" />
           </field>
           <field name="active" position="attributes">
               <attribute name="invisible">1</attribute>
           </field>
       </field>
    </record> 
```
 下面是一个写在arch中的实现在is_done字段之前添加date_deadline字段的具体例子：
 ```
    <xpath expr="//field[@name='is_done']" position="before">
       <field name="date_deadline" />
   </xpath> 
 ```
 快捷方式
 ```
    <field name="is_done" position="before">
      <field name="date_deadline" />
   </field> 
 ```
after:将内容添加到父元素之中，匹配的节点之后。<br>
before:添加内容在匹配节点之前。<br>
inside（默认值）:匹配节点内的追加内容。<br>
replace:替换匹配的节点。如果使用空内容，它将删除该匹配的元素。从Odoo 10开始，它还允许用其他标记包装一个元素，通过在内容中使用$0来表示被替换的元素。<br>
attributes：修改匹配元素的XML属性。在元素内容使用<attribute name =“attr-name”>实现给属性name设置新属性值attr-name。<br>

扩展树视图
```
   <record id="view_tree_todo_task_inherited" model="ir.ui.view">
       <field name="name">Todo Task tree - User extension</field>
       <field name="model">todo.task</field>
       <field name="inherit_id" ref="todo_app.view_tree_todo_task"/>
       <field name="arch" type="xml">
            <field name="name" position="after"> 
                <field name="user_id" />
            </field>
       </field>
    </record>
```

扩展搜索视图
```
   <record id="view_filter_todo_task_inherited" model="ir.ui.view">
       <field name="name">Todo Task tree - User extension</field>
       <field name="model">todo.task</field>
       <field name="inherit_id" ref="todo_app.view_filter_todo_task"/>
       <field name="arch" type="xml">
           <field name="name" position="after">
               <field name="user_id" />
               <filter name="filter_my_tasks" string="My Tasks" domain="[('user_id','in',[uid,False])]" />
               <filter name="filter_not_assigned" string="Not Assigned" domain="[('user_id','=',False)]" />
           </field> 
       </field>
    </record> 
```

<span id="200">(三)业务逻辑</span>
=========

```
from odoo import models, fields, api
@api.multi 
def do_toggle_done(self): 
   for task in self: 
       task.is_done = not task.is_done 
   return True
```
通常表单按钮只能对选定的记录起作用，但在这种情况下，我们希望它也对除当前记录之外的记录起作用，所以使用```@api.model```装饰符
```
from odoo import models, fields, api
@api.model 
def do_clear_done(self): 
   dones = self.search([('is_done', '=', True)]) 
   dones.write({'active': False}) 
   return True
```
写入方法同时对记录集的所有元素设置值。 要写入的值使用字典进行描述。 在这里使用write比遍历记录集更有效率，以便逐一为每个记录集赋值。

<span id="301">访问控制安全</span>
=========

security/ir.model.access.csv。添加以下内容︰
```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
acess_todo_task_group_user,todo.task.user,model_todo_task,base.group_user,1,1,1,1
```

<span id="302">Row-level访问规则</span>
=========
security/todo_access_rules.xml
```
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    
        <record id="todo_task_user_rule" model="ir.rule">
            <field name="name">ToDo Tasks only for owner</field>
            <field name="model_id" ref="model_todo_task"/>
            <field name="domain_force">[('create_uid','=',user.id)]</field>
            <field name="groups" eval="[(4,ref('base.group_user'))]"/>
        </record>
    
</odoo>
```
小心noupdate ="1"属性。这意味着此数据在模块的升级时将不会更新。这就使它能够进行定制，因为后面模块的升级不会破坏用户进行的更改。但请注意，在开发的时候也会这样，所以在开发的时候你可以设置noupdate ="0" ，直到你对你的数据文件满意为止。<br>
在groups字段，你还会发现一个特殊的表达式。它是一个一对多的关系字段，他们有特殊的操作语法。在这种情况下， (4，x) 元组指要追加 x 记录，这里 的x 是关联的员工组，用base.group_user标识的。<br>
修改安全记录规则<br>
覆盖todo_app.todo_task_user_rule以将domain_force字段修改为新值
```
 <？xml version =“1.0”encoding =“utf-8”？>
   <odoo>
    <data noupdate =“1”>
        <record id =“todo_app.todo_task_per_user_rule”model =“ir.rule”>
           <field name =“name”>所有者和关注者的ToDo任务</ field>
           <field name =“model_id”ref =“model_todo_task”/>
           <field name =“groups”eval =“[（4，ref（'base.group_user'））]”/>
            <field name =“domain_force”>['|'，（'user_id'，'in'，[user.id，False]），（'message_follower_ids'，'in'，[user.partner_id.id]）]  </field>
        </ record>
    </ data>
  </ odoo>
```




&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin.jpg)