#### MongoBD 的设计目标时极简、灵活、作为Web应用栈的一部分

#### MongoDB的数据模型时面向文档的，所谓文档是一种类似JSON的结构，（BJSON）



MongoDB的概念：

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| :----------- | :--------------- | :---------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接,MongoDB不支持                |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |



#### MongoDB 有点很多，这里提一下他的缺点：

##### MongoDB仅支持文档内的事务（即类似Mysql一行数据的事务），所以对于需要跨文档（Mysql多行）或跨集合（Mysql多表）事务的应用，请谨慎使用MongoDB。另外，对于需要多表复杂（Join）查询的业务，还是使用关系型数据库为好。

