#GROUP BY和HAVING子句
***
**在通过了WHERE过滤器之后，生成的输入表可以继续用GROUP BY 子句进行分组，然后用HAVING子句选取一些分组行。**
<pre><code>SELECT select_list
    FROM ...
    [WHERE ...]
    GROUP BY grouping_column_reference [, grouping_column_reference]...
</pre></code>

GROUP BY 子句 子句用于将一个表中所有列出的字段值都相等的行分成一组。 字段列出的顺序没什么关系。 效果是将每个拥有相同值的行集合并为一组，代表该组中的所有行。 这样就可以删除输出里的重复，和/或计算应用于这些组的聚合。 比如：
    <pre><code>=> SELECT * FROM test1;
 x | y
---+---
 a | 3
 c | 2
 b | 5
 a | 1
(4 rows)

=> SELECT x FROM test1 GROUP BY x;
 x
 a
 b
 c
(3 rows)
</pre></code>
在第二个查询里，我们不能写成SELECT * FROM test1 GROUP BY x， 因为字段y里没有哪个值可以和每个组关联起来。 被分组的字段可以在选择列表中引用是因为它们每个组都有单一的数值。

通常，如果一个表被分了组，不在GROUP BY中列出的字段只能在总表达式中被引用。 一个带聚合表达式的例子是：
    <pre><code>=> SELECT x, sum(y) FROM test1 GROUP BY x;
 x | sum
---+-----
 a |   4
 b |   5
 c |   2
(3 rows)
</pre></code>
在HAVING子句中的表达式可以引用分组的表达式和未分组的表达式 (后者必须涉及一个聚合函数)。
<pre><code>SELECT product_id, p.name, (sum(s.units) * (p.price - p.cost)) AS profit
    FROM products p LEFT JOIN sales s USING (product_id)
    WHERE s.date > CURRENT_DATE - INTERVAL '4 weeks'
    GROUP BY product_id, p.name, p.price, p.cost
    HAVING sum(p.price * s.units) > 5000;
</pre></code>
在上面的例子里，WHERE子句根据未分组的字段选择数据行 (表达式只是对那些最近四周发生的销售为真)。而HAVING 子句在分组之后选择那些销售总额超过5000的组。 请注意聚合表达式不需要在查询中的所有地方都一样。

如果一个查询调用了聚合函数，但没有GROUP BY子句，分组仍然发生： 结果是单一组行（或者如果单一行被HAVING所淘汰，那么也许没有行）。 同样，它包含一个HAVING子句，甚至没有任何聚合函数的调用或GROUP BY子句。
