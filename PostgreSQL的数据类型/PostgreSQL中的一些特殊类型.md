# PostgreSQL中的一些特殊类型 #
***
<pre><code>CREATE TABLE postgresql_data_type (
id SERIAL,  
pay money   #插入数据后会自动保留两位小数而且会有钱币符号
)

INSERT INTO postgresql_data_type VALUES(DEFAULT, 100)

ALTER TABLE postgresql_data_type ADD text_area TEXT

ALTER TABLE postgresql_data_type ADD bool BOOLEAN    #boolean类型

CREATE TYPE enum_type AS ENUM('sad','happy','ok')    #枚举类型

ALTER TABLE postgresql_data_type ADD enumtype enum_type

CREATE TYPE twoint AS(
a INTEGER,
b FLOAT
)

ALTER TABLE postgresql_data_type ADD twoint twoint #复合类型

INSERT INTO postgresql_data_type VALUES(DEFAULT,200.99,'asfdasdfad','t','ok','(1,2)') #插入数据
</pre></code>

> 枚举类型中，值的顺序是创建枚举类型时定义的顺序。 所有的比较标准运算符及其相关的聚合函数都可支持枚举类型。
> 每个枚举类型都是独立的，不能与其他枚举类型结合。举标签对大小写是敏感的，因此'happy'不等于'HAPPY'。 标签中的空格也是一样。