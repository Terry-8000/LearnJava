# mysql约束

### 约束简介

关系型数据库系统和文件系统的一个不同点是，关系型数据库本身能保证存储数据的完整性，不需要应用程序的控制。一般来说数据完整性有以下三种形式：

- **实体完整性**，保证表中有一个主键。可以通过Primary Key和Unique Key约束保证实体完整性，也可以通过触发器；
- **域完整性**，保证数据中每列的值满足特定的条件，可以通过以下几种方式保证域完整性：
  - 选择合适的数据类型来保证数据值满足特定条件；
  - 外键Foreign Key约束；
  - 编写触发器；
  - 还可以考虑用DEFAULT约束作为强制域完整性的一个方面。
- **参照完整性**，保证两张表之间的关系，可以通过外键或者触发器来保证参照完整性。

### 外键约束

例子：

```sql
-> create table child(
-> id int, 
-> parent_id int, 
-> foreign key (parent_id) references parent(id)
-> );
```

一般来说，被引用的表称为父表，引用的表称为子表。当父表update和delete时，对子表的所做的操作可以有：

- CASCADE：父表update和delete时，子表也进行update和delete；
- SET NULL：父表update和delete时，子表相应的数据更新为NULL，子表的列必须允许为NULL；
- NO ACTION：父表update和delete时，抛出错误，不允许这样操作；
- RESTRICT：同NO ACTION，是默认的外键设置。

在mysql中外键建立时，会自动给该列加一个索引。

对于参照性约束，外键起到一个很好的作用，但是对于数据的导入操作，因为外键的即时检查，对导入的每一行数据都会进行外键检查，会导致花费大量的时间。在用户导入数据的过程中可以忽视外键的检查：

```sql
-> set foreign_key_checks = 0;
-> loading...
-> set foreign_key_checks = 1;
```

