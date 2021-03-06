# 设计通则
## 数据库表设计原则
### 权衡规范与反规范
- 3NF vs 数据冗余

原则上满足3NF（第三范式）。有时为了提高某些查询或应用的性能，可以破坏规范规则通过冗余数据提高性能。

- 读多写少 vs 写多读少

TBD

### 数据库表分类
- 基本数据表：描述业务实体的基本信息。例如，用户基本信息、单位基本信息等。
- 标准编码表（字典表）：描述属性的列表值。例如，职称、民族、状态等。
- 业务数据表：记录业务发生的过程和结果。例如，商品订单表，购物车表等。
- 系统信息表：存放与系统操作、业务控制有关的参数。例如，用户信息、权限、用户配置信息等。
- 统计数据表：存放业务数据统计值。例如，订单统计、访问统计等。
- 临时处理表：存放业务处理过程中的中间结果。
- 其他类型表：存放应用层的日志、消息记录等。

### 字段设计原则
- 使用最小类型

使用能正确存储和表示数据的最小类型。如果不确定需要什么数据类型，则选择不会超出范围的最小类型。

- 选择更简单的数据类型。

整型比字符型代价小。

If you have a fixed size then you can use char(8) -> 8B , mysql works much faster with fixed size fields, but if you chose int -> 4B you will save space and also if you will have a index on that field it will work faster then the char.[1]

- 字段尽量增加not null限制。

性能更好

- 单表字段上限尽量不要超过80
- 0:假，1:真
- 使用VARCHAR 而不是NVARCHAR
- 字段尽可能有默认值，字符型的默认值为一个空字符值串，数字型的默认值为数值0

### 键设计原则
- 为关联字段创建外键
- 所有的键都必须唯一
- 尽可能避免使用复合键
- 外键总是关联唯一的键字段
- 尽可能使用系统生成（如序列SEQUENCE产生）的主键
- 可选键有时可做主键
- 一个表中组合主键的字段个数尽可能少

### 索引设计原则
- 外键字段必须建索引
- 对于索引选择性高的列使用B-Tree索引
- 对于索引选择性低的列使用位图索引
- HASH索引只适用于相等比较
- 不要索引大型字段（有很多字符的字段）
- 不要索引常用的小型表

## 完整性设计

实现数据库中数据得完整性。

1. 主键约束

每个表必须有主健，主健字段或组合字段必须满足非空属性和唯一性要求。

2. 外键约束（业务层控制）
3. Null值转换非空默认值

由于NULL值在参加任何运算时，结果均为NULL，所以必须利用NVL()函数把可能为NULL值得字段或变量转换为非NULL的默认值。

4. CHECK条件（业务层控制）
5.触发器（TBD，用到再定）
6.视图（TBD，用到再定）

## 命名规范
1. 总则
- 所有命名采用26个英文大小写字母和0－9这十个自然数，加上下划线_组成。不能出现其他字符（注释除外）。
- 实际名字尽量描述实体的内容，由英文单词、单词组合或单词缩写组成，不以数字和_开头。
- 命名单词以下划线分割
- 命名中禁止使用SQL关键字。
2. 表
- 表名不使用复数名词
- <前缀>_<含义单词1>_<含义单词2>_...

- ec_: 电商表
- saas_: saas表
- crm_: 客户关系表
- sys_: 系统表 

3. 字段

<含义单词1>_<含义单词2>_...

4. 主键

id

5. 外键（以索引代替）

<主表名含义部分>_id（user_id）

6. 索引
- IDX_<表名含义部分>_<构成索引的字段名>
- IDX_<表名含义部分>_COMPOSITE_<编号>（复合索引情况，编号从1自增）
7. 视图（TBD，用到再定）
8. 存储过程

SP_<系统标识>_<存储过程标识>

9. 函数（TBD，用到再定）
10. 触发器（TBD，用到再定）
11. 用户定义数据类型（TBD，用到再定）
12. 序列

SEQ_<序列标识>

13. 局部变量

L_<变量标识>

14. 全局变量

G_<变量标识>

15. 游标变量（TBD，用到再定）
16. 存储过程或函数定义中的参数
-IN型参数：P_<参数标识>
-OUT型参数：R_<参数标识>
-函数返回值：R_<变量标识>

## 安全性设计（需要随实践完善）

### 用户密码管理

用户帐号的密码必须进行加密处理，确保在任何地方查询都不会出现密码的明文。

### 应用级用户设计

应用级的用户帐号密码不能与数据库相同，防止用户直接操作数据库。用户只能用帐号登录到应用软件，通过应用软件访问数据库，而没有其它途径操作数据库。

## SQL语句编写
1. 字符类型数据

SQL中的字符类型数据应该统一使用单引号。特别对纯数字的字符串，必须用单引号，否则会导致内部转换而引起性能问题或索引失效问题。利用TRIM(),LOWER()等函数格式化匹配条件。

2. 复杂SQL

对于非常复杂的SQL（特别是有多层嵌套，带子句或相关子查询的），应该先考虑是否设计不当引起的。对于一些复杂SQL可以考虑使用程序实现。

3. 避免IN子句

使用 IN 或 NOT IN 子句时，特别是当子句中有多个值且表数据较多时，速度会明显下降。可以采用连接查询, 外连接查询或exists子查询来提高性能。

4. 避免使用SELECT * 语句

如果不必要取出所有数据，不要用 * 来代替，应给出字段列表。

5. 避免不必要的排序

不必要的数据排序大大的降低系统性能。

6. INSERT语句

使用INSERT语句一定要给出插入值的字段列表，这样即使表加了字段也不会影响现有系统的运行。

7. 多表连接

做多表操作时，应该给每个表取一个别名，每个表字段都应该标明其所属哪个表。

 8. 参数的传递

SQL语句的编写，变量尽量使用“?”绑定变量。

9. 存储过程、函数中的注释，示例如下：
``` sql
/*
目的：
作者：
创建日期：
*/

/*
修改顺序号：
修改者：
修改日期：
修改原因：（具体原因详细描述）
*/
```
## 建模管理方法
1. 建模工具

统一使用PowerDesigner软件建模。推荐版本PowerDesigner 16.5。

2. 建模步骤
- 逻辑建模

根据数据库设计和命名规范先在PowerDesigner中建立模型。要求每个表和字段都要有注释说明；Check Model不能出现错误(tools->check model)。

- 生成数据库结构及其相应的SQL脚本。
- 由项目经理负责建模的最终定稿
3. 模型维护
- 所有关于数据库的表、字段及关系、说明等均以模型文件为准。
- 由开发人员将变更需求提交项目负责人审批。
- 项目负责人同意变更后由相应开发人员负责编写变更脚本提交DBA（项目经理暂代）。
- DBA（项目经理暂代）更新数据库及其相关文档，并维护所有部分的一致性。
- 由项目经理负责模型维护

# 实践原则
1. 所有的数据库表设计均需以template表为基础扩展字段。必有字段包括id, uuid, delete_flag, add_time, add_user_id, update_time, update_user_id。以下是pmd截图和表的sql语句详情
``` sql
/*==============================================================*/
/* Table: template                                              */
/*==============================================================*/
create table template
(
   id                   bigint(20) not null auto_increment comment 'ID编号',
   uuid                 varchar(64) not null comment 'UUID',
   delete_flag          int(1) default 0 comment '0:正常, 1:删除',
   add_user_id          bigint(20) default 0 comment '添加人ID',
   add_time             datetime default CURRENT_TIMESTAMP comment '添加时间',
   update_user_id       bigint(20) default 0 comment '更新人ID',
   update_time          datetime default CURRENT_TIMESTAMP comment '更新时间',
   primary key (id)
);

/*==============================================================*/
/* Index: idx_template_uuid                                     */
/*==============================================================*/
create unique index idx_template_uuid on template
(
   uuid
);

/*==============================================================*/
/* Index: idx_template_delete_flag                              */
/*==============================================================*/
create index idx_template_delete_flag on template
(
   delete_flag
);
```

2. 字段索引规范
- id作为所有表的主键且必须建立主键索引
- uuid为了隐藏id自增逻辑，使用全表唯一值，必须建立唯一索引
- 所有存放枚举值的列必须创建索引
- 所有作为外键关联的列必须创建索引

3. sql语句编写（最佳实践）
- 小数类型为decimal，禁止使用float和double。 说明：float和double在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不正确的结果。如果存储的数据范围超过decimal的范围，建议将数据拆成整数和小数分开存储。
- 如果存储的字符串长度相对固定，使用char定长字符串类型。
- 所有boolean类型的状态字段需要用‘*_flag’表示，类型是int(1)，0表示否1表示是。
- 所有多对多的中间映射表表名称为 ‘主_从_map’。

# 数据库选择
## mysql数据库

版本：5.7.x (支持建表标准时间格式，支持json数据类型)

#Reference
1. http://blog.csdn.net/swarb/article/details/22919817
2. https://stackoverflow.com/questions/15896055/mysql-char-vs-int
