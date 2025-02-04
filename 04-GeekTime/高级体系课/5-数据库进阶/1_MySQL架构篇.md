## 1. MySQL 架构图
### 逻辑架构图



### MySQL 日志文件



### MySQL 数据文件



### 案例-一条SQL语句完整的执行流程：

分析SQL语句如下：
```sql
select c_id,first_name,last_name from customer where c_id=14;
```
大体来说，MySQL 可以分为 Server层和存储引擎层两部分：
1. Server层
	- 包括连接器、查询缓存、分析器、优化器、执行器等
	- 涵盖MySQL的大多数核心服务功能
	- 所有内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现。比如：存储过程、触发器、视图等
2. 存储引擎层
	- 负责数据的存储和提取
	- 可插拔式存储引擎：InnDB、MyISAM、Memory等
	- 最常用的存储引擎是InnDB
	- 从MySQL5.5版本开始，默认的存储引擎是InnDB

![](https://picture-bed01.oss-cn-beijing.aliyuncs.com/imgs/202402151828291.png)
**第一步：连接到数据库**

连接器

**第二步：查询缓存**

判断缓存是否存在，缓存命中直接返回，否则执行后续阶段，执行完成后也会将执行结果放入查询缓存。

注意：MySQL8.0版本直接将缓存功能删除掉了。

**第三步：分析SQL语句**

对请求的字符串做分析，判断语法是否正确、提取要查询的表、列、查询条件等，本质上是对一个SQL语句编译的过程，涉及**词法分析、语法分析、预处理器**等。

- 词法分析：词法分析就是把一个完整的 SQL 语句分割成一个个的字符串
- 语法分析：语法分析器根据词法分析的结果做语法检查，判断你输入的SQL 语句是否满足 MySQL 语法
- 预处理器：预处理器则会进一步去检查解析树是否合法，比如表名是否存在，语句中表的列是否存 在等等，在这一步MySQL会检验用户是否有表的操作权限

**第四步：优化SQL语句**

对查询进行优化。根据解析树生成不同的执行计划，然后选择最优的执行计划。

举例：有多个索引可用时，决定使用哪个；有多表关联时，决定各个表的连接顺序，以哪张表为基准表

**第五步：执行SQL语句**

首先判断执行权限，如果有权限，调用存储引擎接口查询。

举例：

- c_id是主键：
  - 调用 InnoDB 引擎接口，从主键索引中检索c_id=14的记录
  - 主键索引等值查询只会查询出一条记录，直接将该记录返回客户端
- c_id非主键：
  - 调用 InnoDB 引擎接口取这个表的第一行，判断c_id  值是不是 14，如果不是则跳过，如果是 则将这行缓存在结果集中
  - 调用引擎接口取“下一行”，重复相同的判断逻辑，直到取到这个表的最后一行
  - 执行器将上述遍历过程中所有满足条件的行组成的结果集返回给客户端



## 2. MySQL 的存储引擎之 InnoDB



