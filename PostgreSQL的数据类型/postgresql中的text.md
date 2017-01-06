#PostgreSQL与MySQL中 varchar和text对比
***

##在MySQL中
char

定长，最大255字节。存储空间未满，会在数据右侧填充空格。

varchar

不定长，最大65536字节，当varchar长度小于4时,自动的把varchar转换成char.。

Blob

存储二进制字符串

TinyBlob 最大 255字节

Blob 最大 65K

MediumBlob 最大 16M

LongBlob 最大 4G

Text

存储大文本

TINYTEXT: 256 bytes
TEXT: 65,535 bytes => ~64kb
MEDIUMTEXT: 16,777,215 bytes => ~16MB
LONGTEXT: 4,294,967,295 bytes => ~4GB

char varchar性能对比

不同的场景，选用的类型不一样。

char：

优点：update较varchar效率高

缺点：定长 会浪费磁盘空间，存储内容不够定填充空格，由于占用磁盘空间大的原因，查询时候效率较varchar低。

varchar：

优点：select查询效率高，较char相比节省磁盘空间，因此查询速度快

缺点：update时，如果新数据比原数据大，数据库需要重新开辟空间，这一点会有性能损耗。效率没有定长的char高

##在PostgreSQL中
<table>
<tr><td>名字</td><td>描述</td></tr>
<tr><td>character varying(n), varchar(n)</td><td>	变长，有长度限制</td></tr>
<tr><td>character(n), char(n)</td><td>定长，不足补空白</td></tr>
<tr><td>text</td><td>变长，无长度限制</td></tr>
</table>
>**提示:** 这三种类型之间没有性能差别，除了当使用填充空白类型时的增加存储空间， 和当存储长度约束的列时一些检查存入时长度的额外的CPU周期。 虽然在某些其它的数据库系统里，character(n) 有一定的性能优势，但在PostgreSQL里没有。 事实上，character(n)通常是这三个中最慢的， 因为额外存储成本。在大多数情况下，应该使用text 或character varying。


#比较
  在PostgreSQL中使用text和character在性能上没有区别

